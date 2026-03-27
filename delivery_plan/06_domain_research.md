# Domain Research: Norwegian Public Sector LMS Deployments

## 1. Norwegian Public Sector IT Procurement Framework

### 1.1 SSA-L (Statens standardavtale - Løsningskontrakt)

SSA-L (Avtale om løpende tjenestekjøp) is the Norwegian government's standard contract template for ongoing service purchases, specifically designed for standardized services delivered over the internet. Updated in 2026, it is the mandatory contract vehicle for SaaS/cloud procurements by Norwegian public sector entities.

**Scope and Application:**
- Cloud-based services (SaaS, PaaS, IaaS)
- On-premise ASP (Application Service Provider) solutions
- Supplementary services including customizations and integrations
- This is the contract type specified by Statens vegvesen for their LMS procurement

**Key Supplier Obligations:**
- Services must comply with customer requirements specified in Bilag 1 (Appendix 1) and the supplier's own service description in Bilag 2
- The supplier bears full responsibility that services meet customer needs as a whole
- Logical separation of customer data from third-party data
- Proportional measures to ensure information security
- Protection against unauthorized access (confidentiality)
- Protection against data changes and loss (integrity)
- Personal data processing per Norwegian law and customer requirements
- Description of third-party service obligations imposed on the customer

**Penalty Provisions (Dagbøter):**
- Daily penalty: 0.15% of agreed compensation for the first six months (excl. VAT) per calendar day of delay
- Maximum cap applies (defined per contract)
- Minimum NOK 1,500/day for documentation obligation breaches
- While daily penalties are running, the customer cannot terminate the agreement
- This creates a "cure period" incentive for suppliers to resolve delays

**Data Protection Requirements:**
- Updated for GDPR Article 82 liability provisions
- Personal data processing must comply with Personopplysningsloven and GDPR
- Data processing agreements (DPA) are integral parts of SSA-L
- EU/EEA data storage is standard requirement

**Contract Templates Available:**
- Bokmål (2026 - current version)
- Nynorsk (2018)
- English (2018, with GDPR updates)

**Best Practices for Bidders:**
- Carefully complete all appendices (Bilag) - they form binding contractual obligations
- Pay particular attention to SLA definitions in Bilag 4 (service levels)
- Ensure data processing agreement alignment with GDPR requirements
- Document all sub-processors and data flows transparently
- Define clear change management procedures for service updates

### 1.2 FOA (Forskrift om offentlige anskaffelser) - Competition with Negotiation

The Statens vegvesen LMS procurement uses "Konkurranse med forhandling" (competitive procedure with negotiation) per FOA section 13-1. This is the Norwegian implementation of EU Directive 2014/24/EU Article 29.

**When It Can Be Used:**
- When the procuring authority's needs cannot be met without adapting available solutions
- When the acquisition involves design or innovative solutions
- When complexity, legal/financial factors, or associated risks necessitate negotiation
- When technical specifications cannot be precisely described by reference to standards
- This procedure is well-suited for complex LMS procurements with significant integration requirements

**Two-Phase Process Structure:**

*Phase 1 - Pre-qualification:*
- Open to all interested suppliers
- Only suppliers meeting qualification requirements proceed
- Procuring authority may set minimum threshold (no fewer than three suppliers)
- If more suppliers qualify than set limit, selection uses objective, non-discriminatory criteria
- For this tender: 3-6 suppliers will be selected (weighted 65% experience, 35% capacity)

*Phase 2 - Negotiation and Tender:*
- Multiple negotiation rounds permitted, progressively reducing participants
- Common deadline for final bids announced before final round
- Final bids cannot be renegotiated

**Negotiation Rules:**
- *May negotiate:* Commercial terms, pricing, technical specifications, all aspects requiring improvement
- *Cannot negotiate:* Award criteria themselves, absolute requirements from tender documents
- Equal treatment: All remaining suppliers receive equivalent opportunities
- Identical meeting formats and simultaneous information access
- Discussion topics may vary by supplier circumstances
- Confidentiality: Supplier-sensitive information protected without explicit consent

**Implications for Our Bid:**
- Expect 2 negotiation rounds (per the timeline: Round 1 on May 26-27, Round 2 on June 16-17)
- Demonstrations on May 11-12 are critical - evaluators will test hands-on
- Final bid deadline is July 1, 2026
- Quality weighs 80% vs. Price 20% - strong incentive to optimize solution quality
- Functional requirements alone are 50% of the total score

### 1.3 DFO Standards and Government IT Context

DFO (Direktoratet for forvaltning og okonomistyring) is the Norwegian government's expert agency for financial management, organization and management, and procurement in the public sector.

**DFO's Role in Government IT:**
- Delivers salary and accounting services to government agencies
- Operates SAP ERP and SAP SuccessFactors for 200,000+ salary recipients (~90% of state employees)
- Provides HR solutions integrated with SAP payroll
- Also delivers shift management through Visma GAT
- The SSA contract templates are maintained by Digitaliseringsdirektoratet (Digdir), though DFO provides procurement guidance

**HR System Landscape in Norwegian Government:**
- DFO's SAP-based HR/payroll is the dominant system for state agencies
- Statens vegvesen likely uses DFO's systems for core HR/payroll
- This means competency data in the new LMS must potentially integrate with or complement DFO's HR systems
- ServiceNow at SVV serves as the competency register - the LMS must sync learning records to ServiceNow

### 1.4 Doffin/TED Procurement Portal

- Doffin is Norway's national notification database for public procurement
- All public tender notices above NOK 100,000 must be published on Doffin
- For above-EEA-threshold procurements (which this is, at 20-35 MNOK), simultaneous publication on TED (Tenders Electronic Daily) is required
- This tender's case number is 25/223727, published February 25, 2026

**Procurement Landscape Changes:**
- A new public procurement framework is expected to come into force in 2025-2026
- New legislation may affect procedures and thresholds during this contract period
- Bidders should monitor regulatory developments

---

## 2. Statens vegvesen - Organization and Digital Strategy

### 2.1 Organizational Profile

- ~5,000 employees across Vegdirektoratet and 6 divisions
- IT Division based in Drammen with ~400 employees
- IT Division covers: strategy, development, architecture, testing, security, infrastructure, operations, support, maintenance, and project management
- IT Director: Lars B. Kalfoss
- IT Division has a strategic unit of ~15 employees managing strategy, finance, portfolio, HR, information, and IT procurement
- Annual budget: 55.3B NOK (total organization)

### 2.2 Digital Strategy: "Digital Forst" (Digital First)

Statens vegvesen's corporate strategy includes a dedicated strategic direction for digital transformation with three core pillars:

**Pillar 1 - Digitalized Value Chain for Roads:**
- Digital workflows across planning, construction, maintenance
- Digital twins (digital tvilling) for road infrastructure
- Standardized terminology and data management
- Automation and process standardization

**Pillar 2 - The Digital Road (Fremtidens Digitale Vei):**
- Intelligent Transport Systems for connected/autonomous vehicles
- Real-time event detection and management
- AI and predictive analytics for decision-making
- Digital communication of traffic rules

**Pillar 3 - Digital Customer Services:**
- Automated approvals, permits, case processing
- Real-time information tools for travelers/businesses
- Proactive, personalized service delivery
- ~5 million annual citizen inquiries to be handled digitally

**Approach:** "Select digital solutions first" - standardized tools, cross-sector collaboration

### 2.3 IT Landscape and Key Systems

**ServiceNow - Central Digitalization Platform:**
- Selected in 2021 after competitive procurement
- "Flyt-prosjektet" (Flow Project) with implementation partner Sopra Steria
- 15-year usage agreement at 15 MNOK/year (225 MNOK total)
- Used for: case management, communication, workflows, process automation
- ServiceNow serves as the **competency register** with job families, job roles, and skills
- Center of Excellence established with Sopra Steria architects and service designers
- LowCode/NoCode framework for rapid service development

**ForgeRock Access Management (AM/IDM):**
- Confirmed in use (ForgeRock AM instance at vegvesen.no/openam/)
- Provides IAM infrastructure for all SVV applications
- Supports SCIM provisioning, OAuth2/OIDC, SAML 2.0
- Integrates with ID-porten and BankID for external users
- The new LMS MUST integrate with ForgeRock for SSO, MFA, and user provisioning

**Kilden - Current LMS:**
- Built and operated by Storyboard AS
- 297 active e-learning courses
- 18,500 annual course completions
- 318 classroom courses/events in 2025
- Contract expires 2027
- Used for internal training including the "Digitalt pa vei" digital competency program
- Pain points: not integrated with other systems, limited analytics, aging platform

**KKP Portal (Kurs- og kompetanseportal):**
- Operated by Apropos/Dossier Solutions
- Contract 2023-2029
- 350,000 total competency certificates issued
- 15,000 new certificates annually
- Primary users: external workers in arbeidsvarsling (work zone safety) and vinterdrift (winter road maintenance)
- Launched new version May 13, 2024
- Competency certificates valid for 5 years
- National register of all valid certificate holders
- Only approved course providers can deliver courses

**Content Tools (Not Integrated):**
- Articulate 360 (Storyline/Rise) for e-learning authoring
- Junglemap for competency mapping
- Vyond for video content creation

### 2.4 Digital Competency Initiative ("Digitalt pa vei")

Launched in 2020, this initiative created a "digital competency ladder" for all ~4,500 employees:

**Competency Levels:**
1. Digital bruker (Digital User) - lowest
2. Digital utforsker (Digital Explorer)
3. Digitalt fornyer (Digital Innovator)
4. Digital paddriver (Digital Champion) - highest

**Implementation:**
- Developed in collaboration with Direktoratet for hoyere utdanning og kompetanse, Norstat, and Task
- Created extensive resource bank: video instructions, nano-learning, animations, conversation tools, webinar recordings
- Self-directed "buffet of learning" approach
- Deployed via Kilden (current LMS)
- Over 200 IT employees received 2 full days for learning, experimentation, and networking

**Relevance to New LMS:** The new platform must support this type of organization-wide competency development program with self-paced, diverse content formats and tracking of competency progression.

---

## 3. Norwegian Compliance Requirements

### 3.1 Personopplysningsloven (Personal Data Act) / GDPR

Norway's Personal Data Act (LOV-2018-06-15-38), effective July 20, 2018, incorporates the EU General Data Protection Regulation (GDPR) into Norwegian law via the EEA Agreement.

**Key Requirements for LMS:**
- **Legal basis for processing:** Must establish lawful basis for processing employee learning data (likely Article 6(1)(b) - employment contract, or (f) - legitimate interest)
- **Data Processing Agreement (DPA):** Mandatory between SVV (controller) and LMS provider (processor)
- **Data minimization:** Only collect data necessary for learning management purposes
- **Purpose limitation:** Learning data cannot be repurposed without additional consent/basis
- **Storage limitation:** Define retention periods for learning records, completion data, assessment results
- **Data subject rights:** Support access, rectification, erasure, portability, objection
- **DPIA (Data Protection Impact Assessment):** Required for large-scale processing of employee data
- **Sub-processor controls:** All sub-processors must be documented and approved
- **Data breach notification:** 72-hour notification requirement to Datatilsynet
- **Cross-border transfers:** EU/EEA storage mandatory per tender requirements; any transfers require additional safeguards

**Datatilsynet (Norwegian DPA):**
- Enforcement powers include fines up to EUR 20M or 4% of global turnover
- Active enforcement in Norway - multiple significant fines issued
- Particular focus on employee monitoring and surveillance

**Specific Tender Requirements (PEO-13 to PEO-29):**
- EU/EEA data storage is an absolute requirement
- Transfer documentation must be provided
- Sub-processor transparency and control mechanisms
- Data return/deletion procedures on contract termination

### 3.2 Arkivloven (Archive Act) - Competency Certificate Archiving

A new Archive Act (Act of 20 June 2025 No. 96 on Documentation and Archives) replaces the 1992 Act and entered into force on January 1, 2026.

**Key Requirements:**
- Public bodies must register and preserve documents created as part of their activities
- Includes documents in computer systems, databases, and interactive solutions
- Archival-mandatory documents must be stored on media and in formats meeting durability and accessibility requirements
- Must ensure authenticity, reliability, integrity, and usability

**Noark-5 Standard:**
- Norway's national standard for electronic records management, mandated by law for all government agencies
- Metadata must be transformable into Noark-XML code
- PDF/A variants required for written text in archival formats
- Noark-5 based systems must be certified by the National Archives

**Implications for LMS - Competency Certificates:**
- Competency certificates (kompetansebevis) for arbeidsvarsling and vinterdrift are archival-mandatory (arkivplikt)
- The LMS must be able to export competency certificates in Noark-5 compatible formats
- Integration with Mime (SVV's archive and journal system) is specified as an option (L24)
- Estimated 150 hours for Mime archive integration
- 350,000 existing certificates must be preserved according to archive standards
- Certificate metadata must be structured for Noark-5 compliance

### 3.3 Likestillings- og diskrimineringsloven sections 17-18 (Universal Design)

**Legal Framework:**
- Equality and Anti-Discrimination Act sections 17 and 18 mandate universal design of ICT
- EU Web Accessibility Directive (WAD) incorporated into Norwegian law
- Enforced by Tilsynet for universell utforming av IKT (Universal Design Supervisory Authority)

**WCAG Requirements for Public Sector:**
- Must follow 48 of 78 success criteria in WCAG 2.1 at levels A and AA
- Applies to websites, mobile applications, and self-service solutions
- From 2025: EU Accessibility Act expands requirements to more services and products

**LMS-Specific Implications:**
- All learning content must be accessible (alt text, captions, keyboard navigation, screen reader support)
- Mobile-responsive design is mandatory
- Assessment and exam interfaces must meet WCAG 2.1 AA
- Content authoring tools should promote accessible content creation
- External-facing portals (KKP equivalent) must fully comply
- ISO 9241-11:2018 (Efficacy, Efficiency, Satisfaction) and Nielsen's 10 heuristics are explicitly referenced in the evaluation criteria

### 3.4 Sikkerhetsloven (Security Act)

Norway's national Security Act (Lov om nasjonal sikkerhet, 2018) applies to public bodies handling classified information or supporting fundamental national functions.

**Applicability Assessment:**
- Statens vegvesen manages critical transport infrastructure (10,600 km roads, 5,600 bridges, 610 tunnels)
- While the LMS itself likely does not process classified information, security requirements cascade
- The National Security Authority (NSM) has cross-sector supervisory responsibility

**Data Classification Levels:**
- TOP SECRET, SECRET, CONFIDENTIAL, RESTRICTED
- LMS data is unlikely to be classified, but competency data for certain roles may have sensitivity implications
- Any data related to emergency preparedness, security personnel certifications, or critical infrastructure operations should be assessed

**Cloud Requirements:**
- Norwegian government cloud strategy emphasizes data sovereignty
- Minister of Digitalisation urged all organizations to prepare cloud exit strategies (March 2025)
- Government regards data centers as critical digital infrastructure
- Most critical digital services should be delivered from data centers in Norway or close allies
- For the LMS: EU/EEA hosting is mandatory; Norwegian hosting may be preferred for competitive advantage

### 3.5 ID-porten Integration Requirements

ID-porten is Norway's national authentication solution for citizen-facing government services, operated by Digitaliseringsdirektoratet (Digdir).

**Technical Architecture:**
- OpenID Connect (OIDC) is the primary and recommended protocol
- Authorization code flow with mandatory PKCE, state, and nonce
- SAML 2.0 support was deprecated; Digdir discontinued SAML on January 1, 2026
- New ID-porten runs on domain idporten.no with issuer `iss=https://idporten.no/`

**Authentication Flow:**
1. User redirected to ID-porten with authentication request
2. User authenticates (via BankID, MinID, etc.)
3. Authorization code returned to service
4. Service exchanges code for ID token at /token endpoint
5. Local session established

**Security Levels (acr_values):**
- `idporten-loa-substantial` - Standard level (MinID)
- `idporten-loa-high` - High assurance (BankID)

**Key Claims in ID Token:**
- `sub` - Pairwise user identifier
- `pid` - Norwegian national identifier (fodselsnummer)
- `acr` - Security level achieved
- `amr` - Authentication method used

**Session Management:**
- Maximum SSO session: 120 minutes from first authentication
- Inactivity timeout: 30 minutes
- Mandatory single logout via /endsession endpoints
- Front-channel logout handling required

**Client Authentication:**
- JWT-based authentication recommended (X.509 Norwegian business certificates, RS256/RS384/RS512)
- Static client secret supported but not recommended for sensitive services

**LMS Integration Implications:**
- External users (~1,100 Kilden users + ~15,000 KKP certificate users) authenticate via ID-porten
- Must support both `substantial` and `high` security levels
- BankID integration through ID-porten provides highest assurance level
- Self-registration flows must integrate with ID-porten authentication
- All redirect URIs and post-logout URIs must be pre-registered

### 3.6 BankID Authentication

**Role in LMS Context:**
- BankID is Norway's most widely used electronic identification method
- Used on thousands of websites/digital services in private and public sectors
- Accessed through ID-porten for government services

**Assurance Levels:**
- BankID with password: eIDAS "High" assurance - suitable for digital signatures, sensitive information
- BankID with biometrics: eIDAS "Substantial" assurance - suitable for most authentication needs

**Integration:**
- OIDC-based integration through ID-porten
- No direct BankID integration needed - ID-porten handles the authentication broker role

### 3.7 Maskinporten (Machine-to-Machine API Authentication)

Maskinporten is Norway's national solution for machine-to-machine authentication between organizations, operated by Digdir.

**Technical Implementation:**
- OAuth2 JWT bearer grant type (urn:ietf:params:oauth:grant-type:jwt-bearer)
- Client authenticates via certificate in JWT (no separate client authentication needed)
- Access tokens must be validated by API resource servers
- 99.9% uptime target

**Registration Requirements:**
- Organization must be registered in Norwegian Entity Registry with organization number
- Or possess EU Trusted List enterprise certificate

**LMS Relevance:**
- Required for API integrations between the LMS and other government systems
- Kursoppslag proxy API may require Maskinporten for government-to-government data exchange
- ServiceNow integration for competency data synchronization
- PowerBI data export automation

---

## 4. Similar Norwegian Government LMS Procurements

### 4.1 KS Laering -> KS Kunnskap (Municipal Sector)

**Background:**
- KS (Kommune- og fylkesorganisasjonen) is the Norwegian Association of Local and Regional Authorities
- 200+ municipalities collaborated on shared learning resources in KS Laering
- Previous technical platform reached end of life

**Procurement Details:**
- Published on Doffin and TED: April 22, 2024
- Contract signed: Late autumn 2024
- **Winner: Task and Valamis** - selected as new shared learning platform
- Rebranded to "KS Kunnskap"

**Implementation:**
- Pilot municipalities: Oslo, Tromso, Midt-Telemark
- Full rollout started early 2025
- Training for superusers and key personnel as first step
- SSA-L contract used (same as SVV tender)

**Relevance:** Demonstrates appetite for modern, cloud-based LMS in Norwegian public sector. Task (Norwegian company) partnered with Valamis (Finnish learning platform) - showing that Nordic partnerships are viable bid structures.

### 4.2 Sikt/Canvas (Higher Education Sector)

**Background:**
- Sikt (formerly UNINETT/Unit) is the IT service provider for Norway's higher education sector
- Conducted new LMS procurement on behalf of all universities and university colleges

**Procurement Details:**
- Procurement conducted 2024-2025
- **Winner: Instructure (Canvas LMS)**
- Contract type: SSA-L
- Agreement signed: March 2025
- Duration: 3 years from April 1, 2025, with automatic 1-year renewals
- Sikt became reseller of Canvas LMS to the Norwegian knowledge sector
- Replaced the 2016 framework agreement

**Scope:** Covered three central services: LMS, digital exam, and plagiarism detection

**Relevance:** Canvas LMS is now the dominant platform in Norwegian higher education. Shows that international platforms (US-based Instructure) can win large Norwegian public sector contracts when properly localized and compliant.

### 4.3 Miljoedirektoratet (Norwegian Environment Agency)

**Procurement Details:**
- Published on Doffin: 2025
- Purpose: LMS for "Miljoedirektoratskolen" - internal training/competency development
- Scope: ~700 permanent employees + 100 part-time + 300 Statsforvalter employees
- **Winner: Storyboard AS (Learning Center)**
- Contract value: NOK 962,500
- Contract signed: May 28, 2025

**Relevance:** Storyboard AS (SVV's current Kilden provider) won this contract. Demonstrates Storyboard's continued presence in the Norwegian public sector LMS market. However, this is a much smaller scope than SVV's procurement.

### 4.4 Norwegian Armed Forces (Forsvaret)

**Platform:** itslearning
- Implemented in 2015
- Used across all defense branches: Army, Navy, Air Force, Home Guard
- Also used by Defence University College
- Selected after testing several major LMS platforms

**Key Features:**
- Specifically configured for military leadership development model
- Structured career progression tracking from conscript to senior officer
- Operates on specially developed infrastructure meeting NSM security requirements
- Layered security: segmentation, end-to-end encryption, MFA with military ID cards, RBAC, continuous monitoring, comprehensive audit logs

**Relevance:** Shows that Norwegian government agencies can and do deploy sophisticated LMS platforms with advanced security. itslearning is a Bergen-based Norwegian company, demonstrating preference for domestic/Nordic solutions in security-sensitive contexts.

### 4.5 Market Summary

| Organization | Platform | Year | Scope |
|---|---|---|---|
| KS (municipalities) | Task/Valamis | 2024 | 200+ municipalities |
| Sikt (universities) | Canvas (Instructure) | 2025 | All UH sector |
| Miljoedirektoratet | Storyboard Learning Center | 2025 | ~1,100 users |
| Forsvaret | itslearning | 2015 | All defense branches |
| SVV (current) | Storyboard Kilden | Expiring 2027 | ~5,300 internal |

**Key Observations:**
- No single platform dominates Norwegian public sector LMS
- Both Nordic (itslearning, Task/Valamis, Storyboard) and international (Canvas/Instructure) platforms compete
- SSA-L is the standard contract vehicle
- Security, GDPR compliance, and Norwegian language support are table-stakes
- Integration capabilities (SSO, SCIM, API) are consistently important requirements
- Recent trend toward cloud-based SaaS solutions over on-premise deployments

---

## 5. The 70/20/10 Learning Model

### 5.1 Model Overview

The 70/20/10 learning model, created by Morgan McCall, Robert Eichinger, and Michael Lombardo at the Center for Creative Leadership, proposes that effective workplace learning comprises:

- **70% Experiential Learning** - Learning through doing, on-the-job experiences, challenges, practice
- **20% Social Learning** - Learning through others via mentoring, coaching, peer interaction, feedback
- **10% Formal Learning** - Structured courses, workshops, e-learning, classroom training

This model is explicitly cited in Statens vegvesen's vision for the new LMS (Key Objective #2).

### 5.2 LMS Features Supporting Each Component

**10% - Formal Learning (Foundation):**
- Course catalog management (SCORM, xAPI, cmi5 content)
- Learning paths and curricula
- Mandatory course assignment and tracking
- Assessments, quizzes, and certification
- Classroom/webinar scheduling and enrollment
- Completion tracking and certification management
- Content authoring tools
- Mobile-responsive course delivery

**20% - Social Learning (Connection):**
- Discussion forums and communities of practice
- Mentoring and coaching program management with interaction tracking
- Peer-to-peer knowledge sharing (articles, videos, tips)
- Expert directories and professional networks
- Social feeds and activity streams
- User-generated content capabilities
- Group workspaces by role, division, or topic
- Rating, commenting, and bookmarking of content
- Integration with collaboration tools (Teams, etc.)

**70% - Experiential Learning (Application):**
- On-the-job activity logging and verification
- Project-based learning assignments
- Performance support tools (job aids, checklists, quick references)
- Simulations and scenario-based training
- Competency-based progression tracking
- Manager verification of workplace activities
- Task and challenge assignments
- Reflection journals and learning logs
- Integration with performance management systems

### 5.3 Implementation Best Practices for Enterprise LMS

**Strategic Alignment:**
- The model is a guideline, not rigid - proportions should adapt to context
- Safety-critical roles (e.g., arbeidsvarsling) may require more formal training (>10%)
- New employee onboarding may weight more heavily toward formal learning initially
- Senior professionals may skew more toward experiential and social

**Manager Enablement:**
- Managers as learning facilitators, not just compliance enforcers
- Dashboard visibility into team learning across all three dimensions
- Tools to assign experiential activities and verify completion
- Coaching and mentoring program support

**Measurement and Analytics:**
- Track formal learning: completion rates, assessment scores, certification status
- Track social learning: participation in forums, mentoring interactions, content sharing
- Track experiential learning: activity logs, competency assessments, project outcomes
- Connect learning metrics to business outcomes (performance KPIs, safety records)

**Technology Integration:**
- LMS as the hub connecting formal, social, and experiential learning
- Integration with collaboration platforms for social learning data
- Integration with HR/performance systems for experiential learning context
- Mobile access for learning in the flow of work
- xAPI (Experience API) for capturing learning across systems and contexts

### 5.4 Application to Statens vegvesen Context

**Formal (10%):**
- Mandatory HMS/safety courses
- Arbeidsvarsling and vinterdrift certification courses
- Regulatory compliance training
- Technical skills training (Articulate content)
- Digital competency program modules

**Social (20%):**
- Cross-divisional professional networks (6 divisions)
- Mentoring programs for career development
- Communities of practice for specialized roles (bridge engineers, traffic planners, etc.)
- Knowledge sharing from the ~400 IT division employees to the broader organization
- "Digital paddriver" (Digital Champion) peer support network

**Experiential (70%):**
- On-the-job training for traffic station work
- Simulator training (specified in tender)
- Field inspections and emergency drills
- Job shadowing and rotation programs
- Re-authorization activities
- Project-based learning during infrastructure projects
- Coaching and mentoring interactions

---

## 6. Statens vegvesen Integration Ecosystem

### 6.1 ForgeRock AM/IDM Integration Architecture

ForgeRock serves as the centralized Identity and Access Management platform for Statens vegvesen.

**Required Integration Points:**
- **SCIM 2.0 Provisioning:** Automated user lifecycle management (create, update, deactivate) from ForgeRock IDM to LMS
- **OAuth2/OIDC:** Authorization for API access between LMS and other SVV systems
- **SAML 2.0:** Legacy SSO support (though OIDC is preferred direction)
- **SSO:** Single sign-on for all internal users (~5,300 employees + ~1,100 consultants)
- **MFA:** Multi-factor authentication enforcement per SVV security policy
- **ID-porten broker:** ForgeRock likely brokers ID-porten authentication for external users
- **BankID:** High-assurance authentication for external certificate holders

**Key Considerations:**
- ForgeRock SCIM connector supports mapping managed users to external systems
- User attributes must sync: roles, division, job family, competency requirements
- Role-based access control (RBAC) mapped from ForgeRock role definitions
- Session management must align with ForgeRock session policies
- Group and organizational unit structures must propagate

### 6.2 ServiceNow Integration Architecture

ServiceNow serves dual purposes at SVV: as a digitalization platform and as the competency register.

**Competency Register Integration:**
- Job families, job roles, and skills defined in ServiceNow
- Learning log synchronization: completed courses/certifications must flow from LMS to ServiceNow
- Competency gap identification: ServiceNow defines required competencies; LMS tracks attainment
- REST APIs for bidirectional data exchange
- Estimated 100 hours for implementation

**Data Flow:**
- ServiceNow -> LMS: Required competencies per role, job family mappings, skill frameworks
- LMS -> ServiceNow: Course completions, certifications earned, competency assessments, learning records

### 6.3 Kursoppslag Proxy API

This is an existing proxy API that abstracts the LMS for internal consumers.

**Requirements:**
- Must maintain existing API contract (critical - other systems depend on this)
- REST API with endpoints for: course participants, course info, learning plan participants/info
- The new LMS must either replicate this API or the proxy must be updated
- Breaking changes to downstream consumers must be avoided

### 6.4 PowerBI Integration

SVV uses PowerBI as its enterprise reporting platform.

**Requirements:**
- Structured data export compatible with PowerBI consumption
- Estimated 100 hours for implementation
- Must support: learning analytics, completion reports, competency gap analysis, compliance status
- Data model must be documented and maintainable

### 6.5 Optional Integrations

**eTeori (Option L22, 70h):**
- Exam registration and results system
- REST API integration
- Prerequisite for KKP portal migration (L21)

**Mime Archive Integration (Option L24, 150h):**
- Archive and journal system for archival-mandatory competency certificates
- Noark-5 compliance requirements
- Must handle 350,000+ existing certificates and ~15,000 new annually

---

## 7. Norwegian Government Cloud and Data Sovereignty Context

### 7.1 Current Policy Direction

- Norway's Minister of Digitalisation (Karianne Tung) urged all government organizations to prepare cloud exit strategies (March 2025)
- Driven by concerns about US cloud provider dependency and potential regulatory conflicts
- Government views data centers as critical digital infrastructure
- Policy direction: Norwegian data on Norwegian soil where possible
- Most critical digital services from Norwegian data centers or close allies

### 7.2 Implications for LMS Procurement

- EU/EEA data storage is an absolute requirement (explicitly stated in tender)
- Norwegian-hosted solutions may receive favorable evaluation
- Cloud exit strategy/data portability should be addressed in the bid
- Vendor lock-in concerns are heightened in current political climate
- Open standards and data export capabilities are competitive advantages
- Multi-cloud or hybrid approaches may score well on resilience

### 7.3 EU Regulatory Context

- EU Data Governance Act, Digital Markets Act, Data Act (2023-2025)
- EU Cloud Sovereignty Framework
- EUCS (EU Cloud Services Scheme) cybersecurity certification
- Norway, as EEA member, aligns with most EU digital regulations
- Schrems II implications continue to affect US-based cloud services

---

## 8. Key Domain Insights and Bid Implications

### 8.1 Competitive Landscape Signals

1. **Storyboard AS** is the incumbent (Kilden) but their platform is being replaced due to age/limitations. They recently won Miljoedirektoratet but at small scale (NOK 962K). They may bid but face "why replace yourself" positioning challenge.

2. **Task/Valamis** just won KS Laering (200+ municipalities) - demonstrates strong Norwegian public sector positioning. Valamis has 70/20/10 capabilities and Nordic compliance expertise.

3. **Canvas/Instructure** won Sikt/UH sector but is more education-focused than corporate LMS. Unlikely competitor for enterprise/government LMS.

4. **itslearning** has deep Norwegian government experience (Forsvaret) but is more education-oriented. Could pivot to corporate market.

5. **International platforms** (Cornerstone, Docebo, Totara, SAP SuccessFactors Learning) have enterprise LMS capabilities but may lack Norwegian market presence and compliance readiness.

### 8.2 Critical Success Factors for This Bid

1. **ForgeRock integration expertise** - This is a specific, complex requirement. Demonstrable SCIM/OIDC integration with ForgeRock is a differentiator.

2. **ServiceNow competency register integration** - Bidirectional REST API integration with ServiceNow's competency model is complex and critical.

3. **70/20/10 model support** - Not just formal courses but social learning, experiential tracking, and communities of practice. This is a stated vision.

4. **External user management at scale** - 15,000 annual competency certificates via ID-porten is a significant scale requirement with high regulatory stakes.

5. **Norwegian language and localization** - Full Norwegian Bokmal (and ideally Nynorsk) support is essential.

6. **WCAG 2.1 AA compliance** - Will be tested during demonstration; must be authentic, not retrofitted.

7. **Migration capability** - 297 courses from Kilden + potentially 350,000 certificates from KKP. Migration risk is real.

8. **Data sovereignty positioning** - EU/EEA hosting mandatory; Norwegian hosting a plus. Cloud exit strategy important given current government direction.

### 8.3 Risk Factors to Address

1. **New Archive Act (2026)** - Requirements may evolve during contract period; competency certificate archiving is a moving regulatory target.

2. **ID-porten SAML deprecation** - SAML fully discontinued January 1, 2026. Any SAML-based integration must use OIDC.

3. **Procurement law changes** - New public procurement framework expected 2025-2026; may affect contract administration.

4. **Cloud sovereignty policy shifts** - Government may tighten cloud policies during the 6-year contract period.

5. **ServiceNow evolution** - SVV's ServiceNow platform is under active development; integration interfaces may change.

6. **KKP portal dependency** - The option to migrate KKP (L21) depends on eTeori integration (L22); this creates a dependency chain that must be planned for.

### 8.4 Terminology Reference

| Norwegian Term | English Translation | Context |
|---|---|---|
| Læringsplattform | Learning Platform | LMS |
| Kompetansebevis | Competency Certificate | Certification document |
| Arbeidsvarsling | Work Zone Safety/Traffic Control | Mandatory training area |
| Vinterdrift | Winter Road Maintenance | Mandatory training area |
| Dagbøter | Daily penalties | Contract penalty mechanism |
| Arkivplikt | Archival obligation | Legal requirement to preserve records |
| Universell utforming | Universal design | Accessibility requirements |
| Personopplysningsloven | Personal Data Act | GDPR implementation |
| Fødselsnummer | National identity number | Used in ID-porten pid claim |
| Virksomhetsstrategi | Corporate strategy | SVV's organizational strategy |
| Kompetansetrapp | Competency ladder | SVV's digital skill progression model |
| Bilag | Appendix/Annex | Contract attachments in SSA-L |
| HMS | Health, Environment, Safety | Norwegian workplace safety framework |

---

*Research compiled: March 22, 2026*
*Sources: Anskaffelser.no, Lovdata.no, Digdir documentation, Doffin.no, vegvesen.no, DFO.no, Sopra Steria press releases, KS Digital, Sikt.no, itslearning.com, Storyboard.no, various Norwegian legal and procurement resources*
