# Architecture Revision: AI-Built Stack
## Reconsidering tech choices when AI does the building

---

## What Changes When AI Builds

The original architecture chose TypeScript everywhere primarily for:
- "Largest talent pool in Norway" -- **irrelevant if AI builds**
- "Easy to hire for" -- **irrelevant**
- "Shared types reduce team size requirements" -- **irrelevant**
- "Full-stack TypeScript reduces context switching" -- **AI doesn't context-switch**

What STILL matters:
- **Performance**: The best runtime for a multi-tenant SaaS serving 250K+ users
- **Correctness**: Type safety, memory safety, fewer production bugs
- **Ecosystem**: Library quality for SCORM, OIDC, SCIM, etc.
- **Deployment**: Operability, binary size, cold start, resource efficiency
- **Long-term maintainability**: Even if AI builds, humans review and reason about it
- **AI code quality**: Which language does AI generate the most correct, idiomatic code in?

---

## Revised Stack Options

### Option A: Rust Backend (Performance-Optimized)

| Layer | Choice | Rationale |
|---|---|---|
| **Backend** | Rust (Axum) | Best performance, memory safety, zero-cost abstractions. Multi-tenant SaaS benefits enormously from Rust's efficiency -- lower hosting costs at scale. AI generates excellent Rust code in 2026. |
| **Frontend** | SvelteKit or Next.js | Svelte: smaller bundles, simpler reactivity, great DX. Next.js: larger ecosystem, better SSR story. |
| **AI/ML services** | Python (FastAPI) | ML ecosystem is Python-native. Separate microservice for AI features. |
| **Database** | PostgreSQL 16 | Unchanged -- still the best choice |

**Pros**: Best possible performance. A Rust backend uses 5-10x less memory than Node.js under equivalent load. At 250K users across multiple tenants, this means significantly lower cloud costs. Memory safety eliminates entire classes of bugs. Single static binary deployment.

**Cons**: Ecosystem for OIDC/SCIM/SCORM is thinner in Rust. Some libraries would need wrapping or building. AI generates good Rust but the review surface is harder for humans unfamiliar with Rust.

**Verdict**: Best for a product you want to run at scale for 10+ years. Overkill if speed-to-market matters more than efficiency.

### Option B: Go Backend (Balanced)

| Layer | Choice | Rationale |
|---|---|---|
| **Backend** | Go (stdlib + Chi/Echo) | Excellent concurrency, simple deployment (single binary), great for cloud-native SaaS. Strong library ecosystem for auth, HTTP, database. |
| **Frontend** | Next.js 15 (React) | Keep React for component ecosystem and accessibility tooling. SSR for WCAG. |
| **AI/ML services** | Python (FastAPI) | Separate service for AI features. |
| **Database** | PostgreSQL 16 | Unchanged |

**Pros**: Go compiles to a single binary with no runtime dependencies. Goroutines handle 10,000+ concurrent connections trivially. Excellent for API servers. Strong OIDC libraries (go-oidc, dex). Docker images are tiny (< 20MB). AI generates very clean Go code. Humans find Go easy to review even if AI wrote it.

**Cons**: No generics until Go 1.18 (now available), less expressive than Rust. SCIM and SCORM libraries are sparse -- would need to build or port. Error handling is verbose (though AI doesn't mind).

**Verdict**: Best balance of performance, simplicity, and ecosystem. Ideal for a cloud-native SaaS where operational simplicity matters.

### Option C: Python Backend (AI-Native)

| Layer | Choice | Rationale |
|---|---|---|
| **Backend** | Python (FastAPI) | AI/ML and the API in the same language. FastAPI is async, typed (Pydantic), and auto-generates OpenAPI docs. |
| **Frontend** | Next.js 15 or SvelteKit | Keep JS framework for frontend |
| **AI/ML** | Built-in (same service) | No separate AI service -- embeddings, recommendations, LLM calls all in the main app |
| **Database** | PostgreSQL 16 | Unchanged |

**Pros**: Unified language for backend + AI eliminates a service boundary. FastAPI performance is good (async + uvicorn). Pydantic models give strong typing. Richest ecosystem for AI/ML. AI generates excellent Python. Simplest architecture (fewer services).

**Cons**: Python is slower than Go/Rust/Node.js for pure API workloads. GIL limits true parallelism (mitigated by async). Memory usage is higher. Multi-tenancy patterns are less established than in Go or Rust.

**Verdict**: Best choice if AI features are the core differentiator and you want the simplest possible architecture. The performance gap vs Go/Rust may not matter until 100K+ concurrent users.

### Option D: Keep TypeScript (Ecosystem-Optimized)

| Layer | Choice | Rationale |
|---|---|---|
| **Backend** | NestJS or Hono (lighter) | Largest npm ecosystem, most libraries for SCORM (scorm-again), SCIM (SCIMMY), OIDC. |
| **Frontend** | Next.js 15 | Unchanged |
| **AI/ML** | Python (FastAPI) | Separate service |
| **Database** | PostgreSQL 16 | Unchanged |

**Pros**: The ecosystem argument is still valid even with AI building. `scorm-again`, `SCIMMY`, `openid-client`, `novu`, `pdfme` -- all the reusable components from our research are npm packages. Using TypeScript means zero adapter code for these. AI generates the most code in JS/TS (largest training corpus).

**Cons**: Node.js performance ceiling is lower than Go/Rust. V8 memory overhead in multi-tenant scenarios.

**Verdict**: Fastest path to MVP because all identified open-source components are npm-native. The ecosystem advantage is about the LIBRARIES, not the developers.

---

## Recommendation

### For this project: **Option B (Go backend) or Option C (Python backend)**

**If speed to market is priority #1**: **Python (FastAPI)** -- single language for backend + AI, simplest architecture, AI generates excellent Python. You ship the MVP fastest. Upgrade to Go later if performance becomes an issue (it won't for the first 50K users).

**If you're building for 10-year scale**: **Go** -- single binary deployment, excellent performance, operationally simple, great for multi-tenant SaaS. Add Python microservice for AI features.

**What I'd actually recommend**:

**Python (FastAPI) for the whole backend + AI layer.** Here's why:

1. AI is the #1 differentiator in this product. Having AI capabilities in the same codebase (not a separate service) means faster iteration on the features that actually win tenders.
2. FastAPI's async performance is sufficient for an LMS serving 5,000-50,000 users per tenant.
3. Pydantic models give you type safety comparable to TypeScript.
4. SQLAlchemy/Alembic for PostgreSQL is battle-tested.
5. AI generates more correct Python than any other language.
6. When you need to call embedding models, run inference, or process data -- you're already in Python. No service boundary, no serialization overhead, no deployment coordination.

**Frontend**: Next.js stays. The React ecosystem for accessible UI components (Radix, Designsystemet) and SSR for WCAG compliance is genuinely the best choice regardless of who builds it.

**The SCORM concern**: `scorm-again` is a JavaScript library that runs in the browser (not the backend). It stays regardless of backend language. The backend only needs to receive and store SCORM data model commits via a REST endpoint -- any language does this trivially.

**The SCIMMY concern**: SCIMMY is Node.js, but SCIM 2.0 is a simple REST protocol. Implementing a SCIM server in Python/FastAPI is straightforward -- there's `scim2-api` (Python) and the spec is well-documented.

---

## Revised Tech Stack

| Layer | Original | Revised | Why |
|---|---|---|---|
| **Backend** | NestJS (TypeScript) | **FastAPI (Python)** | AI-native, unified with ML layer, simpler architecture |
| **Frontend** | Next.js 15 (React) | Next.js 15 (React) | Unchanged -- best ecosystem for accessible UI |
| **Database** | PostgreSQL 16 | PostgreSQL 16 | Unchanged |
| **Cache** | Redis 7 | Redis 7 | Unchanged |
| **Search** | Meilisearch | Meilisearch | Unchanged |
| **File Storage** | S3-compatible | S3-compatible | Unchanged |
| **Job Queue** | BullMQ (Node.js) | **Celery or ARQ** (Python) | Python equivalent |
| **SCORM Runtime** | scorm-again (browser JS) | scorm-again (browser JS) | Unchanged -- runs in browser |
| **xAPI LRS** | SQL LRS | SQL LRS or **custom Python** | Could build LRS into main app |
| **Auth** | Passport.js | **Authlib or python-jose** | Mature Python OIDC/SAML |
| **SCIM Server** | SCIMMY (Node.js) | **scim2-api or custom** | SCIM is simple REST |
| **AI** | Separate Python service | **Built-in** | Same codebase -- the whole point |
| **Infrastructure** | AWS eu-north-1, EKS | AWS eu-north-1, **ECS or Lambda** | Python deploys well on simpler infra |

### Key Architecture Changes

1. **Monolith-first**: One Python application handles API, SCIM, AI, and background jobs. Extract services later if needed. AI doesn't need microservices overhead.
2. **AI in the hot path**: Recommendations, semantic search, and content analysis run in-process. No HTTP round-trip to a separate AI service.
3. **Simpler deployment**: FastAPI + uvicorn + Celery workers. No Kubernetes required initially -- ECS Fargate or even a few EC2 instances suffice for <50K users.
4. **Frontend unchanged**: Next.js deployed separately (Vercel or S3+CloudFront). Communicates with FastAPI backend via REST API.

---

## What This Means for the Build Plan

| Metric | Original (TypeScript) | Revised (Python) |
|---|---|---|
| MVP timeline | 26 weeks | **22-24 weeks** (AI generates Python faster, simpler architecture) |
| Services to deploy | 3 (Next.js + NestJS + Python AI) | **2 (Next.js + FastAPI)** |
| Team needed | 5 FTE | **3-4 FTE + AI** (Python unifies backend + AI roles) |
| Infrastructure complexity | Kubernetes (EKS) | **ECS Fargate** (simpler, cheaper) |
| SCORM libraries | npm native | Unchanged (browser JS) |
| SCIM libraries | npm native (SCIMMY) | Python equivalent or custom (simple) |
| Risk | Low (proven stack) | Low (FastAPI is production-proven at scale) |

---

## Alternative consideration: Elixir/Phoenix

Worth mentioning: **Elixir/Phoenix** would be excellent for an LMS:
- BEAM VM handles millions of concurrent connections
- LiveView eliminates the need for a separate frontend framework for admin UIs
- Built-in PubSub for real-time notifications
- Fault-tolerant by design (supervision trees)
- Great for long-lived WebSocket connections (SCORM player communication)

However: smaller ecosystem, AI generates less Elixir code (smaller training corpus), and the React component ecosystem (Designsystemet, Radix) would need a different integration approach. Consider for v2/rewrite if the product succeeds.
