# 04 - Reusable Components & Open-Source Building Blocks

This document catalogues open-source components, libraries, and frameworks that can be reused when building a Norwegian public sector LMS from scratch. For each component we list the name, URL, license, maturity assessment, integration approach, and effort estimate.

---

## Table of Contents

1. [SCORM / xAPI / cmi5 Runtimes & Players](#1-scorm--xapi--cmi5-runtimes--players)
2. [Learning Record Stores (LRS)](#2-learning-record-stores-lrs)
3. [Interactive Content & Authoring](#3-interactive-content--authoring)
4. [Norwegian Government APIs & SDKs](#4-norwegian-government-apis--sdks)
5. [Norwegian Design System](#5-norwegian-design-system)
6. [LTI Integration](#6-lti-integration)
7. [User Provisioning (SCIM)](#7-user-provisioning-scim)
8. [Quiz & Assessment Engines](#8-quiz--assessment-engines)
9. [Certificate Generation (PDF)](#9-certificate-generation-pdf)
10. [Video Player with Learning Analytics](#10-video-player-with-learning-analytics)
11. [Notification Infrastructure](#11-notification-infrastructure)
12. [Patterns from Existing Open-Source LMS Platforms](#12-patterns-from-existing-open-source-lms-platforms)
13. [Integration Effort Summary](#13-integration-effort-summary)

---

## 1. SCORM / xAPI / cmi5 Runtimes & Players

### 1.1 scorm-again (Recommended)

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/jcputney/scorm-again |
| **npm** | `scorm-again` (v3.0.1, published March 2026) |
| **License** | MIT |
| **Stars** | 302+ |
| **Maturity** | High -- 938 commits, actively maintained, full test suite |

**What it does:** A modern, fully-tested JavaScript runtime library for SCORM 1.2 and SCORM 2004. Provides the complete SCORM API that content packages expect to find on `window.API` (1.2) or `window.API_1484_11` (2004).

**Key features:**
- Complete SCORM 2004 sequencing and navigation (activity trees, rollup rules)
- Cross-frame communication for sandboxed iframes
- Offline data storage with LMS synchronization (mobile-ready)
- LMS-agnostic -- can run without a backing LMS, logging all function calls
- SCORM-compliant synchronous commits that return actual LMS success/failure
- Tree-shakeable ES modules

**Integration approach for our React LMS:**
1. Install: `npm i scorm-again`
2. Create a SCORM player React component that:
   - Extracts and serves the SCORM package files (ZIP handling on backend)
   - Instantiates `Scorm2004API` or `Scorm12API` with config pointing to our backend commit endpoint
   - Attaches the API instance to `window.API_1484_11` / `window.API`
   - Renders the SCORM content in a sandboxed `<iframe>`
   - Listens for events (`.on()` method) to update React state (progress, score, completion)
3. Backend endpoint receives `cmi` data model commits and persists to database
4. Convert SCORM data to xAPI statements for unified analytics

**Effort estimate:** 2-3 weeks for full SCORM player integration (package extraction, player UI, backend persistence, progress tracking)

### 1.2 pipwerks SCORM API Wrapper

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/pipwerks/scorm-api-wrapper |
| **npm** | `pipwerks-scorm-api-wrapper` |
| **License** | MIT |
| **Maturity** | Mature but less actively maintained (created 2008) |

**What it does:** A lightweight abstraction layer between SCORM content and the SCORM API. Version-agnostic (works with both SCORM 1.2 and 2004).

**When to use:** If building custom e-learning content (not hosting third-party packages). For hosting existing SCORM packages, scorm-again is the better choice because it provides the full runtime, not just a wrapper.

**Effort estimate:** 1 week (simpler than scorm-again but less capable)

### 1.3 cmi5.js (Rustici Software)

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/RusticiSoftware/cmi5.js |
| **npm** | `@rusticisoftware/cmi5` |
| **License** | Apache-2.0 |
| **Maturity** | Moderate -- reference implementation by the company behind SCORM Cloud |

**What it does:** JavaScript implementation of the cmi5 AU (Assignable Unit) runtime. cmi5 is the modern successor to SCORM built on top of xAPI.

**Integration approach:** Use alongside an xAPI LRS. The cmi5 AU runtime handles the launch sequence, sends xAPI statements following the cmi5 profile to the LRS.

**Effort estimate:** 1-2 weeks (requires xAPI LRS to be set up first)

### 1.4 @xapi/cmi5

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/xapijs/cmi5 |
| **npm** | `@xapi/cmi5` |
| **License** | MIT |
| **Maturity** | Moderate -- strongly typed, fully compliant with cmi5 Quartz 1st Edition |

**What it does:** Alternative cmi5 library with strong TypeScript support. Complete implementation of the cmi5 specification.

**Effort estimate:** 1-2 weeks

### 1.5 SCORM-to-xAPI Wrapper (ADL)

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/adlnet/SCORM-to-xAPI-Wrapper |
| **License** | Apache-2.0 |
| **Maturity** | Reference implementation by ADL (Advanced Distributed Learning) |

**What it does:** Wraps the standard SCORM APIWrapper.js with embedded xAPI calls. Translates SCORM data model elements to xAPI statements automatically.

**Integration approach:** Drop-in replacement for existing SCORM API wrappers that adds xAPI statement generation. Useful for sending SCORM data to both the traditional SCORM backend and an xAPI LRS simultaneously.

**Effort estimate:** 1 week

---

## 2. Learning Record Stores (LRS)

### 2.1 SQL LRS (Recommended)

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/yetanalytics/lrsql |
| **Website** | https://www.sqllrs.com |
| **License** | Apache-2.0 |
| **Version** | v0.9.3 (November 2025) |
| **Maturity** | High -- 2,241 commits, backed by Yet Analytics |

**What it does:** A SQL-based xAPI Learning Record Store. Unlike most LRS implementations that use NoSQL, SQL LRS stores xAPI statements in relational databases, making it much easier to integrate with enterprise analytics tools.

**Key features:**
- Multi-database support: PostgreSQL 14-18, SQLite, MariaDB, MySQL
- OpenID Connect authentication support
- Docker deployment with pre-built images
- "Reactions" engine: conditional logic that analyzes incoming activity streams, applies deterministic rules, and generates new xAPI statements representing validated outcomes
- Direct SQL access for BI tools (Power BI, Tableau, Looker)
- TLS/HTTPS configuration
- Free and open source forever

**Why recommended over Learning Locker:**
- SQL-based (fits our PostgreSQL stack; no MongoDB dependency)
- Apache-2.0 license (more permissive than Learning Locker's AGPL)
- Built-in Reactions engine for automated learning analytics
- Direct SQL access for reporting without additional ETL

**Integration approach:**
1. Deploy as a Docker container alongside our application
2. Configure OIDC authentication to use the same identity provider
3. Our LMS backend sends xAPI statements via the xAPI REST API
4. SCORM data is translated to xAPI using the SCORM-to-xAPI wrapper
5. Analytics dashboards query the LRS directly via SQL

**Effort estimate:** 1-2 weeks for deployment and integration

### 2.2 Learning Locker

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/LearningLocker/learninglocker |
| **License** | AGPL-3.0 (open source) / Commercial (enterprise) |
| **Maturity** | High -- world's most installed open-source LRS |

**What it does:** Full-featured LRS with dashboards, data visualization, and xAPI conformance.

**Key features:**
- Built-in dashboard and visualization tools
- Statement forwarding to other LRS instances
- Multiple Docker deployment options
- Large community and documentation

**Considerations:**
- Requires MongoDB (adds infrastructure complexity to our PostgreSQL-based stack)
- AGPL license requires sharing modifications
- Owned by Learning Pool (commercial entity); community edition may lag

**Effort estimate:** 1-2 weeks for deployment

### 2.3 ADL LRS

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/adlnet/ADL_LRS |
| **License** | Apache-2.0 |
| **Maturity** | Moderate -- reference implementation by ADL, updated for xAPI 2.0 |

**What it does:** Reference LRS implementation by the organization that created xAPI. Recently updated to IEEE 9274.1.1 (xAPI 2.0).

**Effort estimate:** 1-2 weeks

---

## 3. Interactive Content & Authoring

### 3.1 H5P + Lumi H5P Node.js Library (Recommended)

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/Lumieducation/H5P-Nodejs-library |
| **Website** | https://docs.lumi.education |
| **License** | GPL-3.0 |
| **Version** | v10.0.5 (March 2026) |
| **Maturity** | High -- active monorepo with 9 packages, TypeScript, well-documented |

**What it does:** Complete H5P server and client implementation for Node.js. H5P is the most widely adopted open-source framework for creating interactive educational content (50+ content types: interactive video, quizzes, drag-and-drop, branching scenarios, etc.).

**Monorepo packages:**
| Package | Purpose |
|---------|---------|
| `@lumieducation/h5p-server` | Core backend -- framework-agnostic |
| `@lumieducation/h5p-express` | Express middleware integration |
| `@lumieducation/h5p-react` | React player and editor components |
| `@lumieducation/h5p-webcomponents` | Native web components |
| `@lumieducation/h5p-mongos3` | MongoDB + S3 storage adapters |
| `@lumieducation/h5p-redis-lock` | Redis-based distributed locking |
| `@lumieducation/h5p-html-exporter` | Export H5P as standalone HTML |
| `@lumieducation/h5p-svg-sanitizer` | XSS protection for SVG uploads |
| `@lumieducation/h5p-clamav-scanner` | Virus scanning for uploads |

**Integration approach:**
1. Install `@lumieducation/h5p-server` + `@lumieducation/h5p-express` for backend
2. Install `@lumieducation/h5p-react` for React player/editor components
3. Configure storage backend (S3 for content, PostgreSQL adapter for metadata)
4. Expose H5P editor to course creators for authoring interactive content
5. Embed H5P player in course viewer for learners
6. H5P natively generates xAPI statements -- route these to our LRS

**Important note:** GPL-3.0 license means our H5P integration code must also be GPL. This can be managed by keeping it in a separate service/microservice with a clean API boundary.

**Effort estimate:** 3-4 weeks (backend integration, React component setup, storage configuration, xAPI routing)

### 3.2 Adapt Learning Framework

| Field | Detail |
|-------|--------|
| **URL** | https://www.adaptlearning.org |
| **GitHub** | https://github.com/adaptlearning |
| **License** | GPL-3.0 |
| **Maturity** | High -- long-standing open-source project with active community |

**What it does:** Open-source framework for creating responsive HTML5 e-learning courses. Includes both a developer framework and a web-based authoring tool for non-technical users.

**Key features:**
- WYSIWYG authoring tool (web-based, no code required)
- Exports to SCORM 2004, xAPI/TinCan, AICC, and cmi5
- Responsive design out of the box
- Plugin architecture for extensibility
- Tracks completion status and assessments

**Integration approach:** Deploy Adapt Authoring Tool alongside our LMS for internal course creation. Export courses as SCORM packages that our SCORM player (scorm-again) can deliver.

**Effort estimate:** 2-3 weeks for deployment and integration with our content pipeline

### 3.3 Open eLearning

| Field | Detail |
|-------|--------|
| **URL** | https://www.openelearning.org |
| **License** | GPL-3.0 |
| **Maturity** | Moderate |

**What it does:** Desktop-based e-learning authoring tool that produces HTML5 content with SCORM support. Less relevant for a web-first approach but could serve as an offline authoring option.

**Effort estimate:** Minimal (standalone tool, no direct integration needed)

---

## 4. Norwegian Government APIs & SDKs

### 4.1 ID-porten (OIDC Authentication)

| Field | Detail |
|-------|--------|
| **Documentation** | https://docs.digdir.no/docs/idporten/oidc/oidc_guide_idporten |
| **Example App** | https://github.com/felleslosninger/idporten-oidc-client-example |
| **Protocol** | OpenID Connect (Authorization Code Flow) |
| **Maturity** | Production -- used by all Norwegian public sector services |

**What it does:** National authentication service for Norwegian citizens. Supports BankID, MinID, Buypass, and Commfides as identity methods. Provides the `pid` (fødselsnummer/d-nummer) of authenticated users.

**Key technical details:**
- Standard OIDC Authorization Code Flow
- Pairwise subject identifiers (different `sub` per client)
- SSO session: 120 min max, 30 min inactivity timeout
- Must implement single logout via `/endsession`
- Scopes: `openid`, `profile`, `idporten:user.log.read`

**Official example app:** BFF pattern with React frontend + Java/Spring Boot backend. However, we can use any OIDC-compliant library.

**Recommended Node.js libraries:**
| Library | npm | Purpose |
|---------|-----|---------|
| `openid-client` | `openid-client` | OIDC client for Node.js (by panva, OIDC certified) |
| `passport` + `passport-openidconnect` | Various | Express middleware for OIDC |

**Integration approach:**
1. Register our application with Digdir's Samarbeidsportalen
2. Use `openid-client` in our Node.js backend to implement the OIDC flow
3. Implement BFF pattern: React frontend -> Node.js backend -> ID-porten
4. Store session with secure cookies, independent of ID-porten session
5. Implement single logout endpoint

**Effort estimate:** 2 weeks (registration, implementation, testing against test environment)

### 4.2 Maskinporten (Machine-to-Machine Auth)

| Field | Detail |
|-------|--------|
| **Documentation** | https://docs.digdir.no/docs/Maskinporten/ |
| **npm packages** | `@vtfk/maskinporten-auth`, NAV's `maskinporten-client` |
| **Protocol** | OAuth 2.0 JWT Bearer assertion (server-to-server) |
| **Maturity** | Production -- standard for Norwegian M2M auth |

**What it does:** Authentication and access control for machine-to-machine communication between organizations. Uses enterprise certificates for authentication.

**Key Node.js packages:**
- `@vtfk/maskinporten-auth` -- Simple token retrieval using PEM certificate
- `navikt/maskinporten-client` -- NAV's implementation with Maskinporten config

**Integration approach:**
1. Obtain enterprise certificate from Commfides or Buypass
2. Register scopes in Samarbeidsportalen
3. Use `@vtfk/maskinporten-auth` to obtain JWT tokens
4. Use tokens to call external government APIs (Altinn, KRR, etc.)

**Effort estimate:** 1 week

### 4.3 Altinn APIs

| Field | Detail |
|-------|--------|
| **Documentation** | https://docs.altinn.studio/api/ |
| **Protocol** | REST (JSON) + SOAP |
| **Maturity** | Production -- Altinn 3 is the latest platform |

**What it does:** Norway's integration hub for communication between citizens, businesses, and public entities. 1,500+ forms and digital services.

**Relevant for LMS:**
- User/organization lookup
- Digital correspondence/notifications
- Authorization and role delegation
- Integration with public registries

**Note:** No official Node.js SDK exists. Use REST API directly with standard HTTP clients (e.g., `axios` or `fetch`). Authenticate via Maskinporten or API keys.

**Effort estimate:** 2-3 weeks depending on which Altinn services we integrate

### 4.4 Kontakt- og reservasjonsregisteret (KRR)

For looking up citizens' contact information (email, phone, digital mailbox preferences). Accessed via Maskinporten tokens.

**Effort estimate:** 1 week

---

## 5. Norwegian Design System

### 5.1 Designsystemet (Recommended)

| Field | Detail |
|-------|--------|
| **URL** | https://designsystemet.no |
| **GitHub** | https://github.com/digdir/designsystemet |
| **npm** | `@digdir/designsystemet-react` |
| **Version** | v1.13.0 (March 2026) |
| **License** | MIT |
| **Maturity** | High -- V1 released, 230+ organizations in community |

**What it does:** Norway's official public sector design system. Provides accessible, consistent UI components for digital public services. Used by Altinn Studio (70 agencies) and many other government services.

**npm packages:**
| Package | Purpose |
|---------|---------|
| `@digdir/designsystemet-react` | React components |
| `@digdir/designsystemet-css` | CSS implementation with default theme |
| `@digdir/designsystemet-types` | TypeScript type definitions |
| `@digdir/designsystemet` | CLI tools |

**Key components include:**
- Form elements: Textfield, Select, Checkbox, Radio, Switch, Search
- Layout: Card, Details (accordion), Dialog, Popover
- Navigation: SkipLink, ToggleGroup
- Feedback: ErrorSummary
- Theming via token-based system at theme.designsystemet.no

**Integration approach:**
1. Install: `npm i @digdir/designsystemet-react @digdir/designsystemet-css`
2. Generate custom theme using Theme Builder (theme.designsystemet.no)
3. Use components as the base UI layer across the entire LMS
4. Extend with custom components where needed (Designsystemet covers basics, not domain-specific components)

**Why this is important:** Using Designsystemet signals to Norwegian public sector buyers that the LMS follows government standards, is accessible (WCAG compliant), and will feel familiar to users of other government services.

**Effort estimate:** 1 week initial setup, then ongoing as we build UI (components are drop-in)

---

## 6. LTI Integration

### 6.1 ltijs (Recommended)

| Field | Detail |
|-------|--------|
| **URL** | https://cvmcosta.me/ltijs/ |
| **npm** | `ltijs` |
| **License** | Apache-2.0 |
| **Maturity** | High -- LTI Advantage Complete Certified by IMS Global |

**What it does:** Turns your Node.js application into a fully functional LTI 1.3 tool provider. Implements the complete LTI Advantage specification including:
- Core LTI 1.3 launch and authentication
- Assignment and Grade Services (AGS)
- Names and Roles Provisioning Services (NRPS)
- Deep Linking
- Dynamic Registration (first library to implement this)

**Key features:**
- Express-based server with preconfigured routes
- Handles all security and validation automatically
- MongoDB for state storage (supports database plugins for alternatives)
- IMS Global certified

**Integration approach:**
- **As LTI Platform (LMS side):** Our LMS acts as the platform that launches external LTI tools. We need to implement the platform side of LTI 1.3. ltijs primarily focuses on the tool provider side, but understanding its patterns helps.
- **As LTI Tool Provider:** If we want our content to be launchable from other LMS platforms, ltijs handles this completely.
- For the platform side, consider building custom implementation using `openid-client` for the OIDC-based security layer.

**Effort estimate:** 2-3 weeks (tool provider: 1 week with ltijs; platform side: 2 weeks custom implementation)

### 6.2 lti-node-library

| Field | Detail |
|-------|--------|
| **npm** | `lti-node-library` |
| **License** | MIT |
| **Maturity** | Moderate |

Simpler alternative with LTI 1.3 Core and limited AGS support. Good for basic integrations.

---

## 7. User Provisioning (SCIM)

### 7.1 SCIMMY + SCIMMY Routers (Recommended)

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/scimmyjs/scimmy |
| **npm** | `scimmy`, `scimmy-routers` |
| **License** | MIT |
| **Version** | v1.3.5 (March 2025) |
| **Maturity** | Good -- full SCIM 2.0 (RFC 7643/7644), tested with Microsoft Entra ID |

**What it does:** Complete SCIM 2.0 server library for Node.js. SCIM (System for Cross-domain Identity Management) enables automated user provisioning from identity providers (Azure AD/Entra ID, Okta, etc.) to our LMS.

**Key features:**
- Full SCIM 2.0 protocol implementation
- Handler-based architecture (ingress/egress/degress)
- `scimmy-routers`: Express middleware that exposes SCIM endpoints
- Tested compatibility with Microsoft Entra ID
- Bundled core resource schemas (Users, Groups)

**Integration approach:**
1. Install `scimmy` + `scimmy-routers`
2. Define handlers that map SCIM operations to our user database
3. Mount the Express router at `/scim/v2`
4. Configure enterprise customers' identity providers to provision users via SCIM

**Why SCIM matters for public sector:**
Norwegian organizations typically use Azure AD/Entra ID. SCIM enables automatic user creation, updates, and deactivation when employees join, change roles, or leave -- critical for compliance and reducing admin overhead.

**Effort estimate:** 2 weeks

### 7.2 SCIM Gateway

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/jelhub/scimgateway |
| **npm** | `scimgateway` |
| **License** | MIT |
| **Maturity** | Good -- TypeScript, multiple backend plugins |

Alternative that acts as a gateway between SCIM clients and various backend systems. More complex but more flexible for multi-system environments.

**Effort estimate:** 2-3 weeks

---

## 8. Quiz & Assessment Engines

### 8.1 SurveyJS Form Library (Recommended)

| Field | Detail |
|-------|--------|
| **URL** | https://surveyjs.io |
| **GitHub** | https://github.com/surveyjs/survey-library |
| **npm** | `survey-react-ui` |
| **License** | MIT (Form Library), Commercial (Creator/Dashboard) |
| **Maturity** | Very High -- mature product, extensive documentation |

**What it does:** JavaScript form builder library that supports dynamic forms, surveys, quizzes, and assessments. Provides 20+ question types with scoring, timing, conditional logic, and validation.

**Key features for assessments:**
- Multi-page forms with progress tracking
- Scoring and grading with customizable logic
- Timer per question or per quiz
- Conditional visibility and branching
- 50+ language localizations (including Norwegian)
- JSON-based schema (easy to store and version)
- React, Angular, Vue support
- Client-side only -- no vendor lock-in for data storage

**Integration approach:**
1. Install `survey-react-ui`
2. Define quiz schemas in JSON (store in our database)
3. Build quiz editor UI for course creators (or use SurveyJS Creator -- commercial license)
4. Render quizzes in course player
5. Capture results and send as xAPI statements to LRS

**Note on licensing:** The Form Library (renderer) is MIT and free. The visual Creator/Designer component requires a commercial license ($899/developer one-time). For MVP, we can build our own simple quiz editor and use the free renderer.

**Effort estimate:** 2-3 weeks (custom quiz editor + renderer integration + xAPI statement generation)

### 8.2 H5P Quiz Content Types

H5P (section 3.1) includes multiple quiz/assessment content types:
- Multiple Choice
- True/False
- Fill in the Blanks
- Drag and Drop
- Interactive Video with embedded questions
- Question Set (combined quizzes)
- Branching Scenario

These are already covered by H5P integration and generate xAPI statements natively.

---

## 9. Certificate Generation (PDF)

### 9.1 pdfme (Recommended)

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/pdfme/pdfme |
| **Website** | https://pdfme.com |
| **npm** | `@pdfme/generator`, `@pdfme/ui` |
| **License** | MIT |
| **Maturity** | High -- actively maintained, TypeScript + React |

**What it does:** Open-source PDF generation library with a WYSIWYG template designer. Templates are JSON-based, making them easy to create, store, and version.

**Key features:**
- **Designer:** Visual drag-and-drop template editor (runs in browser)
- **Generator:** Generates PDFs from template + data (works in browser and Node.js)
- **Form:** Input form based on template (for collecting dynamic data)
- **Viewer:** Preview generated PDFs
- JSON templates -- easy to store in database
- Works with both browser and Node.js

**Integration approach for certificates:**
1. Install `@pdfme/generator` (backend) and `@pdfme/ui` (admin frontend)
2. Create certificate templates using the visual Designer (admin interface)
3. Store templates as JSON in database
4. When a learner completes a course:
   - Backend generates PDF using template + learner data (name, course title, date, score)
   - Store PDF in object storage (S3)
   - Send download link to learner
5. Optional: Add QR code with verification URL

**Effort estimate:** 1-2 weeks

### 9.2 pdf-lib

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/Hopding/pdf-lib |
| **License** | MIT |
| **Maturity** | High -- popular, well-documented |

Lower-level PDF creation/modification library. More control but requires more code to build templates. Better suited as a fallback or for complex PDF manipulation needs.

---

## 10. Video Player with Learning Analytics

### 10.1 Video.js + xAPI Video Profile

| Field | Detail |
|-------|--------|
| **URL** | https://videojs.com + https://github.com/jhaag75/xapi-videojs |
| **License** | Apache-2.0 (Video.js) |
| **Maturity** | Video.js: Very High; xAPI integration: Moderate (reference implementations) |

**What it does:** Video.js is the most popular open-source HTML5 video player. The xAPI Video Profile (by ADL) is a standardized way to track video interactions via xAPI statements.

**Key tracking capabilities (via xAPI Video Profile):**
- Play, pause, seek, complete events
- Watched segments and completion percentage
- Device, OS, browser, screen size metadata
- Time spent watching vs. total duration
- Cross-device resume (via xAPI state)

**Integration approach:**
1. Use Video.js as the video player component in our React app
2. Implement xAPI Video Profile event tracking using the reference implementation
3. Send xAPI statements to our LRS on video events
4. Build analytics dashboards showing video engagement metrics

**Effort estimate:** 2 weeks (player integration + xAPI event tracking + analytics)

### 10.2 H5P Interactive Video

H5P's Interactive Video content type provides video playback with:
- Embedded quiz questions at specific timestamps
- Bookmarks and navigation
- Summary screens
- Full xAPI tracking of all interactions

This is covered under the H5P integration (section 3.1) and requires no additional video player work.

---

## 11. Notification Infrastructure

### 11.1 Novu (Recommended)

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/novuhq/novu |
| **Website** | https://novu.co |
| **License** | MIT (core) |
| **Maturity** | Very High -- well-funded, large community |

**What it does:** Open-source notification infrastructure providing a unified API for multi-channel notifications: Email, SMS, Push, In-App, and Chat.

**Key features:**
- Unified API for all notification channels
- Workflow automation with conditional logic
- Embeddable in-app notification center (React component)
- Template management with variable interpolation
- Subscriber preferences (users control which channels they receive)
- Real-time delivery via WebSocket
- Self-hostable via Docker

**Tech stack:** Node.js + TypeScript, NestJS, MongoDB, Redis + BullMQ, Socket.io

**Integration approach:**
1. Self-host Novu via Docker Compose
2. Define notification workflows (course completion, assignment due, new content available)
3. Embed the React notification center component in our LMS UI
4. Trigger notifications from our backend via Novu's API
5. Let users manage notification preferences

**Consideration:** Novu requires MongoDB and Redis, adding infrastructure complexity. For a simpler MVP, we could build basic email notifications using a library like `nodemailer` and add Novu later for multi-channel support.

**Effort estimate:** 2-3 weeks (self-hosting + workflow definition + React component + backend integration)

---

## 12. Patterns from Existing Open-Source LMS Platforms

These platforms are NOT recommended for forking, but studying their architecture provides valuable patterns.

### 12.1 Moodle

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/moodle/moodle |
| **License** | GPL-3.0 |
| **Language** | PHP |

**Patterns to learn from:**
- **SCORM handling:** Moodle is SCORM 1.2 conformant only. SCORM 2004 support was partially implemented then abandoned. Lesson: full SCORM 2004 support is complex; using scorm-again avoids this pitfall.
- **Activity module architecture:** Pluggable activity types (quiz, SCORM, assignment, forum). Each module has a standard interface for adding/viewing/grading.
- **Gradebook:** Flexible grade calculation with categories, weights, and aggregation methods.
- **Mobile SCORM player:** Retrieves SCO list and default values via API, launches player, handles LMSInitialize/LMSSetValue/LMSCommit sequence.
- **Role-based access:** Context-based permissions (site, course, module levels).

### 12.2 Canvas LMS

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/instructure/canvas-lms |
| **License** | AGPL-3.0 |
| **Stack** | Ruby on Rails + React + PostgreSQL + Redis |

**Patterns to learn from:**
- **Modern frontend:** React-based SPA with Rails API backend -- similar to our planned architecture.
- **LTI implementation:** Canvas is one of the most LTI-compliant platforms; studying their LTI 1.3 platform implementation is valuable.
- **API-first design:** Comprehensive REST API that powers both the web UI and mobile apps.
- **SpeedGrader:** Efficient grading workflow with inline document viewing and annotation.
- **PostgreSQL at scale:** Canvas proves PostgreSQL can handle enterprise LMS workloads.

### 12.3 Open edX

| Field | Detail |
|-------|--------|
| **URL** | https://github.com/openedx |
| **License** | AGPL-3.0 |
| **Stack** | Python/Django + React |

**Patterns to learn from:**
- **xAPI via event-routing-backends:** Plugin that transforms edX tracking events into xAPI statements. Architecture: events -> transformation layer -> LRS. This event-driven pattern is worth adopting.
- **Aspects analytics:** Converts raw tracking JSON to xAPI for storage and analysis. Shows how to build a learning analytics pipeline.
- **Microservice boundaries:** Course catalog, enrollment, grading, and certificates are separate services communicating via events.
- **XBlock architecture:** Pluggable content components (like H5P content types). Each XBlock defines its own view, edit, and grading logic.

---

## 13. Integration Effort Summary

### Recommended Component Stack

| Category | Component | License | Effort | Priority |
|----------|-----------|---------|--------|----------|
| **SCORM Runtime** | scorm-again | MIT | 2-3 weeks | P0 (MVP) |
| **xAPI LRS** | SQL LRS | Apache-2.0 | 1-2 weeks | P0 (MVP) |
| **Design System** | Designsystemet | MIT | 1 week | P0 (MVP) |
| **Authentication** | ID-porten (via openid-client) | N/A | 2 weeks | P0 (MVP) |
| **Interactive Content** | H5P (Lumi Node.js) | GPL-3.0 | 3-4 weeks | P1 |
| **Quiz/Assessment** | SurveyJS Form Library | MIT | 2-3 weeks | P1 |
| **Certificates** | pdfme | MIT | 1-2 weeks | P1 |
| **Video Player** | Video.js + xAPI Profile | Apache-2.0 | 2 weeks | P1 |
| **LTI** | ltijs | Apache-2.0 | 2-3 weeks | P2 |
| **SCIM Provisioning** | SCIMMY | MIT | 2 weeks | P2 |
| **Notifications** | Novu | MIT | 2-3 weeks | P2 |
| **M2M Auth** | Maskinporten | N/A | 1 week | P2 |
| **cmi5 Runtime** | @xapi/cmi5 | MIT | 1-2 weeks | P3 |
| **Content Authoring** | Adapt Learning | GPL-3.0 | 2-3 weeks | P3 |

### Total Estimated Integration Effort

| Priority | Components | Effort Range |
|----------|-----------|--------------|
| **P0 (MVP)** | SCORM, LRS, Design System, Auth | 6-8 weeks |
| **P1 (Post-MVP)** | H5P, Quiz, Certificates, Video | 8-12 weeks |
| **P2 (Growth)** | LTI, SCIM, Notifications, Maskinporten | 7-9 weeks |
| **P3 (Future)** | cmi5, Authoring Tools | 3-5 weeks |

### License Considerations

| License | Components | Implication |
|---------|-----------|-------------|
| **MIT** | scorm-again, Designsystemet, SCIMMY, SurveyJS, pdfme | No restrictions -- can use freely in proprietary product |
| **Apache-2.0** | SQL LRS, Video.js, ltijs, cmi5.js | Permissive -- must include license notice and state changes |
| **GPL-3.0** | H5P (Lumi), Adapt Learning | Copyleft -- integration code must be GPL. Mitigate by running as separate service with API boundary |
| **AGPL-3.0** | Learning Locker (if used) | Network copyleft -- must share source if accessed over network |

### Key Architecture Decisions Enabled by Component Choices

1. **PostgreSQL as unified database:** SQL LRS uses PostgreSQL, aligning with our main application database. No need for MongoDB.
2. **xAPI as the learning data standard:** All components (SCORM, H5P, Video.js, quizzes) can generate xAPI statements routed to a single LRS for unified analytics.
3. **Microservice boundary for GPL components:** H5P runs as a separate service behind an API, keeping the main application under a permissive license.
4. **Standards-first approach:** SCORM, xAPI, cmi5, LTI, SCIM -- using open standards ensures interoperability with existing government systems and third-party tools.
5. **Norwegian government alignment:** Designsystemet + ID-porten + Maskinporten ensures the platform meets government expectations for accessibility, authentication, and integration.
