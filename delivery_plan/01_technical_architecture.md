# Technical Architecture Analysis
## Statens vegvesen LMS Delivery -- Case 25/223727

---

## 1. Executive Summary

This document provides a comprehensive technical architecture analysis for the Statens vegvesen (SVV) LMS procurement. The solution must function as a cloud-based SaaS standard platform integrated into SVV's existing enterprise landscape through six primary integration touchpoints: IAM (ForgeRock), Kursoppslag (proxy API), ServiceNow, PowerBI, eTeori (option), and Mime (option). The architecture must satisfy strict Norwegian government requirements for data sovereignty (EU/EEA), GDPR compliance, archival obligations (arkivplikt), and universal design (WCAG).

The core architectural challenge is not building an LMS -- SVV explicitly requires a proven SaaS standard solution -- but rather integrating that solution into a tightly governed identity, competency, and reporting ecosystem while preserving existing API contracts and data flows.

---

## 2. Integration Architecture Overview

### 2.1 System Context

The LMS sits at the center of a hub-and-spoke integration topology with six surrounding systems. Based on the architecture diagram (Bilag 1 vedlegg 4) and the detailed component diagram from the bidder conference, the following systems participate:

| System | Category | Direction | Protocol | Priority |
|--------|----------|-----------|----------|----------|
| ForgeRock AM/IDM (IAM) | Identity & Access | Inbound to LMS | SCIM, OAuth2/OIDC, SAML 2.0 | Mandatory (L04) |
| Kursoppslag | Proxy API | LMS exposes data | REST API (JSON) | Mandatory (part of L03) |
| ServiceNow | Competency Register | Bidirectional | REST API | Mandatory (L05) |
| PowerBI | Reporting | Outbound from LMS | REST API / file export | Mandatory (L06) |
| eTeori | Exam System | Bidirectional | REST API (OAuth2) | Option (L22) |
| Mime (via Mime WS) | Archive/Journal | Outbound from LMS | SOAP (write) + REST (read) | Option (L24) |
| Internt brukerregister | User Directory | To IAM (not directly to LMS) | Internal | Indirect |
| Eksterne laeringssystemer | External content (Udemy, Coursera, MS Learn) | Optional content federation | Varies | Future consideration |

### 2.2 Architectural Principles

The bidder conference and tender documents establish these binding architectural principles:

1. **API-first**: All data exchange between systems must follow an API-first approach
2. **Identity from IAM only**: Internal user identity, roles, and organizational affiliation come exclusively from SVV's identity solutions -- never created locally in the LMS
3. **Kursoppslag abstraction**: Internal SVV consumers never communicate directly with the LMS API; they go through the Kursoppslag proxy. The LMS must expose data that maintains the existing Kursoppslag contract
4. **Archive from the start**: Competency certificates are archival-mandatory (arkivpliktige). This must be considered in the architecture from day one, whether through Mime integration or built-in archival capabilities
5. **Standard solution, minimal customization**: SVV wants a proven SaaS product, not a custom build. Configuration over customization

---

## 3. Integration Specifications

### 3.1 IAM Integration (ForgeRock AM/IDM)

**Complexity: HIGH | Deliverable: L04 (fixed price)**

This is the most architecturally significant integration. SVV's IAM platform (ForgeRock) is the single source of truth for all user lifecycle management.

#### 3.1.1 Authentication and Authorization (ForgeRock AM)

The LMS must support the following authentication mechanisms:

| Mechanism | User Type | Use Case |
|-----------|-----------|----------|
| OAuth2/OIDC | Internal employees, consultants | Primary SSO for ~6,400 internal users |
| SAML 2.0 | Federated / legacy systems | Fallback federation protocol |
| SSO | All internal users | Seamless access across SVV systems |
| MFA | All users (risk-based) | Additional authentication factor |
| ID-porten | External users (~1,100 from Kilden + KKP users) | Norwegian national eID login |
| BankID | External users (strong identity) | High-assurance identity verification |
| MinID | External users | Government-issued eID |
| Maskinporten | System-to-system | Machine-to-machine authentication for API calls |

**ADR-001: Authentication Protocol Selection**

- **Status**: Proposed
- **Context**: SVV operates ForgeRock AM supporting both OIDC and SAML 2.0. Modern LMS platforms typically support OIDC natively with better refresh token handling and API authorization.
- **Decision**: Implement OAuth2/OIDC as the primary authentication protocol for all user types. Support SAML 2.0 as a secondary federation path for backward compatibility. Use Maskinporten for all server-to-server API authentication where SVV requires it.
- **Consequences**: OIDC gives us token-based API authorization for free (bearer tokens for Kursoppslag, ServiceNow). SAML 2.0 support adds configuration complexity but is required for SVV's multi-protocol environment.

#### 3.1.2 User Provisioning (ForgeRock IDM via SCIM)

The LMS must receive and process SCIM-based provisioning events. User lifecycle is managed exclusively in SVV's master sources.

**Required SCIM attributes:**
- Name (fornavn, etternavn)
- Email address
- Organizational affiliation (avdeling/enhet)
- Job function / role (stillingsfunksjon)
- Status (active/inactive)
- Identifiers (brukerIdent)

**Organizational hierarchy data:**
- Unit name and hierarchical placement
- Parent unit
- Changes in organizational structure

**Data flow (textual):**

```
[Internt brukerregister] --> [ForgeRock IDM] --SCIM--> [LMS]
                                                        |
                                                  Create/Update/
                                                  Deactivate user
                                                        |
                                                  Map to LMS roles,
                                                  org structure,
                                                  access permissions
```

**Key requirements:**
- SCIM 2.0 compliance (RFC 7643/7644) preferred, or equivalent mechanism by agreement
- Support for organizational unit hierarchy synchronization
- Deprovisioning must remove access without deleting learning history (GDPR retention vs. arkivplikt tension)
- No manual user creation in the LMS for internal users

#### 3.1.3 External User Authentication

External users authenticate via ID-porten (Norwegian national eID infrastructure). The LMS must:
- Support ID-porten integration (likely brokered through ForgeRock AM)
- Allow self-registration for external course participants
- Issue and manage competency certificates (kompetansebevis) for external users
- Handle ~15,000 competency certificates annually (KKP portal scope)

### 3.2 Kursoppslag Integration (Proxy API)

**Complexity: MEDIUM | Deliverable: Part of L03/L04 (fixed price)**

Kursoppslag is SVV's internal proxy API that abstracts the LMS from internal consumers. This is a critical architectural constraint: the LMS must expose data in a way that preserves the existing Kursoppslag API contract.

#### 3.2.1 API Contract (Must Be Maintained)

The following endpoints define the contract that internal SVV systems (including ServiceNow) consume:

**Endpoint 1: `/api/v2/kursoppslag/kursdeltakere` (GET)**
- Query params: `kursId` (int, opt), `endretFor` (datetime, opt), `endretEtter` (datetime, opt)
- Response fields: `brukerKursId`, `kursId`, `kursNavn`, `brukerIdent`, `fornavn`, `etternavn`, `statusId`, `status`, `gyldigFra`, `gyldigTil`, `sistEndret`

**Endpoint 2: `/api/v2/kursoppslag/kursinfo` (GET)**
- Query params: `kursId` (int, opt), `kursNavn` (string, opt)
- Response fields: `kursId`, `kursNavn`, `kursBeskrivelse`, `kursType`, `fagomraader` (array), `maalgrupper` (array), `kursUrl`, `kursBilde`

**Endpoint 3: `/api/laeringsplanoppslag/laeringsplandeltakere` (GET)**
- Query params: `laeringsplanId` (int, opt), `endretFor` (datetime, opt), `endretEtter` (datetime, opt)
- Response fields: `brukerLaeringsplanId`, `laeringsPlanId`, `brukerIdent`, `fornavn`, `etternavn`, `utfortProsent`, `statusId`, `status`, `gyldigFra`, `sistEndret`

**Endpoint 4: `/api/laeringsplanoppslag/laeringsplaninfo` (GET)**
- Query params: `laeringsplanId` (int, opt)
- Response fields: `laeringsplanId`, `laeringsplanNavn`, `antallDeltakere`

**Legacy endpoints (v1):** `/api/kursoppslag/kursdeltakere` and `/api/kursoppslag/kursinfo` with slightly different schemas (v1 includes `startDato`/`sluttDato`; v2 includes `fagomraader`/`maalgrupper`/`kursUrl`/`kursBilde`).

#### 3.2.2 Integration Architecture

```
[Internal SVV systems]
        |
        v
  [Kursoppslag API]  <-- SVV maintains this proxy
        |
        v
  [LMS REST API]  <-- LMS vendor must expose equivalent data
```

**Key requirement**: The LMS API must be able to return all fields listed above so that SVV's Kursoppslag proxy can map them. The bidder must detail:
- API endpoints that expose course participant data with delta queries (endretFor/endretEtter)
- API endpoints for course metadata including subject areas and target groups
- API endpoints for learning plan (laeringsplan) participation and metadata
- Authentication method for API access (OAuth2 bearer tokens via Maskinporten preferred)

**ADR-002: Kursoppslag Data Mapping Strategy**

- **Status**: Proposed
- **Context**: The existing Kursoppslag contract uses Norwegian field names and SVV-specific data structures. The LMS vendor's API will use the vendor's own schema.
- **Decision**: The mapping between the LMS API schema and the Kursoppslag contract is owned by SVV's Kursoppslag team. The LMS vendor's responsibility is to expose all required data points via stable, documented REST APIs. SVV will update the Kursoppslag proxy to map the new LMS API.
- **Consequences**: This avoids requiring the LMS vendor to implement Norwegian-language API endpoints. The LMS exposes standard APIs; SVV maintains the translation layer. Risk: field mapping gaps may emerge during integration testing.

### 3.3 ServiceNow Integration

**Complexity: HIGH | Deliverable: L05 (T&M, estimated 100h)**

ServiceNow is SVV's competency register and contains the master data for job families, job roles, and skills. The integration is bidirectional.

#### 3.3.1 Data Flows

**Inbound to LMS (from ServiceNow):**
- Job families (~60 families)
- Job roles (~300 roles, ~40 cross-divisional)
- Skills/competency markers (ferdigheter)
- Person-to-role assignments
- Person-to-skill assignments

**Outbound from LMS (to ServiceNow):**
- Learning log (leeringslogg) -- completed learning activities
- Course participation and results
- Competency achievements linked to skills

**Data flow:**

```
[ServiceNow]                                    [LMS]
     |                                             |
     |-- Job families, job roles, skills --------> |  (Inbound: reference data)
     |                                             |
     |<-- Learning log, completions, results ----- |  (Outbound: activity data)
     |                                             |
     |-- Person-role/skill assignments ----------> |  (Inbound: for content curation)
     |                                             |
     |<-- Achieved skills per person ------------- |  (Outbound: skill writeback)
```

#### 3.3.2 Job Role Data Structure

Each job role contains:
- **Jobbrollenavn**: Role name (e.g., "Byggeleder elektro")
- **Jobbfamilie**: Family grouping (~60 families, e.g., "Byggeleder" with 13 sub-roles)
- **Hovedoppgaver**: Key tasks description
- **Hovedkompetanse**: Required competencies
- **Kapittel**: Catalog chapter (e.g., "DoV-UTB", "Felles for alle driftsenheter")
- **Additional markers**: Future-relevant for content curation

#### 3.3.3 Content Curation Requirements

Job roles and skills must be usable within the LMS for:
- Tagging learning activities with applicable job roles and skills
- Displaying job role/skill catalogs in the LMS for content tagging
- Marking which skills are achieved upon activity completion
- Writing achieved skills back to ServiceNow

**ADR-003: ServiceNow Sync Pattern**

- **Status**: Proposed
- **Context**: ServiceNow is the master for job roles and skills. The LMS is the master for learning activities and completions. Both systems need each other's data.
- **Decision**: Implement event-driven synchronization where possible: ServiceNow pushes role/skill changes to LMS via webhook or scheduled REST calls; LMS pushes completion events to ServiceNow via REST API. Fall back to scheduled batch sync (e.g., nightly) for full reconciliation. Kursoppslag handles some of this data flow for course participation data; direct integration handles job roles and skills.
- **Consequences**: Event-driven sync reduces latency for learning log updates (important for compliance tracking). Nightly batch reconciliation catches any missed events. Complexity increases due to dual sync pattern, but data freshness improves significantly.

#### 3.3.4 Effort Estimate Context

The 100h estimate (L05) must cover:
- API mapping and field alignment (~20h)
- Inbound sync: job families, roles, skills, assignments (~30h)
- Outbound sync: learning log, completions, skill writeback (~30h)
- Testing and validation (~20h)

**Risk**: 100h is tight for bidirectional sync with ~300 roles and skill writeback. The ferdigheter/skills framework is described as "will probably be mastered in ServiceNow or a specialized application" -- indicating the data model is not yet finalized. This creates scope uncertainty.

### 3.4 PowerBI Integration

**Complexity: MEDIUM | Deliverable: L06 (T&M, estimated 100h)**

SVV has a mature PowerBI environment with a dedicated team, governed data practices, and standardized reporting patterns.

#### 3.4.1 Data Requirements for PowerBI Compatibility

The LMS must provide data that supports a star schema reporting model:

**Dimension tables:**
- User (brukerIdent, name, division, department, job role)
- Course/Activity (kursId, name, type, subject area, target group, duration)
- Organization (division, section, unit hierarchy)
- Competency area (fagomrade, skill categories)

**Fact tables:**
- Course completions (brukerKursId, timestamps for registration/start/completion, score, status, time spent)
- Learning plan progress (laeringsplanId, utfortProsent, status)
- Competency achievements (skill, date achieved, expiry date)

**Required data characteristics:**
1. Structured formats: CSV, Excel (.xlsx), or REST API with JSON responses
2. Stable unique IDs for users, courses, completions (primary/foreign keys)
3. Consistent date format (ISO 8601)
4. Historical data preservation (not just latest status -- full event history with timestamps)
5. Separation of dimension and fact data
6. Standardized metadata (course category, learning objectives, level, duration)

#### 3.4.2 Data Access Method

Two options, both should be supported:

**Option A: REST API with authentication (preferred for dynamic reporting)**
- OAuth2 bearer token authentication
- Paginated endpoints for large datasets
- Delta/incremental query support (changed since timestamp)
- Rate limiting appropriate for batch data extraction

**Option B: Scheduled file export (fallback)**
- Automated export to secure storage (Azure Blob / SFTP)
- Scheduled intervals (daily/weekly)
- Structured CSV or JSON format

#### 3.4.3 Effort Estimate Context

The 100h estimate (L06) must cover:
- Data model mapping to star schema (~15h)
- API endpoint configuration or export pipeline setup (~30h)
- Authentication and security setup (~15h)
- Testing with SVV's PowerBI team, sample reports (~25h)
- Documentation (~15h)

### 3.5 eTeori Integration (Option)

**Complexity: LOW-MEDIUM | Deliverable: L22 (option, estimated 70h)**

eTeori is SVV's theoretical exam system. The integration enables automated exam registration and result retrieval.

#### 3.5.1 API Specification

Two REST API endpoints with JSON contracts:

**Exam Registration (outbound from LMS):**
- LMS sends candidate registration to eTeori
- Includes candidate identification and exam details
- Response: confirmation or error

**Result Retrieval (inbound to LMS):**
- LMS queries eTeori for exam results
- Response: pass/fail results and scores

**Authentication**: Currently OAuth2. SVV indicates Maskinporten may be introduced later.

**Data flow:**

```
[LMS] --REST/JSON--> [eTeori]  : Register candidate for exam
[LMS] <--REST/JSON-- [eTeori]  : Retrieve exam results
```

#### 3.5.2 Dependency

L22 (eTeori) is a prerequisite for L21 (KKP portal migration), as arbeidsvarsling exams are conducted via eTeori.

### 3.6 Mime Archive Integration (Option)

**Complexity: HIGH | Deliverable: L24 (option, estimated 150h)**

Mime is SVV's archive and journal system (based on Public 360), compliant with the Noark 5 standard. Competency certificates are archival-mandatory documents under Arkivforskriften.

#### 3.6.1 Architecture

```
[LMS] --VPN--> [Mime WS Sak] ---> [Mime 360 / Public 360]
                (Integration layer)     (Archive system)
```

**Mime WS Sak** provides:
- **SOAP services**: All write operations (create/modify case folders, journal entries)
- **REST services**: Read operations (retrieve files, metadata, master data)

#### 3.6.2 Typical Archival Flow

1. Create saksmappe (case folder) in Mime
2. Create journalpost (journal entry) linked to the case folder
3. Attach competency certificate document
4. Finalize and expedite the journal entry
5. Optional: close case folder when complete

#### 3.6.3 Requirements

- VPN between LMS cloud environment and SVV's Mime services
- SOAP client implementation for write operations
- Noark 5 compliant metadata (classification, access control, screening)
- Development in dedicated Mime development environment, then promotion to acceptance and production

**Alternative path**: Arkivforskriften allows built-in archival in the solution itself if it provides secure, audit-safe storage. If the LMS vendor can demonstrate compliant built-in archival capabilities, this may be agreed as an alternative to Mime integration.

#### 3.6.4 Effort Estimate Context

150h is reasonable for Mime integration given:
- SOAP client implementation (~30h)
- Noark 5 metadata mapping (~20h)
- Case folder and journal entry workflow (~40h)
- VPN setup and security (~15h)
- Testing across dev/acceptance/production environments (~30h)
- Documentation (~15h)

---

## 4. Protocol and Standards Mapping

### 4.1 Identity and Access Protocols

| Protocol | Usage | Direction |
|----------|-------|-----------|
| SCIM 2.0 (RFC 7643/7644) | User provisioning from ForgeRock IDM | Inbound to LMS |
| OAuth 2.0 / OIDC | SSO for internal users, API authorization | Bidirectional |
| SAML 2.0 | Federated authentication (secondary) | Inbound to LMS |
| Maskinporten | Machine-to-machine authentication for government APIs | Outbound from LMS |

### 4.2 Learning Standards

| Standard | Usage | Notes |
|----------|-------|-------|
| SCORM 1.2 / 2004 | E-learning course packaging and tracking | 297 existing courses likely in SCORM format |
| xAPI (Tin Can) | Activity tracking for workplace-based learning | Needed for befaringer, hospitering, simulatortrening |
| cmi5 | Modern SCORM successor | Should be supported for future content |
| IMS LTI | External tool integration | For connecting external learning systems |

### 4.3 Integration Protocols

| Protocol | System | Operations |
|----------|--------|------------|
| REST/JSON | Kursoppslag, ServiceNow, PowerBI, eTeori | All CRUD operations |
| SOAP/XML | Mime WS Sak | Write operations (case/journal creation) |
| REST | Mime WS Sak | Read operations (file/metadata retrieval) |

### 4.4 Archival and Compliance Standards

| Standard | Application |
|----------|-------------|
| Noark 5 | Archive metadata structure for Mime integration |
| Arkivforskriften | Legal basis for archival-mandatory competency certificates |
| WCAG 2.1 AA | Universal design requirement for all user interfaces |
| ISO 9241-11:2018 | Usability evaluation framework (efficacy, efficiency, satisfaction) |

---

## 5. Security Architecture

### 5.1 Data Protection

| Requirement | Implementation |
|-------------|----------------|
| Encryption at rest | AES-256 or equivalent for all stored data |
| Encryption in transit | TLS 1.2+ for all communications |
| End-to-end data flow security | No unencrypted data in any transit path (APP-08) |
| Mobile encryption | Encrypted storage on mobile devices (APP-12, APP-13) |
| EU/EEA data storage | All data processing and storage within EU/EEA borders (POD-02) |
| Backup to 2 locations | Geographically separated backup sites within EU/EEA (POD-10) |
| Tested recovery | Regular recovery testing with documented results |
| DDoS protection | Network-level DDoS mitigation (NET-13) |

### 5.2 Access Control Model

```
[ForgeRock AM] --> [SSO/OIDC Token] --> [LMS]
                                          |
                                    Role mapping:
                                    - Internal employee
                                    - Consultant/contractor
                                    - Leader (personalansvar)
                                    - Administrator
                                    - Content manager
                                    - Division administrator
                                    - External course participant
                                    - External course instructor (KKP)
```

Role-based access control (RBAC) must be mapped from SVV's organizational model:
- Roles derived from ForgeRock IDM provisioning
- Organizational hierarchy determines content visibility
- Division-level admin permissions for decentralized management
- Leader roles see their team's learning status and certification expiry

### 5.3 Audit and Logging

- All activity must be logged for traceability (LOG-04)
- Audit logs must include: who, what, when, from where
- Log integration with SVV's monitoring infrastructure
- Immutable audit trail for competency-related actions (arkivplikt connection)
- GDPR-compliant retention periods for log data

### 5.4 GDPR and Privacy

- Data Processing Agreement (DPA) required, preferably based on SSA template (Bilag 5)
- Sub-processor register with advance notification of changes
- Data portability: structured, machine-readable export on contract termination
- Right to erasure balanced against arkivplikt retention requirements
- Privacy by design: minimal data collection principle (RO-05)
- Transfer documentation for any data movement (even within EU/EEA)

### 5.5 Security Assessments

- ROS (risk and vulnerability assessment) required (RO-01)
- Vulnerability disclosure process (RO-02)
- Penetration testing (RO-03)
- Security review by SVV's IT security and privacy counsel

---

## 6. Infrastructure Recommendations

### 6.1 Cloud Hosting

**Requirement**: Cloud-based SaaS solution with EU/EEA data residency.

**Recommended approach**:
- Primary hosting in EU/EEA data center region (e.g., Azure North Europe / West Europe, AWS eu-north-1 Stockholm, or equivalent)
- Backup to second EU/EEA location (geographically separated)
- CDN with EU/EEA edge nodes for static content delivery
- The LMS vendor operates the SaaS platform; SVV does not manage infrastructure

### 6.2 Network Architecture

```
[SVV Internal Network]
        |
        |-- ForgeRock AM/IDM (SCIM/OIDC/SAML)
        |-- Kursoppslag proxy (REST over HTTPS)
        |-- ServiceNow (REST over HTTPS)
        |
        v
   [Internet / VPN]
        |
        v
[LMS SaaS Platform - EU/EEA]
        |
        |-- PowerBI data access (REST/file export)
        |-- eTeori (REST over HTTPS)
        |-- Mime WS (SOAP/REST over VPN)
```

**VPN requirement**: Mime integration specifically requires VPN between SVV and the LMS hosting environment. Other integrations can use HTTPS with mutual TLS or Maskinporten tokens over public internet.

### 6.3 Environment Strategy

| Environment | Purpose | SVV Integration |
|-------------|---------|-----------------|
| Development | Vendor development and initial integration | Connected to SVV test IAM, Mime dev |
| Test/Acceptance | SVV acceptance testing | Connected to SVV test environments for all integrations |
| Production | Live service | Connected to SVV production systems |

Test environment must be available from 07.05.2026 for POC demonstrations (11-12.05.2026).

---

## 7. Migration Strategy

### 7.1 Scope: Kilden Migration (L08 -- 150h T&M)

**Source system**: Kilden (by Storyboard AS), contract expiring 2027.

**Migration inventory:**
- ~297 active e-learning courses
- ~18,500 annual course completions (historical data scope TBD)
- ~318 classroom courses/events per year
- User accounts and learning history
- Course metadata, categories, and structures
- Content packages (likely SCORM format from Articulate 360)

### 7.2 Migration Approach

**Phase 1: Discovery and Mapping (est. 30h)**
- Catalog all Kilden content with format, size, and dependencies
- Map Kilden data model to new LMS data model
- Identify content requiring format conversion vs. direct import
- Define scope of historical data migration (how many years of completions)
- Establish data quality criteria

**Phase 2: Content Migration (est. 50h)**
- Export SCORM/e-learning packages from Kilden
- Import and validate in new LMS
- Handle content that requires re-packaging (e.g., if Kilden uses proprietary formats)
- Map course metadata (fagomraader, maalgrupper, kursType)
- Preserve course IDs or establish ID mapping for Kursoppslag continuity

**Phase 3: Data Migration (est. 40h)**
- Migrate user learning history (completions, scores, dates)
- Migrate learning plan structures and enrollment data
- Migrate certificate/kompetansebevis records
- Validate data integrity: cross-reference migrated records with Kilden source

**Phase 4: Validation and Cutover (est. 30h)**
- Parallel run period where both systems are operational
- Validate all Kursoppslag API responses against new LMS data
- User acceptance testing with sample users from each division
- Cutover plan with rollback procedure

### 7.3 Migration Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Kilden uses proprietary content format | High -- courses may need rebuild | Early format analysis in Phase 1; engage Storyboard AS |
| Historical data volume exceeds estimate | Medium -- more hours needed | Define clear cutoff date for historical data early |
| ID mapping breaks Kursoppslag contract | High -- internal consumers fail | Establish ID mapping table; test all Kursoppslag endpoints |
| Kilden cooperation issues | Medium -- data access delays | Contract with Storyboard AS for migration support |
| Content quality degradation | Medium -- user experience impact | Visual QA of every migrated course |

### 7.4 KKP Portal Migration (Option L21 -- 250h)

Significantly larger scope:
- 350,000 competency certificates
- ~15,000 certificates issued annually
- Seasonal usage patterns (winter operations training)
- Requires completed eTeori integration (L22) as prerequisite
- Own learning surface with separate design and learning architecture
- Migration of external user accounts (ID-porten authenticated)
- Timing constraint: migration cannot happen during peak season

---

## 8. Technical Risk Register

| # | Risk | Probability | Impact | Mitigation | Owner |
|---|------|-------------|--------|------------|-------|
| R1 | SCIM provisioning incompatibility between ForgeRock IDM and LMS vendor's SCIM implementation | Medium | High | Request SCIM compliance details in bid; test early with SVV's ForgeRock test environment | Integration lead |
| R2 | Kursoppslag field mapping gaps -- LMS API cannot expose all required fields | Medium | High | Detailed field-by-field mapping during L01; identify gaps before configuration starts | Solution architect |
| R3 | ServiceNow skills framework not finalized when integration starts | High | Medium | Design integration with extensible skill model; plan for iterative rollout of skill sync | Product owner (SVV) |
| R4 | 100h insufficient for ServiceNow bidirectional integration | Medium | Medium | Prioritize must-have data flows; defer nice-to-have (e.g., skill writeback) to L12 change hours | Project manager |
| R5 | Mime SOAP integration adds significant complexity and delays | Medium | Medium | Evaluate built-in archival as alternative; start Mime integration early if chosen | Integration lead |
| R6 | GDPR/data sovereignty conflict during vendor selection | Low | High | Verify EU/EEA data center locations and sub-processor chains before contract | Legal/privacy |
| R7 | Content migration from Kilden requires more than 150h | Medium | Medium | Scope historical data migration tightly; automate what can be automated | Migration lead |
| R8 | External user scale for KKP (15,000 certs/year) creates performance issues | Low | High | Load testing during acceptance; ensure SaaS can handle seasonal peaks | Performance engineer |
| R9 | LMS vendor's learning standard support gaps (xAPI for workplace activities) | Medium | Medium | Verify xAPI support for non-course activities during POC | Solution architect |
| R10 | Multiple simultaneous integration workstreams create resource bottleneck at SVV | High | High | Stagger integration delivery; ensure SVV assigns dedicated integration contacts per system | Project manager |

---

## 9. Recommended Technology Decisions

### 9.1 LMS Platform Selection Criteria (Technical)

Based on the requirements analysis, the LMS platform must natively support:

1. **SCIM 2.0 provisioning endpoint** -- non-negotiable for ForgeRock integration
2. **OAuth2/OIDC and SAML 2.0** -- dual protocol support for authentication
3. **Comprehensive REST API** -- with delta query support for Kursoppslag integration
4. **SCORM 1.2/2004, xAPI, and cmi5** -- full learning standard coverage
5. **Multi-tenant or sub-domain capability** -- for external learning surfaces (KKP portal)
6. **Built-in reporting API or data warehouse export** -- for PowerBI integration
7. **EU/EEA data center hosting** -- with backup to second location
8. **WCAG 2.1 AA compliance** -- Norwegian universal design law
9. **AI capabilities** -- personalized learning recommendations, content generation assistance
10. **Norwegian language support** -- UI and system support

### 9.2 Integration Middleware Consideration

**ADR-004: Direct Integration vs. Integration Middleware**

- **Status**: Proposed
- **Context**: SVV has 6+ integration touchpoints. Some data flows are complex (bidirectional ServiceNow sync). An integration middleware (e.g., MuleSoft, Azure Integration Services) could centralize integration management.
- **Decision**: Recommend direct point-to-point integrations for the initial delivery. The Kursoppslag proxy already serves as an integration layer for course data. Adding middleware adds cost and complexity that is not justified for the initial scope of 4 mandatory integrations.
- **Consequences**: Simpler initial architecture. If the number of integrations grows significantly in future (L21-L24 options, additional external learning systems), middleware should be reconsidered. Each direct integration must be well-documented and follow consistent patterns (authentication, error handling, retry logic).

### 9.3 Data Synchronization Strategy

| Data Type | Master System | Sync Pattern | Frequency |
|-----------|---------------|--------------|-----------|
| Users, org structure | ForgeRock IDM | Push (SCIM) | Real-time on change |
| Job families, job roles | ServiceNow | Scheduled pull or push | Daily or on change |
| Skills/ferdigheter | ServiceNow (or specialized app) | Scheduled pull or push | Daily or on change |
| Learning completions | LMS | Event-driven push | Near real-time |
| Skill achievements | LMS | Event-driven push | Near real-time |
| Course/activity catalog | LMS | Pull (via Kursoppslag) | On demand |
| Reporting data | LMS | Scheduled export or API pull | Daily |
| Exam registrations | LMS | Event-driven push to eTeori | On action |
| Exam results | eTeori | Pull from eTeori | On demand or scheduled |
| Archival documents | LMS | Event-driven push to Mime | On certificate issuance |

---

## 10. Effort Estimates Per Integration

Referencing the pricing schema hours from the tender:

| Deliverable | Integration | Estimated Hours | Pricing Model | Effort Breakdown |
|-------------|-------------|-----------------|---------------|------------------|
| L04 | IAM (ForgeRock SSO + SCIM) | Included in fixed price (1A) | Fixed | OIDC/SAML config: 20h, SCIM provisioning: 40h, ID-porten setup: 20h, Testing: 20h |
| L05 | ServiceNow | 100h | T&M (1B) | API mapping: 20h, Inbound sync: 30h, Outbound sync: 30h, Testing: 20h |
| L06 | PowerBI | 100h | T&M (1B) | Data model: 15h, API/export setup: 30h, Auth/security: 15h, Testing: 25h, Docs: 15h |
| L08 | Kilden Migration | 150h | T&M (1B) | Discovery: 30h, Content migration: 50h, Data migration: 40h, Validation: 30h |
| L03 (partial) | Kursoppslag API compatibility | Included in fixed price (1A) | Fixed | API analysis: 15h, Field mapping: 20h, Testing: 15h |
| L22 (option) | eTeori | 70h | Option | API integration: 30h, Workflow: 20h, Testing: 20h |
| L24 (option) | Mime | 150h | Option | SOAP client: 30h, Noark 5 mapping: 20h, Workflow: 40h, VPN/security: 15h, Testing: 30h, Docs: 15h |
| L21 (option) | KKP Portal | 250h | Option | Learning surface: 50h, Content migration: 80h, User migration: 40h, Certificate register: 50h, Testing: 30h |

**Total mandatory T&M (L05+L06+L08)**: 350h
**Total options (L21+L22+L24)**: 470h

---

## 11. Phase-by-Phase Technical Delivery Approach

### Phase 0: Pre-Contract (Now -- September 2026)

- Prepare technical architecture response for qualification (by 07.04.2026)
- Prepare bid documentation for Bid 1 (by 06.05.2026)
- Prepare POC demonstration environment (by 07.05.2026)
- Demonstrate 5 role-based scenarios (11-12.05.2026)
- Negotiate technical details (May-June 2026)
- Final bid (01.07.2026)
- Contract award (21.08.2026), signing (01.09.2026)

### Phase 1: Establishment -- Startup (L01) [Weeks 1-4]

**Technical activities:**
- Verify technical prerequisites (API endpoints, authentication, network connectivity)
- Establish development and test environments
- Connect test environment to SVV's ForgeRock test instance
- Define test plan and acceptance criteria jointly with SVV
- Train SVV project team on the LMS platform

**Integration milestones:**
- ForgeRock test connectivity established
- SCIM provisioning tested with sample users
- Kursoppslag API field mapping documented

### Phase 2: Core Platform (L02, L03, L04) [Weeks 4-12]

**Technical activities:**
- L02: Needs analysis across divisions (workshops for learning surface design)
- L03: Solution configuration per SVV requirements
- L04: IAM integration -- SCIM provisioning, SSO (OIDC/SAML), MFA, ID-porten

**Integration milestones:**
- SCIM provisioning operational with full user base
- SSO working for internal and external users
- Role-based access verified across all user types
- Kursoppslag API compatibility confirmed

### Phase 3: Integrations (L05, L06) [Weeks 8-16]

**Technical activities:**
- L05: ServiceNow bidirectional integration
  - Sprint 1: Inbound -- job roles, families, skills into LMS
  - Sprint 2: Outbound -- learning log to ServiceNow
  - Sprint 3: Content curation tagging and skill writeback
- L06: PowerBI data pipeline
  - Sprint 1: Data model alignment and API setup
  - Sprint 2: Sample reports with SVV PowerBI team
  - Sprint 3: Automated data refresh pipeline

**Integration milestones:**
- ServiceNow role data visible in LMS
- Learning completions flowing to ServiceNow
- PowerBI connected and producing sample reports

### Phase 4: Content and Migration (L07, L08) [Weeks 12-20]

**Technical activities:**
- L07: Learning surfaces configured per approved architecture from L02
- L08: Kilden migration
  - Content export and import
  - Historical data migration
  - Validation and QA of migrated content
  - Kursoppslag regression testing with migrated data

**Integration milestones:**
- All 297 courses migrated and validated
- Historical learning data imported
- Kursoppslag API returns correct data from new LMS

### Phase 5: Training, Testing, Go-Live (L09, L10) [Weeks 18-24]

**Technical activities:**
- L09: Administrator and superuser training
- L10: Acceptance testing per agreed test plan
  - Functional testing of all configured features
  - Integration testing across all touchpoints
  - Performance testing (especially for external user scale)
  - Security testing and penetration test
  - WCAG compliance verification

**Integration milestones:**
- All integrations pass acceptance tests
- End-to-end data flow verified: user provisioning -> course enrollment -> completion -> ServiceNow -> PowerBI
- Cutover plan approved

### Phase 6: Post-Delivery (Ongoing)

**Technical activities:**
- L11: Support and helpdesk operational
- L12: Change requests and enhancements (200h estimated, discounted rate)
- Platform upgrades and security patches
- Integration monitoring and maintenance

**Optional deliverables (can be triggered during contract period):**
- L21: KKP portal migration (depends on L22)
- L22: eTeori integration
- L23: Integrated payment solution
- L24: Mime archive integration

---

## 12. Key Architectural Decisions Summary

| ADR | Decision | Rationale |
|-----|----------|-----------|
| ADR-001 | OIDC as primary auth, SAML 2.0 as secondary | Modern token-based auth, API authorization built-in |
| ADR-002 | LMS exposes standard API; SVV maintains Kursoppslag mapping | Avoids vendor lock-in in proxy layer; SVV controls translation |
| ADR-003 | Event-driven sync with nightly batch reconciliation for ServiceNow | Balances data freshness with reliability |
| ADR-004 | Direct point-to-point integrations, no middleware initially | Justified by scope (4 mandatory integrations); revisit if options expand landscape |

---

## Appendix A: Architecture Diagram Reference

The authoritative integration topology is documented in:
- `Bilag 1 vedlegg 4 - Arkitekturskisse.jpg` -- high-level component diagram
- Bidder conference slide 16 (12.03.2026) -- detailed component diagram with data flow annotations

Both diagrams show the same topology: LMS at center, IAM (ForgeRock) at top providing provisioning and authentication, Kursoppslag and ServiceNow on the left handling competency data, PowerBI below for reporting, eTeori for exam integration, and Mime at bottom for archival.

## Appendix B: Kursoppslag API Contract Reference

Full API contract with field-level specifications is documented in `Bilag 1 vedlegg 3 - LMS Integrasjoner.docx`, sections 2.2.1 through 2.2.6.

## Appendix C: Learning Standards Required

The LMS must support the following learning activity types (from `Bilag 1 vedlegg 5`):
- Classroom courses (physical and digital)
- Webinars (Teams integration)
- E-learning (SCORM from Articulate Rise/Storyline, external providers)
- Quizzes and tests (standalone quiz engine required)
- Assignment submissions (text, video, audio with assessment workflow)
- Learning paths (blended, customizable per division)
- Workplace activities (befaringer, hospitering, simulator, re-authorization)
- Self-registered learning with manager approval
- Coaching/mentoring activities

All activity types must generate tracking data compatible with ServiceNow learning log and PowerBI reporting.
