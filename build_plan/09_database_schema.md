# 09 - Database Schema & Data Architecture

## Document Status

| Field | Value |
|-------|-------|
| Status | Proposed |
| Date | 2026-03-22 |
| Scope | Complete PostgreSQL schema for multi-tenant Norwegian public sector LMS |
| Target DB | PostgreSQL 16 + pgvector |
| Multi-tenancy | Schema-per-tenant with shared platform schema |
| Migration tool | Alembic (SQLAlchemy, Python/FastAPI backend) |

---

## Table of Contents

1. [Schema Review: Current State Critique](#1-schema-review-current-state-critique)
2. [Multi-Tenancy Deep Dive](#2-multi-tenancy-deep-dive)
3. [Best Practices from Moodle, Canvas, and Industry](#3-best-practices-from-moodle-canvas-and-industry)
4. [Data Model for 70/20/10 Learning](#4-data-model-for-702010-learning)
5. [Missing Tables: Full DDL for All 11 Gaps](#5-missing-tables-full-ddl-for-all-11-gaps)
6. [Performance Considerations](#6-performance-considerations)
7. [Migration Strategy](#7-migration-strategy)
8. [Full Revised Schema DDL](#8-full-revised-schema-ddl)

---

## 1. Schema Review: Current State Critique

### 1.1 What the current schema does well

The existing schema from `01_architecture.md` section 6 has a solid foundation:

**Good decisions:**
- **UUID primary keys everywhere** -- correct for a multi-tenant SaaS where IDs must be globally unique and not leak sequential ordering to clients.
- **Separate `completions` table from `enrollments`** -- keeping the immutable audit trail of completions distinct from the mutable enrollment state is the right pattern. Moodle conflates these, which causes endless reporting headaches.
- **JSONB for flexible metadata** -- using `metadata JSONB` on courses and `cmi_data JSONB` on SCORM runtime is the right hybrid approach. The SCORM CMI data model has 100+ fields that vary by SCORM version; relational columns for all of them would be impractical.
- **Partitioning on `completions` and `audit_log`** -- time-range partitioning on high-volume append-only tables is correct. These tables will grow unboundedly.
- **`search_vector TSVECTOR`** on courses -- Norwegian full-text search built into the schema rather than bolted on later.
- **`org_unit_id` scoping on `user_roles`** -- this enables the scoped RBAC that SVV requires (division admin manages only their division).
- **Schema-per-tenant architecture** -- appropriate for Norwegian public sector where data isolation is a regulatory requirement, not just a preference.

### 1.2 Issues that need fixing

**Problem 1: Missing foreign key indexes**

The current schema defines foreign keys but only creates indexes for a few query patterns. Every foreign key column that participates in JOINs needs its own index, or those JOINs will trigger sequential scans.

Missing indexes:
- `enrollments.course_id` (has index, good)
- `enrollments.assigned_by` (no index -- needed for "who assigned this" queries)
- `course_contents.course_id` (no index -- every course detail page JOINs this)
- `learning_path_items.path_id` (no index -- learning path rendering JOINs this)
- `learning_path_items.course_id` (no index)
- `learning_path_enrollments.path_id` (no index)
- `learning_path_enrollments.user_id` (no index)
- `competencies.framework_id` (no index)
- `competencies.parent_id` (no index -- tree traversal queries)
- `user_competencies.competency_id` (no index)
- `certificates.course_id` (no index)
- `certificates.template_id` (no index)
- `events.course_id` (no index)
- `events.instructor_id` (no index)
- `notifications.user_id` (has partial index for unread, good)
- `users.org_unit_id` (no index -- every leader dashboard query filters by this)
- `users.manager_id` (no index -- direct reports queries)

**Problem 2: `user_roles` composite primary key includes `org_unit_id` which can be NULL**

The current PK is `(user_id, role_id, org_unit_id)`. If `org_unit_id` is NULL (for global roles like System Admin), PostgreSQL treats each NULL as distinct, allowing duplicate `(user_id, role_id, NULL)` rows. Fix: use a surrogate PK and a unique constraint with `COALESCE`.

**Problem 3: `user_competencies` has `UNIQUE (user_id, competency_id)` but no history**

A user can acquire a competency, let it expire, and re-acquire it. The unique constraint prevents storing multiple records for the same competency. The table should track competency history, with a separate "current" view.

**Problem 4: `completions` partitioned by `completed_at` but missing partition key in indexes**

When using range partitioning, the partition key (`completed_at`) must be part of any unique constraint or primary key. The current `id UUID PRIMARY KEY` will fail on partitioned tables in PostgreSQL because the partition key is not included.

**Problem 5: No `updated_at` triggers**

Several tables have `updated_at TIMESTAMPTZ DEFAULT now()` but no trigger to automatically update it on row modification. Without a trigger, `updated_at` will always equal `created_at`.

**Problem 6: `courses.is_mandatory` is a boolean but should be per-target-group**

GAP #3 explicitly requires: same course mandatory for group A, optional for group B, with different certification validity durations. A single boolean on the course table cannot model this.

**Problem 7: `certificates.certificate_no VARCHAR(50) UNIQUE`**

Certificate numbers need to be unique per tenant, which schema-per-tenant handles. But the generation strategy is not defined. Recommend a deterministic format: `{tenant_slug}-{year}-{sequence}` (e.g., `SVV-2027-000142`).

**Problem 8: No soft-delete pattern**

The schema uses `ON DELETE CASCADE` in several places. For a public sector LMS where audit trails matter, hard deletes are dangerous. Courses, users, and completions should use soft-delete (`deleted_at TIMESTAMPTZ`) with the actual record preserved.

**Problem 9: `xapi_statements` stores raw JSONB but has no extracted indexed columns**

Querying xAPI statements by verb, actor, or object requires GIN indexes on JSONB, which work but are slower than B-tree indexes on extracted columns. The current schema has `idx_xapi_actor` as a GIN index, but common query patterns (find all "completed" statements for a user) would benefit from extracted columns.

**Problem 10: No `version` or `schema_version` tracking in tenant schemas**

There is no mechanism to track which migration version each tenant schema is at. This is critical for schema-per-tenant where migrations must be applied to all schemas.

---

## 2. Multi-Tenancy Deep Dive

### 2.1 Schema-per-tenant: operational reality

Schema-per-tenant is the correct choice for Norwegian public sector. Here is the operational analysis:

#### Why schema-per-tenant over RLS-only

| Concern | Schema-per-tenant | Shared + RLS |
|---------|-------------------|--------------|
| **Data isolation** | Physical separation at namespace level | Logical separation via policies -- one bug leaks everything |
| **Norwegian public sector audit** | Passes security reviews -- auditors understand schemas | Requires detailed RLS policy audit; harder to explain |
| **Per-tenant backup/restore** | `pg_dump -n tenant_svv` -- trivial | Must filter by `tenant_id` during restore -- error-prone |
| **Per-tenant data deletion** | `DROP SCHEMA tenant_xyz CASCADE` -- instant, complete | Must DELETE from every table WHERE tenant_id = X -- slow, fragmented |
| **Performance isolation** | Separate indexes, separate table statistics per tenant | Shared indexes grow with all tenants; skewed statistics |
| **Cross-tenant analytics** | Harder -- requires aggregation pipeline | Easier -- query across all tenants directly |
| **Migration complexity** | Must apply to N schemas | Apply once to shared tables |
| **Connection pooling** | SET search_path per transaction | Simpler -- no schema switching |
| **Max tenants** | Practical limit ~100-500 schemas per database | Thousands of tenants easily |

**Verdict**: Schema-per-tenant is correct for our use case (government agencies, expected 5-50 tenants, strong isolation requirements). If we later serve hundreds of smaller organizations, we can introduce a shared-schema tier for small tenants while keeping schema-per-tenant for large agencies.

#### Connection pooling with PgBouncer

PgBouncer must run in **transaction mode** (not session mode) for schema-per-tenant to work correctly:

```
; pgbouncer.ini
[databases]
lms = host=db.internal port=5432 dbname=lms_production

[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 50
reserve_pool_size = 10
server_lifetime = 3600
server_idle_timeout = 600
```

**Critical**: In transaction mode, session-level `SET search_path` does not persist between transactions. Every database operation must use `SET LOCAL search_path` within a transaction, or qualify table names explicitly.

FastAPI middleware pattern:

```python
# middleware/tenant.py
async def set_tenant_schema(request: Request, call_next):
    tenant = resolve_tenant(request)  # from subdomain or JWT
    schema_name = f"tenant_{tenant.slug}"

    # Each request gets a connection scoped to the tenant
    async with db.begin() as conn:
        await conn.execute(
            text(f"SET LOCAL search_path TO {schema_name}, shared, public")
        )
        request.state.db = conn
        response = await call_next(request)

    return response
```

#### Migration execution

With Alembic, migrations must be applied to all tenant schemas. The recommended approach:

```python
# alembic/env.py
def run_migrations_online():
    engine = create_engine(config.get_main_option("sqlalchemy.url"))

    # Get all tenant schemas
    with engine.connect() as conn:
        schemas = conn.execute(text(
            "SELECT schema_name FROM shared.tenants WHERE is_active = true"
        )).fetchall()

    # Apply migration to each tenant schema
    for (schema_name,) in schemas:
        with engine.connect() as conn:
            conn.execute(text(f"SET search_path TO {schema_name}"))
            context.configure(connection=conn, target_metadata=target_metadata)
            with context.begin_transaction():
                context.run_migrations()
```

There is also the `alembic-multischema` PyPI package that provides this functionality out of the box.

#### Monitoring

Key metrics to track per tenant:

```sql
-- Table sizes per schema (detect runaway tenants)
SELECT schemaname, tablename,
       pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename)) as total_size
FROM pg_tables
WHERE schemaname LIKE 'tenant_%'
ORDER BY pg_total_relation_size(schemaname || '.' || tablename) DESC;

-- Active connections per tenant
SELECT query, usename, state, wait_event,
       (regexp_match(query, 'search_path TO (\w+)'))[1] as tenant_schema
FROM pg_stat_activity
WHERE state = 'active';

-- Slow queries per tenant (requires pg_stat_statements)
SELECT query, calls, mean_exec_time, total_exec_time
FROM pg_stat_statements
WHERE query LIKE '%SET LOCAL search_path%'
ORDER BY mean_exec_time DESC
LIMIT 20;
```

#### Backup strategy

```bash
# Per-tenant backup (for tenant data export / GDPR requests)
pg_dump -h db.internal -U lms -n tenant_svv -F c -f tenant_svv_$(date +%Y%m%d).dump lms_production

# Full database backup (nightly)
pg_dump -h db.internal -U lms -F c -f lms_full_$(date +%Y%m%d).dump lms_production

# Restore single tenant (disaster recovery or data migration)
pg_restore -h db.internal -U lms -n tenant_svv -d lms_production tenant_svv_20270115.dump
```

### 2.2 Schema layout

```
PostgreSQL database: lms_production
|
+-- schema: public
|   +-- Extensions: pgvector, pg_trgm, pgcrypto, btree_gist
|
+-- schema: shared
|   +-- tenants                    -- Tenant registry
|   +-- tenant_features            -- Feature flags per tenant
|   +-- platform_admins            -- Platform-level admin accounts
|   +-- migration_history          -- Tracks migration version per tenant schema
|   +-- global_activity_types      -- Default activity type templates
|   +-- global_certificate_templates -- Shared certificate templates
|
+-- schema: tenant_svv             -- Statens vegvesen
|   +-- [all tenant tables]
|
+-- schema: tenant_nve             -- Example: Norges vassdrags- og energidirektorat
|   +-- [all tenant tables]
|
+-- schema: analytics              -- Read-only aggregated analytics
    +-- cross_tenant_metrics       -- Anonymized platform-wide metrics
```

---

## 3. Best Practices from Moodle, Canvas, and Industry

### 3.1 What we adopt from Moodle

Moodle has approximately 200 tables (50 core, the rest for plugins/modules). Key patterns worth adopting:

**Competency framework modeling**: Moodle's competency subsystem (`competency_framework`, `competency`, `competency_usercomp`, `competency_evidence`, `competency_plan`, `competency_plancomp`) provides a mature model. Their approach includes:
- Frameworks contain competencies in a tree structure (parent_id)
- Competencies have a configurable proficiency scale (not hardcoded levels)
- User competency records include evidence linking (which course completion, which activity gave this competency)
- Learning plans connect competencies to users with deadlines

**What we adopt**: Configurable proficiency scales per framework, evidence linking from completions to competencies, and learning plans as a first-class concept.

**What we do differently**: Moodle stores competency evidence as a separate join table referencing multiple source types. We use a single `evidence_type` + `evidence_id` polymorphic reference, which is simpler and sufficient for our use cases.

### 3.2 What we adopt from Canvas LMS

Canvas (Instructure) is PostgreSQL-native and open source. Key patterns:

**Assignment and submission model**: Canvas separates assignments (the definition) from submissions (each student's work). Submissions have a `workflow_state` (submitted, graded, pending_review) and support multiple submission types (online_text_entry, online_upload, online_url, media_recording).

**Rubrics and grading**: Canvas uses `rubrics`, `rubric_assessments`, and `rubric_associations` tables. Rubric criteria are stored as JSONB with structured scoring. This is more flexible than a fully normalized rubric model.

**Assignment overrides**: Canvas supports per-group or per-student overrides on due dates, availability, and grading criteria through `assignment_overrides` and `assignment_override_students` tables.

**What we adopt**: The assignment/submission separation, workflow states on submissions, and the rubric-as-JSONB pattern for assessment checklists.

### 3.3 What we adopt from xAPI / SQL LRS

Yet Analytics' SQL LRS (open source, Apache 2.0) provides the reference PostgreSQL schema for xAPI storage. Key insights:

**Statement storage**: Store the full JSON statement as JSONB for conformance, but extract commonly queried fields (actor IRI, verb IRI, object ID, timestamp) into indexed columns for fast filtering.

**Statement references and voiding**: xAPI supports statement references (one statement referring to another) and voiding (marking a previous statement as void). The schema must support both.

**What we adopt**: Hybrid storage -- full JSONB statement plus extracted indexed columns for the query patterns we actually use (actor, verb, activity, timestamp range).

### 3.4 Industry patterns

**Soft-delete everywhere**: Enterprise LMS platforms never hard-delete learning records. We add `deleted_at TIMESTAMPTZ` to all major entities and filter with `WHERE deleted_at IS NULL` in application queries.

**Temporal records for competencies**: A user's competency level changes over time. Rather than UPDATE in place, INSERT a new record and track the history. This is critical for auditing ("when did this person lose their certification?").

**Event sourcing for SCORM/xAPI**: Store every CMI data model commit as an event, not just the latest state. This enables debugging ("why did the score change from 85 to 72?") and replay.

---

## 4. Data Model for 70/20/10 Learning

SVV's vision centers on the 70/20/10 model. The schema must treat all three learning modes as first-class citizens, not bolt workplace learning onto a course-centric model.

### 4.1 Unified learning record architecture

```
                        +---------------------------+
                        |   LEARNING RECORD          |
                        |   (unified activity log)   |
                        +---------------------------+
                               /     |     \
                              /      |      \
                     +-------+  +----+----+  +----------+
                     | 10%   |  | 20%     |  | 70%      |
                     | FORMAL|  | SOCIAL  |  | EXPERIEN |
                     +-------+  +---------+  +----------+
                     |         |            |
              +------+------+ | +---------+----------+
              |             | | |                     |
         courses      learning | workplace       self-
         events       paths    | activities      training
         SCORM/xAPI          | checklists
         quizzes             | practical exams
         assignments         | field inspections
                             |
                        mentoring
                        coaching
                        peer observation
                        knowledge sharing
```

### 4.2 The `learning_records` approach

Rather than scattering learning evidence across disconnected tables, we introduce a **unified `learning_records`** table that acts as the central timeline. Every learning event -- whether completing a SCORM course, finishing a field inspection, or logging a mentoring session -- creates a record here.

This is conceptually similar to xAPI statements but optimized for our reporting and dashboard queries. The xAPI statement store (`xapi_statements`) remains separate for LRS conformance, but `learning_records` is our operational reporting backbone.

```sql
-- The unified learning timeline
-- Every learning event across all three modes creates a record here
CREATE TABLE learning_records (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id),
    record_type     VARCHAR(30) NOT NULL,
    -- FORMAL: 'course_completion', 'quiz_attempt', 'assignment_submission',
    --         'scorm_session', 'event_attendance'
    -- SOCIAL: 'mentoring_session', 'peer_observation', 'knowledge_share'
    -- EXPERIENTIAL: 'workplace_activity', 'self_training', 'practical_exam',
    --              'field_inspection', 'emergency_drill', 'simulator_session'
    learning_mode   VARCHAR(15) NOT NULL CHECK (learning_mode IN ('formal', 'social', 'experiential')),
    source_type     VARCHAR(50) NOT NULL,
    -- 'course', 'event', 'workplace_activity', 'assignment', 'external'
    source_id       UUID,                            -- FK to source table (polymorphic)
    title           VARCHAR(500) NOT NULL,
    description     TEXT,
    occurred_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    duration_min    INT,
    score           DECIMAL(5,2),
    status          VARCHAR(20) NOT NULL DEFAULT 'completed',
    -- 'completed', 'in_progress', 'pending_approval', 'approved', 'rejected'
    competency_ids  UUID[],                          -- Competencies this contributes to
    evidence_url    VARCHAR(500),                    -- Link to evidence (PDF, photo, etc.)
    metadata        JSONB DEFAULT '{}',
    approved_by     UUID REFERENCES users(id),
    approved_at     TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
) PARTITION BY RANGE (occurred_at);
```

This table serves the learner dashboard ("show me everything I have learned this year"), the leader dashboard ("show me my team's learning across all modes"), and the analytics layer ("what percentage of learning is experiential vs. formal?").

---

## 5. Missing Tables: Full DDL for All 11 Gaps

### GAP #1 & #11: Workplace-Based Activities

```sql
-- Configurable activity types
-- Seeded with SVV's specific types but extensible per tenant
CREATE TABLE activity_types (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code            VARCHAR(50) NOT NULL UNIQUE,
    -- 'befaring', 'hospitering', 'beredskapsovelse', 'simulatortrening',
    -- 'praktisk_prove', 'jobbobservasjon', 'egentrening', 'veiledning',
    -- 'ekstern_kurs', 'konferanse', 'mentoring'
    name_nb         VARCHAR(255) NOT NULL,           -- Norwegian bokmal name
    name_nn         VARCHAR(255),                    -- Norwegian nynorsk name
    name_en         VARCHAR(255),                    -- English name
    description     TEXT,
    learning_mode   VARCHAR(15) NOT NULL DEFAULT 'experiential'
                    CHECK (learning_mode IN ('formal', 'social', 'experiential')),
    requires_approval       BOOLEAN DEFAULT true,
    requires_evidence       BOOLEAN DEFAULT false,
    requires_checklist      BOOLEAN DEFAULT false,
    requires_sensor         BOOLEAN DEFAULT false,    -- For practical exams
    default_competency_id   UUID REFERENCES competencies(id),
    icon            VARCHAR(50),                     -- Icon identifier for UI
    sort_order      INT DEFAULT 0,
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Individual activity records
-- Each workplace activity completed by a learner
CREATE TABLE activity_records (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id),
    activity_type_id UUID NOT NULL REFERENCES activity_types(id),
    title           VARCHAR(500) NOT NULL,
    description     TEXT,
    occurred_at     TIMESTAMPTZ NOT NULL,
    duration_min    INT,
    location        VARCHAR(500),
    -- Approval workflow
    status          VARCHAR(20) NOT NULL DEFAULT 'draft',
    -- 'draft', 'submitted', 'pending_approval', 'approved', 'rejected', 'expired'
    submitted_at    TIMESTAMPTZ,
    approved_by     UUID REFERENCES users(id),
    approved_at     TIMESTAMPTZ,
    rejection_reason TEXT,
    -- Sensor/observer for practical exams and job observation
    observer_name   VARCHAR(255),
    observer_email  VARCHAR(255),
    sensor_user_id  UUID REFERENCES users(id),       -- Internal sensor if registered user
    sensor_confirmed_at TIMESTAMPTZ,
    -- Evidence
    evidence_urls   TEXT[],                           -- S3 paths to photos, PDFs, videos
    evidence_notes  TEXT,
    -- Competency linkage
    competency_id   UUID REFERENCES competencies(id),
    competency_level INT,
    -- Validity
    valid_until     DATE,                            -- Expiry for time-limited competencies
    -- Metadata
    metadata        JSONB DEFAULT '{}',              -- Extensible per activity type
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at      TIMESTAMPTZ                      -- Soft delete
);

CREATE INDEX idx_activity_records_user ON activity_records(user_id, occurred_at DESC);
CREATE INDEX idx_activity_records_type ON activity_records(activity_type_id);
CREATE INDEX idx_activity_records_status ON activity_records(status)
    WHERE status IN ('submitted', 'pending_approval');
CREATE INDEX idx_activity_records_approver ON activity_records(approved_by)
    WHERE approved_by IS NOT NULL;
CREATE INDEX idx_activity_records_competency ON activity_records(competency_id)
    WHERE competency_id IS NOT NULL;
CREATE INDEX idx_activity_records_valid_until ON activity_records(valid_until)
    WHERE valid_until IS NOT NULL AND deleted_at IS NULL;
```

### GAP #2: Activity Approval Workflows

```sql
-- Generic approval workflow engine
-- Used for workplace activities, assignment reviews, and content publishing
CREATE TABLE approval_workflows (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    entity_type     VARCHAR(50) NOT NULL,
    -- 'activity_record', 'assignment_submission', 'course_publish', 'content_page'
    steps           JSONB NOT NULL DEFAULT '[]',
    -- Array of steps: [{"order": 1, "approver_role": "leader", "required": true},
    --                  {"order": 2, "approver_role": "sensor", "required": false}]
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Individual approval requests
CREATE TABLE approval_requests (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    workflow_id     UUID REFERENCES approval_workflows(id),
    entity_type     VARCHAR(50) NOT NULL,
    entity_id       UUID NOT NULL,                   -- Polymorphic FK
    requested_by    UUID NOT NULL REFERENCES users(id),
    requested_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    current_step    INT NOT NULL DEFAULT 1,
    status          VARCHAR(20) NOT NULL DEFAULT 'pending',
    -- 'pending', 'approved', 'rejected', 'cancelled'
    completed_at    TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_approval_requests_entity ON approval_requests(entity_type, entity_id);
CREATE INDEX idx_approval_requests_status ON approval_requests(status)
    WHERE status = 'pending';

-- Individual approval decisions within a workflow
CREATE TABLE approval_decisions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    request_id      UUID NOT NULL REFERENCES approval_requests(id) ON DELETE CASCADE,
    step_number     INT NOT NULL,
    decided_by      UUID NOT NULL REFERENCES users(id),
    decision        VARCHAR(20) NOT NULL CHECK (decision IN ('approved', 'rejected', 'returned')),
    comment         TEXT,
    decided_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_approval_decisions_request ON approval_decisions(request_id);
```

### GAP #3: Checklists and Assessment Forms

```sql
-- Checklist/assessment form templates
-- Used for job observation, practical exams, and structured assessments
CREATE TABLE checklist_templates (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    activity_type_id UUID REFERENCES activity_types(id),
    -- Which activity type this checklist is for (e.g., 'jobbobservasjon')
    version         INT NOT NULL DEFAULT 1,
    items           JSONB NOT NULL,
    -- Structured checklist definition:
    -- [
    --   {
    --     "id": "uuid",
    --     "section": "Sikkerhet",
    --     "text": "Bruker riktig verneutstyr",
    --     "type": "pass_fail",           -- 'pass_fail', 'rating_1_5', 'yes_no', 'text', 'numeric'
    --     "required": true,
    --     "weight": 1.0,
    --     "guidance": "Sjekk hjelm, vernebriller, hansker"
    --   }
    -- ]
    scoring_method  VARCHAR(20) DEFAULT 'all_pass',
    -- 'all_pass' (every required item must pass),
    -- 'weighted_average' (calculate weighted score),
    -- 'minimum_score' (must reach minimum percentage)
    minimum_score   DECIMAL(5,2),                    -- For 'minimum_score' method
    is_active       BOOLEAN DEFAULT true,
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Completed checklist instances
CREATE TABLE checklist_responses (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    template_id     UUID NOT NULL REFERENCES checklist_templates(id),
    template_version INT NOT NULL,
    activity_record_id UUID REFERENCES activity_records(id) ON DELETE CASCADE,
    assessed_user_id UUID NOT NULL REFERENCES users(id), -- The learner being assessed
    assessor_id     UUID NOT NULL REFERENCES users(id),  -- The observer/sensor
    responses       JSONB NOT NULL,
    -- Responses matching template items:
    -- [
    --   {"item_id": "uuid", "value": "pass", "comment": "God praksis"},
    --   {"item_id": "uuid", "value": "fail", "comment": "Mangler hjelm"}
    -- ]
    overall_result  VARCHAR(20),                     -- 'pass', 'fail', 'conditional'
    overall_score   DECIMAL(5,2),
    assessor_comment TEXT,
    completed_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_checklist_responses_activity ON checklist_responses(activity_record_id);
CREATE INDEX idx_checklist_responses_assessed ON checklist_responses(assessed_user_id, completed_at DESC);
CREATE INDEX idx_checklist_responses_template ON checklist_responses(template_id);
```

### GAP #4 & #7: Learning Areas / Fagsider with Access Control

```sql
-- Learning areas (omrader/fagsider)
-- Content spaces with configurable access control per user group
CREATE TABLE learning_areas (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title           VARCHAR(500) NOT NULL,
    slug            VARCHAR(100) NOT NULL UNIQUE,     -- URL-friendly identifier
    description     TEXT,
    -- Branding
    hero_image_url  VARCHAR(500),
    icon            VARCHAR(50),
    color_theme     VARCHAR(7),                      -- Hex color
    -- Hierarchy
    parent_id       UUID REFERENCES learning_areas(id),
    sort_order      INT DEFAULT 0,
    -- Ownership
    owner_org_unit_id UUID REFERENCES org_units(id), -- Which division owns this area
    created_by      UUID REFERENCES users(id),
    -- Status
    status          VARCHAR(20) NOT NULL DEFAULT 'draft',
    -- 'draft', 'published', 'archived'
    published_at    TIMESTAMPTZ,
    -- Metadata
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at      TIMESTAMPTZ
);

CREATE INDEX idx_learning_areas_parent ON learning_areas(parent_id);
CREATE INDEX idx_learning_areas_owner ON learning_areas(owner_org_unit_id);
CREATE INDEX idx_learning_areas_status ON learning_areas(status)
    WHERE deleted_at IS NULL;

-- Access control for learning areas
CREATE TABLE learning_area_access (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    area_id         UUID NOT NULL REFERENCES learning_areas(id) ON DELETE CASCADE,
    -- Grant can be to a target_group, org_unit, role, or individual user
    grant_type      VARCHAR(20) NOT NULL,
    -- 'target_group', 'org_unit', 'role', 'user'
    grant_target_id UUID NOT NULL,                   -- FK depends on grant_type
    permission      VARCHAR(10) NOT NULL CHECK (permission IN ('read', 'edit', 'admin')),
    granted_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (area_id, grant_type, grant_target_id, permission)
);

CREATE INDEX idx_learning_area_access_area ON learning_area_access(area_id);
CREATE INDEX idx_learning_area_access_target ON learning_area_access(grant_type, grant_target_id);

-- Content items within learning areas
CREATE TABLE learning_area_items (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    area_id         UUID NOT NULL REFERENCES learning_areas(id) ON DELETE CASCADE,
    item_type       VARCHAR(30) NOT NULL,
    -- 'course', 'document', 'link', 'announcement', 'video', 'page'
    -- Polymorphic reference to the actual content
    course_id       UUID REFERENCES courses(id),     -- If item_type = 'course'
    page_id         UUID REFERENCES content_pages(id), -- If item_type = 'page'
    -- Direct content (for links, announcements)
    title           VARCHAR(500),
    url             VARCHAR(500),
    body            TEXT,                             -- Rich text for announcements
    -- Display
    sort_order      INT DEFAULT 0,
    is_featured     BOOLEAN DEFAULT false,
    is_pinned       BOOLEAN DEFAULT false,
    -- Status
    published_at    TIMESTAMPTZ,
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_learning_area_items_area ON learning_area_items(area_id, sort_order);
```

### GAP #5: Landing Page CMS Content Management

```sql
-- CMS pages for landing pages and content areas
CREATE TABLE content_pages (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    slug            VARCHAR(200) NOT NULL,           -- URL path
    title           VARCHAR(500) NOT NULL,
    page_type       VARCHAR(30) NOT NULL,
    -- 'landing_internal', 'landing_external', 'info_page', 'area_page'
    -- Page content as structured blocks (block editor pattern)
    blocks          JSONB NOT NULL DEFAULT '[]',
    -- Array of content blocks:
    -- [
    --   {"type": "hero", "image_url": "/img/hero.jpg", "title": "Velkommen", "subtitle": "..."},
    --   {"type": "text", "content": "<p>Rich text HTML</p>"},
    --   {"type": "featured_courses", "course_ids": ["uuid1", "uuid2"], "layout": "carousel"},
    --   {"type": "announcements", "max_items": 5},
    --   {"type": "image", "url": "/img/photo.jpg", "alt": "Description", "caption": "..."},
    --   {"type": "link_list", "links": [{"title": "...", "url": "...", "icon": "..."}]}
    -- ]
    -- Versioning
    version         INT NOT NULL DEFAULT 1,
    is_published    BOOLEAN DEFAULT false,
    published_at    TIMESTAMPTZ,
    published_by    UUID REFERENCES users(id),
    -- Targeting
    audience        VARCHAR(20) DEFAULT 'all',
    -- 'all', 'internal', 'external'
    org_unit_id     UUID REFERENCES org_units(id),   -- Scoped to org unit (for division pages)
    -- Metadata
    meta_title      VARCHAR(200),
    meta_description VARCHAR(500),
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at      TIMESTAMPTZ
);

CREATE UNIQUE INDEX idx_content_pages_slug_audience
    ON content_pages(slug, audience)
    WHERE deleted_at IS NULL AND is_published = true;

-- Page version history for undo/restore
CREATE TABLE content_page_versions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    page_id         UUID NOT NULL REFERENCES content_pages(id) ON DELETE CASCADE,
    version         INT NOT NULL,
    blocks          JSONB NOT NULL,
    edited_by       UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (page_id, version)
);
```

### GAP #6: Custom Event Registration Forms

```sql
-- Custom registration form definitions for events
CREATE TABLE event_registration_forms (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_id        UUID REFERENCES events(id) ON DELETE CASCADE,
    -- Can also be a template not attached to a specific event
    is_template     BOOLEAN DEFAULT false,
    template_name   VARCHAR(255),
    fields          JSONB NOT NULL,
    -- Array of form field definitions:
    -- [
    --   {
    --     "id": "uuid",
    --     "name": "allergier",
    --     "label_nb": "Allergier eller matintoleranser",
    --     "label_nn": "Allergiar eller matintoleransar",
    --     "type": "text",                 -- 'text', 'textarea', 'dropdown', 'checkbox',
    --                                     -- 'radio', 'email', 'phone', 'date'
    --     "required": false,
    --     "options": null,                -- For dropdown/radio: ["Glutenfri", "Laktosefri", ...]
    --     "placeholder": "F.eks. nøtteallergi",
    --     "sort_order": 1
    --   },
    --   {
    --     "id": "uuid",
    --     "name": "overnatting",
    --     "label_nb": "Trenger du overnatting?",
    --     "type": "radio",
    --     "required": true,
    --     "options": ["Ja", "Nei"],
    --     "sort_order": 2
    --   },
    --   {
    --     "id": "uuid",
    --     "name": "tilgjengelighet",
    --     "label_nb": "Tilgjengelighetsbehov",
    --     "type": "textarea",
    --     "required": false,
    --     "sort_order": 3
    --   }
    -- ]
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_event_registration_forms_event ON event_registration_forms(event_id);

-- Individual registration form responses
CREATE TABLE event_registration_responses (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    form_id         UUID NOT NULL REFERENCES event_registration_forms(id) ON DELETE CASCADE,
    event_id        UUID NOT NULL REFERENCES events(id),
    user_id         UUID NOT NULL REFERENCES users(id),
    responses       JSONB NOT NULL,
    -- Key-value responses matching form fields:
    -- {"allergier": "Nøtteallergi", "overnatting": "Ja", "tilgjengelighet": null}
    submitted_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (form_id, user_id)
);

CREATE INDEX idx_event_reg_responses_event ON event_registration_responses(event_id);
CREATE INDEX idx_event_reg_responses_user ON event_registration_responses(user_id);
```

### GAP #7: Per-Group Certification Validity Durations

```sql
-- Target group definitions
-- Reusable audience segments used for mandatory assignments,
-- certification rules, report filtering, and content targeting
CREATE TABLE target_groups (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    -- Rule-based membership (evaluated dynamically)
    rules           JSONB NOT NULL,
    -- Membership rules:
    -- {
    --   "operator": "AND",              -- "AND" or "OR"
    --   "conditions": [
    --     {"field": "org_unit_id", "op": "in_subtree", "value": "uuid-of-division"},
    --     {"field": "role", "op": "eq", "value": "feltarbeider"},
    --     {"field": "user_type", "op": "in", "value": ["internal", "consultant"]},
    --     {"field": "metadata.job_family", "op": "eq", "value": "arbeidsvarsling"}
    --   ]
    -- }
    -- Cached member count (updated periodically)
    cached_member_count INT DEFAULT 0,
    last_evaluated_at TIMESTAMPTZ,
    created_by      UUID REFERENCES users(id),
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Static target group membership (for manually maintained groups)
CREATE TABLE target_group_members (
    target_group_id UUID NOT NULL REFERENCES target_groups(id) ON DELETE CASCADE,
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    added_by        UUID REFERENCES users(id),
    added_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (target_group_id, user_id)
);

CREATE INDEX idx_target_group_members_user ON target_group_members(user_id);

-- Course-target-group rules
-- Links courses to target groups with specific mandatory/optional and certification rules
CREATE TABLE course_target_rules (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id       UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
    target_group_id UUID NOT NULL REFERENCES target_groups(id),
    is_mandatory    BOOLEAN NOT NULL DEFAULT false,
    -- Certification configuration per target group
    cert_validity_months INT,                        -- NULL = no expiry
    cert_template_id UUID REFERENCES certificate_templates(id),
    -- Auto-enrollment
    auto_enroll     BOOLEAN DEFAULT false,
    -- Due date rules
    due_within_days INT,                             -- Days after enrollment to complete
    -- Recertification
    recert_reminder_days INT DEFAULT 30,             -- Remind N days before expiry
    recert_course_id UUID REFERENCES courses(id),    -- Different course for recertification?
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (course_id, target_group_id)
);

CREATE INDEX idx_course_target_rules_course ON course_target_rules(course_id);
CREATE INDEX idx_course_target_rules_group ON course_target_rules(target_group_id);
```

This solves GAP #3: the same course can have `cert_validity_months = 24` for target group "Arbeidsvarsling Klasse 2" and `cert_validity_months = 12` for "Arbeidsvarsling Klasse 3", with `is_mandatory = true` for one and `is_mandatory = false` for the other.

### GAP #8: Granular Per-Course Report Access Delegation

```sql
-- Object-level permission grants
-- Allows division admin to grant specific users access to reports for specific courses
CREATE TABLE object_permissions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    -- Who is granted access
    grantee_user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    -- What object type
    object_type     VARCHAR(30) NOT NULL,
    -- 'course', 'learning_path', 'learning_area', 'event'
    -- Which specific object
    object_id       UUID NOT NULL,
    -- What permission
    permission      VARCHAR(30) NOT NULL,
    -- 'view_reports', 'manage_enrollments', 'edit_content', 'view_content'
    -- Who granted it and scope
    granted_by      UUID NOT NULL REFERENCES users(id),
    -- Optional scope restriction
    org_unit_id     UUID REFERENCES org_units(id),   -- Limit to this org unit's data
    -- Metadata
    expires_at      TIMESTAMPTZ,                     -- Optional expiry
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (grantee_user_id, object_type, object_id, permission)
);

CREATE INDEX idx_object_permissions_grantee ON object_permissions(grantee_user_id, object_type)
    WHERE is_active = true;
CREATE INDEX idx_object_permissions_object ON object_permissions(object_type, object_id)
    WHERE is_active = true;
```

### GAP #9: Custom Role Configuration

```sql
-- Permission definitions
-- Granular permissions that can be assembled into custom roles
CREATE TABLE permissions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code            VARCHAR(100) NOT NULL UNIQUE,
    -- 'courses.view', 'courses.create', 'courses.edit', 'courses.delete',
    -- 'courses.manage_enrollments', 'courses.view_reports',
    -- 'users.view', 'users.manage', 'users.impersonate',
    -- 'learning_areas.create', 'learning_areas.edit',
    -- 'content_pages.edit', 'content_pages.publish',
    -- 'reports.view_own_team', 'reports.view_division', 'reports.view_all',
    -- 'settings.manage', 'roles.manage', 'integrations.manage'
    name_nb         VARCHAR(255) NOT NULL,
    name_en         VARCHAR(255),
    description     TEXT,
    category        VARCHAR(50) NOT NULL,
    -- 'courses', 'users', 'reports', 'content', 'settings', 'integrations'
    is_system       BOOLEAN DEFAULT false,           -- System permissions cannot be revoked from superadmin
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Role-permission mapping
-- Roles now reference the permissions table instead of using JSONB
CREATE TABLE role_permissions (
    role_id         UUID NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    permission_id   UUID NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
    PRIMARY KEY (role_id, permission_id)
);

CREATE INDEX idx_role_permissions_role ON role_permissions(role_id);
```

This replaces the existing `roles.permissions JSONB` column with a proper many-to-many relationship. The `roles` table is updated to add `is_custom BOOLEAN DEFAULT false` and `is_system BOOLEAN DEFAULT false` columns to distinguish system roles from custom roles.

### GAP #10: Assignment Submissions

```sql
-- Assignments (the definition -- what learners must submit)
CREATE TABLE assignments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id       UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
    title           VARCHAR(500) NOT NULL,
    description     TEXT,
    instructions    TEXT,                             -- Rich text instructions
    -- Submission configuration
    submission_types VARCHAR(30)[] NOT NULL DEFAULT ARRAY['file_upload'],
    -- 'file_upload', 'text_entry', 'url', 'media_recording'
    allowed_file_types VARCHAR(20)[],                -- e.g., ['pdf', 'docx', 'mp4', 'mp3']
    max_file_size_mb INT DEFAULT 100,
    max_files       INT DEFAULT 5,
    -- Grading
    grading_type    VARCHAR(20) NOT NULL DEFAULT 'pass_fail',
    -- 'pass_fail', 'points', 'percentage', 'rubric', 'ungraded'
    max_points      DECIMAL(7,2),
    passing_points  DECIMAL(7,2),
    rubric_id       UUID REFERENCES checklist_templates(id),  -- Reuse checklist as rubric
    -- Resubmission
    max_attempts    INT DEFAULT 1,
    allow_resubmit  BOOLEAN DEFAULT false,
    -- Deadlines
    due_at          TIMESTAMPTZ,
    lock_at         TIMESTAMPTZ,                     -- No submissions after this
    -- Peer review
    peer_review_enabled BOOLEAN DEFAULT false,
    peer_review_count   INT DEFAULT 1,               -- How many peers review each submission
    -- Status
    position        INT DEFAULT 0,                   -- Order within course
    is_published    BOOLEAN DEFAULT false,
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_assignments_course ON assignments(course_id);

-- Assignment submissions (each learner's work)
CREATE TABLE assignment_submissions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    assignment_id   UUID NOT NULL REFERENCES assignments(id) ON DELETE CASCADE,
    user_id         UUID NOT NULL REFERENCES users(id),
    attempt_number  INT NOT NULL DEFAULT 1,
    -- Submission content
    submission_type VARCHAR(30) NOT NULL,
    -- 'file_upload', 'text_entry', 'url', 'media_recording'
    body            TEXT,                             -- For text_entry
    url             VARCHAR(500),                    -- For url type
    file_urls       TEXT[],                          -- S3 paths for file uploads
    file_names      VARCHAR(255)[],                  -- Original file names
    file_sizes      BIGINT[],                        -- File sizes in bytes
    -- Workflow
    workflow_state  VARCHAR(20) NOT NULL DEFAULT 'draft',
    -- 'draft', 'submitted', 'under_review', 'graded', 'returned', 'resubmit_requested'
    submitted_at    TIMESTAMPTZ,
    -- Grading
    graded_by       UUID REFERENCES users(id),
    graded_at       TIMESTAMPTZ,
    score           DECIMAL(7,2),
    grade           VARCHAR(20),                     -- 'pass', 'fail', or letter grade
    grading_comment TEXT,
    rubric_assessment JSONB,                         -- Structured rubric scores if rubric-graded
    -- Metadata
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_submissions_assignment ON assignment_submissions(assignment_id);
CREATE INDEX idx_submissions_user ON assignment_submissions(user_id, assignment_id);
CREATE INDEX idx_submissions_state ON assignment_submissions(workflow_state)
    WHERE workflow_state IN ('submitted', 'under_review');
CREATE UNIQUE INDEX idx_submissions_unique_attempt
    ON assignment_submissions(assignment_id, user_id, attempt_number);
```

### GAP #11: Additional Workplace Activity Support (covered by GAP #1/#11 tables above)

The `activity_types`, `activity_records`, `checklist_templates`, and `checklist_responses` tables from GAP #1 and GAP #3 cover all workplace-based activity types: befaring, hospitering, beredskapsovelser, simulatortrening, praktiske prover, jobbobservasjon, egentrening, and veiledning.

---

## 6. Performance Considerations

### 6.1 Indexing strategy

**Principle**: Every foreign key gets a B-tree index. Every common filter/sort combination gets a composite index. Partial indexes for common WHERE clauses.

```sql
-- ============================================================
-- CORE ENTITY INDEXES
-- ============================================================

-- Users
CREATE INDEX idx_users_org_unit ON users(org_unit_id) WHERE is_active = true;
CREATE INDEX idx_users_manager ON users(manager_id) WHERE manager_id IS NOT NULL;
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_external_id ON users(external_id) WHERE external_id IS NOT NULL;
CREATE INDEX idx_users_user_type ON users(user_type) WHERE is_active = true;
CREATE INDEX idx_users_search ON users USING GIN(search_vector);

-- Org units (tree traversal)
CREATE INDEX idx_org_units_parent ON org_units(parent_id);
CREATE INDEX idx_org_units_level ON org_units(level);

-- Courses
CREATE INDEX idx_courses_search ON courses USING GIN(search_vector);
CREATE INDEX idx_courses_status ON courses(status, published_at DESC) WHERE deleted_at IS NULL;
CREATE INDEX idx_courses_category ON courses(category_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_courses_type ON courses(type) WHERE status = 'published' AND deleted_at IS NULL;
CREATE INDEX idx_courses_created_by ON courses(created_by);
-- pgvector index for semantic search
CREATE INDEX idx_courses_embedding ON courses USING hnsw (embedding vector_cosine_ops)
    WITH (m = 16, ef_construction = 64);

-- Course contents
CREATE INDEX idx_course_contents_course ON course_contents(course_id);

-- Enrollments
CREATE INDEX idx_enrollments_user_status ON enrollments(user_id, status);
CREATE INDEX idx_enrollments_course ON enrollments(course_id);
CREATE INDEX idx_enrollments_due_date ON enrollments(due_date)
    WHERE status NOT IN ('completed', 'withdrawn') AND due_date IS NOT NULL;
CREATE INDEX idx_enrollments_assigned_by ON enrollments(assigned_by)
    WHERE assigned_by IS NOT NULL;

-- Completions (partitioned -- indexes created on each partition)
CREATE INDEX idx_completions_user ON completions(user_id, completed_at DESC);
CREATE INDEX idx_completions_course ON completions(course_id, completed_at DESC);
CREATE INDEX idx_completions_valid_until ON completions(valid_until)
    WHERE valid_until IS NOT NULL;

-- SCORM runtime
CREATE INDEX idx_scorm_runtime_enrollment ON scorm_runtime_data(enrollment_id);

-- Learning paths
CREATE INDEX idx_lp_items_path ON learning_path_items(path_id, position);
CREATE INDEX idx_lp_items_course ON learning_path_items(course_id);
CREATE INDEX idx_lp_enrollments_user ON learning_path_enrollments(user_id);
CREATE INDEX idx_lp_enrollments_path ON learning_path_enrollments(path_id);

-- Competencies
CREATE INDEX idx_competencies_framework ON competencies(framework_id);
CREATE INDEX idx_competencies_parent ON competencies(parent_id);
CREATE INDEX idx_user_competencies_user ON user_competencies(user_id);
CREATE INDEX idx_user_competencies_competency ON user_competencies(competency_id);
CREATE INDEX idx_user_competencies_expires ON user_competencies(expires_at)
    WHERE expires_at IS NOT NULL;

-- Certificates
CREATE INDEX idx_certificates_user ON certificates(user_id);
CREATE INDEX idx_certificates_course ON certificates(course_id);
CREATE INDEX idx_certificates_verification ON certificates(verification_code);
CREATE INDEX idx_certificates_valid_until ON certificates(valid_until)
    WHERE revoked = false;
CREATE INDEX idx_certificates_competency ON certificates(competency_id);

-- Events
CREATE INDEX idx_events_course ON events(course_id);
CREATE INDEX idx_events_instructor ON events(instructor_id);
CREATE INDEX idx_events_starts_at ON events(starts_at)
    WHERE status = 'scheduled';

-- Notifications
CREATE INDEX idx_notifications_user_unread ON notifications(user_id, created_at DESC)
    WHERE is_read = false;

-- Audit log (partitioned)
CREATE INDEX idx_audit_log_entity ON audit_log(entity_type, entity_id);
CREATE INDEX idx_audit_log_user ON audit_log(user_id, created_at DESC);

-- xAPI statements (partitioned)
CREATE INDEX idx_xapi_verb ON xapi_statements(verb_id, timestamp DESC);
CREATE INDEX idx_xapi_actor ON xapi_statements(actor_id, timestamp DESC);
CREATE INDEX idx_xapi_activity ON xapi_statements(activity_id, timestamp DESC);
CREATE INDEX idx_xapi_registration ON xapi_statements(registration)
    WHERE registration IS NOT NULL;

-- Learning records (partitioned)
CREATE INDEX idx_learning_records_user ON learning_records(user_id, occurred_at DESC);
CREATE INDEX idx_learning_records_mode ON learning_records(learning_mode, occurred_at DESC);
CREATE INDEX idx_learning_records_source ON learning_records(source_type, source_id);
CREATE INDEX idx_learning_records_status ON learning_records(status)
    WHERE status = 'pending_approval';
```

### 6.2 Partitioning strategy

**Tables to partition and method:**

| Table | Partition Method | Key | Granularity | Rationale |
|-------|-----------------|-----|-------------|-----------|
| `completions` | RANGE | `completed_at` | Yearly | Moderate write volume, long retention, common date-range queries |
| `audit_log` | RANGE | `created_at` | Monthly | High write volume, retention policy (24 months), rarely queried beyond recent months |
| `xapi_statements` | RANGE | `timestamp` | Monthly | High volume from SCORM/xAPI content, analytics queries always time-bounded |
| `learning_records` | RANGE | `occurred_at` | Yearly | Moderate volume, long retention, dashboard queries span current year |
| `notifications` | RANGE | `created_at` | Monthly | High volume, short retention (90 days), old partitions dropped |
| `scorm_runtime_data` | None | -- | -- | Low volume per enrollment, no time-series pattern |

**Partition creation automation:**

```sql
-- Function to auto-create future partitions
CREATE OR REPLACE FUNCTION create_monthly_partition(
    parent_table TEXT, partition_date DATE
) RETURNS VOID AS $$
DECLARE
    partition_name TEXT;
    start_date DATE;
    end_date DATE;
BEGIN
    start_date := date_trunc('month', partition_date);
    end_date := start_date + INTERVAL '1 month';
    partition_name := parent_table || '_' || to_char(start_date, 'YYYY_MM');

    EXECUTE format(
        'CREATE TABLE IF NOT EXISTS %I PARTITION OF %I
         FOR VALUES FROM (%L) TO (%L)',
        partition_name, parent_table, start_date, end_date
    );
END;
$$ LANGUAGE plpgsql;

-- Called by a scheduled job (pg_cron or Celery beat) to create partitions 3 months ahead
```

### 6.3 Common query patterns and optimization

**Query 1: Learner dashboard -- "My learning"**

This is the highest-frequency query in the system. Every logged-in user hits this on every visit.

```sql
-- Optimized: uses composite index on enrollments(user_id, status)
-- and partial index on notifications
EXPLAIN ANALYZE
SELECT
    e.id, e.status, e.progress_pct, e.due_date, e.enrolled_at,
    c.title, c.type, c.format, c.duration_min,
    c.short_desc
FROM enrollments e
JOIN courses c ON c.id = e.course_id
WHERE e.user_id = $1
  AND e.status IN ('enrolled', 'in_progress')
ORDER BY
    CASE WHEN e.due_date IS NOT NULL AND e.due_date < CURRENT_DATE + INTERVAL '7 days'
         THEN 0 ELSE 1 END,
    e.due_date ASC NULLS LAST,
    e.enrolled_at DESC;
```

**Query 2: Leader dashboard -- "Team learning status"**

```sql
-- Requires: idx_users_manager, idx_enrollments_user_status
-- For leaders with large teams, this benefits from a materialized view
SELECT
    u.id, u.display_name, u.email,
    COUNT(e.id) FILTER (WHERE e.status = 'completed') AS completed_count,
    COUNT(e.id) FILTER (WHERE e.status IN ('enrolled', 'in_progress')) AS active_count,
    COUNT(e.id) FILTER (WHERE e.due_date < CURRENT_DATE AND e.status != 'completed') AS overdue_count,
    MIN(cert.valid_until) FILTER (WHERE cert.valid_until IS NOT NULL AND cert.revoked = false)
        AS earliest_cert_expiry
FROM users u
LEFT JOIN enrollments e ON e.user_id = u.id
LEFT JOIN certificates cert ON cert.user_id = u.id
WHERE u.manager_id = $1  -- Or: u.org_unit_id IN (subtree of leader's org unit)
  AND u.is_active = true
GROUP BY u.id
ORDER BY overdue_count DESC, earliest_cert_expiry ASC NULLS LAST;
```

**Query 3: Certification expiry report**

```sql
-- Requires: idx_certificates_valid_until partial index
-- This runs as a scheduled job for notification generation
SELECT
    c.id, c.certificate_no, c.valid_until,
    u.id AS user_id, u.display_name, u.email, u.manager_id,
    comp.name AS competency_name,
    crs.title AS course_title,
    ctr.recert_course_id,
    ctr.recert_reminder_days
FROM certificates c
JOIN users u ON u.id = c.user_id
JOIN competencies comp ON comp.id = c.competency_id
JOIN courses crs ON crs.id = c.course_id
LEFT JOIN course_target_rules ctr ON ctr.course_id = c.course_id
WHERE c.revoked = false
  AND c.valid_until BETWEEN CURRENT_DATE AND CURRENT_DATE + INTERVAL '90 days'
ORDER BY c.valid_until ASC;
```

**Query 4: 70/20/10 analytics -- "Learning mode distribution"**

```sql
-- Requires: idx_learning_records_mode, partitioned by occurred_at
SELECT
    learning_mode,
    COUNT(*) AS record_count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) AS percentage,
    SUM(duration_min) AS total_minutes,
    COUNT(DISTINCT user_id) AS unique_learners
FROM learning_records
WHERE occurred_at >= date_trunc('year', CURRENT_DATE)
  AND status IN ('completed', 'approved')
GROUP BY learning_mode
ORDER BY
    CASE learning_mode
        WHEN 'experiential' THEN 1
        WHEN 'social' THEN 2
        WHEN 'formal' THEN 3
    END;
```

**Query 5: Semantic course search with pgvector**

```sql
-- Search for courses semantically similar to a query
-- embedding is generated by the application using Multilingual E5
SELECT
    c.id, c.title, c.short_desc, c.type, c.format,
    1 - (c.embedding <=> $1::vector) AS similarity_score
FROM courses c
WHERE c.status = 'published'
  AND c.deleted_at IS NULL
  AND c.embedding IS NOT NULL
  AND 1 - (c.embedding <=> $1::vector) > 0.5  -- Similarity threshold
ORDER BY c.embedding <=> $1::vector
LIMIT 20;
```

### 6.4 Materialized views for reporting

```sql
-- Materialized view: team learning summary
-- Refreshed every 15 minutes by a Celery beat task
CREATE MATERIALIZED VIEW mv_team_learning_summary AS
SELECT
    u.org_unit_id,
    u.manager_id,
    COUNT(DISTINCT u.id) AS team_size,
    COUNT(DISTINCT e.id) FILTER (WHERE e.status = 'completed') AS total_completions,
    COUNT(DISTINCT e.id) FILTER (
        WHERE e.due_date < CURRENT_DATE AND e.status NOT IN ('completed', 'withdrawn')
    ) AS total_overdue,
    COUNT(DISTINCT c2.id) FILTER (
        WHERE c2.valid_until < CURRENT_DATE + INTERVAL '30 days' AND c2.revoked = false
    ) AS expiring_certs_30d,
    COUNT(DISTINCT lr.id) FILTER (
        WHERE lr.learning_mode = 'experiential'
        AND lr.occurred_at >= date_trunc('quarter', CURRENT_DATE)
    ) AS experiential_activities_qtd
FROM users u
LEFT JOIN enrollments e ON e.user_id = u.id
LEFT JOIN certificates c2 ON c2.user_id = u.id
LEFT JOIN learning_records lr ON lr.user_id = u.id
WHERE u.is_active = true
GROUP BY u.org_unit_id, u.manager_id;

CREATE UNIQUE INDEX ON mv_team_learning_summary(org_unit_id, manager_id);

-- Refresh command (called by Celery beat)
-- REFRESH MATERIALIZED VIEW CONCURRENTLY mv_team_learning_summary;
```

---

## 7. Migration Strategy

### 7.1 Tooling: Alembic with multi-schema support

**Directory structure:**

```
backend/
+-- alembic/
|   +-- versions/              -- Migration files (shared across all schemas)
|   |   +-- 001_initial_schema.py
|   |   +-- 002_add_activity_types.py
|   |   +-- 003_add_learning_areas.py
|   +-- env.py                 -- Multi-schema migration runner
|   +-- script.py.mako         -- Migration template
+-- alembic.ini
```

**Key principles:**

1. **One migration set, N schemas**: Migrations are authored once and applied to all tenant schemas. Never write tenant-specific migrations.

2. **Forward-only with rollback scripts**: Every migration has both `upgrade()` and `downgrade()` functions. But in production, we prefer forward fixes (new migration to fix issues) over rollbacks.

3. **No-lock migrations**: Large table modifications use patterns that avoid `ACCESS EXCLUSIVE` locks:
   - `ALTER TABLE ADD COLUMN ... DEFAULT` (PostgreSQL 11+ does not rewrite the table)
   - `CREATE INDEX CONCURRENTLY` (does not lock the table)
   - Avoid `ALTER TABLE ALTER COLUMN TYPE` on large tables; instead, add new column, backfill, swap

4. **Migration tracking per schema**: The `shared.migration_history` table tracks which version each tenant is at, enabling partial rollouts and canary migrations.

### 7.2 Safe migration patterns

```python
# Example: Adding a new column with backfill (no locks)
def upgrade():
    # Step 1: Add column with default (instant, no rewrite in PG 11+)
    op.add_column('courses', sa.Column('embedding', sa.LargeBinary(), nullable=True))

    # Step 2: Create index concurrently (outside transaction)
    # NOTE: Alembic cannot run CONCURRENTLY inside a transaction
    # This must be run with non_transactional_ddl=True
    op.execute("""
        CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_courses_embedding
        ON courses USING hnsw (embedding vector_cosine_ops)
    """)

def downgrade():
    op.drop_index('idx_courses_embedding', 'courses')
    op.drop_column('courses', 'embedding')
```

```python
# Example: Renaming a column safely (zero downtime)
def upgrade():
    # Step 1: Add new column
    op.add_column('courses', sa.Column('short_description', sa.String(500)))

    # Step 2: Backfill (in batches for large tables)
    op.execute("""
        UPDATE courses SET short_description = short_desc
        WHERE short_desc IS NOT NULL
    """)

    # Step 3: Application code reads from BOTH columns during transition
    # Step 4: Next migration drops old column after application is fully deployed

def downgrade():
    op.drop_column('courses', 'short_description')
```

### 7.3 Data migration from Kilden

The existing Kilden LMS (by Storyboard AS) has:
- 297 active e-learning courses
- 18,500 completions
- 318 classroom courses/events

Migration approach:
1. Export Kilden data via API or database export
2. Transform to our schema using Python ETL scripts
3. Load into `tenant_svv` schema
4. Verify counts and data integrity
5. Run parallel operation period (both systems active)
6. Cut over

Key mapping challenges:
- Kilden user IDs to our UUIDs (use `external_id` column)
- Kilden course structures to our `courses` + `course_contents` model
- Historical completions to our `completions` table (preserve original dates)
- Kilden certificates to our `certificates` table (preserve certificate numbers)

---

## 8. Full Revised Schema DDL

This is the complete, production-ready schema for one tenant schema. All tables, constraints, indexes, and comments are included.

### 8.1 Extensions and shared schema

```sql
-- ============================================================
-- EXTENSIONS (in public schema, shared across all tenants)
-- ============================================================
CREATE EXTENSION IF NOT EXISTS "pgcrypto";           -- gen_random_uuid()
CREATE EXTENSION IF NOT EXISTS "pg_trgm";            -- Trigram similarity for fuzzy search
CREATE EXTENSION IF NOT EXISTS "btree_gist";         -- GiST indexes for exclusion constraints
CREATE EXTENSION IF NOT EXISTS "vector";             -- pgvector for semantic search

-- ============================================================
-- SHARED SCHEMA (platform-level, not tenant-scoped)
-- ============================================================
CREATE SCHEMA IF NOT EXISTS shared;

CREATE TABLE shared.tenants (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    slug            VARCHAR(50) NOT NULL UNIQUE,      -- 'svv', 'nve', etc.
    name            VARCHAR(255) NOT NULL,             -- 'Statens vegvesen'
    schema_name     VARCHAR(100) NOT NULL UNIQUE,      -- 'tenant_svv'
    -- IAM configuration
    iam_provider    VARCHAR(30) NOT NULL DEFAULT 'forgerock',
    iam_config      JSONB NOT NULL DEFAULT '{}',
    -- Features
    features        JSONB NOT NULL DEFAULT '{}',
    -- Branding
    branding        JSONB NOT NULL DEFAULT '{}',
    -- Settings
    default_locale  VARCHAR(5) NOT NULL DEFAULT 'nb',
    data_retention_days INT NOT NULL DEFAULT 2555,     -- ~7 years
    timezone        VARCHAR(50) NOT NULL DEFAULT 'Europe/Oslo',
    -- Status
    is_active       BOOLEAN NOT NULL DEFAULT true,
    provisioned_at  TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE shared.migration_history (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL REFERENCES shared.tenants(id),
    schema_name     VARCHAR(100) NOT NULL,
    migration_version VARCHAR(50) NOT NULL,
    applied_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    duration_ms     INT,
    success         BOOLEAN NOT NULL DEFAULT true,
    error_message   TEXT,
    UNIQUE (schema_name, migration_version)
);

CREATE TABLE shared.platform_admins (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email           VARCHAR(255) NOT NULL UNIQUE,
    display_name    VARCHAR(255) NOT NULL,
    password_hash   VARCHAR(255),                     -- Only for emergency access
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### 8.2 Tenant schema -- complete DDL

```sql
-- ============================================================
-- TENANT SCHEMA
-- All tables below are created per tenant schema.
-- Example: CREATE SCHEMA tenant_svv; SET search_path TO tenant_svv;
-- ============================================================

-- ============================================================
-- UTILITY: updated_at trigger function
-- ============================================================
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- ============================================================
-- 1. ORGANIZATIONAL STRUCTURE
-- ============================================================

-- Organizational hierarchy
CREATE TABLE org_units (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    code            VARCHAR(50),                      -- Internal org code
    parent_id       UUID REFERENCES org_units(id),
    external_id     VARCHAR(255),                     -- ServiceNow org ID
    level           INT NOT NULL DEFAULT 0,
    -- 0 = organization, 1 = division, 2 = department, 3 = section, 4 = team
    path            TEXT,                             -- Materialized path: '/uuid1/uuid2/uuid3'
    sort_order      INT DEFAULT 0,
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_org_units_parent ON org_units(parent_id);
CREATE INDEX idx_org_units_level ON org_units(level);
CREATE INDEX idx_org_units_path ON org_units USING GIST (path gist_trgm_ops);
CREATE INDEX idx_org_units_external ON org_units(external_id) WHERE external_id IS NOT NULL;

-- ============================================================
-- 2. USERS
-- ============================================================

CREATE TABLE users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    external_id     VARCHAR(255) UNIQUE,              -- IdP subject identifier
    scim_id         VARCHAR(255) UNIQUE,              -- SCIM resource ID
    email           VARCHAR(255),
    display_name    VARCHAR(255) NOT NULL,
    given_name      VARCHAR(100),
    family_name     VARCHAR(100),
    pid_hash        VARCHAR(128),                     -- SHA-512 hashed national ID (external users)
    user_type       VARCHAR(20) NOT NULL DEFAULT 'internal'
                    CHECK (user_type IN ('internal', 'external', 'consultant', 'service_account')),
    locale          VARCHAR(5) NOT NULL DEFAULT 'nb',
    -- Organization
    org_unit_id     UUID REFERENCES org_units(id),
    manager_id      UUID REFERENCES users(id),
    job_title       VARCHAR(255),
    department      VARCHAR(255),
    -- Profile
    phone           VARCHAR(30),
    avatar_url      VARCHAR(500),
    metadata        JSONB DEFAULT '{}',               -- Extensible profile fields (job_family, etc.)
    -- Search
    search_vector   TSVECTOR,
    -- Status
    is_active       BOOLEAN NOT NULL DEFAULT true,
    last_login_at   TIMESTAMPTZ,
    deactivated_at  TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at      TIMESTAMPTZ                       -- Soft delete
);

CREATE INDEX idx_users_email ON users(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_external_id ON users(external_id) WHERE external_id IS NOT NULL;
CREATE INDEX idx_users_org_unit ON users(org_unit_id) WHERE is_active = true AND deleted_at IS NULL;
CREATE INDEX idx_users_manager ON users(manager_id) WHERE manager_id IS NOT NULL;
CREATE INDEX idx_users_user_type ON users(user_type) WHERE is_active = true AND deleted_at IS NULL;
CREATE INDEX idx_users_pid_hash ON users(pid_hash) WHERE pid_hash IS NOT NULL;
CREATE INDEX idx_users_search ON users USING GIN(search_vector);

CREATE TRIGGER trg_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- ============================================================
-- 3. ROLES & PERMISSIONS (RBAC)
-- ============================================================

CREATE TABLE permissions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code            VARCHAR(100) NOT NULL UNIQUE,
    name_nb         VARCHAR(255) NOT NULL,
    name_en         VARCHAR(255),
    description     TEXT,
    category        VARCHAR(50) NOT NULL,
    is_system       BOOLEAN DEFAULT false,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE roles (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(100) NOT NULL UNIQUE,
    description     TEXT,
    is_system       BOOLEAN DEFAULT false,            -- System roles cannot be deleted
    is_custom       BOOLEAN DEFAULT false,            -- Custom roles created by admins
    sort_order      INT DEFAULT 0,
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE role_permissions (
    role_id         UUID NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    permission_id   UUID NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
    PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE user_roles (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role_id         UUID NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    org_unit_id     UUID REFERENCES org_units(id),    -- NULL = global scope
    granted_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    granted_by      UUID REFERENCES users(id),
    expires_at      TIMESTAMPTZ,
    is_active       BOOLEAN DEFAULT true
);

-- Prevent duplicate role assignments (handling NULL org_unit_id)
CREATE UNIQUE INDEX idx_user_roles_unique
    ON user_roles(user_id, role_id, COALESCE(org_unit_id, '00000000-0000-0000-0000-000000000000'))
    WHERE is_active = true;

CREATE INDEX idx_user_roles_user ON user_roles(user_id) WHERE is_active = true;
CREATE INDEX idx_user_roles_role ON user_roles(role_id);
CREATE INDEX idx_user_roles_org_unit ON user_roles(org_unit_id) WHERE org_unit_id IS NOT NULL;

-- Object-level permissions (for GAP #8: granular report access)
CREATE TABLE object_permissions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    grantee_user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    object_type     VARCHAR(30) NOT NULL,
    object_id       UUID NOT NULL,
    permission      VARCHAR(30) NOT NULL,
    granted_by      UUID NOT NULL REFERENCES users(id),
    org_unit_id     UUID REFERENCES org_units(id),
    expires_at      TIMESTAMPTZ,
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (grantee_user_id, object_type, object_id, permission)
);

CREATE INDEX idx_object_perms_grantee ON object_permissions(grantee_user_id, object_type)
    WHERE is_active = true;
CREATE INDEX idx_object_perms_object ON object_permissions(object_type, object_id)
    WHERE is_active = true;

-- ============================================================
-- 4. CATEGORIES
-- ============================================================

CREATE TABLE categories (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    slug            VARCHAR(100) NOT NULL,
    description     TEXT,
    parent_id       UUID REFERENCES categories(id),
    sort_order      INT DEFAULT 0,
    icon            VARCHAR(50),
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_categories_parent ON categories(parent_id);
CREATE UNIQUE INDEX idx_categories_slug ON categories(slug) WHERE parent_id IS NULL;

CREATE TRIGGER trg_categories_updated_at BEFORE UPDATE ON categories
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- ============================================================
-- 5. TARGET GROUPS (GAP #7: audience definitions)
-- ============================================================

CREATE TABLE target_groups (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    -- Dynamic membership rules
    rules           JSONB NOT NULL DEFAULT '{}',
    -- Static or dynamic?
    membership_type VARCHAR(10) NOT NULL DEFAULT 'dynamic'
                    CHECK (membership_type IN ('dynamic', 'static', 'hybrid')),
    cached_member_count INT DEFAULT 0,
    last_evaluated_at TIMESTAMPTZ,
    created_by      UUID REFERENCES users(id),
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TRIGGER trg_target_groups_updated_at BEFORE UPDATE ON target_groups
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TABLE target_group_members (
    target_group_id UUID NOT NULL REFERENCES target_groups(id) ON DELETE CASCADE,
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    added_by        UUID REFERENCES users(id),
    added_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (target_group_id, user_id)
);

CREATE INDEX idx_target_group_members_user ON target_group_members(user_id);

-- ============================================================
-- 6. COMPETENCY FRAMEWORK
-- ============================================================

CREATE TABLE competency_frameworks (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    external_id     VARCHAR(255),                     -- ServiceNow framework ID
    -- Taxonomy labels per level (e.g., "Omrade" > "Kompetanse" > "Ferdighet")
    taxonomy        JSONB DEFAULT '[]',
    version         INT NOT NULL DEFAULT 1,
    status          VARCHAR(20) NOT NULL DEFAULT 'draft'
                    CHECK (status IN ('draft', 'active', 'archived')),
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TRIGGER trg_competency_frameworks_updated_at BEFORE UPDATE ON competency_frameworks
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TABLE competencies (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    framework_id    UUID NOT NULL REFERENCES competency_frameworks(id) ON DELETE CASCADE,
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    parent_id       UUID REFERENCES competencies(id),
    external_id     VARCHAR(255),                     -- ServiceNow skill ID
    -- Proficiency scale (configurable per competency or inherited from framework)
    proficiency_scale JSONB,
    -- Example: [
    --   {"level": 1, "name_nb": "Grunnleggende", "name_en": "Basic", "description": "..."},
    --   {"level": 2, "name_nb": "Videregaende", "name_en": "Intermediate"},
    --   {"level": 3, "name_nb": "Avansert", "name_en": "Advanced"},
    --   {"level": 4, "name_nb": "Ekspert", "name_en": "Expert"}
    -- ]
    sort_order      INT DEFAULT 0,
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_competencies_framework ON competencies(framework_id);
CREATE INDEX idx_competencies_parent ON competencies(parent_id);
CREATE INDEX idx_competencies_external ON competencies(external_id) WHERE external_id IS NOT NULL;

CREATE TRIGGER trg_competencies_updated_at BEFORE UPDATE ON competencies
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- User competency records (supports history -- no unique constraint on user+competency)
CREATE TABLE user_competencies (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    competency_id   UUID NOT NULL REFERENCES competencies(id),
    level           INT,
    acquired_at     DATE NOT NULL,
    expires_at      DATE,
    -- Evidence linking
    source          VARCHAR(50) NOT NULL,
    -- 'course_completion', 'workplace_activity', 'manual', 'servicenow',
    -- 'assessment', 'external', 'import'
    evidence_type   VARCHAR(30),                      -- 'completion', 'certificate', 'activity_record'
    evidence_id     UUID,                             -- FK to the evidence entity
    -- Status
    is_current      BOOLEAN NOT NULL DEFAULT true,    -- Denormalized: latest record for this competency
    superseded_by   UUID REFERENCES user_competencies(id),
    notes           TEXT,
    granted_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_user_competencies_user ON user_competencies(user_id)
    WHERE is_current = true;
CREATE INDEX idx_user_competencies_competency ON user_competencies(competency_id);
CREATE INDEX idx_user_competencies_expires ON user_competencies(expires_at)
    WHERE expires_at IS NOT NULL AND is_current = true;
CREATE INDEX idx_user_competencies_current ON user_competencies(user_id, competency_id)
    WHERE is_current = true;

-- ============================================================
-- 7. COURSES
-- ============================================================

CREATE TABLE courses (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title           VARCHAR(500) NOT NULL,
    description     TEXT,
    short_desc      VARCHAR(500),
    type            VARCHAR(50) NOT NULL
                    CHECK (type IN ('e-learning', 'classroom', 'webinar', 'blended',
                                    'assignment', 'external', 'video', 'learning_path')),
    format          VARCHAR(50),
                    -- 'scorm_12', 'scorm_2004', 'xapi', 'cmi5', 'native', 'h5p', NULL
    duration_min    INT,
    -- Categorization
    category_id     UUID REFERENCES categories(id),
    competency_id   UUID REFERENCES competencies(id), -- Primary competency this course develops
    tags            TEXT[] DEFAULT '{}',               -- Free-form tags for search/filtering
    -- Publishing
    status          VARCHAR(20) NOT NULL DEFAULT 'draft'
                    CHECK (status IN ('draft', 'review', 'published', 'archived')),
    created_by      UUID REFERENCES users(id),
    published_at    TIMESTAMPTZ,
    archived_at     TIMESTAMPTZ,
    -- Versioning
    version         INT NOT NULL DEFAULT 1,
    -- Search
    search_vector   TSVECTOR,                         -- Norwegian FTS
    -- Semantic search (pgvector)
    embedding       vector(768),                      -- Multilingual E5 embeddings
    -- Metadata
    metadata        JSONB DEFAULT '{}',
    -- Language
    language        VARCHAR(5) DEFAULT 'nb',
    -- External reference
    external_id     VARCHAR(255),                     -- For imported/migrated courses
    -- Timestamps
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at      TIMESTAMPTZ                       -- Soft delete
);

CREATE INDEX idx_courses_search ON courses USING GIN(search_vector);
CREATE INDEX idx_courses_embedding ON courses USING hnsw (embedding vector_cosine_ops)
    WITH (m = 16, ef_construction = 64)
    WHERE embedding IS NOT NULL;
CREATE INDEX idx_courses_status ON courses(status, published_at DESC)
    WHERE deleted_at IS NULL;
CREATE INDEX idx_courses_category ON courses(category_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_courses_type ON courses(type)
    WHERE status = 'published' AND deleted_at IS NULL;
CREATE INDEX idx_courses_competency ON courses(competency_id) WHERE competency_id IS NOT NULL;
CREATE INDEX idx_courses_created_by ON courses(created_by);
CREATE INDEX idx_courses_tags ON courses USING GIN(tags) WHERE deleted_at IS NULL;
CREATE INDEX idx_courses_external ON courses(external_id) WHERE external_id IS NOT NULL;

CREATE TRIGGER trg_courses_updated_at BEFORE UPDATE ON courses
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Trigger to update search_vector on insert/update
CREATE OR REPLACE FUNCTION update_course_search_vector()
RETURNS TRIGGER AS $$
BEGIN
    NEW.search_vector :=
        setweight(to_tsvector('norwegian', COALESCE(NEW.title, '')), 'A') ||
        setweight(to_tsvector('norwegian', COALESCE(NEW.short_desc, '')), 'B') ||
        setweight(to_tsvector('norwegian', COALESCE(NEW.description, '')), 'C') ||
        setweight(to_tsvector('norwegian', COALESCE(array_to_string(NEW.tags, ' '), '')), 'B');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_courses_search_vector BEFORE INSERT OR UPDATE ON courses
    FOR EACH ROW EXECUTE FUNCTION update_course_search_vector();

-- Course-target-group rules (GAP #7: per-group mandatory/cert rules)
CREATE TABLE course_target_rules (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id       UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
    target_group_id UUID NOT NULL REFERENCES target_groups(id),
    is_mandatory    BOOLEAN NOT NULL DEFAULT false,
    -- Certification
    cert_validity_months INT,                         -- NULL = no expiry
    cert_template_id UUID,                            -- FK added after certificate_templates
    -- Auto-enrollment
    auto_enroll     BOOLEAN DEFAULT false,
    -- Deadlines
    due_within_days INT,
    -- Recertification
    recert_reminder_days INT DEFAULT 30,
    recert_course_id UUID REFERENCES courses(id),
    -- Metadata
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (course_id, target_group_id)
);

CREATE INDEX idx_course_target_rules_course ON course_target_rules(course_id);
CREATE INDEX idx_course_target_rules_group ON course_target_rules(target_group_id);

CREATE TRIGGER trg_course_target_rules_updated_at BEFORE UPDATE ON course_target_rules
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Course content packages (SCORM, xAPI, video, documents)
CREATE TABLE course_contents (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id       UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
    content_type    VARCHAR(50) NOT NULL
                    CHECK (content_type IN ('scorm_package', 'xapi_package', 'cmi5_package',
                                            'h5p_package', 'video', 'document', 'quiz',
                                            'audio', 'html', 'link')),
    -- Package metadata
    scorm_version   VARCHAR(20),
    entry_point     VARCHAR(500),                     -- Relative path to launch file
    storage_path    VARCHAR(500) NOT NULL,             -- S3 path
    original_filename VARCHAR(255),
    file_size_bytes BIGINT,
    mime_type       VARCHAR(100),
    -- Parsed manifest data
    manifest_data   JSONB,
    -- Sequencing (for multi-SCO packages)
    sco_tree        JSONB,                            -- SCORM 2004 activity tree
    -- Versioning
    version         INT NOT NULL DEFAULT 1,
    is_active       BOOLEAN DEFAULT true,
    -- Timestamps
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_course_contents_course ON course_contents(course_id);

CREATE TRIGGER trg_course_contents_updated_at BEFORE UPDATE ON course_contents
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- ============================================================
-- 8. ENROLLMENTS
-- ============================================================

CREATE TABLE enrollments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    course_id       UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
    status          VARCHAR(20) NOT NULL DEFAULT 'enrolled'
                    CHECK (status IN ('enrolled', 'in_progress', 'completed', 'failed',
                                      'expired', 'withdrawn', 'waiting_list')),
    enrolled_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    started_at      TIMESTAMPTZ,
    completed_at    TIMESTAMPTZ,
    due_date        DATE,
    score           DECIMAL(5,2),
    progress_pct    SMALLINT NOT NULL DEFAULT 0 CHECK (progress_pct BETWEEN 0 AND 100),
    attempts        INT NOT NULL DEFAULT 0,
    time_spent_sec  INT DEFAULT 0,
    -- Source tracking
    source          VARCHAR(50) NOT NULL DEFAULT 'manual'
                    CHECK (source IN ('manual', 'auto_assign', 'manager', 'self',
                                      'learning_path', 'target_rule', 'scim', 'import')),
    assigned_by     UUID REFERENCES users(id),
    -- Target rule that triggered this enrollment
    target_rule_id  UUID REFERENCES course_target_rules(id),
    -- Learning path context (if enrolled via a learning path)
    learning_path_enrollment_id UUID,                 -- FK added after lp_enrollments table
    -- Metadata
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (user_id, course_id)
);

CREATE INDEX idx_enrollments_user_status ON enrollments(user_id, status);
CREATE INDEX idx_enrollments_course ON enrollments(course_id);
CREATE INDEX idx_enrollments_due_date ON enrollments(due_date)
    WHERE status NOT IN ('completed', 'withdrawn') AND due_date IS NOT NULL;
CREATE INDEX idx_enrollments_assigned_by ON enrollments(assigned_by)
    WHERE assigned_by IS NOT NULL;
CREATE INDEX idx_enrollments_target_rule ON enrollments(target_rule_id)
    WHERE target_rule_id IS NOT NULL;

CREATE TRIGGER trg_enrollments_updated_at BEFORE UPDATE ON enrollments
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- ============================================================
-- 9. COMPLETIONS (partitioned, immutable audit trail)
-- ============================================================

CREATE TABLE completions (
    id              UUID NOT NULL DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL,                    -- No FK on partitioned table (validated by app)
    course_id       UUID NOT NULL,
    completed_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    score           DECIMAL(5,2),
    time_spent_sec  INT,
    attempt_number  INT NOT NULL DEFAULT 1,
    certificate_id  UUID,                             -- FK to certificates
    valid_until     DATE,
    -- Snapshot of CMI data at completion
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (id, completed_at)                    -- Partition key must be in PK
) PARTITION BY RANGE (completed_at);

-- Create partitions for initial years
CREATE TABLE completions_2026 PARTITION OF completions
    FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
CREATE TABLE completions_2027 PARTITION OF completions
    FOR VALUES FROM ('2027-01-01') TO ('2028-01-01');
CREATE TABLE completions_2028 PARTITION OF completions
    FOR VALUES FROM ('2028-01-01') TO ('2029-01-01');
CREATE TABLE completions_default PARTITION OF completions DEFAULT;

CREATE INDEX idx_completions_user ON completions(user_id, completed_at DESC);
CREATE INDEX idx_completions_course ON completions(course_id, completed_at DESC);
CREATE INDEX idx_completions_valid_until ON completions(valid_until)
    WHERE valid_until IS NOT NULL;

-- ============================================================
-- 10. SCORM RUNTIME DATA
-- ============================================================

CREATE TABLE scorm_runtime_data (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    enrollment_id   UUID NOT NULL REFERENCES enrollments(id) ON DELETE CASCADE,
    sco_id          VARCHAR(255),                     -- SCO identifier (for multi-SCO packages)
    -- Core CMI data as JSONB (flexible across SCORM versions)
    cmi_data        JSONB NOT NULL DEFAULT '{}',
    -- Frequently accessed fields extracted for indexing
    lesson_status   VARCHAR(30),                      -- SCORM 1.2: passed, failed, completed, etc.
    completion_status VARCHAR(30),                     -- SCORM 2004: completed, incomplete, etc.
    success_status  VARCHAR(30),                      -- SCORM 2004: passed, failed, unknown
    score_raw       DECIMAL(7,2),
    score_scaled    DECIMAL(5,4),                     -- -1.0 to 1.0
    -- Suspend/resume data
    suspend_data    TEXT,                              -- Can be very large (base64 encoded state)
    bookmark        VARCHAR(1000),                    -- cmi.core.lesson_location / cmi.location
    -- Time tracking
    total_time      VARCHAR(20),                      -- SCORM formatted time string
    total_time_sec  INT,                              -- Computed seconds for queries
    -- Session management
    attempt_number  INT NOT NULL DEFAULT 1,
    session_count   INT NOT NULL DEFAULT 0,
    -- Timestamps
    last_updated    TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (enrollment_id, sco_id, attempt_number)
);

CREATE INDEX idx_scorm_runtime_enrollment ON scorm_runtime_data(enrollment_id);

-- SCORM runtime event log (append-only, for debugging and replay)
CREATE TABLE scorm_runtime_events (
    id              UUID NOT NULL DEFAULT gen_random_uuid(),
    enrollment_id   UUID NOT NULL,
    sco_id          VARCHAR(255),
    event_type      VARCHAR(30) NOT NULL,
    -- 'initialize', 'set_value', 'get_value', 'commit', 'terminate'
    element         VARCHAR(255),                     -- CMI element name
    value           TEXT,                             -- Value set
    timestamp       TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (id, timestamp)
) PARTITION BY RANGE (timestamp);

CREATE TABLE scorm_events_2026 PARTITION OF scorm_runtime_events
    FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
CREATE TABLE scorm_events_2027 PARTITION OF scorm_runtime_events
    FOR VALUES FROM ('2027-01-01') TO ('2028-01-01');
CREATE TABLE scorm_events_default PARTITION OF scorm_runtime_events DEFAULT;

CREATE INDEX idx_scorm_events_enrollment ON scorm_runtime_events(enrollment_id, timestamp DESC);

-- ============================================================
-- 11. LEARNING PATHS
-- ============================================================

CREATE TABLE learning_paths (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title           VARCHAR(500) NOT NULL,
    description     TEXT,
    short_desc      VARCHAR(500),
    is_sequential   BOOLEAN NOT NULL DEFAULT true,
    competency_id   UUID REFERENCES competencies(id),
    -- Search
    search_vector   TSVECTOR,
    embedding       vector(768),
    -- Publishing
    status          VARCHAR(20) NOT NULL DEFAULT 'draft'
                    CHECK (status IN ('draft', 'published', 'archived')),
    created_by      UUID REFERENCES users(id),
    published_at    TIMESTAMPTZ,
    -- Metadata
    estimated_duration_min INT,
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at      TIMESTAMPTZ
);

CREATE INDEX idx_learning_paths_status ON learning_paths(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_learning_paths_search ON learning_paths USING GIN(search_vector);

CREATE TRIGGER trg_learning_paths_updated_at BEFORE UPDATE ON learning_paths
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TABLE learning_path_items (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    path_id         UUID NOT NULL REFERENCES learning_paths(id) ON DELETE CASCADE,
    -- Item can be a course, event, assignment, or workplace activity type
    item_type       VARCHAR(30) NOT NULL DEFAULT 'course'
                    CHECK (item_type IN ('course', 'event', 'assignment', 'activity_type', 'external')),
    course_id       UUID REFERENCES courses(id),
    activity_type_id UUID,                            -- FK to activity_types (for workplace items)
    -- Ordering
    position        INT NOT NULL,
    is_required     BOOLEAN NOT NULL DEFAULT true,
    -- Overrides
    title_override  VARCHAR(500),                     -- Override course title in path context
    description_override TEXT,
    -- Unlock conditions
    unlock_after_position INT,                        -- Unlock after this position is completed
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (path_id, position)
);

CREATE INDEX idx_lp_items_path ON learning_path_items(path_id, position);
CREATE INDEX idx_lp_items_course ON learning_path_items(course_id) WHERE course_id IS NOT NULL;

CREATE TABLE learning_path_enrollments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    path_id         UUID NOT NULL REFERENCES learning_paths(id) ON DELETE CASCADE,
    status          VARCHAR(20) NOT NULL DEFAULT 'enrolled'
                    CHECK (status IN ('enrolled', 'in_progress', 'completed', 'withdrawn')),
    enrolled_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    started_at      TIMESTAMPTZ,
    completed_at    TIMESTAMPTZ,
    progress_pct    SMALLINT DEFAULT 0 CHECK (progress_pct BETWEEN 0 AND 100),
    -- Track per-item progress
    item_progress   JSONB DEFAULT '{}',
    -- {"item_uuid_1": {"status": "completed", "completed_at": "..."}, ...}
    source          VARCHAR(50) DEFAULT 'manual',
    assigned_by     UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (user_id, path_id)
);

CREATE INDEX idx_lp_enrollments_user ON learning_path_enrollments(user_id);
CREATE INDEX idx_lp_enrollments_path ON learning_path_enrollments(path_id);

CREATE TRIGGER trg_lp_enrollments_updated_at BEFORE UPDATE ON learning_path_enrollments
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Add FK from enrollments to learning_path_enrollments
ALTER TABLE enrollments
    ADD CONSTRAINT fk_enrollments_lp_enrollment
    FOREIGN KEY (learning_path_enrollment_id)
    REFERENCES learning_path_enrollments(id);

-- ============================================================
-- 12. CERTIFICATES
-- ============================================================

CREATE TABLE certificate_templates (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    html_template   TEXT NOT NULL,                     -- Jinja2/Handlebars template for PDF
    css_styles      TEXT,
    -- Template variables available: {{user_name}}, {{course_title}}, {{completion_date}},
    -- {{certificate_no}}, {{valid_until}}, {{competency_name}}, {{score}}, {{qr_code_url}}
    thumbnail_url   VARCHAR(500),
    is_default      BOOLEAN DEFAULT false,
    is_active       BOOLEAN DEFAULT true,
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TRIGGER trg_cert_templates_updated_at BEFORE UPDATE ON certificate_templates
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Add FK from course_target_rules to certificate_templates
ALTER TABLE course_target_rules
    ADD CONSTRAINT fk_course_target_rules_cert_template
    FOREIGN KEY (cert_template_id)
    REFERENCES certificate_templates(id);

CREATE TABLE certificates (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id),
    competency_id   UUID REFERENCES competencies(id),
    course_id       UUID REFERENCES courses(id),
    template_id     UUID REFERENCES certificate_templates(id),
    -- Certificate identity
    certificate_no  VARCHAR(50) NOT NULL UNIQUE,       -- Format: {TENANT}-{YEAR}-{SEQ}
    -- Issuance
    issued_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
    valid_until     DATE,
    -- PDF
    pdf_storage_path VARCHAR(500),
    -- Verification
    verification_code VARCHAR(64) UNIQUE NOT NULL,     -- For public verification URL
    verification_url VARCHAR(500),
    -- Status
    revoked         BOOLEAN NOT NULL DEFAULT false,
    revoked_at      TIMESTAMPTZ,
    revoked_reason  TEXT,
    revoked_by      UUID REFERENCES users(id),
    -- Archive integration (Mime)
    archived_to     VARCHAR(255),                      -- Mime archive reference
    archived_at     TIMESTAMPTZ,
    -- Metadata
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_certificates_user ON certificates(user_id);
CREATE INDEX idx_certificates_course ON certificates(course_id);
CREATE INDEX idx_certificates_competency ON certificates(competency_id);
CREATE INDEX idx_certificates_verification ON certificates(verification_code);
CREATE INDEX idx_certificates_valid_until ON certificates(valid_until)
    WHERE revoked = false AND valid_until IS NOT NULL;
CREATE INDEX idx_certificates_certificate_no ON certificates(certificate_no);

-- Certificate number sequence (per tenant, per year)
CREATE SEQUENCE cert_number_seq START 1;

-- ============================================================
-- 13. EVENTS (Classroom, Webinar, Hybrid)
-- ============================================================

CREATE TABLE events (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id       UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
    title           VARCHAR(500),
    description     TEXT,
    -- Scheduling
    starts_at       TIMESTAMPTZ NOT NULL,
    ends_at         TIMESTAMPTZ NOT NULL,
    timezone        VARCHAR(50) DEFAULT 'Europe/Oslo',
    -- Location
    location        VARCHAR(500),
    location_type   VARCHAR(20) NOT NULL DEFAULT 'physical'
                    CHECK (location_type IN ('physical', 'virtual', 'hybrid')),
    virtual_url     VARCHAR(500),
    address         TEXT,
    room            VARCHAR(100),
    -- Capacity
    max_participants INT,
    min_participants INT,
    waitlist_enabled BOOLEAN DEFAULT true,
    -- Instructors (can have multiple)
    instructor_id   UUID REFERENCES users(id),
    -- Status
    status          VARCHAR(20) NOT NULL DEFAULT 'scheduled'
                    CHECK (status IN ('draft', 'scheduled', 'open_registration', 'closed',
                                      'in_progress', 'completed', 'cancelled')),
    -- Registration
    registration_opens_at TIMESTAMPTZ,
    registration_closes_at TIMESTAMPTZ,
    -- Cost
    cost_nok        DECIMAL(10,2),
    -- Metadata
    metadata        JSONB DEFAULT '{}',
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_events_course ON events(course_id);
CREATE INDEX idx_events_instructor ON events(instructor_id);
CREATE INDEX idx_events_starts_at ON events(starts_at) WHERE status IN ('scheduled', 'open_registration');
CREATE INDEX idx_events_status ON events(status);

CREATE TRIGGER trg_events_updated_at BEFORE UPDATE ON events
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Event attendees (separate from enrollments to support waitlist, attendance tracking)
CREATE TABLE event_attendees (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_id        UUID NOT NULL REFERENCES events(id) ON DELETE CASCADE,
    user_id         UUID NOT NULL REFERENCES users(id),
    status          VARCHAR(20) NOT NULL DEFAULT 'registered'
                    CHECK (status IN ('registered', 'waitlisted', 'confirmed', 'attended',
                                      'no_show', 'cancelled')),
    registered_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    confirmed_at    TIMESTAMPTZ,
    cancelled_at    TIMESTAMPTZ,
    attendance_marked_at TIMESTAMPTZ,
    attendance_marked_by UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (event_id, user_id)
);

CREATE INDEX idx_event_attendees_event ON event_attendees(event_id);
CREATE INDEX idx_event_attendees_user ON event_attendees(user_id);

CREATE TRIGGER trg_event_attendees_updated_at BEFORE UPDATE ON event_attendees
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Event instructors (many-to-many for multiple instructors)
CREATE TABLE event_instructors (
    event_id        UUID NOT NULL REFERENCES events(id) ON DELETE CASCADE,
    user_id         UUID NOT NULL REFERENCES users(id),
    role            VARCHAR(30) DEFAULT 'instructor'
                    CHECK (role IN ('instructor', 'co_instructor', 'facilitator', 'guest')),
    PRIMARY KEY (event_id, user_id)
);

-- Custom registration forms for events (GAP #6)
CREATE TABLE event_registration_forms (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_id        UUID REFERENCES events(id) ON DELETE CASCADE,
    is_template     BOOLEAN DEFAULT false,
    template_name   VARCHAR(255),
    fields          JSONB NOT NULL,
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_event_reg_forms_event ON event_registration_forms(event_id);

CREATE TRIGGER trg_event_reg_forms_updated_at BEFORE UPDATE ON event_registration_forms
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TABLE event_registration_responses (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    form_id         UUID NOT NULL REFERENCES event_registration_forms(id) ON DELETE CASCADE,
    event_id        UUID NOT NULL REFERENCES events(id),
    user_id         UUID NOT NULL REFERENCES users(id),
    responses       JSONB NOT NULL,
    submitted_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (form_id, user_id)
);

CREATE INDEX idx_event_reg_responses_event ON event_registration_responses(event_id);
CREATE INDEX idx_event_reg_responses_user ON event_registration_responses(user_id);

-- ============================================================
-- 14. ASSIGNMENTS & SUBMISSIONS (GAP #10)
-- ============================================================

CREATE TABLE assignments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id       UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
    title           VARCHAR(500) NOT NULL,
    description     TEXT,
    instructions    TEXT,
    -- Submission config
    submission_types TEXT[] NOT NULL DEFAULT ARRAY['file_upload'],
    allowed_file_types TEXT[],
    max_file_size_mb INT DEFAULT 100,
    max_files       INT DEFAULT 5,
    -- Grading
    grading_type    VARCHAR(20) NOT NULL DEFAULT 'pass_fail'
                    CHECK (grading_type IN ('pass_fail', 'points', 'percentage',
                                            'rubric', 'ungraded')),
    max_points      DECIMAL(7,2),
    passing_points  DECIMAL(7,2),
    rubric_id       UUID,                             -- FK to checklist_templates
    -- Resubmission
    max_attempts    INT DEFAULT 1,
    allow_resubmit  BOOLEAN DEFAULT false,
    -- Deadlines
    due_at          TIMESTAMPTZ,
    lock_at         TIMESTAMPTZ,
    -- Peer review
    peer_review_enabled BOOLEAN DEFAULT false,
    peer_review_count   INT DEFAULT 1,
    -- Display
    position        INT DEFAULT 0,
    is_published    BOOLEAN DEFAULT false,
    -- Metadata
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_assignments_course ON assignments(course_id);

CREATE TRIGGER trg_assignments_updated_at BEFORE UPDATE ON assignments
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TABLE assignment_submissions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    assignment_id   UUID NOT NULL REFERENCES assignments(id) ON DELETE CASCADE,
    user_id         UUID NOT NULL REFERENCES users(id),
    attempt_number  INT NOT NULL DEFAULT 1,
    -- Content
    submission_type VARCHAR(30) NOT NULL,
    body            TEXT,
    url             VARCHAR(500),
    file_urls       TEXT[],
    file_names      VARCHAR(255)[],
    file_sizes      BIGINT[],
    -- Workflow
    workflow_state  VARCHAR(20) NOT NULL DEFAULT 'draft'
                    CHECK (workflow_state IN ('draft', 'submitted', 'under_review',
                                              'graded', 'returned', 'resubmit_requested')),
    submitted_at    TIMESTAMPTZ,
    -- Grading
    graded_by       UUID REFERENCES users(id),
    graded_at       TIMESTAMPTZ,
    score           DECIMAL(7,2),
    grade           VARCHAR(20),
    grading_comment TEXT,
    rubric_assessment JSONB,
    -- Timestamps
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_submissions_unique ON assignment_submissions(assignment_id, user_id, attempt_number);
CREATE INDEX idx_submissions_assignment ON assignment_submissions(assignment_id);
CREATE INDEX idx_submissions_user ON assignment_submissions(user_id);
CREATE INDEX idx_submissions_state ON assignment_submissions(workflow_state)
    WHERE workflow_state IN ('submitted', 'under_review');

CREATE TRIGGER trg_submissions_updated_at BEFORE UPDATE ON assignment_submissions
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- ============================================================
-- 15. WORKPLACE ACTIVITIES (GAP #1, #11)
-- ============================================================

CREATE TABLE activity_types (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code            VARCHAR(50) NOT NULL UNIQUE,
    name_nb         VARCHAR(255) NOT NULL,
    name_nn         VARCHAR(255),
    name_en         VARCHAR(255),
    description     TEXT,
    learning_mode   VARCHAR(15) NOT NULL DEFAULT 'experiential'
                    CHECK (learning_mode IN ('formal', 'social', 'experiential')),
    requires_approval       BOOLEAN DEFAULT true,
    requires_evidence       BOOLEAN DEFAULT false,
    requires_checklist      BOOLEAN DEFAULT false,
    requires_sensor         BOOLEAN DEFAULT false,
    default_competency_id   UUID REFERENCES competencies(id),
    default_validity_months INT,                       -- Default validity if time-limited
    icon            VARCHAR(50),
    sort_order      INT DEFAULT 0,
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TRIGGER trg_activity_types_updated_at BEFORE UPDATE ON activity_types
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TABLE activity_records (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id),
    activity_type_id UUID NOT NULL REFERENCES activity_types(id),
    title           VARCHAR(500) NOT NULL,
    description     TEXT,
    occurred_at     TIMESTAMPTZ NOT NULL,
    duration_min    INT,
    location        VARCHAR(500),
    -- Approval
    status          VARCHAR(20) NOT NULL DEFAULT 'draft'
                    CHECK (status IN ('draft', 'submitted', 'pending_approval',
                                      'approved', 'rejected', 'expired')),
    submitted_at    TIMESTAMPTZ,
    approved_by     UUID REFERENCES users(id),
    approved_at     TIMESTAMPTZ,
    rejection_reason TEXT,
    -- Observer/sensor
    observer_name   VARCHAR(255),
    observer_email  VARCHAR(255),
    sensor_user_id  UUID REFERENCES users(id),
    sensor_confirmed_at TIMESTAMPTZ,
    -- Evidence
    evidence_urls   TEXT[],
    evidence_notes  TEXT,
    -- Competency
    competency_id   UUID REFERENCES competencies(id),
    competency_level INT,
    -- Validity
    valid_until     DATE,
    -- Metadata
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at      TIMESTAMPTZ
);

CREATE INDEX idx_activity_records_user ON activity_records(user_id, occurred_at DESC);
CREATE INDEX idx_activity_records_type ON activity_records(activity_type_id);
CREATE INDEX idx_activity_records_status ON activity_records(status)
    WHERE status IN ('submitted', 'pending_approval');
CREATE INDEX idx_activity_records_competency ON activity_records(competency_id)
    WHERE competency_id IS NOT NULL;
CREATE INDEX idx_activity_records_valid_until ON activity_records(valid_until)
    WHERE valid_until IS NOT NULL AND deleted_at IS NULL;

CREATE TRIGGER trg_activity_records_updated_at BEFORE UPDATE ON activity_records
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- ============================================================
-- 16. CHECKLISTS & ASSESSMENTS (GAP #3)
-- ============================================================

CREATE TABLE checklist_templates (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    activity_type_id UUID REFERENCES activity_types(id),
    version         INT NOT NULL DEFAULT 1,
    items           JSONB NOT NULL,
    scoring_method  VARCHAR(20) DEFAULT 'all_pass'
                    CHECK (scoring_method IN ('all_pass', 'weighted_average',
                                              'minimum_score', 'manual')),
    minimum_score   DECIMAL(5,2),
    is_active       BOOLEAN DEFAULT true,
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TRIGGER trg_checklist_templates_updated_at BEFORE UPDATE ON checklist_templates
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Add FK from assignments to checklist_templates (rubric)
ALTER TABLE assignments
    ADD CONSTRAINT fk_assignments_rubric
    FOREIGN KEY (rubric_id)
    REFERENCES checklist_templates(id);

CREATE TABLE checklist_responses (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    template_id     UUID NOT NULL REFERENCES checklist_templates(id),
    template_version INT NOT NULL,
    activity_record_id UUID REFERENCES activity_records(id) ON DELETE CASCADE,
    submission_id   UUID REFERENCES assignment_submissions(id) ON DELETE CASCADE,
    -- Either activity_record_id OR submission_id, not both
    assessed_user_id UUID NOT NULL REFERENCES users(id),
    assessor_id     UUID NOT NULL REFERENCES users(id),
    responses       JSONB NOT NULL,
    overall_result  VARCHAR(20) CHECK (overall_result IN ('pass', 'fail', 'conditional')),
    overall_score   DECIMAL(5,2),
    assessor_comment TEXT,
    completed_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    CHECK (
        (activity_record_id IS NOT NULL AND submission_id IS NULL) OR
        (activity_record_id IS NULL AND submission_id IS NOT NULL) OR
        (activity_record_id IS NULL AND submission_id IS NULL)
    )
);

CREATE INDEX idx_checklist_responses_activity ON checklist_responses(activity_record_id)
    WHERE activity_record_id IS NOT NULL;
CREATE INDEX idx_checklist_responses_submission ON checklist_responses(submission_id)
    WHERE submission_id IS NOT NULL;
CREATE INDEX idx_checklist_responses_assessed ON checklist_responses(assessed_user_id, completed_at DESC);

-- ============================================================
-- 17. APPROVAL WORKFLOWS (GAP #2)
-- ============================================================

CREATE TABLE approval_workflows (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    entity_type     VARCHAR(50) NOT NULL,
    steps           JSONB NOT NULL DEFAULT '[]',
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE approval_requests (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    workflow_id     UUID REFERENCES approval_workflows(id),
    entity_type     VARCHAR(50) NOT NULL,
    entity_id       UUID NOT NULL,
    requested_by    UUID NOT NULL REFERENCES users(id),
    requested_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    current_step    INT NOT NULL DEFAULT 1,
    status          VARCHAR(20) NOT NULL DEFAULT 'pending'
                    CHECK (status IN ('pending', 'approved', 'rejected', 'cancelled')),
    completed_at    TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_approval_requests_entity ON approval_requests(entity_type, entity_id);
CREATE INDEX idx_approval_requests_pending ON approval_requests(status)
    WHERE status = 'pending';
CREATE INDEX idx_approval_requests_requester ON approval_requests(requested_by);

CREATE TABLE approval_decisions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    request_id      UUID NOT NULL REFERENCES approval_requests(id) ON DELETE CASCADE,
    step_number     INT NOT NULL,
    decided_by      UUID NOT NULL REFERENCES users(id),
    decision        VARCHAR(20) NOT NULL
                    CHECK (decision IN ('approved', 'rejected', 'returned')),
    comment         TEXT,
    decided_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_approval_decisions_request ON approval_decisions(request_id);

-- ============================================================
-- 18. LEARNING AREAS & CMS (GAP #4, #5, #7, #9)
-- ============================================================

CREATE TABLE learning_areas (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title           VARCHAR(500) NOT NULL,
    slug            VARCHAR(100) NOT NULL UNIQUE,
    description     TEXT,
    hero_image_url  VARCHAR(500),
    icon            VARCHAR(50),
    color_theme     VARCHAR(7),
    parent_id       UUID REFERENCES learning_areas(id),
    sort_order      INT DEFAULT 0,
    owner_org_unit_id UUID REFERENCES org_units(id),
    created_by      UUID REFERENCES users(id),
    status          VARCHAR(20) NOT NULL DEFAULT 'draft'
                    CHECK (status IN ('draft', 'published', 'archived')),
    published_at    TIMESTAMPTZ,
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at      TIMESTAMPTZ
);

CREATE INDEX idx_learning_areas_parent ON learning_areas(parent_id);
CREATE INDEX idx_learning_areas_owner ON learning_areas(owner_org_unit_id);
CREATE INDEX idx_learning_areas_slug ON learning_areas(slug) WHERE deleted_at IS NULL;

CREATE TRIGGER trg_learning_areas_updated_at BEFORE UPDATE ON learning_areas
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TABLE learning_area_access (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    area_id         UUID NOT NULL REFERENCES learning_areas(id) ON DELETE CASCADE,
    grant_type      VARCHAR(20) NOT NULL
                    CHECK (grant_type IN ('target_group', 'org_unit', 'role', 'user')),
    grant_target_id UUID NOT NULL,
    permission      VARCHAR(10) NOT NULL CHECK (permission IN ('read', 'edit', 'admin')),
    granted_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (area_id, grant_type, grant_target_id, permission)
);

CREATE INDEX idx_learning_area_access_area ON learning_area_access(area_id);
CREATE INDEX idx_learning_area_access_target ON learning_area_access(grant_type, grant_target_id);

-- Content pages (CMS)
CREATE TABLE content_pages (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    slug            VARCHAR(200) NOT NULL,
    title           VARCHAR(500) NOT NULL,
    page_type       VARCHAR(30) NOT NULL
                    CHECK (page_type IN ('landing_internal', 'landing_external',
                                         'info_page', 'area_page', 'help_page')),
    blocks          JSONB NOT NULL DEFAULT '[]',
    version         INT NOT NULL DEFAULT 1,
    is_published    BOOLEAN DEFAULT false,
    published_at    TIMESTAMPTZ,
    published_by    UUID REFERENCES users(id),
    audience        VARCHAR(20) DEFAULT 'all'
                    CHECK (audience IN ('all', 'internal', 'external')),
    org_unit_id     UUID REFERENCES org_units(id),
    area_id         UUID REFERENCES learning_areas(id),
    meta_title      VARCHAR(200),
    meta_description VARCHAR(500),
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at      TIMESTAMPTZ
);

CREATE UNIQUE INDEX idx_content_pages_slug_audience
    ON content_pages(slug, audience)
    WHERE deleted_at IS NULL AND is_published = true;

CREATE TRIGGER trg_content_pages_updated_at BEFORE UPDATE ON content_pages
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TABLE content_page_versions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    page_id         UUID NOT NULL REFERENCES content_pages(id) ON DELETE CASCADE,
    version         INT NOT NULL,
    blocks          JSONB NOT NULL,
    edited_by       UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (page_id, version)
);

-- Learning area items
CREATE TABLE learning_area_items (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    area_id         UUID NOT NULL REFERENCES learning_areas(id) ON DELETE CASCADE,
    item_type       VARCHAR(30) NOT NULL
                    CHECK (item_type IN ('course', 'learning_path', 'document',
                                         'link', 'announcement', 'video', 'page')),
    course_id       UUID REFERENCES courses(id),
    learning_path_id UUID REFERENCES learning_paths(id),
    page_id         UUID REFERENCES content_pages(id),
    -- Direct content
    title           VARCHAR(500),
    url             VARCHAR(500),
    body            TEXT,
    -- Display
    sort_order      INT DEFAULT 0,
    is_featured     BOOLEAN DEFAULT false,
    is_pinned       BOOLEAN DEFAULT false,
    published_at    TIMESTAMPTZ,
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_learning_area_items_area ON learning_area_items(area_id, sort_order);

CREATE TRIGGER trg_learning_area_items_updated_at BEFORE UPDATE ON learning_area_items
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- ============================================================
-- 19. UNIFIED LEARNING RECORDS (70/20/10)
-- ============================================================

CREATE TABLE learning_records (
    id              UUID NOT NULL DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL,
    record_type     VARCHAR(30) NOT NULL,
    learning_mode   VARCHAR(15) NOT NULL
                    CHECK (learning_mode IN ('formal', 'social', 'experiential')),
    source_type     VARCHAR(50) NOT NULL,
    source_id       UUID,
    title           VARCHAR(500) NOT NULL,
    description     TEXT,
    occurred_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    duration_min    INT,
    score           DECIMAL(5,2),
    status          VARCHAR(20) NOT NULL DEFAULT 'completed'
                    CHECK (status IN ('completed', 'in_progress', 'pending_approval',
                                      'approved', 'rejected')),
    competency_ids  UUID[],
    evidence_url    VARCHAR(500),
    metadata        JSONB DEFAULT '{}',
    approved_by     UUID,
    approved_at     TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (id, occurred_at)
) PARTITION BY RANGE (occurred_at);

CREATE TABLE learning_records_2026 PARTITION OF learning_records
    FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
CREATE TABLE learning_records_2027 PARTITION OF learning_records
    FOR VALUES FROM ('2027-01-01') TO ('2028-01-01');
CREATE TABLE learning_records_2028 PARTITION OF learning_records
    FOR VALUES FROM ('2028-01-01') TO ('2029-01-01');
CREATE TABLE learning_records_default PARTITION OF learning_records DEFAULT;

CREATE INDEX idx_learning_records_user ON learning_records(user_id, occurred_at DESC);
CREATE INDEX idx_learning_records_mode ON learning_records(learning_mode, occurred_at DESC);
CREATE INDEX idx_learning_records_source ON learning_records(source_type, source_id);
CREATE INDEX idx_learning_records_status ON learning_records(status)
    WHERE status = 'pending_approval';

-- ============================================================
-- 20. xAPI STATEMENT STORE (LRS)
-- ============================================================

CREATE TABLE xapi_statements (
    id              UUID NOT NULL DEFAULT gen_random_uuid(),
    statement_id    UUID NOT NULL,                     -- xAPI statement UUID
    -- Extracted indexed columns for fast querying
    actor_id        VARCHAR(255),                      -- Actor IRI/mbox
    verb_id         VARCHAR(255) NOT NULL,             -- Verb IRI (e.g., 'http://adlnet.gov/expapi/verbs/completed')
    activity_id     VARCHAR(500),                      -- Object activity IRI
    registration    UUID,                              -- Context registration UUID
    -- Full statement (for conformance and full-fidelity retrieval)
    actor           JSONB NOT NULL,
    verb            JSONB NOT NULL,
    object          JSONB NOT NULL,
    result          JSONB,
    context         JSONB,
    -- xAPI metadata
    timestamp       TIMESTAMPTZ NOT NULL,
    stored          TIMESTAMPTZ NOT NULL DEFAULT now(),
    authority       JSONB,
    -- Voiding
    is_voided       BOOLEAN NOT NULL DEFAULT false,
    voided_by_statement_id UUID,
    -- Primary key includes partition key
    PRIMARY KEY (id, timestamp)
) PARTITION BY RANGE (timestamp);

CREATE TABLE xapi_statements_2026 PARTITION OF xapi_statements
    FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
CREATE TABLE xapi_statements_2027 PARTITION OF xapi_statements
    FOR VALUES FROM ('2027-01-01') TO ('2028-01-01');
CREATE TABLE xapi_statements_2028 PARTITION OF xapi_statements
    FOR VALUES FROM ('2028-01-01') TO ('2029-01-01');
CREATE TABLE xapi_statements_default PARTITION OF xapi_statements DEFAULT;

-- Unique constraint on statement_id (within partition)
CREATE UNIQUE INDEX idx_xapi_statement_id ON xapi_statements(statement_id, timestamp);
CREATE INDEX idx_xapi_verb ON xapi_statements(verb_id, timestamp DESC);
CREATE INDEX idx_xapi_actor ON xapi_statements(actor_id, timestamp DESC);
CREATE INDEX idx_xapi_activity ON xapi_statements(activity_id, timestamp DESC);
CREATE INDEX idx_xapi_registration ON xapi_statements(registration)
    WHERE registration IS NOT NULL;

-- xAPI state documents (activity state, agent profile, activity profile)
CREATE TABLE xapi_documents (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    doc_type        VARCHAR(20) NOT NULL
                    CHECK (doc_type IN ('state', 'agent_profile', 'activity_profile')),
    activity_id     VARCHAR(500),
    agent_id        VARCHAR(255),
    state_id        VARCHAR(255),
    registration    UUID,
    content         JSONB,
    content_type    VARCHAR(100) DEFAULT 'application/json',
    etag            VARCHAR(64) NOT NULL,
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (doc_type, activity_id, agent_id, state_id, COALESCE(registration::text, ''))
);

-- ============================================================
-- 21. NOTIFICATIONS
-- ============================================================

CREATE TABLE notifications (
    id              UUID NOT NULL DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL,
    type            VARCHAR(50) NOT NULL,
    -- 'course_assigned', 'course_due_soon', 'cert_expiring', 'approval_needed',
    -- 'approval_result', 'event_reminder', 'learning_path_step', 'system'
    title           VARCHAR(255) NOT NULL,
    body            TEXT,
    link            VARCHAR(500),
    -- Delivery
    channels        TEXT[] DEFAULT ARRAY['in_app'],
    -- 'in_app', 'email', 'push'
    email_sent      BOOLEAN DEFAULT false,
    push_sent       BOOLEAN DEFAULT false,
    -- Read status
    is_read         BOOLEAN NOT NULL DEFAULT false,
    read_at         TIMESTAMPTZ,
    -- Metadata
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (id, created_at)
) PARTITION BY RANGE (created_at);

CREATE TABLE notifications_2026 PARTITION OF notifications
    FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
CREATE TABLE notifications_2027 PARTITION OF notifications
    FOR VALUES FROM ('2027-01-01') TO ('2028-01-01');
CREATE TABLE notifications_default PARTITION OF notifications DEFAULT;

CREATE INDEX idx_notifications_user_unread ON notifications(user_id, created_at DESC)
    WHERE is_read = false;
CREATE INDEX idx_notifications_user ON notifications(user_id, created_at DESC);

-- ============================================================
-- 22. AUDIT LOG
-- ============================================================

CREATE TABLE audit_log (
    id              UUID NOT NULL DEFAULT gen_random_uuid(),
    user_id         UUID,
    action          VARCHAR(100) NOT NULL,
    -- 'create', 'update', 'delete', 'login', 'logout', 'enroll', 'complete',
    -- 'approve', 'reject', 'grant_role', 'revoke_role', 'export', 'import'
    entity_type     VARCHAR(50) NOT NULL,
    entity_id       UUID,
    old_values      JSONB,
    new_values      JSONB,
    ip_address      INET,
    user_agent      TEXT,
    -- Additional context
    session_id      VARCHAR(64),
    request_id      VARCHAR(64),                      -- Correlation ID
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (id, created_at)
) PARTITION BY RANGE (created_at);

CREATE TABLE audit_log_2026_01 PARTITION OF audit_log
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
CREATE TABLE audit_log_2026_02 PARTITION OF audit_log
    FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
CREATE TABLE audit_log_2026_03 PARTITION OF audit_log
    FOR VALUES FROM ('2026-03-01') TO ('2026-04-01');
CREATE TABLE audit_log_2026_04 PARTITION OF audit_log
    FOR VALUES FROM ('2026-04-01') TO ('2026-05-01');
CREATE TABLE audit_log_2026_05 PARTITION OF audit_log
    FOR VALUES FROM ('2026-05-01') TO ('2026-06-01');
CREATE TABLE audit_log_2026_06 PARTITION OF audit_log
    FOR VALUES FROM ('2026-06-01') TO ('2026-07-01');
CREATE TABLE audit_log_2026_07 PARTITION OF audit_log
    FOR VALUES FROM ('2026-07-01') TO ('2026-08-01');
CREATE TABLE audit_log_2026_08 PARTITION OF audit_log
    FOR VALUES FROM ('2026-08-01') TO ('2026-09-01');
CREATE TABLE audit_log_2026_09 PARTITION OF audit_log
    FOR VALUES FROM ('2026-09-01') TO ('2026-10-01');
CREATE TABLE audit_log_2026_10 PARTITION OF audit_log
    FOR VALUES FROM ('2026-10-01') TO ('2026-11-01');
CREATE TABLE audit_log_2026_11 PARTITION OF audit_log
    FOR VALUES FROM ('2026-11-01') TO ('2026-12-01');
CREATE TABLE audit_log_2026_12 PARTITION OF audit_log
    FOR VALUES FROM ('2026-12-01') TO ('2027-01-01');
CREATE TABLE audit_log_default PARTITION OF audit_log DEFAULT;

CREATE INDEX idx_audit_log_entity ON audit_log(entity_type, entity_id);
CREATE INDEX idx_audit_log_user ON audit_log(user_id, created_at DESC);

-- ============================================================
-- 23. INTEGRATION SYNC TRACKING
-- ============================================================

CREATE TABLE integration_sync_log (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    integration     VARCHAR(50) NOT NULL,
    -- 'servicenow', 'forgerock_scim', 'powerbi', 'kursoppslag', 'mime'
    direction       VARCHAR(10) NOT NULL CHECK (direction IN ('inbound', 'outbound')),
    entity_type     VARCHAR(50),
    entity_id       UUID,
    external_id     VARCHAR(255),
    action          VARCHAR(20) NOT NULL,
    -- 'create', 'update', 'delete', 'sync'
    status          VARCHAR(20) NOT NULL DEFAULT 'pending'
                    CHECK (status IN ('pending', 'success', 'failed', 'retrying')),
    request_payload JSONB,
    response_payload JSONB,
    error_message   TEXT,
    retry_count     INT DEFAULT 0,
    next_retry_at   TIMESTAMPTZ,
    started_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    completed_at    TIMESTAMPTZ
);

CREATE INDEX idx_integration_sync_status ON integration_sync_log(integration, status)
    WHERE status IN ('pending', 'failed', 'retrying');
CREATE INDEX idx_integration_sync_entity ON integration_sync_log(entity_type, entity_id);

-- ============================================================
-- 24. MATERIALIZED VIEWS FOR REPORTING
-- ============================================================

-- Team learning summary (refreshed every 15 minutes)
CREATE MATERIALIZED VIEW mv_team_learning_summary AS
SELECT
    u.org_unit_id,
    u.manager_id,
    COUNT(DISTINCT u.id) AS team_size,
    COUNT(DISTINCT e.id) FILTER (WHERE e.status = 'completed') AS total_completions,
    COUNT(DISTINCT e.id) FILTER (
        WHERE e.due_date < CURRENT_DATE AND e.status NOT IN ('completed', 'withdrawn')
    ) AS total_overdue,
    COUNT(DISTINCT cert.id) FILTER (
        WHERE cert.valid_until < CURRENT_DATE + INTERVAL '30 days'
        AND cert.revoked = false
    ) AS expiring_certs_30d
FROM users u
LEFT JOIN enrollments e ON e.user_id = u.id
LEFT JOIN certificates cert ON cert.user_id = u.id
WHERE u.is_active = true AND u.deleted_at IS NULL
GROUP BY u.org_unit_id, u.manager_id;

CREATE UNIQUE INDEX ON mv_team_learning_summary(org_unit_id, manager_id);

-- 70/20/10 distribution by org unit (refreshed hourly)
CREATE MATERIALIZED VIEW mv_learning_mode_distribution AS
SELECT
    u.org_unit_id,
    lr.learning_mode,
    date_trunc('month', lr.occurred_at) AS month,
    COUNT(*) AS record_count,
    COUNT(DISTINCT lr.user_id) AS unique_learners,
    SUM(lr.duration_min) AS total_minutes
FROM learning_records lr
JOIN users u ON u.id = lr.user_id
WHERE lr.status IN ('completed', 'approved')
  AND lr.occurred_at >= date_trunc('year', CURRENT_DATE) - INTERVAL '1 year'
GROUP BY u.org_unit_id, lr.learning_mode, date_trunc('month', lr.occurred_at);

CREATE UNIQUE INDEX ON mv_learning_mode_distribution(org_unit_id, learning_mode, month);
```

### 8.3 Seed data

```sql
-- ============================================================
-- SEED DATA: Default roles, permissions, and activity types
-- ============================================================

-- Default permissions
INSERT INTO permissions (code, name_nb, name_en, category, is_system) VALUES
    -- Courses
    ('courses.view', 'Se kurs', 'View courses', 'courses', true),
    ('courses.create', 'Opprette kurs', 'Create courses', 'courses', false),
    ('courses.edit', 'Redigere kurs', 'Edit courses', 'courses', false),
    ('courses.delete', 'Slette kurs', 'Delete courses', 'courses', false),
    ('courses.publish', 'Publisere kurs', 'Publish courses', 'courses', false),
    ('courses.manage_enrollments', 'Administrere paamelding', 'Manage enrollments', 'courses', false),
    ('courses.view_reports', 'Se kursrapporter', 'View course reports', 'courses', false),
    -- Users
    ('users.view', 'Se brukere', 'View users', 'users', true),
    ('users.manage', 'Administrere brukere', 'Manage users', 'users', false),
    ('users.manage_own_team', 'Administrere eget team', 'Manage own team', 'users', false),
    -- Reports
    ('reports.view_own', 'Se egne rapporter', 'View own reports', 'reports', true),
    ('reports.view_team', 'Se teamrapporter', 'View team reports', 'reports', false),
    ('reports.view_division', 'Se divisjonsrapporter', 'View division reports', 'reports', false),
    ('reports.view_all', 'Se alle rapporter', 'View all reports', 'reports', false),
    ('reports.export', 'Eksportere rapporter', 'Export reports', 'reports', false),
    -- Content
    ('content.view', 'Se innhold', 'View content', 'content', true),
    ('content.create', 'Opprette innhold', 'Create content', 'content', false),
    ('content.edit', 'Redigere innhold', 'Edit content', 'content', false),
    ('content.publish', 'Publisere innhold', 'Publish content', 'content', false),
    -- Learning areas
    ('areas.view', 'Se laeringsomrader', 'View learning areas', 'areas', true),
    ('areas.create', 'Opprette laeringsomrader', 'Create learning areas', 'areas', false),
    ('areas.edit', 'Redigere laeringsomrader', 'Edit learning areas', 'areas', false),
    -- Activities
    ('activities.register', 'Registrere aktiviteter', 'Register activities', 'activities', true),
    ('activities.approve', 'Godkjenne aktiviteter', 'Approve activities', 'activities', false),
    -- Settings
    ('settings.view', 'Se innstillinger', 'View settings', 'settings', false),
    ('settings.manage', 'Administrere innstillinger', 'Manage settings', 'settings', false),
    ('roles.manage', 'Administrere roller', 'Manage roles', 'settings', false),
    ('integrations.manage', 'Administrere integrasjoner', 'Manage integrations', 'settings', false),
    -- Competencies
    ('competencies.view', 'Se kompetanserammeverk', 'View competency frameworks', 'competencies', true),
    ('competencies.manage', 'Administrere kompetanser', 'Manage competencies', 'competencies', false),
    -- Certificates
    ('certificates.issue', 'Utstede sertifikater', 'Issue certificates', 'certificates', false),
    ('certificates.revoke', 'Tilbakekalle sertifikater', 'Revoke certificates', 'certificates', false);

-- Default roles
INSERT INTO roles (name, description, is_system, sort_order) VALUES
    ('Learner', 'Standard learner role with basic access', true, 1),
    ('Leader', 'Team leader with team management and reporting', true, 2),
    ('Content Manager', 'Course and content creation and management', true, 3),
    ('Division Admin', 'Division-scoped administration', true, 4),
    ('System Admin', 'Full system administration', true, 5);

-- Role-permission assignments (example for Learner and Leader)
-- Learner gets: courses.view, content.view, reports.view_own, activities.register, areas.view, competencies.view
-- Leader gets: all Learner permissions + reports.view_team, activities.approve, users.manage_own_team, courses.manage_enrollments
-- Content Manager gets: all Learner + courses.create/edit/publish, content.create/edit/publish
-- (Full permission mapping done in application seed script)

-- Default activity types for Norwegian public sector (SVV)
INSERT INTO activity_types (code, name_nb, name_nn, name_en, learning_mode, requires_approval, requires_evidence, requires_checklist, requires_sensor, sort_order) VALUES
    ('befaring', 'Befaring', 'Synfaring', 'Field inspection', 'experiential', true, false, false, false, 1),
    ('hospitering', 'Hospitering', 'Hospitering', 'Job shadowing/internship', 'experiential', true, false, false, false, 2),
    ('beredskapsovelse', 'Beredskapsovelse', 'Beredskapsovelse', 'Emergency drill', 'experiential', true, false, false, false, 3),
    ('simulatortrening', 'Simulatortrening', 'Simulatortrening', 'Simulator training', 'experiential', true, false, false, false, 4),
    ('praktisk_prove', 'Praktisk prove', 'Praktisk prove', 'Practical exam', 'experiential', true, true, true, true, 5),
    ('jobbobservasjon', 'Jobbobservasjon', 'Jobbobservasjon', 'Job observation', 'social', true, false, true, false, 6),
    ('egentrening', 'Egentrening', 'Eigentrening', 'Self-directed training', 'experiential', false, false, false, false, 7),
    ('veiledning', 'Veiledning/mentoring', 'Rettleiing/mentoring', 'Coaching/mentoring', 'social', false, false, false, false, 8),
    ('ekstern_kurs', 'Eksternt kurs', 'Eksternt kurs', 'External course', 'formal', true, true, false, false, 9),
    ('konferanse', 'Konferanse/seminar', 'Konferanse/seminar', 'Conference/seminar', 'formal', false, false, false, false, 10),
    ('kollegaveiledning', 'Kollegaveiledning', 'Kollegaveiledning', 'Peer coaching', 'social', false, false, false, false, 11),
    ('reautoriserering', 'Re-autorisering', 'Re-autorisering', 'Re-authorization', 'formal', true, true, false, true, 12);
```

### 8.4 Table count summary

| Category | Tables | Partitioned |
|----------|--------|-------------|
| Organization & Users | 2 | 0 |
| RBAC & Permissions | 5 | 0 |
| Categories & Target Groups | 3 | 0 |
| Competency Framework | 3 | 0 |
| Courses & Content | 3 | 0 |
| Enrollments & Completions | 2 | 1 |
| SCORM Runtime | 2 | 1 |
| Learning Paths | 3 | 0 |
| Certificates | 2 | 0 |
| Events | 5 | 0 |
| Assignments & Submissions | 2 | 0 |
| Workplace Activities | 2 | 0 |
| Checklists & Assessments | 2 | 0 |
| Approval Workflows | 3 | 0 |
| Learning Areas & CMS | 5 | 0 |
| Learning Records (70/20/10) | 1 | 1 |
| xAPI LRS | 2 | 1 |
| Notifications | 1 | 1 |
| Audit Log | 1 | 1 |
| Integration Sync | 1 | 0 |
| Materialized Views | 2 | -- |
| **Total** | **50 tables** | **6 partitioned** |
| **Shared schema** | **3 tables** | -- |

---

## Appendix A: ER Diagram (Simplified)

```
                                    ORGANIZATION
                                    +----------+
                                    | org_units|<---------+
                                    +----+-----+          |
                                         |parent          |
                                         |                |
          +-------------------+     +----+-----+     +----+------+
          | target_groups     |<-+  |  users   |---->| user_roles|
          | target_group_     |  |  +----+-----+     +-----+-----+
          |   members         |  |       |                 |
          +-------------------+  |       |manager     +----+-----+
                                 |       |            |  roles   |
   +--------+                    |  +----+------+     +----+-----+
   |competency|                  |  |enrollments|          |
   |frameworks|                  |  +----+------+     +----+------+
   +----+-----+                  |       |            |role_perms |
        |                        |  +----+------+     +----+------+
   +----+------+                 |  |completions|          |
   |competencies|                |  +-----------+     +----+------+
   +----+-------+               |                    |permissions|
        |                        |  +----------+     +-----------+
   +----+---------+              |  |certificates|
   |user_competencies|           |  +----------+
   +------------------+         |
                                 |  +-----------+    +-------------+
   +--------+   +---------+     |  |  courses   |--->|course_target|
   |activity |-->|activity |     +--|           |    |  _rules     |
   | _types  |   |_records |        +-----+-----+    +-------------+
   +---------+   +----+----+              |
                      |            +------+------+
                 +----+------+     |course_contents|
                 |checklist_ |     +------+-------+
                 | responses |            |
                 +----+------+     +------+-------+
                      |           |learning_paths |
                 +----+------+    +------+--------+
                 |checklist_ |           |
                 | templates |    +------+--------+
                 +-----------+    |lp_items       |
                                  +------+--------+
   +-----------+                         |
   |  events   |                  +------+--------+
   +-----+-----+                  |lp_enrollments |
         |                        +---------------+
   +-----+-------+
   |event_attendees|    +-----------+    +-----------+
   +--------------+    |assignments |    |learning_  |
                        +-----+-----+    |  records  |
   +---------------+         |          +-----------+
   |learning_areas |   +-----+-------+
   +-------+-------+   |assignment_  |    +-----------+
           |            |submissions |    |xapi_      |
   +-------+-------+   +-------------+   |statements |
   |area_access    |                      +-----------+
   +---------------+   +-------------+
                        |content_pages|    +-----------+
   +---------------+   +-------+-----+    |audit_log  |
   |notifications  |           |          +-----------+
   +---------------+   +-------+--------+
                        |page_versions  |  +-----------+
   +---------------+   +---------------+  |approval_  |
   |integration_   |                      |workflows  |
   |  sync_log     |                      +-----+-----+
   +---------------+                            |
                                          +-----+-------+
                                          |approval_    |
                                          | requests    |
                                          +-----+-------+
                                                |
                                          +-----+-------+
                                          |approval_    |
                                          | decisions   |
                                          +-------------+
```

---

## Appendix B: Research Sources

This document was informed by the following research:

- [Debugg - PostgreSQL Multitenancy: RLS vs Schemas vs Separate DBs](https://debugg.ai/resources/postgres-multitenancy-rls-vs-schemas-vs-separate-dbs-performance-isolation-migration-playbook-2025) -- Comprehensive 2025 comparison of multi-tenancy approaches
- [Crunchy Data - Row Level Security for Tenants](https://www.crunchydata.com/blog/row-level-security-for-tenants-in-postgres) -- RLS implementation patterns
- [AWS - Multi-Tenant Data Isolation with PostgreSQL RLS](https://aws.amazon.com/blogs/database/multi-tenant-data-isolation-with-postgresql-row-level-security/) -- AWS recommended patterns
- [DZone - PgBouncer at Scale: 10K+ Connections Multi-Tenant Postgres](https://dzone.com/articles/database-connection-pooling-at-scale-pgbouncer-mul) -- Connection pooling operational reality
- [Moodle - Database Schema Introduction](https://docs.moodle.org/dev/Database_schema_introduction) -- Moodle's 200-table schema design
- [Moodle - Competency API](https://docs.moodle.org/dev/Competency_API) -- Competency framework implementation
- [Prisma - Canvas LMS PostgreSQL Schema](https://github.com/prisma/database-schema-examples/blob/main/postgres/canvas-lms/schema.sql) -- Canvas database patterns
- [Yet Analytics - SQL LRS](https://github.com/yetanalytics/lrsql) -- PostgreSQL-based xAPI LRS reference implementation
- [SCORM.com - Run-Time Reference](https://scorm.com/scorm-explained/technical-scorm/run-time/run-time-reference/) -- SCORM CMI data model specification
- [Alembic Cookbook - Multi-schema Migrations](https://alembic.sqlalchemy.org/en/latest/cookbook.html) -- Multi-schema migration patterns
- [PyPI - alembic-multischema](https://pypi.org/project/alembic-multischema/) -- Multi-schema migration tooling
- [MergeBoard - Multitenancy with FastAPI, SQLAlchemy and PostgreSQL](https://mergeboard.com/blog/6-multitenancy-fastapi-sqlalchemy-postgresql/) -- FastAPI multi-tenancy implementation
- [Calmops - PostgreSQL Vector Search 2026](https://calmops.com/database/postgresql/postgresql-vector-search-pgvector-complete-guide-2026/) -- pgvector production guide
- [Medium - 9 Postgres Partitioning Strategies for Time-Series](https://medium.com/@connect.hashblock/9-postgres-partitioning-strategies-for-time-series-at-scale-c1b764a9b691) -- Partitioning strategy guide
