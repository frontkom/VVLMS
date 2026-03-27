# MVP Feature Set and Epic Breakdown
## Norwegian Public Sector LMS -- Product Build Plan

**Date**: 2026-03-22
**Target customer**: Statens vegvesen (SVV) -- Case 25/223727
**Product vision**: A purpose-built, cloud-native LMS SaaS for the Norwegian public sector
**Demo target**: POC demonstration 11-12.05.2026 (~7 weeks from today)
**MVP target**: Working demo environment by 07.05.2026

---

## Phasing Strategy

| Phase | Timeline | Goal |
|---|---|---|
| **MVP** | Now -- 06.05.2026 (~7 weeks) | Demonstrate all 5 POC scenarios to SVV evaluators. Must be functional enough for hands-on testing. |
| **Phase 2** | 05.2026 -- 01.01.2027 | Production-ready for SVV go-live. Full integrations, migration tooling, compliance. |
| **Phase 3** | 2027+ | Multi-tenant market expansion, advanced AI, payment, marketplace. |

### MVP Scoping Principle

The MVP is driven entirely by the **5 POC demonstration scenarios** (Bilag 1 vedlegg 7) and the **hands-on evaluator testing**. Everything not needed for those two evaluation events is deferred. Quality (80% weight) and usability (12.5% of quality) are the dominant award criteria -- the demo must be polished, not just functional.

---

## EPIC 1: Platform Foundation

**Description**: Multi-tenant core platform with user management, RBAC, organization hierarchy, notifications, and audit logging. The skeleton on which everything else is built.
**Phase**: MVP (core) / Phase 2 (full multi-tenancy, advanced audit)
**Complexity**: XL
**Dependencies**: None -- this is the foundation.

### User Stories

#### MVP

**PF-01**: As a platform operator, I want to provision a new tenant with its own branding and configuration, so that each customer organization operates in an isolated environment.
- **Acceptance criteria**:
  - Tenant can be created via admin CLI/API
  - Tenant has isolated data (users, courses, completions)
  - Tenant-specific branding (logo, colors, name) renders on all pages
  - At least one demo tenant (SVV) is fully configured for POC

**PF-02**: As an administrator, I want to manage users with role-based access control, so that people see only what their role permits.
- **Acceptance criteria**:
  - Roles: Learner, Leader (personnel responsibility), Content Manager, Division Admin, System Admin
  - Permissions are enforced on both UI and API level
  - Role assignment can be manual or via SCIM attribute mapping
  - Demo accounts exist for all 5 POC roles

**PF-03**: As an administrator, I want to model my organizational hierarchy (divisions, sections, units), so that reporting, enrollment, and access follow the org structure.
- **Acceptance criteria**:
  - Tree-structured org model (SVV: Vegdirektoratet + 6 divisions)
  - Users belong to one primary org unit
  - Leaders see their unit and sub-units
  - Org structure is importable (CSV or SCIM)

**PF-04**: As a user, I want to receive notifications about assigned courses, deadlines, and completions, so that I stay informed without checking the platform constantly.
- **Acceptance criteria**:
  - In-app notification center with unread count
  - Email notifications for: course assignment, deadline approaching (7 days, 1 day), completion confirmation
  - Notification preferences (opt-out per category)
  - Leader notifications for team certification expiry

**PF-05**: As a system administrator, I want all user actions logged for audit purposes, so that we meet traceability requirements (LOG-04).
- **Acceptance criteria**:
  - Login/logout events logged
  - Course start/complete events logged
  - Admin actions (user changes, course changes) logged
  - Logs include timestamp, user ID, action, target object
  - Logs are queryable by admin

#### Phase 2

**PF-06**: As a platform operator, I want automated tenant onboarding with self-service configuration, so that new public sector customers can be set up efficiently.

**PF-07**: As a system administrator, I want detailed audit log export and retention policies, so that we comply with arkivloven requirements for long-term traceability.

**PF-08**: As a platform operator, I want tenant usage metering, so that I can bill customers based on active users or site license.

---

## EPIC 2: Authentication & Identity

**Description**: SSO via OIDC/SAML, SCIM 2.0 user provisioning, ID-porten for external users, MFA, and Maskinporten for M2M API auth. This is the most architecturally critical integration for SVV.
**Phase**: MVP (OIDC + SCIM + ID-porten stub) / Phase 2 (full ForgeRock, Maskinporten, MFA)
**Complexity**: XL
**Dependencies**: EPIC 1 (Platform Foundation)

### User Stories

#### MVP

**AU-01**: As an internal SVV employee, I want to log in via SSO (OIDC), so that I access the LMS seamlessly with my existing credentials.
- **Acceptance criteria**:
  - OIDC authorization code flow with PKCE implemented
  - Redirect to IdP login page, return with token
  - User session created on first login (JIT provisioning if SCIM user exists)
  - Session timeout configurable (default 8 hours)
  - SSO logout (front-channel or back-channel)
  - Demo: works with a configured OIDC provider (Keycloak or ForgeRock sandbox)

**AU-02**: As an IT administrator at SVV, I want users provisioned automatically via SCIM 2.0, so that the LMS user base stays in sync with our identity system without manual work.
- **Acceptance criteria**:
  - SCIM 2.0 server endpoints: /Users, /Groups
  - Supports CREATE, UPDATE, DELETE (deactivate), PATCH operations
  - Maps SCIM attributes to LMS user profile: name, email, org unit, role, status
  - Handles organizational hierarchy via SCIM group or custom extension
  - Bearer token authentication for SCIM endpoint
  - Reconciliation: bulk import/sync capability

**AU-03**: As an external user (e.g., contractor for arbeidsvarsling), I want to log in via ID-porten, so that I can access courses and certificates using my national eID.
- **Acceptance criteria**:
  - ID-porten OIDC flow (via ForgeRock as broker, or direct)
  - User identified by fødselsnummer (national ID) -- stored securely
  - First login creates external user profile
  - External users see only the external portal (EPIC 6)
  - Demo: functional login flow (can use ID-porten test environment or mock)

**AU-04**: As a user, I want SAML 2.0 SSO as a fallback federation option, so that SVV's multi-protocol IAM environment is fully supported.
- **Acceptance criteria**:
  - SP-initiated SAML 2.0 flow
  - Metadata exchange with IdP
  - Attribute mapping from SAML assertions to user profile

#### Phase 2

**AU-05**: As a system administrator, I want MFA enforced for all admin and external users, so that we meet IAM-07 and IAM-09 requirements.
- **Acceptance criteria**:
  - MFA via TOTP or push notification
  - Configurable per role (mandatory for admin, optional for learners)
  - MFA can be delegated to ForgeRock AM (preferred for SVV)

**AU-06**: As a backend system, I want to authenticate via Maskinporten, so that server-to-server API calls (Kursoppslag, ServiceNow) use standard Norwegian government M2M auth.
- **Acceptance criteria**:
  - Maskinporten JWT grant flow implemented
  - Scoped API access based on Maskinporten claims
  - Token caching and refresh

**AU-07**: As an external user, I want to log in via BankID for high-assurance identity verification, so that my competency certificates are legally tied to my verified identity.

---

## EPIC 3: Content Engine

**Description**: SCORM 1.2/2004 player, xAPI/cmi5 support, content upload/management, metadata/tagging, and built-in quiz/assessment engine. Must support SVV's existing 297 Articulate-produced courses.
**Phase**: MVP (SCORM player + content management + basic quiz) / Phase 2 (xAPI, cmi5, advanced assessments)
**Complexity**: XL
**Dependencies**: EPIC 1 (Platform Foundation)

### User Stories

#### MVP

**CE-01**: As a content manager, I want to upload SCORM 1.2 and SCORM 2004 packages, so that our existing e-learning courses work on the new platform.
- **Acceptance criteria**:
  - Upload .zip SCORM packages via drag-and-drop or file picker
  - Package validation (imsmanifest.xml parsing)
  - SCORM 1.2 API adapter (LMSInitialize, LMSGetValue, LMSSetValue, LMSCommit, LMSFinish)
  - SCORM 2004 RTE API adapter (Initialize, GetValue, SetValue, Commit, Terminate)
  - Tracks: completion_status, success_status, score, suspend_data, session_time
  - Content renders in embedded iframe with proper sandboxing
  - **POC demo**: Upload and launch an Articulate Storyline SCORM package

**CE-02**: As a content manager, I want to create and manage courses with rich metadata, so that learners can discover relevant content.
- **Acceptance criteria**:
  - Course fields: title, description (rich text), thumbnail, duration estimate, language, difficulty level
  - Tagging: free-text tags + controlled vocabulary (competency area, division, mandatory/optional)
  - Course status workflow: Draft > Published > Archived
  - Version management: update course content without losing completion history
  - **POC demo**: Create a new course, add SCORM content, set metadata, publish

**CE-03**: As a content manager, I want to build quizzes and assessments within the platform, so that I can evaluate learner knowledge without external tools.
- **Acceptance criteria**:
  - Question types: multiple choice, multiple select, true/false, fill-in-the-blank, ordering/ranking
  - Question bank with tagging and reuse
  - Quiz settings: pass score, attempt limits, time limit, randomization, show/hide correct answers
  - Score tracking integrated with progress/completion
  - **POC demo**: Create a quiz, take it as a learner, view results as content manager

**CE-04**: As a content manager, I want to organize content into a structured catalog with categories, so that learners can browse by topic area.
- **Acceptance criteria**:
  - Hierarchical content categories (e.g., HMS > Arbeidsvarsling > Grunnkurs)
  - Course listing with filters (category, format, duration, mandatory/optional)
  - Thumbnail grid and list views
  - Content manager can assign courses to categories

**CE-05**: As a content manager, I want to update a course's content while preserving existing learner history, so that course updates don't erase progress records.
- **Acceptance criteria**:
  - New content version replaces the active version
  - Completions against prior versions are retained and marked with version
  - Option to require re-completion after major update
  - **POC demo (scenario 4.3)**: Update a course and archive an old version

#### Phase 2

**CE-06**: As a platform, I want to emit and receive xAPI statements, so that learning data from external systems can be integrated and our data can be consumed externally.
- **Acceptance criteria**:
  - xAPI statement generation for all learning events
  - LRS (Learning Record Store) built-in or integrated
  - xAPI statement forwarding to external LRS

**CE-07**: As a content manager, I want cmi5 package support, so that we can use the modern content standard that combines SCORM portability with xAPI data richness.

**CE-08**: As a content manager, I want to upload and manage non-SCORM content (PDF, video, links, documents), so that all learning materials live in one place.

---

## EPIC 4: Learning Management

**Description**: Course catalog and discovery, enrollment/registration, progress tracking, completion/grading, learning paths, and classroom/event management. This is the core learning experience.
**Phase**: MVP (catalog, enrollment, progress, learning paths, basic classroom) / Phase 2 (advanced event management, waitlists, virtual classroom integrations)
**Complexity**: XL
**Dependencies**: EPIC 1 (Platform Foundation), EPIC 3 (Content Engine)

### User Stories

#### MVP

**LM-01**: As a learner, I want a personalized landing page showing my assigned courses, progress, and upcoming deadlines, so that I know what to do next.
- **Acceptance criteria**:
  - Dashboard shows: assigned courses, in-progress courses, upcoming deadlines, recent completions
  - Progress indicators (percentage, status badges)
  - Quick-launch buttons to resume courses
  - Responsive design (desktop + tablet + mobile)
  - **POC demo (scenario 1.1)**: Internal user lands on personalized page

**LM-02**: As a learner, I want to search and browse the course catalog, so that I can find relevant learning on my own.
- **Acceptance criteria**:
  - Full-text search across course title, description, tags
  - Filter by: category, format (digital/classroom), duration, mandatory/optional, language
  - Sort by: relevance, newest, popularity, deadline
  - Search results show thumbnail, title, duration, format icon, rating
  - **POC demo (scenario 1.1)**: Search for a course, view details, self-enroll

**LM-03**: As a learner, I want to enroll in courses (self-enrollment or assigned), so that I can start learning.
- **Acceptance criteria**:
  - Self-enrollment for optional courses (one-click)
  - Admin/leader assignment (individual or bulk by org unit)
  - Enrollment confirmation notification
  - Enrollment with deadline (mandatory courses)
  - Enrollment history visible to learner and leader

**LM-04**: As a learner, I want my progress tracked automatically as I work through content, so that I can resume where I left off.
- **Acceptance criteria**:
  - SCORM suspend_data enables resume
  - Progress percentage tracked and displayed
  - Time spent tracked per session and total
  - Completion triggered by SCORM completion_status or manual marking (for non-SCORM)
  - **POC demo (scenario 1.1)**: Resume a partially completed course

**LM-05**: As a leader, I want to see my team's learning status at a glance, so that I can ensure compliance and support development.
- **Acceptance criteria**:
  - Team dashboard showing: enrolled, in-progress, completed, overdue per team member
  - Filter by: course, time period, status
  - Drill-down to individual learner detail
  - Alert/highlight for overdue mandatory courses
  - **POC demo (scenario 3.1)**: Leader views team learning status

**LM-06**: As a content manager, I want to create learning paths (ordered sequences of courses), so that learners follow a structured progression.
- **Acceptance criteria**:
  - Learning path = ordered list of courses/activities with optional prerequisites
  - Progress tracking at path level (X of Y completed)
  - Prerequisite enforcement (must complete A before starting B)
  - Learning path can be assigned like a single course
  - Visual progress indicator (timeline/checklist view)
  - **POC demo (scenario 1.1)**: Learner progresses through a learning path
  - **POC demo (scenario 4.2)**: Content manager creates a learning path

**LM-07**: As a leader, I want to assign courses and learning paths to my team members, so that I can direct their development.
- **Acceptance criteria**:
  - Assign to individual or entire org unit
  - Set deadline (optional)
  - Add personal message/context for assignment
  - Assignee receives notification
  - **POC demo (scenario 3.3)**: Leader assigns a course to a team member

**LM-08**: As a content manager, I want to manage classroom courses and events (physical and virtual), so that instructor-led training is handled in the same platform.
- **Acceptance criteria**:
  - Event fields: title, description, instructor, location (physical address or virtual link), date/time, capacity
  - Registration with capacity limits
  - Waitlist when full
  - Attendance marking (by instructor or admin)
  - Calendar view of upcoming events
  - **POC demo (scenario 4.1)**: Create a classroom course with sessions

**LM-09**: As a leader, I want to approve or reject activity completions that require verification, so that workplace-based learning is validated.
- **Acceptance criteria**:
  - Activity types that require leader approval (job observation, practical exam, emergency drill)
  - Approval workflow: learner submits > leader reviews > approve/reject with comment
  - **POC demo (scenario 3.3)**: Leader approves a learning activity

**LM-10**: As a content manager, I want to configure courses as mandatory or optional and set them per org unit, so that compliance training is enforced for the right people.
- **Acceptance criteria**:
  - Mandatory flag per course
  - Mandatory assignment scoped to org unit(s)
  - Mandatory courses surface prominently on learner dashboard
  - Overdue mandatory courses trigger escalation notifications
  - **POC demo (scenario 4.4)**: Configure mandatory vs. optional

#### Phase 2

**LM-11**: As a learner, I want to join virtual classroom sessions via integrated video conferencing (Teams, Zoom), so that I can attend from any location.

**LM-12**: As a content manager, I want advanced waitlist management with automatic promotion when spots open, so that classroom events are efficiently utilized.

**LM-13**: As a learner, I want to submit assignments (file upload, text) for instructor review, so that workplace-based activities are documented.

---

## EPIC 5: Competency Framework

**Description**: Competency/skills definitions, role-to-competency mapping, individual profiles, gap analysis, certification management with PDF generation. Critical for SVV's KKP portal use case and HMS/safety compliance.
**Phase**: MVP (competency model, certifications, basic gap analysis) / Phase 2 (advanced gap analysis, predictions, ServiceNow sync)
**Complexity**: L
**Dependencies**: EPIC 1 (Platform Foundation), EPIC 4 (Learning Management)

### User Stories

#### MVP

**CF-01**: As an administrator, I want to define competencies and skills with levels, so that we have a structured vocabulary for what people need to know.
- **Acceptance criteria**:
  - Competency = named skill area with description
  - Levels (e.g., Grunnleggende, Erfaren, Ekspert) configurable per competency
  - Competencies organized in a hierarchy (domains > competencies > levels)
  - Bulk import from CSV/Excel

**CF-02**: As an administrator, I want to map competencies to roles (job families), so that we can define what each role requires.
- **Acceptance criteria**:
  - Role = named job function (e.g., "Prosjektleder Utbygging")
  - Map required competencies + levels to each role
  - A user can have one or more roles
  - Role definitions importable from ServiceNow (Phase 2: automated sync)

**CF-03**: As a learner, I want to see my individual competency profile, so that I understand my current skill levels and what's needed for my role.
- **Acceptance criteria**:
  - Visual display of acquired competencies vs. required (for assigned roles)
  - Competency sources: course completion, certification, manual attestation
  - Gap indicators (missing competency, insufficient level)
  - **POC demo (scenario 3.1, leader view)**: See team member competency status

**CF-04**: As a leader, I want a competency gap analysis for my team, so that I can identify development priorities.
- **Acceptance criteria**:
  - Team-level view: required vs. acquired per competency
  - Heat map or table showing gaps across team members
  - Filter by competency domain, role, urgency
  - Drill-down to individual gap detail
  - **POC demo (scenario 3.1)**: Leader views team competency gaps

**CF-05**: As a platform, I want to issue, track, and manage certifications with expiry dates, so that compliance-critical competencies are monitored.
- **Acceptance criteria**:
  - Certification = competency proof with issue date, expiry date, issuing authority
  - Auto-issue on course/path completion (configurable rules)
  - Expiry tracking with alerts (90 days, 30 days, 7 days before expiry)
  - Renewal workflow: complete refresher course > new certificate issued
  - Certificate status: valid, expiring soon, expired, revoked
  - **POC demo (scenario 3.2)**: Leader sees certification expiry alerts

**CF-06**: As a user, I want to receive a competency certificate as a PDF, so that I have a verifiable document of my qualification.
- **Acceptance criteria**:
  - PDF/A format generated from template
  - Certificate contains: name, fødselsnummer (masked), competency/course, date, expiry, unique certificate ID
  - QR code or URL for third-party verification (05.019c)
  - Digital signature (optional Phase 2)
  - Downloadable by user and administrator
  - Batch generation capability (for KKP scenario: 15,000/year)

#### Phase 2

**CF-07**: As a leader, I want competency gap predictions based on upcoming certification expiries and role changes, so that I can plan training proactively.

**CF-08**: As an administrator, I want bidirectional competency sync with ServiceNow, so that job families, roles, and skills are maintained in one place and reflected in both systems.

**CF-09**: As an administrator, I want Noark-5 compatible certificate archiving, so that competency certificates meet arkivloven requirements without manual processes.

---

## EPIC 6: External Portal

**Description**: Self-registration portal for external users (contractors, partners), branded per tenant, with its own course catalog, certificate issuance, and access control. Models SVV's KKP portal (Kurs- og kompetanseportal for arbeidsvarsling og vinterdrift).
**Phase**: MVP (basic external portal with self-registration and certificates) / Phase 2 (full KKP migration, branded sub-portals, payment)
**Complexity**: L
**Dependencies**: EPIC 2 (Authentication -- ID-porten), EPIC 3 (Content Engine), EPIC 5 (Competency Framework)

### User Stories

#### MVP

**EP-01**: As an external user, I want to register and create an account using ID-porten, so that I can access courses relevant to my work.
- **Acceptance criteria**:
  - Self-registration flow: ID-porten login > profile completion (employer, role, phone)
  - External user profile created with limited permissions
  - Welcome/onboarding screen on first login
  - **POC demo (scenario 2.1)**: External user registers and logs in for the first time

**EP-02**: As an external user, I want to see a catalog of courses available to me, so that I can find and enroll in relevant training.
- **Acceptance criteria**:
  - External catalog is a curated subset of the full catalog
  - Courses flagged as "external-visible" by content manager
  - Browse/search/filter within external catalog
  - Self-enrollment for external courses

**EP-03**: As an external user, I want to earn and download competency certificates, so that I can prove my qualifications to employers and authorities.
- **Acceptance criteria**:
  - Certificate issued on course completion (same as CF-06)
  - Certificate accessible from external user's profile
  - Historical certificates viewable
  - Third-party verification via QR code / URL

**EP-04**: As an administrator, I want to control which courses and content are visible to external users, so that internal-only materials remain restricted.
- **Acceptance criteria**:
  - Course visibility: internal-only, external-only, both
  - External portal has separate navigation and branding
  - External users cannot access internal content or internal user data

#### Phase 2

**EP-05**: As a platform operator, I want tenant-specific branded sub-portals for external audiences, so that each customer organization presents its own branded learning experience to external users.

**EP-06**: As an external user, I want to pay for courses online, so that paid training is accessible without invoicing.

**EP-07**: As an administrator, I want to migrate KKP portal data (350,000 historical certificates, 15,000/year issuance), so that the external portal replaces KKP without data loss.

---

## EPIC 7: Integrations

**Description**: ForgeRock IAM connector, ServiceNow connector, PowerBI data export, Kursoppslag-compatible API, and webhook framework. These are the mandatory integration touchpoints for SVV.
**Phase**: MVP (Kursoppslag API stub, webhook framework) / Phase 2 (ServiceNow, PowerBI, full Kursoppslag, eTeori, Mime)
**Complexity**: XL
**Dependencies**: EPIC 1, EPIC 2 (IAM is covered in EPIC 2), EPIC 4 (Learning Management), EPIC 5 (Competency Framework)

### User Stories

#### MVP

**IN-01**: As the SVV Kursoppslag proxy, I want the LMS to expose REST API endpoints matching the existing Kursoppslag contract, so that internal SVV consumers continue to work without changes.
- **Acceptance criteria**:
  - REST API endpoints for: course participants, course info, learning plan participants, learning plan info (6 endpoints per Bilag 1 vedlegg 3)
  - JSON response format compatible with existing field names
  - OAuth2 bearer token authentication
  - API documentation (OpenAPI/Swagger)
  - **Note**: For MVP demo, API must be functional and demonstrable. Full field-level compatibility validated in Phase 2.

**IN-02**: As a developer integrating with the LMS, I want a webhook framework, so that external systems can subscribe to learning events in real time.
- **Acceptance criteria**:
  - Configurable webhook subscriptions per event type (enrollment, completion, certification)
  - HTTP POST with JSON payload to subscriber URL
  - Retry logic with exponential backoff
  - Webhook management UI for admins
  - Event log for debugging

**IN-03**: As a developer, I want comprehensive REST API documentation, so that I can build integrations efficiently.
- **Acceptance criteria**:
  - OpenAPI 3.0 specification
  - Interactive API explorer (Swagger UI or equivalent)
  - Authentication guide
  - Rate limiting documented

#### Phase 2

**IN-04**: As SVV's ServiceNow instance, I want bidirectional sync of competency data (job families, roles, skills, learning log), so that competency management is unified across systems.
- **Acceptance criteria**:
  - REST API integration with ServiceNow
  - Event-driven sync with nightly batch reconciliation (ADR-003)
  - Competency definitions: ServiceNow -> LMS
  - Learning completions: LMS -> ServiceNow
  - Sopra Steria coordination plan documented

**IN-05**: As SVV's PowerBI environment, I want structured data export from the LMS, so that learning and competency analytics are available in SVV's standard reporting tool.
- **Acceptance criteria**:
  - Scheduled data export (daily/weekly) in structured format
  - Data model documentation for PowerBI consumption
  - REST API for on-demand data queries
  - Standard datasets: completions, enrollments, certifications, competency gaps

**IN-06**: As SVV's Kursoppslag API, I want full field-level compatibility with the existing API contract, so that zero changes are needed in downstream consumers.

**IN-07**: As SVV's eTeori exam system, I want bidirectional integration for exam registration and results, so that theory exams are linked to competency certificates (Option L22).

**IN-08**: As SVV's Mime archive system, I want automated certificate archiving via SOAP/REST, so that arkivpliktige documents are archived without manual steps (Option L24).

---

## EPIC 8: AI & Personalization

**Description**: Content recommendations, semantic search (Norwegian language), AI content assistant, and competency gap predictions. Must comply with KI-01 (AI transparency) and Norwegian AI ethics standards.
**Phase**: MVP (content recommendations, Norwegian search) / Phase 2 (AI content assistant, chatbot) / Phase 3 (predictive models, advanced generation)
**Complexity**: L (MVP) / XL (full)
**Dependencies**: EPIC 3 (Content Engine), EPIC 4 (Learning Management), EPIC 5 (Competency Framework)

### User Stories

#### MVP

**AI-01**: As a learner, I want personalized course recommendations on my landing page, so that I discover relevant learning without searching.
- **Acceptance criteria**:
  - Content-based recommendations using: my role, competency gaps, colleagues' completions, course metadata
  - "Recommended for you" section on dashboard (3-6 courses)
  - Recommendation reasons displayed ("Based on your role", "Popular in your division")
  - **Safety override**: Mandatory safety courses always surface first, never suppressed by AI
  - **POC demo (scenario 1.1)**: Learner sees personalized recommendations

**AI-02**: As a learner, I want intelligent search that understands Norwegian language and synonyms, so that I find relevant content even with imprecise queries.
- **Acceptance criteria**:
  - Full-text search with Norwegian stemming and lemmatization
  - Semantic search: "trafikksikkerhet" finds courses tagged "traffic safety" and "vegtrafikkloven"
  - Typo tolerance
  - Search results ranked by relevance (combining text match + metadata + popularity)
  - **POC demo (scenario 1.1)**: Search demonstrates Norwegian language understanding

#### Phase 2

**AI-03**: As a content manager, I want an AI assistant that helps me create course outlines, quiz questions, and learning objectives, so that content creation is faster.
- **Acceptance criteria**:
  - Generate course outline from topic description
  - Generate quiz questions from learning content (text/PDF input)
  - Suggest learning objectives (Bloom's taxonomy)
  - EU-hosted LLM (Azure OpenAI EU or equivalent)
  - Human review/approval before publishing
  - KI-01 compliance: technology, training data, and GDPR basis documented

**AI-04**: As a learner, I want an AI chatbot that answers questions about available courses and platform features, so that I get help without waiting for support.

#### Phase 3

**AI-05**: As a leader, I want predictive competency gap analysis, so that I can anticipate training needs before certifications expire or roles change.

**AI-06**: As an administrator, I want AI-generated training ROI analysis, so that learning investments can be evaluated against competency outcomes.

**AI-07**: As a content manager, I want AI to auto-generate full e-learning modules from source documents, so that content production scales dramatically.

---

## EPIC 9: Admin & Reporting

**Description**: Admin dashboard, reports (completion, compliance, engagement), data export, system configuration, and branding/theming per tenant. The admin experience for division administrators and system admins.
**Phase**: MVP (dashboard, basic reports, branding) / Phase 2 (advanced analytics, custom reports, PowerBI integration)
**Complexity**: L
**Dependencies**: EPIC 1 (Platform Foundation), EPIC 4 (Learning Management), EPIC 5 (Competency Framework)

### User Stories

#### MVP

**AR-01**: As a division administrator, I want a dashboard showing key learning metrics for my division, so that I can monitor training compliance at a glance.
- **Acceptance criteria**:
  - Metrics: total enrollments, completions, overdue, completion rate, active learners
  - Filter by: time period, org unit, course, mandatory/optional
  - Visual charts (bar, line, pie)
  - Comparison across org units within my division
  - **POC demo (scenario 5.1)**: Division admin views dashboard

**AR-02**: As a division administrator, I want to generate and export reports, so that I can share learning data with stakeholders.
- **Acceptance criteria**:
  - Pre-built reports: completion report, compliance report (mandatory course status), enrollment report, certification status report
  - Filters: date range, org unit, course, user group
  - Export to CSV and PDF
  - **POC demo (scenario 5.1)**: Generate and export a report

**AR-03**: As a system administrator, I want to configure platform branding (logo, colors, terminology) per tenant, so that each customer organization has a branded experience.
- **Acceptance criteria**:
  - Configurable: logo, primary/secondary colors, favicon, platform name
  - Preview before applying
  - Separate branding for external portal

**AR-04**: As a system administrator, I want to manage system-wide settings (notifications, defaults, feature toggles), so that the platform behavior can be tailored per customer.
- **Acceptance criteria**:
  - Settings UI for: default language, notification templates, enrollment defaults, session timeout
  - Feature toggles for optional capabilities (AI recommendations, self-enrollment, etc.)

**AR-05**: As a division administrator, I want to see data visualizations of learning engagement trends, so that I can identify patterns and act on declining engagement.
- **Acceptance criteria**:
  - Time-series charts: completions over time, active learners over time
  - Engagement metrics: average time to complete, completion rate by course type
  - Exportable charts

#### Phase 2

**AR-06**: As a division administrator, I want custom report builder capability, so that I can create reports tailored to my division's specific needs.

**AR-07**: As a system administrator, I want PowerBI-ready data exports on a scheduled basis, so that SVV's central analytics team can build custom dashboards.

**AR-08**: As a system administrator, I want a system health dashboard showing API usage, error rates, and performance, so that I can proactively manage platform health.

---

## EPIC 10: Compliance & Security

**Description**: GDPR tools, archiving compliance, WCAG 2.1 AA, encryption, DDoS protection, and penetration testing framework. Non-negotiable for Norwegian public sector.
**Phase**: MVP (WCAG, encryption, GDPR basics) / Phase 2 (full GDPR tools, Noark-5, pentest framework)
**Complexity**: L
**Dependencies**: All other epics (cross-cutting)

### User Stories

#### MVP

**CS-01**: As a user with disabilities, I want the platform to meet WCAG 2.1 AA standards, so that I can use it effectively regardless of my abilities.
- **Acceptance criteria**:
  - All pages pass WCAG 2.1 AA automated checks (axe/Lighthouse)
  - Keyboard navigation for all interactive elements
  - Screen reader compatibility tested (NVDA/VoiceOver)
  - Color contrast ratios meet AA thresholds
  - Alt text on all images, ARIA labels on interactive elements
  - Responsive design: 320px to 2560px width
  - Norwegian language in all UI elements

**CS-02**: As a system administrator, I want all data encrypted at rest and in transit, so that we meet POD-02/POD-10 requirements.
- **Acceptance criteria**:
  - TLS 1.3 for all connections
  - AES-256 encryption for data at rest
  - Database-level encryption
  - Encryption key management (HSM or cloud KMS)

**CS-03**: As a data protection officer, I want basic GDPR controls, so that we can respond to data subject requests.
- **Acceptance criteria**:
  - User data export (downloadable by admin)
  - User data deletion (anonymization preserving aggregate statistics)
  - Consent tracking for optional processing (AI recommendations, analytics)
  - Data processing log accessible to admin

**CS-04**: As a platform operator, I want DDoS protection, so that the platform remains available during attacks (NET-13).
- **Acceptance criteria**:
  - CDN/WAF with DDoS mitigation (Cloudflare, AWS Shield, or equivalent)
  - Rate limiting on API endpoints
  - Monitoring alerts for unusual traffic patterns

**CS-05**: As a developer, I want the platform built with OWASP Top 10 protections, so that common vulnerabilities are prevented.
- **Acceptance criteria**:
  - Input validation and output encoding (XSS prevention)
  - Parameterized queries (SQL injection prevention)
  - CSRF protection
  - Security headers (CSP, HSTS, X-Frame-Options)
  - Dependency vulnerability scanning in CI/CD

#### Phase 2

**CS-06**: As a data protection officer, I want comprehensive GDPR tools including automated data retention and right-to-erasure workflows, so that compliance is systematic and auditable.

**CS-07**: As an archivist, I want Noark-5 compatible export for arkivpliktige documents, so that competency certificates and other records meet Norwegian archival law requirements.

**CS-08**: As a security officer, I want annual penetration testing with results shared with SVV, so that vulnerabilities are identified and remediated (RO-05).

**CS-09**: As a security officer, I want a continuous vulnerability disclosure program, so that SVV is informed of security findings in near real time (RO-03).

---

## EPIC 11: Infrastructure & DevOps

**Description**: Cloud infrastructure (EU data centers), CI/CD pipeline, monitoring/alerting, backup/DR, and environment management. The operational backbone.
**Phase**: MVP (cloud setup, CI/CD, basic monitoring, dev/staging/prod) / Phase 2 (advanced monitoring, DR testing, auto-scaling)
**Complexity**: L
**Dependencies**: None -- runs in parallel with EPIC 1.

### User Stories

#### MVP

**ID-01**: As a platform operator, I want the platform deployed on EU/EEA cloud infrastructure, so that data sovereignty requirements are met from day one.
- **Acceptance criteria**:
  - All compute, storage, and networking in EU/EEA data centers
  - Cloud provider: AWS (eu-north-1 Stockholm) or Azure (Norway East) or equivalent
  - Infrastructure as Code (Terraform/Pulumi)
  - No data leaves EU/EEA for any purpose

**ID-02**: As a developer, I want automated CI/CD pipelines, so that code changes are tested and deployed reliably.
- **Acceptance criteria**:
  - Automated build on every commit
  - Automated test suite execution (unit + integration)
  - Automated deployment to staging on merge to main
  - Manual promotion gate for production deployment
  - Rollback capability (deploy previous version)

**ID-03**: As a platform operator, I want monitoring and alerting, so that issues are detected before users notice.
- **Acceptance criteria**:
  - Application performance monitoring (response times, error rates)
  - Infrastructure monitoring (CPU, memory, disk, network)
  - Alert channels: email, Slack/Teams webhook
  - Uptime monitoring with SLA tracking
  - Log aggregation and search

**ID-04**: As a platform operator, I want separate environments (dev, staging, prod), so that development and testing don't affect production.
- **Acceptance criteria**:
  - Three environments with identical architecture
  - Environment-specific configuration (secrets, URLs, feature flags)
  - Data isolation between environments
  - Staging mirrors production (same cloud region, similar scale)

**ID-05**: As a platform operator, I want automated backups with tested recovery, so that data loss is prevented (POD-07/08/09).
- **Acceptance criteria**:
  - Daily automated database backups
  - Backups stored in two separate EU/EEA locations
  - Point-in-time recovery capability
  - Recovery tested quarterly (documented)
  - RPO: 24 hours, RTO: 4 hours

#### Phase 2

**ID-06**: As a platform operator, I want auto-scaling based on load, so that the platform handles seasonal peaks (KKP winter season) without manual intervention.

**ID-07**: As a platform operator, I want a disaster recovery plan with automated failover, so that the platform survives a full data center outage.

**ID-08**: As a platform operator, I want blue-green or canary deployment capability, so that production updates have zero downtime and can be rolled back instantly.

---

## Phase Summary Matrix

| Epic | MVP (7 weeks) | Phase 2 (post-demo) | Phase 3 (2027+) |
|---|---|---|---|
| **1. Platform Foundation** | Tenant setup, RBAC, org hierarchy, notifications, basic audit | Full multi-tenancy, advanced audit, metering | Self-service tenant onboarding |
| **2. Authentication & Identity** | OIDC SSO, SCIM 2.0, ID-porten, SAML | MFA, Maskinporten, BankID | Federated identity marketplace |
| **3. Content Engine** | SCORM 1.2/2004 player, content management, basic quiz | xAPI, cmi5, non-SCORM content | Content marketplace |
| **4. Learning Management** | Catalog, enrollment, progress, learning paths, classroom events, leader views | Virtual classroom integration, advanced waitlists, assignments | Social learning, forums, gamification |
| **5. Competency Framework** | Competency model, certifications, basic gap analysis, PDF certificates | ServiceNow sync, advanced predictions, Noark-5 archiving | Skills marketplace, industry benchmarking |
| **6. External Portal** | Self-registration, external catalog, certificate access | KKP migration, branded sub-portals, payment | Multi-portal marketplace |
| **7. Integrations** | Kursoppslag API, webhook framework, API docs | ServiceNow, PowerBI, eTeori, Mime | Integration marketplace, iPaaS connector |
| **8. AI & Personalization** | Content recommendations, Norwegian semantic search | AI content assistant, chatbot | Predictive models, auto-generation |
| **9. Admin & Reporting** | Dashboard, basic reports, branding, export | Custom reports, PowerBI integration, health monitoring | Advanced analytics, benchmarking |
| **10. Compliance & Security** | WCAG 2.1 AA, encryption, GDPR basics, DDoS, OWASP | Full GDPR tools, Noark-5, pentest, vulnerability disclosure | ISO 27001, SOC 2 |
| **11. Infrastructure & DevOps** | EU cloud, CI/CD, monitoring, environments, backups | Auto-scaling, DR failover, zero-downtime deploys | Multi-region, edge caching |

---

## MVP Epic Dependency Graph

```
EPIC 11 (Infrastructure) ─────────────────────────────────────────┐
                                                                   │
EPIC 1 (Platform Foundation) ──┬──────────────────────────────────┤
                               │                                   │
                               ├── EPIC 2 (Auth & Identity) ──────┤
                               │                                   │
                               ├── EPIC 3 (Content Engine) ───────┤
                               │       │                           │
                               │       └── EPIC 4 (Learning Mgmt) ┤
                               │               │                   │
                               │               └── EPIC 5 (Competency) ──── EPIC 6 (External Portal)
                               │                       │
                               ├── EPIC 7 (Integrations) ─────────┤
                               │                                   │
                               ├── EPIC 9 (Admin & Reporting) ────┤
                               │                                   │
                               └── EPIC 10 (Compliance) ──────────┘
                                       │
                               EPIC 8 (AI) depends on EPICs 3, 4, 5
```

### Recommended Build Order (MVP)

| Sprint | Weeks | Focus | Epics |
|---|---|---|---|
| **Sprint 0** | W1 | Infra setup, project scaffolding, design system | EPIC 11 (ID-01, ID-02, ID-04), EPIC 1 (PF-01 start) |
| **Sprint 1** | W2-W3 | Platform core, auth, SCORM player | EPIC 1 (PF-01 to PF-05), EPIC 2 (AU-01, AU-02), EPIC 3 (CE-01) |
| **Sprint 2** | W3-W4 | Content management, course catalog, enrollment | EPIC 3 (CE-02 to CE-05), EPIC 4 (LM-01 to LM-04) |
| **Sprint 3** | W4-W5 | Learning paths, leader views, competency model | EPIC 4 (LM-05 to LM-10), EPIC 5 (CF-01 to CF-04) |
| **Sprint 4** | W5-W6 | Certifications, external portal, AI, admin | EPIC 5 (CF-05, CF-06), EPIC 6 (EP-01 to EP-04), EPIC 8 (AI-01, AI-02), EPIC 9 (AR-01 to AR-05) |
| **Sprint 5** | W6-W7 | Integrations, compliance, polish, demo prep | EPIC 2 (AU-03, AU-04), EPIC 7 (IN-01 to IN-03), EPIC 10 (CS-01 to CS-05), POC scenario rehearsal |

---

## POC Scenario Coverage Map

Each POC scenario must be fully covered by MVP stories:

| POC Scenario | Stories Covering It |
|---|---|
| **1.1 Internal user**: Landing page, recommendations, search, notifications, learning path | LM-01, AI-01, LM-02, PF-04, LM-06, LM-04 |
| **1.2 Internal user**: (continuation of 1.1 flow) | LM-03, LM-04 |
| **2.1 External user**: Registration, first login, onboarding | AU-03, EP-01, EP-02 |
| **3.1 Leader**: Team learning status, certification expiry | LM-05, CF-04, CF-05 |
| **3.2 Leader**: Certification expiry alerts | CF-05, PF-04 |
| **3.3 Leader**: Course assignment, activity approval | LM-07, LM-09 |
| **4.1 Content manager**: Course creation (SCORM) | CE-01, CE-02, LM-08 |
| **4.2 Content manager**: Learning path setup | LM-06, CE-04 |
| **4.3 Content manager**: Course update/archiving | CE-05 |
| **4.4 Content manager**: Mandatory vs optional config | LM-10 |
| **5.1 Division admin**: Reports, dashboards, data export | AR-01, AR-02, AR-05 |

---

## Story Count Summary

| Epic | MVP Stories | Phase 2 Stories | Phase 3 Stories | Total |
|---|---|---|---|---|
| 1. Platform Foundation | 5 | 3 | -- | 8 |
| 2. Authentication & Identity | 4 | 3 | -- | 7 |
| 3. Content Engine | 5 | 3 | -- | 8 |
| 4. Learning Management | 10 | 3 | -- | 13 |
| 5. Competency Framework | 6 | 3 | -- | 9 |
| 6. External Portal | 4 | 3 | -- | 7 |
| 7. Integrations | 3 | 5 | -- | 8 |
| 8. AI & Personalization | 2 | 2 | 3 | 7 |
| 9. Admin & Reporting | 5 | 3 | -- | 8 |
| 10. Compliance & Security | 5 | 4 | -- | 9 |
| 11. Infrastructure & DevOps | 5 | 3 | -- | 8 |
| **TOTAL** | **54** | **35** | **3** | **92** |

**MVP**: 54 user stories across 11 epics -- aggressive but focused entirely on what SVV evaluators will test.
