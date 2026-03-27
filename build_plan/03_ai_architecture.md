# AI/ML Architecture -- Custom-Built LMS
## Statens vegvesen LMS Build Plan

---

## 1. Executive Summary

This document defines the AI/ML architecture for a custom-built LMS targeting Norwegian public sector agencies, starting with Statens vegvesen (SVV). It covers build-vs-buy decisions for each AI capability, the technical infrastructure, Norwegian language considerations, EU/GDPR compliance strategy, and a realistic MVP phasing plan.

**Core principle**: Use managed AI services (LLM APIs, embedding models) where they are mature and EU-compliant. Build custom only where Norwegian-specific requirements, domain logic, or competitive differentiation demand it. Minimize self-hosted ML infrastructure in MVP.

**Estimated total AI effort**: 8-12 person-months for MVP, 18-28 person-months through Phase 2.

---

## 2. AI Capabilities Overview

Six AI capabilities are required, each with different build-vs-buy trade-offs:

| Capability | MVP Priority | Build vs Buy | Primary Technology |
|---|---|---|---|
| Recommendation engine | Phase 1 (rule-based) | Build custom | Python + PostgreSQL |
| Semantic search | Phase 1 | Buy + integrate | pgvector + multilingual embeddings |
| Content generation | Phase 1 (basic) | Buy (LLM API) | Claude/GPT via EU-hosted API |
| Chatbot/assistant | Phase 2 | Buy + build RAG layer | LLM API + custom RAG pipeline |
| Competency analytics | Phase 1 (basic) | Build custom | Python + SQL + PowerBI export |
| Norwegian NLP | Cross-cutting | Buy (models) + configure | Multilingual embedding models |

---

## 3. Build vs Buy Analysis

### 3.1 Recommendation Engine

**Decision: BUILD CUSTOM (phased)**

**Why build**: Recommendation logic for an LMS is domain-specific. SVV's requirements -- safety-critical certification overrides, 70/20/10 learning model integration, ServiceNow competency data -- make off-the-shelf recommendation libraries insufficient. The data schema is ours, so we control the feature engineering entirely.

**Why not buy**: Existing recommendation-as-a-service products (Recombee, Amazon Personalize) add latency, cost, and GDPR complexity without solving the domain-specific logic. Open-source libraries like LensKit or Surprise handle the math but not the business rules.

**Architecture**:

```
Phase 1 (MVP): Rule-based + content-based filtering
Phase 2: Add collaborative filtering
Phase 3: Hybrid model with learning-to-rank

+--------------------------------------------------+
|           Recommendation Service (Python/FastAPI) |
+--------------------------------------------------+
|                                                    |
|  +-------------------+  +----------------------+  |
|  | Rule Engine       |  | Content-Based Filter |  |
|  | - Mandatory first  |  | - Role matching     |  |
|  | - Expiry alerts    |  | - Topic similarity  |  |
|  | - Safety overrides |  | - Difficulty level  |  |
|  +-------------------+  +----------------------+  |
|                                                    |
|  +-------------------+  +----------------------+  |
|  | Collaborative      |  | Feature Store        |  |
|  | Filter (Phase 2)   |  | (PostgreSQL-based)   |  |
|  | - User similarity  |  | - User profiles      |  |
|  | - Item popularity  |  | - Learning history   |  |
|  +-------------------+  +----------------------+  |
+--------------------------------------------------+
```

**Phase 1 -- Rule-based + content-based (MVP)**:
- Hard-coded rules: mandatory courses first, expiring certifications surfaced, safety-critical items cannot be skipped
- Content-based filtering using cosine similarity on course metadata vectors (topic tags, target roles, difficulty, division relevance)
- Data source: user profile (role, division, job family from SCIM/ServiceNow) matched against course metadata
- No ML model training required -- pure heuristics and vector similarity
- Effort: 2-3 person-months

**Phase 2 -- Collaborative filtering**:
- Requires 6+ months of usage data (enrollments, completions, ratings)
- User-item interaction matrix using implicit feedback (completions, time-on-task, assessment scores)
- Algorithm: Alternating Least Squares (ALS) via implicit library, or lightweight matrix factorization
- Blended with content-based scores using weighted combination
- Retraining: weekly batch job
- Effort: 2-3 person-months

**Phase 3 -- Learning-to-rank**:
- Train a ranking model (LightGBM or similar) that combines rule-based, content-based, and collaborative signals
- Features: user-content similarity, popularity, recency, organizational relevance, completion probability
- Evaluate with offline metrics (NDCG, MRR) and online A/B tests
- Effort: 2-3 person-months

**Explainability**: Each recommendation carries a reason code mapped to a Norwegian explanation template. Example: `reason: "role_match"` -> "Anbefalt fordi dette kurset er relevant for din rolle som {role_name}".

### 3.2 Semantic Search

**Decision: BUY components + BUILD integration layer**

**Why not build from scratch**: Building a search engine is unnecessary. The challenge is Norwegian language understanding and vector similarity, not search infrastructure.

**Why not pure SaaS search**: Algolia and similar services are excellent but expensive at scale, add external data dependency, and their Norwegian language support varies. We need tight control over the embedding pipeline for GDPR.

**Architecture**:

```
User query (Norwegian)
        |
        v
+-------------------+
| Query Processing  |  Language detection, query expansion,
| (Python service)  |  abbreviation handling (SVV, HMS, KKP)
+-------------------+
        |
        v
+-------------------+
| Embedding Model   |  Multilingual E5 or Cohere Embed
| (EU-hosted)       |  (self-hosted or EU API)
+-------------------+
        |
        v
+-------------------+
| Hybrid Search     |  Vector similarity (pgvector) +
| (PostgreSQL)      |  Full-text BM25 (tsvector)
+-------------------+
        |
        v
+-------------------+
| Re-ranking &      |  Personalization boost (role, division),
| Personalization   |  faceted filtering
+-------------------+
        |
        v
Search results with relevance scores
```

**Technology choices**:

| Component | Choice | Rationale |
|---|---|---|
| Vector store | **pgvector** (PostgreSQL extension) | Already using PostgreSQL; avoids separate vector DB operational burden; sufficient for <1M vectors; mature, well-supported |
| Full-text search | **PostgreSQL tsvector** with Norwegian dictionary | Built-in, supports Norwegian stemming (snowball), no additional infrastructure |
| Embedding model | **Multilingual E5-large** (self-hosted) or **Cohere Embed v3** (EU API) | Both handle Norwegian well; E5 is open-source and can run on CPU; Cohere has EU endpoint |
| Query processing | Custom Python service | Handles SVV-specific abbreviations, Norwegian compound word splitting |

**Why pgvector over Qdrant/Weaviate/Meilisearch**:
- pgvector runs inside the existing PostgreSQL instance -- zero additional infrastructure
- For our scale (~10,000-50,000 content items), pgvector with HNSW indexing is performant (<50ms queries)
- Simplifies backup, recovery, and data governance (one database to manage)
- If we outgrow pgvector, migration to a dedicated vector DB is straightforward (the embedding pipeline stays the same)
- Meilisearch is excellent for typo-tolerant keyword search but lacks native vector search; adding it means two search systems
- Qdrant/Weaviate add operational overhead for a scale that doesn't warrant it in MVP

**Embedding pipeline**:
1. On content publish/update: extract text from course metadata, descriptions, learning objectives, and SCORM content (where extractable)
2. Generate embedding vector (768 or 1024 dimensions) via the embedding model
3. Store in pgvector alongside content record
4. Index with HNSW for approximate nearest-neighbor search
5. Full-text index updated in parallel for BM25 scoring

**Hybrid scoring**: `final_score = alpha * vector_similarity + (1 - alpha) * bm25_score + personalization_boost`. Alpha tuned empirically (start at 0.6).

**Effort**: 2-3 person-months (including embedding pipeline, search API, query processing)

### 3.3 Content Generation

**Decision: BUY (LLM API via EU-hosted provider)**

**Why buy**: Building or fine-tuning our own generative model is prohibitively expensive and unnecessary. Commercial LLM APIs deliver high-quality Norwegian text generation, quiz creation, and summarization out of the box. The value we add is in the prompt engineering, content validation pipeline, and SVV-specific guardrails.

**Why not self-hosted open model**: Open models (Llama 3, Mistral) can be self-hosted in EU, but they underperform commercial APIs on Norwegian generation quality, require GPU infrastructure (expensive), and demand ongoing model operations expertise. Consider self-hosted only if API costs become prohibitive or data sensitivity requirements change.

**Provider options (ranked)**:

| Provider | EU Hosting | Norwegian Quality | Cost | Notes |
|---|---|---|---|---|
| **Azure OpenAI** (GPT-4o) | Sweden Central, West Europe | Excellent | Medium | Microsoft EU Data Boundary; strong GDPR position |
| **Anthropic Claude** (via AWS EU or GCP EU) | Frankfurt, London | Excellent | Medium | Strong safety/alignment; EU endpoints available |
| **Google Vertex AI** (Gemini) | Finland, Netherlands | Good | Medium | EU regions available |
| **Mistral** (La Plateforme) | France (EU-native) | Good | Lower | EU-native company; competitive on cost |

**Recommended**: Start with **Azure OpenAI (Sweden Central)** or **Anthropic Claude (EU endpoint)**. Both provide contractual guarantees that data is not used for model training and stays within EU. Implement an abstraction layer to switch providers.

**Content generation capabilities**:

| Capability | MVP | Phase 2 |
|---|---|---|
| Quiz question generation from course text | Yes | Enhanced with Bloom's taxonomy tagging |
| Course outline generation from learning objectives | Yes | Full module structure with activity suggestions |
| Content summarization (executive summaries, key takeaways) | Yes | Multi-format output (text, bullet points, infographic prompts) |
| Bokmal/Nynorsk translation assistance | Yes | Improved with SVV terminology glossary |
| Alt-text generation for images | Yes | Batch processing for existing content |
| Full course content drafts | No | Phase 2 |
| Scenario-based assessment generation | No | Phase 2 |

**LLM integration architecture**:

```
Content Manager UI
        |
        v
+-------------------+
| AI Gateway        |  API key management, rate limiting,
| (Python/FastAPI)  |  cost tracking, request/response logging
+-------------------+
        |
        v
+-------------------+
| PII Filter        |  Scan input for personal data;
| (Presidio or      |  redact before sending to LLM;
|  custom regex)    |  restore in output
+-------------------+
        |
        v
+-------------------+
| Prompt Manager    |  Versioned prompt templates per task;
| (config-driven)   |  SVV terminology injection;
|                   |  system prompt with domain constraints
+-------------------+
        |
        v
+-------------------+
| LLM Provider      |  Azure OpenAI / Claude / Mistral
| (EU endpoint)     |  via provider SDK
+-------------------+
        |
        v
+-------------------+
| Output Validator  |  Format validation, safety check,
|                   |  terminology compliance,
|                   |  AI-generated content label
+-------------------+
        |
        v
Draft content for human review
(never published without approval)
```

**Key design decisions**:
- **Human-in-the-loop mandatory**: All AI-generated content requires human review before publication
- **AI labeling**: All generated content tagged with `ai_generated: true` in metadata, displayed as "AI-assistert" in the UI
- **PII filtering**: Microsoft Presidio (open-source, runs locally) scans inputs/outputs for Norwegian PII patterns (personnummer, names, addresses)
- **Prompt versioning**: Prompts stored as versioned config, not hardcoded, enabling iteration without deployment
- **Cost controls**: Per-user and per-feature token budgets; alert at 80% of monthly budget

**Effort**: 2-3 person-months (gateway, PII filter, prompt engineering, validation pipeline, UI integration)

### 3.4 Chatbot / Learning Assistant

**Decision: BUY LLM API + BUILD RAG layer**

**Why this split**: The conversational AI engine (understanding intent, generating natural responses) is a solved problem via LLM APIs. The value we build is the Retrieval-Augmented Generation (RAG) pipeline that grounds responses in SVV's actual course catalog, policies, and FAQ content.

**MVP scope**: Phase 2 (not MVP). Requires indexed knowledge base and sufficient course content to be useful.

**Architecture**:

```
User message (Norwegian)
        |
        v
+-------------------+
| Intent Router     |  Classify: FAQ, course search, progress check,
|                   |  enrollment help, escalate to human
+-------------------+
        |
        v
+-------------------+
| RAG Retrieval     |  Query pgvector for relevant content chunks
| (same embedding   |  from: course descriptions, FAQ, policies,
|  pipeline as      |  help documentation
|  search)          |
+-------------------+
        |
        v
+-------------------+
| LLM Generation    |  System prompt: SVV context, Norwegian language,
| (EU-hosted)       |  scope constraints (learning topics only),
|                   |  citation requirements
+-------------------+
        |
        v
+-------------------+
| Response Filter   |  Check: on-topic? Safe? Cites sources?
|                   |  Identifies as AI? Offers human escalation?
+-------------------+
        |
        v
Chat response with source citations
```

**Shared infrastructure**: The chatbot reuses the embedding pipeline and vector store from semantic search. The RAG retrieval is essentially a search query with different output formatting. This means the marginal cost of adding a chatbot after search is implemented is primarily prompt engineering and UI work.

**Guardrails**:
- Scoped to learning-related topics within SVV context only
- Cannot modify enrollments, grades, or certifications -- read-only
- Always identifies as AI ("Jeg er en AI-assistent for laeringsplattformen")
- Escalation to human support always available
- Session history retained in-session only; not stored long-term without consent
- Available in Bokmal and Nynorsk (determined by user language preference)

**Effort**: 3-4 person-months (RAG pipeline, intent routing, prompt engineering, UI, testing)

### 3.5 Competency Analytics

**Decision: BUILD CUSTOM**

**Why build**: Competency gap analysis is the intersection of ServiceNow data (job families, roles, required skills) and LMS data (completions, certifications). This is domain-specific logic, not a generic ML problem. The analytics are rules and SQL queries, not trained models -- at least for MVP.

**Architecture**:

```
+-------------------+     +-------------------+
| ServiceNow Data   |     | LMS Data          |
| - Job families    |     | - Completions     |
| - Job roles       |     | - Certifications  |
| - Required skills |     | - Assessment scores|
| - Person-role map |     | - Learning paths  |
+-------------------+     +-------------------+
        |                         |
        v                         v
+-------------------------------------------+
| Competency Analytics Engine (Python)      |
|                                           |
| - Individual gap: required vs achieved    |
| - Team/division aggregation               |
| - Certification expiry forecasting        |
| - Compliance status tracking              |
| - Trend analysis (Phase 2)                |
+-------------------------------------------+
        |
        v
+-------------------+     +-------------------+
| API Endpoints     |     | PowerBI Export     |
| (for LMS UI)      |     | (structured data)  |
+-------------------+     +-------------------+
```

**MVP capabilities**:
1. **Individual competency gap view**: Compare user's completed courses/certifications against their role's requirements (from ServiceNow). Display as percentage complete with specific gaps highlighted.
2. **Team/division dashboard**: Aggregate gaps per division. Show compliance rates for mandatory training.
3. **Certification expiry alerts**: Query all certifications with expiry dates; alert at 90, 60, 30 days. Especially critical for arbeidsvarsling (work zone safety) and vinterdrift (winter operations).
4. **PowerBI data export**: Structured JSON/CSV export compatible with SVV's PowerBI instance. Pre-built data model matching SVV's reporting requirements.

**Phase 2 additions**:
- Predictive completion models (who is at risk of not completing mandatory training on time)
- Training effectiveness correlation (does completing course X improve competency assessment scores?)
- Competency trend forecasting
- Natural language queries for admins ("Hvor mange i Utbygging har fullfort HMS-kurset?")

**Technology**: Python (pandas for data processing, FastAPI for endpoints), PostgreSQL for storage, scheduled batch jobs for aggregation. No ML model training needed in MVP -- this is analytics, not prediction.

**Effort**: 2-3 person-months (MVP analytics), 3-4 person-months (Phase 2 predictive models)

### 3.6 Norwegian NLP

**Decision: BUY (multilingual models) + CONFIGURE for Norwegian**

This is not a standalone capability but a cross-cutting concern affecting search, embeddings, content generation, and the chatbot.

**Norwegian language challenges for an LMS**:

| Challenge | Impact | Solution |
|---|---|---|
| Bokmal vs Nynorsk | Two written standards; content may be in either | Multilingual embedding models treat both as Norwegian variants; search must handle both |
| Compound words | "Arbeidsvarslingskurs" = arbeid + varsling + kurs | Query processing with compound word splitting; embedding models handle this reasonably well |
| Domain terminology | SVV-specific terms (HMS, KKP, arbeidsvarsling, vinterdrift, Noark-5) | Custom terminology dictionary injected into prompts and search query expansion |
| Small language | Less training data than English/German | Use multilingual models (not Norwegian-only); validate quality empirically |
| Diacritics and special chars | ae/oe/aa vs unicode | Normalize in search pipeline; support both forms |

**Models that handle Norwegian well**:

| Model | Type | Norwegian Quality | Hosting |
|---|---|---|---|
| **Multilingual E5** (intfloat/multilingual-e5-large) | Embedding | Very good | Self-hosted (CPU or GPU) |
| **Cohere Embed v3** (embed-multilingual-v3.0) | Embedding | Very good | EU API endpoint |
| **NB-BERT** (NbAiLab/nb-bert-large) | Encoder (Norwegian-specific) | Excellent for classification/NER | Self-hosted |
| **NorBERT** | Encoder (Norwegian-specific) | Excellent for understanding tasks | Self-hosted |
| GPT-4o / Claude 3.5+ | Generative | Excellent Bokmal, good Nynorsk | EU API |

**Recommended approach**:
- **Embeddings**: Multilingual E5-large for both search and RAG retrieval. Handles Bokmal and Nynorsk. Open-source, self-hostable on CPU. 1024-dimension vectors.
- **Text generation**: Commercial LLM API (Azure OpenAI or Claude). System prompts specify language (Bokmal or Nynorsk based on user preference).
- **Classification/NER** (if needed later): NB-BERT from the National Library of Norway (NbAiLab). Best-in-class Norwegian-specific model for understanding tasks. Free and open-source.
- **Terminology**: Maintain a custom SVV glossary (50-200 terms) used for query expansion in search and injected into LLM system prompts.

**Effort**: Included in each capability's estimate. The SVV terminology dictionary is ~1 person-week of initial work + ongoing maintenance.

---

## 4. Technical Infrastructure

### 4.1 System Architecture

```
+================================================================+
|                        LMS Application                          |
|  (Next.js frontend + Python/FastAPI backend + PostgreSQL)       |
+================================================================+
        |                    |                    |
        v                    v                    v
+----------------+  +----------------+  +------------------+
| AI Gateway     |  | Embedding      |  | Analytics        |
| (FastAPI)      |  | Service        |  | Service          |
|                |  | (FastAPI)      |  | (Python + SQL)   |
| - LLM routing  |  |                |  |                  |
| - PII filter   |  | - Content      |  | - Gap analysis   |
| - Cost control |  |   indexing     |  | - Expiry alerts  |
| - Audit log    |  | - Query embed  |  | - Compliance     |
| - Rate limit   |  | - Batch reindex|  | - PowerBI export |
+----------------+  +----------------+  +------------------+
        |                    |                    |
        v                    v                    v
+================================================================+
|                     PostgreSQL                                  |
|  - Application data                                             |
|  - pgvector (embeddings, HNSW index)                           |
|  - tsvector (full-text search, Norwegian stemmer)              |
|  - Feature store tables (user profiles, content metadata)      |
|  - AI audit log                                                 |
+================================================================+
        |
        v
+================================================================+
|              External AI Services (EU-hosted)                   |
|  - Azure OpenAI (Sweden Central) or Claude (EU endpoint)       |
|  - Embedding model (self-hosted or Cohere EU)                  |
+================================================================+
```

### 4.2 Data Pipeline

```
Data Sources                  Processing              Storage & Index
+-----------+
| SCIM      | ---> User profile sync ---+
+-----------+                            |
                                         v
+-----------+                  +------------------+
| ServiceNow| ---> Competency  | PostgreSQL       |
| REST API  |     data sync    | - users          |
+-----------+      (nightly)   | - competencies   |
                               | - courses        |
+-----------+                  | - completions    |
| Course    | ---> Content     | - embeddings     |
| Authoring |     indexing     |   (pgvector)     |
+-----------+   (on publish)   | - features       |
                               +------------------+
+-----------+                          |
| xAPI/SCORM| ---> Learning            v
| Events    |     event log    +------------------+
+-----------+                  | Batch Jobs       |
                               | - Rec retraining |
                               | - Analytics agg  |
                               | - Embedding sync |
                               | - PowerBI export |
                               +------------------+
```

### 4.3 AI Gateway Design

The AI Gateway is the central service mediating all LLM interactions. It provides:

**Request pipeline**:
1. Authentication and authorization (which users/roles can access which AI features)
2. Rate limiting (per-user, per-feature, global)
3. PII detection and redaction (Presidio with Norwegian entity patterns)
4. Prompt template resolution (versioned prompts from config store)
5. Provider routing (primary + fallback provider)
6. Request logging (full audit trail, excluding PII)

**Response pipeline**:
1. Output safety check (content filtering for harmful/off-topic content)
2. PII scan on response (catch any PII the LLM might generate)
3. Format validation (expected JSON schema, required fields)
4. AI-generated content labeling
5. Response logging and cost tracking
6. Cache check (for identical/similar requests within TTL)

**Provider abstraction**:

```python
# Simplified provider interface
class LLMProvider(Protocol):
    async def generate(self, messages: list[Message], config: GenerationConfig) -> GenerationResult: ...
    async def embed(self, texts: list[str]) -> list[list[float]]: ...

# Concrete providers implement this interface
# Switching providers = changing config, not code
```

**Cost controls**:
- Token budget per feature per month (e.g., content generation: 2M tokens/month)
- Per-user daily limits (e.g., 20 content generation requests/day)
- Alert at 80% budget consumption
- Auto-fallback to cheaper model tier at 95%
- Monthly cost reporting per feature

### 4.4 Feature Store

Rather than a dedicated feature store system (Feast, Tecton), we use PostgreSQL tables that serve as a lightweight feature store. This avoids infrastructure complexity while providing the core benefit: pre-computed features available at query time.

**Tables**:

```sql
-- User features (updated on profile change or learning event)
CREATE TABLE user_features (
    user_id UUID PRIMARY KEY,
    role VARCHAR,
    division VARCHAR,
    job_family VARCHAR,
    competency_level JSONB,        -- {skill_id: level}
    completed_course_ids INTEGER[],
    active_learning_paths INTEGER[],
    certification_expiry JSONB,    -- {cert_id: expiry_date}
    last_activity_at TIMESTAMPTZ,
    feature_vector VECTOR(128),    -- Learned user embedding (Phase 2)
    updated_at TIMESTAMPTZ
);

-- Content features (updated on content publish/edit)
CREATE TABLE content_features (
    content_id INTEGER PRIMARY KEY,
    content_type VARCHAR,          -- course, learning_path, resource
    topics TEXT[],
    target_roles TEXT[],
    target_divisions TEXT[],
    difficulty_level INTEGER,
    language VARCHAR,              -- nb, nn, en
    completion_count INTEGER,
    avg_rating FLOAT,
    embedding VECTOR(1024),        -- From multilingual E5
    updated_at TIMESTAMPTZ
);

-- Interaction features (aggregated from events)
CREATE TABLE interaction_features (
    user_id UUID,
    content_id INTEGER,
    completed BOOLEAN,
    completion_date TIMESTAMPTZ,
    time_spent_minutes FLOAT,
    assessment_score FLOAT,
    rating INTEGER,
    PRIMARY KEY (user_id, content_id)
);
```

### 4.5 Vector Database Decision: pgvector

**Why pgvector over dedicated vector databases**:

| Factor | pgvector | Qdrant | Weaviate |
|---|---|---|---|
| Operational overhead | Zero (PostgreSQL extension) | New service to deploy, monitor, backup | New service to deploy, monitor, backup |
| Scale (our use case) | Handles <1M vectors easily | Designed for billions | Designed for billions |
| Query latency (50K vectors) | <20ms with HNSW | <10ms | <10ms |
| ACID transactions | Yes (same DB as app data) | No | No |
| Backup/recovery | Existing PostgreSQL backup | Separate backup infrastructure | Separate backup infrastructure |
| GDPR data management | One data store to govern | Additional data store to manage | Additional data store to manage |
| Cost | $0 additional | $200-500/month managed | $200-500/month managed |
| Migration path | Can migrate to dedicated if needed | -- | -- |

**When to reconsider**: If content catalog exceeds 500K items, or if we need sub-10ms vector search latency at scale, migrate the vector index to Qdrant (straightforward -- embeddings are portable).

### 4.6 ML Pipeline (Phase 2+)

For MVP, no ML model training is needed. All AI is either rule-based (recommendations, analytics) or API-based (LLM calls). When we add collaborative filtering and predictive models in Phase 2:

**Lightweight MLOps**:
- Model training: Python scripts executed as scheduled jobs (Kubernetes CronJobs or equivalent)
- Experiment tracking: MLflow (open-source, self-hosted) for logging parameters, metrics, and model artifacts
- Model versioning: MLflow model registry; models stored as artifacts in object storage (S3-compatible, EU region)
- Model serving: Pre-computed scores stored in PostgreSQL feature tables (batch inference, not real-time model serving)
- Monitoring: Custom metrics pushed to the application's existing monitoring stack (Prometheus/Grafana)

**Why not Kubeflow/SageMaker in MVP**: Our ML workloads are small (train on <100K interactions, serve from pre-computed tables). Full MLOps platforms add massive infrastructure complexity for a team of 1-2 ML engineers. MLflow + cron jobs is sufficient until we have multiple models in production.

---

## 5. EU/GDPR Compliance Architecture

### 5.1 Data Flow Compliance

All AI data flows must satisfy PEO-13 through PEO-29 (EU/EEA storage mandatory, sub-processor controls, transfer documentation).

```
+===========================+
| EU/EEA Boundary           |
|                           |
|  +---------------------+  |
|  | LMS Application     |  |
|  | (EU data center)    |  |
|  +---------------------+  |
|           |                |
|           v                |
|  +---------------------+  |
|  | AI Gateway           |  |
|  | (EU data center)     |  |
|  | - PII filtering      |  |
|  | - All logging here   |  |
|  +---------------------+  |
|           |                |
|           v                |
|  +---------------------+  |
|  | LLM API             |  |
|  | (Azure Sweden /     |  |
|  |  Claude EU /        |  |
|  |  Mistral France)    |  |
|  +---------------------+  |
|                           |
| Nothing leaves EU/EEA    |
+===========================+
```

### 5.2 Legal Basis per AI Feature

| AI Feature | Legal Basis | Personal Data Used | Data Minimization |
|---|---|---|---|
| Recommendations | Legitimate interest (Art. 6(1)(f)) | Role, division, learning history | Only role-level features, not individual behavior profiles |
| Semantic search | Legitimate interest (Art. 6(1)(f)) | Search queries (pseudonymized in logs) | Queries logged without user ID for analytics; with user ID only for personalization |
| Content generation | N/A | No personal data in prompts | PII filter strips any personal data before LLM call |
| Chatbot | Consent (Art. 6(1)(a)) | Conversation content (session only) | Not stored beyond session unless user opts in |
| Competency analytics | Performance of contract (Art. 6(1)(b)) | Role, competency profile, completions | Aggregated at team level for dashboards; individual view only accessible to self + manager |
| Predictive models (Phase 2) | Legitimate interest (Art. 6(1)(f)) with DPIA | Learning patterns (anonymized for training) | Models trained on anonymized aggregates; predictions are probabilities, not decisions |

### 5.3 PII Filtering Pipeline

```python
# PII detection entities for Norwegian context
NORWEGIAN_PII_ENTITIES = [
    "PERSON",           # Names (fornavn, etternavn)
    "NO_PERSONNUMMER",  # Norwegian national ID (11 digits)
    "EMAIL_ADDRESS",
    "PHONE_NUMBER",
    "NO_BANKKONTONUMMER",  # Bank account numbers
    "LOCATION",          # Addresses
    "IP_ADDRESS",
]

# Pipeline: detect -> redact -> send to LLM -> restore (if needed)
```

Implementation: Microsoft Presidio with custom Norwegian entity recognizers. Runs as a local service within the AI Gateway -- no PII leaves the gateway.

### 5.4 AI Audit Trail

Every AI interaction is logged for KI-01 compliance and LOG-04 traceability:

```json
{
    "timestamp": "2027-01-15T14:30:00Z",
    "feature": "content_generation",
    "user_id_hash": "sha256:abc...",  // Pseudonymized
    "request_type": "quiz_generation",
    "model_provider": "azure_openai",
    "model_version": "gpt-4o-2025-01",
    "prompt_template_version": "quiz_v3",
    "input_token_count": 450,
    "output_token_count": 320,
    "latency_ms": 2100,
    "pii_detected": false,
    "output_validated": true,
    "cost_usd": 0.012
}
```

Retention: 12 months full detail, then anonymized aggregates retained for model performance tracking.

### 5.5 EU AI Act Classification

All AI features are **limited risk** (transparency obligations only) because:
- AI is advisory only -- no automated decisions affecting employment, certification validity, or safety clearance
- Human review required for all content generation
- Users informed when interacting with AI
- AI-generated content labeled

No features qualify as **high risk** under Annex III because the LMS does not make employment decisions, access decisions to education, or safety-critical determinations autonomously.

Compliance measures:
- Transparency: AI features clearly labeled in UI
- AI-generated content marked in metadata (Art. 50)
- Technical documentation maintained for all AI components (Art. 11 readiness)
- Human oversight mechanisms documented (Art. 14 readiness)

---

## 6. MVP vs Later Phases

### 6.1 Phase 1 -- MVP (Months 1-4)

**Goal**: Ship a working LMS with useful AI that doesn't require accumulated user data.

| AI Feature | MVP Scope | Effort |
|---|---|---|
| **Rule-based recommendations** | Mandatory courses, expiry alerts, role-based content matching | 2-3 PM |
| **Semantic search** | Hybrid search (pgvector + BM25) with Norwegian support | 2-3 PM |
| **Basic content generation** | Quiz generation, summarization, Bokmal/Nynorsk translation | 2-3 PM |
| **Competency gap dashboard** | Individual + team view from ServiceNow data | 1-2 PM |
| **AI Gateway infrastructure** | PII filter, audit log, cost controls, provider abstraction | 1-2 PM |
| **Total MVP AI effort** | | **8-12 PM** |

**What MVP does NOT include** (and why):
- Collaborative filtering (needs 6+ months of usage data)
- Chatbot (needs indexed knowledge base + sufficient content; premature without content migration complete)
- Predictive analytics (needs 12+ months of historical data)
- Advanced content generation (full course drafts -- too risky without established quality baselines)

### 6.2 Phase 2 -- Enhanced Personalization (Months 5-12)

| AI Feature | Phase 2 Scope | Effort | Dependency |
|---|---|---|---|
| **Collaborative filtering** | User-item similarity, blended with content-based | 2-3 PM | 6+ months usage data |
| **Chatbot / learning assistant** | RAG-based conversational AI | 3-4 PM | Content indexed, FAQ created |
| **Learning path optimization** | AI-suggested course sequences | 1-2 PM | Collaborative filtering |
| **Admin intelligence** | Anomaly detection, automated compliance reports | 2-3 PM | Analytics pipeline mature |
| **Enhanced content generation** | Course outline generation, scenario assessments | 1-2 PM | Content team feedback |
| **Total Phase 2 AI effort** | | **10-14 PM** |

### 6.3 Phase 3 -- Advanced Analytics (Months 13-24)

| AI Feature | Phase 3 Scope | Effort | Dependency |
|---|---|---|---|
| **Predictive completion models** | At-risk learner identification | 2-3 PM | 12+ months data |
| **Training effectiveness analysis** | Course impact on competency growth | 2-3 PM | ServiceNow + LMS data correlation |
| **Natural language admin queries** | "How many in Utbygging completed HMS?" | 2-3 PM | Mature data model |
| **Learning-to-rank** | Unified ranking model combining all signals | 2-3 PM | Phase 2 models validated |
| **Total Phase 3 AI effort** | | **8-12 PM** |

### 6.4 Data Accumulation Requirements

| AI Capability | Minimum Data Needed | Estimated Time to Accumulate |
|---|---|---|
| Content-based recommendations | Course metadata + user profiles | Available at launch (from ServiceNow + content migration) |
| Collaborative filtering | ~10,000 user-course interactions | 3-6 months with 5,000 users |
| Predictive completion | ~50,000 enrollment records with outcomes | 6-12 months |
| Training effectiveness | Completion data + competency assessments over time | 12+ months |
| Anomaly detection | Baseline of 6+ months normal patterns | 6 months |

---

## 7. Infrastructure Cost Estimates

### 7.1 MVP Infrastructure (Annual)

| Component | Service | Estimated Cost (NOK/year) | Notes |
|---|---|---|---|
| LLM API (content gen) | Azure OpenAI / Claude | 150,000 - 300,000 | ~500K-1M tokens/day for content gen + search reranking |
| Embedding model | Self-hosted (CPU) or Cohere | 0 - 50,000 | Self-hosted: included in compute; Cohere: ~$0.0001/query |
| PostgreSQL (with pgvector) | Managed (e.g., Neon, Supabase, or cloud provider) | Included in platform | Already part of the LMS infrastructure |
| PII filtering (Presidio) | Self-hosted | 0 | Runs as a sidecar, minimal compute |
| ML pipeline (Phase 2+) | MLflow + compute | 30,000 - 80,000 | Small training jobs, artifact storage |
| Monitoring | Existing stack | 0 | AI metrics added to existing Prometheus/Grafana |
| **Total AI infrastructure** | | **180,000 - 430,000** | |

### 7.2 Scaling Projections

| Scale | Users | Content Items | Vectors | LLM Calls/Month | Est. AI Cost (NOK/year) |
|---|---|---|---|---|---|
| SVV MVP | 5,000 | 500 | 5,000 | 10,000 | 200,000 |
| SVV full | 20,000 | 2,000 | 20,000 | 50,000 | 400,000 |
| Multi-tenant (5 agencies) | 100,000 | 10,000 | 100,000 | 250,000 | 800,000 |
| Multi-tenant (20 agencies) | 500,000 | 50,000 | 500,000 | 1,000,000 | 1,500,000 |

At 500K+ vectors, consider migrating from pgvector to Qdrant (~200K NOK/year additional for managed Qdrant).

### 7.3 Team Cost

| Role | Phase 1 (MVP) | Phase 2 | Phase 3 | Notes |
|---|---|---|---|---|
| Senior AI/ML Engineer | 1.0 FTE | 1.0 FTE | 0.5 FTE | Core AI development |
| Backend Engineer (AI integration) | 0.5 FTE | 0.5 FTE | 0.25 FTE | Gateway, APIs, data pipelines |
| Data Engineer | 0.5 FTE | 0.5 FTE | 0.25 FTE | Embedding pipeline, feature store, analytics |
| **Total AI team** | **2.0 FTE** | **2.0 FTE** | **1.0 FTE** | |

---

## 8. Technology Stack Summary

| Layer | Technology | Version/Variant | Justification |
|---|---|---|---|
| **AI Gateway** | Python + FastAPI | 3.12+ | Same stack as backend; async for LLM streaming |
| **LLM Provider** | Azure OpenAI or Anthropic Claude | GPT-4o / Claude 3.5+ | EU hosting, Norwegian quality, GDPR-compatible |
| **Embedding Model** | Multilingual E5-large | intfloat/multilingual-e5-large | Open-source, Norwegian support, CPU-capable |
| **Vector Store** | pgvector | 0.7+ (HNSW support) | Zero infrastructure overhead; sufficient scale |
| **Full-text Search** | PostgreSQL tsvector | Norwegian snowball stemmer | Built-in, Norwegian stemming support |
| **PII Detection** | Microsoft Presidio | + custom Norwegian recognizers | Open-source, local processing |
| **Analytics** | Python + pandas + SQL | | Lightweight, no Spark needed at this scale |
| **ML Pipeline** | MLflow | Phase 2+ | Open-source experiment tracking and model registry |
| **Feature Store** | PostgreSQL tables | Custom schema | Avoid Feast/Tecton overhead; tables are sufficient |
| **Monitoring** | Prometheus + Grafana | AI-specific dashboards | Reuse existing infrastructure |
| **Norwegian NLP** | NB-BERT (NbAiLab) | Phase 2+ for classification | Best Norwegian-specific model if needed |

---

## 9. Risk Assessment

### 9.1 AI-Specific Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| **LLM API Norwegian quality degrades** | Low | Medium | Monitor output quality; maintain fallback provider; consider Norwegian-specific fine-tuning |
| **pgvector performance at scale** | Low | Medium | Benchmark at projected scale; migration path to Qdrant documented |
| **LLM provider changes EU data terms** | Low | High | Abstraction layer enables 1-week provider switch; monitor policy changes |
| **Content generation hallucination** | Medium | High | Human-in-the-loop mandatory; RAG grounding; factual validation checklist |
| **AI feature scope creep** | High | Medium | Phase discipline; each feature has clear acceptance criteria and data dependency |
| **Insufficient data for collaborative filtering** | Medium | Low | Content-based filtering works without user data; collaborative is an enhancement, not dependency |
| **PII leakage through LLM** | Low | High | Presidio scanning on input/output; no personal data in prompts; audit trail |
| **AI costs exceed budget** | Medium | Medium | Token budgets per feature; automatic fallback to cheaper models; cost monitoring |

### 9.2 Build vs Buy Risk Summary

| Decision | Risk if Wrong | Reversibility | Confidence |
|---|---|---|---|
| Build recommendation engine | Low -- worst case, simple rule-based system still works | Easy to replace with library | High |
| Use pgvector over dedicated vector DB | Low -- migration to Qdrant is straightforward | Medium (data migration needed) | High |
| Buy LLM API (not self-host) | Medium -- provider dependency | Medium (switch providers via abstraction) | High |
| Build analytics (not use BI tool) | Low -- analytics are SQL queries | Easy to migrate to any tool | High |
| Defer chatbot to Phase 2 | Low -- chatbot without content is useless anyway | N/A | High |

---

## 10. Key Architecture Decisions (ADRs)

### ADR-AI-001: pgvector over Dedicated Vector Database
- **Status**: Accepted
- **Context**: Need vector storage for ~10K-50K content embeddings
- **Decision**: Use pgvector extension in existing PostgreSQL
- **Rationale**: Zero operational overhead, sufficient performance, single data store for GDPR, includes HNSW indexing
- **Revisit when**: Content exceeds 500K items or query latency exceeds 50ms at p99

### ADR-AI-002: Commercial LLM API over Self-Hosted Open Model
- **Status**: Accepted
- **Context**: Need text generation for content creation, chatbot, and quiz generation in Norwegian
- **Decision**: Use Azure OpenAI (Sweden Central) or Anthropic Claude (EU endpoint) via API
- **Rationale**: Superior Norwegian quality, no GPU infrastructure needed, contractual GDPR guarantees, faster time-to-market
- **Revisit when**: API costs exceed 500K NOK/year, or data sensitivity requirements change, or open models match Norwegian quality

### ADR-AI-003: Rule-Based Recommendations for MVP
- **Status**: Accepted
- **Context**: Need personalized recommendations at launch without historical user data
- **Decision**: Phase 1 uses rule-based + content-based filtering only; collaborative filtering deferred to Phase 2
- **Rationale**: Collaborative filtering requires 6+ months of usage data; rule-based delivers immediate value for safety-critical compliance alerts and role-based matching
- **Revisit when**: 10K+ user-course interactions accumulated

### ADR-AI-004: Centralized AI Gateway
- **Status**: Accepted
- **Context**: Multiple AI features need LLM access with consistent PII filtering, logging, and cost controls
- **Decision**: All LLM interactions route through a single AI Gateway service
- **Rationale**: Single point for PII filtering, audit logging, cost controls, and provider switching; avoids duplicating these concerns in each feature
- **Revisit when**: Gateway becomes a performance bottleneck (unlikely at this scale)

### ADR-AI-005: PostgreSQL as Feature Store
- **Status**: Accepted
- **Context**: Need pre-computed features for recommendations and analytics
- **Decision**: Use dedicated PostgreSQL tables rather than a feature store system (Feast, Tecton)
- **Rationale**: Our feature count is small (<50 features), update frequency is low (daily batch), and we're already operating PostgreSQL. Feature store systems add significant infrastructure complexity.
- **Revisit when**: Real-time feature computation needed, or feature count exceeds 200, or multiple ML models require consistent feature access

---

## 11. Open Questions

1. **Embedding model hosting**: Self-host Multilingual E5 on CPU (zero cost, ~200ms/query) or use Cohere EU API (~$0.0001/query, ~50ms)? Decision depends on latency requirements and whether we want to avoid any external embedding service.

2. **LLM provider final selection**: Azure OpenAI vs Claude vs Mistral. Need to evaluate: (a) Norwegian Nynorsk quality specifically, (b) EU data processing agreement terms, (c) pricing for expected volume. Recommend running a structured evaluation with SVV-specific prompts.

3. **NB-BERT usage**: The National Library of Norway's NB-BERT is excellent for Norwegian text classification and named entity recognition. Do we need these capabilities in Phase 1 (e.g., auto-tagging course content by topic)? Current assessment: not for MVP, but valuable for Phase 2 content enrichment.

4. **ServiceNow sync frequency**: Competency analytics quality depends on fresh data from ServiceNow. Current plan is nightly batch sync. Is near-real-time needed (e.g., webhook on role change)?

5. **A/B testing infrastructure**: How do we measure recommendation quality? Need agreement on metrics (click-through, completion rate, user satisfaction) and A/B testing framework before Phase 2.
