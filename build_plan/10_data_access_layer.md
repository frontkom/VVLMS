# Data Access Layer Design
## Multi-tenancy abstraction: Schema-per-tenant now, RLS-ready later

**Decision**: ADR-005
**Status**: Accepted
**Date**: 2026-03-22

---

## Decision

**MVP**: Schema-per-tenant in PostgreSQL. Each government agency gets its own schema.

**Future**: When we need 100+ tenants (municipalities), add a shared-schema + RLS tier. The application code should NOT need rewriting -- only the data access layer adapts.

**Principle**: No database access anywhere in the codebase except through the tenant-aware repository layer. No raw SQL with hardcoded schema names. No direct SQLAlchemy session usage outside the repository.

---

## Architecture

```
[FastAPI Route Handler]
        |
        | (never touches DB directly)
        |
[Service Layer]
        |
        | (business logic, no DB awareness)
        |
[Repository Layer]  <-- ALL database access goes through here
        |
        |--- TenantContext (resolves tenant, sets isolation mode)
        |
        +--- SchemaStrategy (MVP: SET search_path)
        |
        +--- RLSStrategy (Future: SET app.tenant_id)
        |
[SQLAlchemy Session]
        |
[PostgreSQL]
```

---

## Implementation

### 1. Tenant Context Middleware

```python
# app/middleware/tenant.py

from dataclasses import dataclass
from enum import Enum
from fastapi import Request
from typing import Optional


class IsolationMode(Enum):
    SCHEMA = "schema"      # Schema-per-tenant (MVP)
    RLS = "rls"            # Row-level security (future)


@dataclass
class TenantContext:
    tenant_id: str
    slug: str
    schema_name: str
    isolation_mode: IsolationMode

    @property
    def is_schema_isolated(self) -> bool:
        return self.isolation_mode == IsolationMode.SCHEMA


async def tenant_middleware(request: Request, call_next):
    """Resolve tenant from subdomain/JWT and attach to request state."""
    tenant = await resolve_tenant(request)
    request.state.tenant = TenantContext(
        tenant_id=tenant.id,
        slug=tenant.slug,
        schema_name=f"tenant_{tenant.slug}",
        isolation_mode=IsolationMode(tenant.isolation_mode),
    )
    response = await call_next(request)
    return response


async def resolve_tenant(request: Request) -> "TenantRecord":
    """Resolve tenant from subdomain, JWT claim, or API key."""
    # 1. Try subdomain: svv.veileder.no
    host = request.headers.get("host", "")
    subdomain = host.split(".")[0] if "." in host else None

    if subdomain and subdomain not in ("www", "api", "app"):
        return await get_tenant_by_slug(subdomain)

    # 2. Try JWT claim
    if hasattr(request.state, "user") and request.state.user:
        return await get_tenant_by_id(request.state.user.tenant_id)

    # 3. Try API key header
    api_key = request.headers.get("X-Tenant-Key")
    if api_key:
        return await get_tenant_by_api_key(api_key)

    raise TenantNotFoundError("Could not resolve tenant")
```

### 2. Database Session Factory

```python
# app/db/session.py

from contextlib import asynccontextmanager
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from sqlalchemy import text

engine = create_async_engine(
    settings.DATABASE_URL,
    pool_size=50,
    max_overflow=20,
    pool_pre_ping=True,
)

async_session_factory = async_sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)


@asynccontextmanager
async def get_tenant_session(tenant: TenantContext):
    """
    Create a database session scoped to a tenant.

    This is the ONLY way to get a database session in the application.
    The isolation strategy is transparent to the caller.
    """
    async with async_session_factory() as session:
        async with session.begin():
            if tenant.isolation_mode == IsolationMode.SCHEMA:
                # MVP: switch to tenant schema
                await session.execute(
                    text(f"SET LOCAL search_path TO {tenant.schema_name}, shared, public")
                )
            elif tenant.isolation_mode == IsolationMode.RLS:
                # Future: set RLS context variable
                await session.execute(
                    text("SET LOCAL search_path TO shared_tenants, shared, public")
                )
                await session.execute(
                    text(f"SET LOCAL app.tenant_id = '{tenant.tenant_id}'")
                )

            yield session


# FastAPI dependency
async def get_db(request: Request) -> AsyncSession:
    """FastAPI dependency that provides a tenant-scoped session."""
    tenant = request.state.tenant
    async with get_tenant_session(tenant) as session:
        yield session
```

### 3. Base Repository (all repos inherit from this)

```python
# app/repositories/base.py

from typing import TypeVar, Generic, Type, Optional, List
from uuid import UUID
from sqlalchemy import select, update, func
from sqlalchemy.ext.asyncio import AsyncSession

from app.models.base import BaseModel

T = TypeVar("T", bound=BaseModel)


class BaseRepository(Generic[T]):
    """
    Base repository that all domain repositories inherit from.

    RULES:
    - All database access goes through repositories
    - Repositories receive an AsyncSession that is already tenant-scoped
    - Repositories NEVER set search_path or tenant context
    - Repositories NEVER import or reference tenant/schema names
    - Repositories use SQLAlchemy models, never raw SQL with table names
    """

    def __init__(self, session: AsyncSession, model_class: Type[T]):
        self.session = session
        self.model_class = model_class

    async def get_by_id(self, id: UUID) -> Optional[T]:
        result = await self.session.get(self.model_class, id)
        return result

    async def get_all(
        self,
        offset: int = 0,
        limit: int = 100,
        filters: dict = None,
    ) -> List[T]:
        query = select(self.model_class)

        if filters:
            for key, value in filters.items():
                if hasattr(self.model_class, key):
                    query = query.where(getattr(self.model_class, key) == value)

        query = query.offset(offset).limit(limit)
        result = await self.session.execute(query)
        return list(result.scalars().all())

    async def create(self, **kwargs) -> T:
        instance = self.model_class(**kwargs)
        self.session.add(instance)
        await self.session.flush()
        return instance

    async def update(self, id: UUID, **kwargs) -> Optional[T]:
        instance = await self.get_by_id(id)
        if instance:
            for key, value in kwargs.items():
                setattr(instance, key, value)
            await self.session.flush()
        return instance

    async def soft_delete(self, id: UUID) -> bool:
        instance = await self.get_by_id(id)
        if instance and hasattr(instance, "deleted_at"):
            instance.deleted_at = func.now()
            await self.session.flush()
            return True
        return False

    async def count(self, filters: dict = None) -> int:
        query = select(func.count()).select_from(self.model_class)
        if filters:
            for key, value in filters.items():
                if hasattr(self.model_class, key):
                    query = query.where(getattr(self.model_class, key) == value)
        result = await self.session.execute(query)
        return result.scalar()
```

### 4. Domain Repositories (examples)

```python
# app/repositories/courses.py

from typing import List, Optional
from uuid import UUID
from sqlalchemy import select, or_
from sqlalchemy.orm import selectinload

from app.models.course import Course, CourseContent, Enrollment
from app.repositories.base import BaseRepository


class CourseRepository(BaseRepository[Course]):
    """
    Course data access. Tenant isolation is handled by the session --
    this repository has ZERO awareness of which tenant it's operating on.
    """

    def __init__(self, session):
        super().__init__(session, Course)

    async def search(
        self,
        query: str,
        category_id: UUID = None,
        is_mandatory: bool = None,
        status: str = "published",
        limit: int = 20,
    ) -> List[Course]:
        stmt = (
            select(Course)
            .where(Course.status == status)
            .where(Course.deleted_at.is_(None))
        )

        if query:
            stmt = stmt.where(
                Course.search_vector.match(query, postgresql_regconfig="norwegian")
            )

        if category_id:
            stmt = stmt.where(Course.category_id == category_id)

        if is_mandatory is not None:
            # Note: in the revised schema, mandatory is per target group,
            # not a simple boolean. This method would be updated to check
            # course_target_rules instead.
            pass

        stmt = stmt.limit(limit)
        result = await self.session.execute(stmt)
        return list(result.scalars().all())

    async def get_with_content(self, course_id: UUID) -> Optional[Course]:
        stmt = (
            select(Course)
            .options(selectinload(Course.contents))
            .where(Course.id == course_id)
        )
        result = await self.session.execute(stmt)
        return result.scalar_one_or_none()

    async def get_user_enrollments(
        self,
        user_id: UUID,
        statuses: List[str] = None,
    ) -> List[Enrollment]:
        stmt = (
            select(Enrollment)
            .options(selectinload(Enrollment.course))
            .where(Enrollment.user_id == user_id)
        )
        if statuses:
            stmt = stmt.where(Enrollment.status.in_(statuses))

        stmt = stmt.order_by(Enrollment.due_date.asc().nullslast())
        result = await self.session.execute(stmt)
        return list(result.scalars().all())


# app/repositories/learning_records.py

from app.models.learning_record import LearningRecord
from app.repositories.base import BaseRepository


class LearningRecordRepository(BaseRepository[LearningRecord]):
    """Unified learning timeline across formal/social/experiential."""

    def __init__(self, session):
        super().__init__(session, LearningRecord)

    async def get_user_timeline(
        self,
        user_id: UUID,
        learning_mode: str = None,
        year: int = None,
        limit: int = 50,
    ) -> List[LearningRecord]:
        stmt = (
            select(LearningRecord)
            .where(LearningRecord.user_id == user_id)
            .where(LearningRecord.status.in_(["completed", "approved"]))
        )

        if learning_mode:
            stmt = stmt.where(LearningRecord.learning_mode == learning_mode)

        if year:
            stmt = stmt.where(
                LearningRecord.occurred_at >= f"{year}-01-01",
                LearningRecord.occurred_at < f"{year + 1}-01-01",
            )

        stmt = stmt.order_by(LearningRecord.occurred_at.desc()).limit(limit)
        result = await self.session.execute(stmt)
        return list(result.scalars().all())

    async def get_70_20_10_breakdown(
        self,
        org_unit_id: UUID = None,
        year: int = None,
    ) -> dict:
        """Get learning mode distribution for 70/20/10 analytics."""
        stmt = (
            select(
                LearningRecord.learning_mode,
                func.count().label("count"),
                func.sum(LearningRecord.duration_min).label("total_minutes"),
            )
            .where(LearningRecord.status.in_(["completed", "approved"]))
            .group_by(LearningRecord.learning_mode)
        )

        if org_unit_id:
            stmt = stmt.join(User).where(User.org_unit_id == org_unit_id)

        if year:
            stmt = stmt.where(
                LearningRecord.occurred_at >= f"{year}-01-01",
                LearningRecord.occurred_at < f"{year + 1}-01-01",
            )

        result = await self.session.execute(stmt)
        rows = result.all()

        total = sum(r.count for r in rows)
        return {
            r.learning_mode: {
                "count": r.count,
                "percentage": round(r.count / total * 100, 1) if total > 0 else 0,
                "total_minutes": r.total_minutes or 0,
            }
            for r in rows
        }
```

### 5. Service Layer (uses repositories, not sessions)

```python
# app/services/course_service.py

from uuid import UUID
from typing import List

from app.repositories.courses import CourseRepository
from app.repositories.learning_records import LearningRecordRepository
from app.schemas.course import CourseCreate, CourseUpdate


class CourseService:
    """
    Business logic for courses.

    RULES:
    - Services receive repositories, not database sessions
    - Services contain business logic, validation, orchestration
    - Services have ZERO awareness of tenancy or database isolation
    """

    def __init__(
        self,
        course_repo: CourseRepository,
        learning_record_repo: LearningRecordRepository,
    ):
        self.courses = course_repo
        self.learning_records = learning_record_repo

    async def complete_course(self, user_id: UUID, course_id: UUID, score: float):
        """Mark a course as completed and create a learning record."""
        enrollment = await self.courses.update_enrollment_status(
            user_id=user_id,
            course_id=course_id,
            status="completed",
            score=score,
        )

        course = await self.courses.get_by_id(course_id)

        # Create unified learning record
        await self.learning_records.create(
            user_id=user_id,
            record_type="course_completion",
            learning_mode="formal",
            source_type="course",
            source_id=course_id,
            title=course.title,
            score=score,
            status="completed",
        )

        # Check if this triggers a certification
        # (business logic, not data access)
        await self._check_certification_rules(user_id, course_id)

        return enrollment
```

### 6. FastAPI Route (wires it all together)

```python
# app/api/routes/courses.py

from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession

from app.db.session import get_db
from app.repositories.courses import CourseRepository
from app.repositories.learning_records import LearningRecordRepository
from app.services.course_service import CourseService

router = APIRouter(prefix="/courses", tags=["courses"])


def get_course_service(db: AsyncSession = Depends(get_db)) -> CourseService:
    """Dependency injection: build service with tenant-scoped repos."""
    return CourseService(
        course_repo=CourseRepository(db),
        learning_record_repo=LearningRecordRepository(db),
    )


@router.get("/")
async def list_courses(
    q: str = None,
    category_id: UUID = None,
    service: CourseService = Depends(get_course_service),
):
    """List courses. Tenant isolation is automatic -- the session is already scoped."""
    return await service.courses.search(query=q, category_id=category_id)


@router.post("/{course_id}/complete")
async def complete_course(
    course_id: UUID,
    score: float,
    user: User = Depends(get_current_user),
    service: CourseService = Depends(get_course_service),
):
    return await service.complete_course(user.id, course_id, score)
```

---

## What This Buys Us for Future RLS Migration

When we need to add the RLS tier for small tenants:

### What changes:
1. **`get_tenant_session()`** -- add the `IsolationMode.RLS` branch (already stubbed above)
2. **SQLAlchemy models** -- add `tenant_id` column to all models in the shared schema
3. **PostgreSQL** -- create RLS policies on shared-schema tables
4. **Tenant registry** -- add `isolation_mode` field to tenant config

### What does NOT change:
- **Repositories** -- zero changes. They use SQLAlchemy models, not raw SQL.
- **Services** -- zero changes. They call repository methods.
- **API routes** -- zero changes. They depend on services.
- **Tests** -- zero changes (if written against repository interfaces).

### Migration path:

```python
# Step 1: Add tenant_id to shared-schema models (Alembic migration)
# Step 2: Create RLS policies
# Step 3: Update TenantContext to route small tenants to shared schema
# Step 4: New small tenants go to shared schema automatically
# Step 5: (Optional) Migrate existing small tenants from schema to shared

# The application code change is literally ONE function:
async def get_tenant_session(tenant: TenantContext):
    async with async_session_factory() as session:
        async with session.begin():
            if tenant.isolation_mode == IsolationMode.SCHEMA:
                await session.execute(
                    text(f"SET LOCAL search_path TO {tenant.schema_name}, shared, public")
                )
            elif tenant.isolation_mode == IsolationMode.RLS:
                await session.execute(
                    text("SET LOCAL search_path TO shared_tenants, shared, public")
                )
                await session.execute(
                    text(f"SET LOCAL app.tenant_id = '{tenant.tenant_id}'")
                )
            yield session
```

---

## Testing Strategy

```python
# tests/conftest.py

import pytest
from app.db.session import get_tenant_session
from app.middleware.tenant import TenantContext, IsolationMode


@pytest.fixture
async def test_tenant():
    """Create a test tenant with its own schema."""
    tenant = TenantContext(
        tenant_id="test-tenant-001",
        slug="test",
        schema_name="tenant_test",
        isolation_mode=IsolationMode.SCHEMA,
    )
    # Create schema for test
    async with engine.begin() as conn:
        await conn.execute(text("CREATE SCHEMA IF NOT EXISTS tenant_test"))
        await conn.execute(text("SET search_path TO tenant_test"))
        # Run migrations...

    yield tenant

    # Cleanup
    async with engine.begin() as conn:
        await conn.execute(text("DROP SCHEMA tenant_test CASCADE"))


@pytest.fixture
async def db(test_tenant):
    """Provide a tenant-scoped database session for tests."""
    async with get_tenant_session(test_tenant) as session:
        yield session


# Tests use repositories exactly like production code
async def test_create_course(db):
    repo = CourseRepository(db)
    course = await repo.create(
        title="HMS Grunnkurs",
        type="e-learning",
        status="published",
    )
    assert course.id is not None
    assert course.title == "HMS Grunnkurs"
```

---

## Rules for the Team

1. **NEVER** import `engine` or `async_session_factory` outside of `app/db/session.py`
2. **NEVER** write raw SQL with schema names (`tenant_svv.courses`) -- use SQLAlchemy models
3. **NEVER** access the database from a service or route handler directly -- go through a repository
4. **ALWAYS** receive the session via dependency injection (`Depends(get_db)`)
5. **ALWAYS** use `BaseRepository` methods or extend them -- don't create one-off session.execute() calls
6. **NEVER** hardcode tenant identification in queries -- the session handles it
7. **Tests** use the same repository layer as production -- no test-specific data access shortcuts
