# Gap Analysis: Build Plan vs. Client Requirements
## Cross-reference of BUILD_DELIVERY_PLAN user stories against SVV tender documents

**Date**: 2026-03-22
**Sources checked**:
- Bilag 1 vedlegg 7 - POC oppgaver til demonstrasjon (all scenarios 1.1-6.3)
- Bilag 1 vedlegg 5 - Laeringsaktiviteter (all activity types)
- SSA-L Bilag 1 - Kravspesifikasjon (deliverables, vision, requirements)
- Bilag 1 vedlegg 3 - Integrasjoner (API contracts)

---

## 1. POC SCENARIOS: Coverage Check

### COVERED (in build plan epics/stories)

| POC # | Scenario | Build Plan Coverage | Stories |
|---|---|---|---|
| 1.1 | Internal user: landing page, recommendations, search, notifications, certs | COVERED | LM-01, AI-01, LM-02, PF-04, CF-05 |
| 1.2 | Internal user: learning path progression | COVERED | LM-06, LM-04 |
| 2.1 | External user: registration, first login, onboarding | COVERED | AU-03, EP-01, EP-02 |
| 3.1 | Leader: team learning status, certification expiry | COVERED | LM-05, CF-04, CF-05 |
| 3.2 | Leader: assign courses (individual + bulk) | COVERED | LM-07 |
| 3.3 | Leader: approve/register activity (in-platform + outside platform) | PARTIALLY | LM-09 covers approval, but see GAP #1 |
| 4.1 | Content manager: create e-learning course (SCORM/xAPI/cmi5 upload) | COVERED | CE-01, CE-02 |
| 4.2 | Content manager: create learning path with e-learning + event + test + certification | PARTIALLY | LM-06, LM-08, CE-03. But see GAP #2 |
| 4.3 | Content manager: update and archive course/learning path | COVERED | CE-05 |
| 4.4 | Content manager: same course mandatory for one group, optional for another, different cert duration | PARTIALLY | LM-10. But see GAP #3 |
| 5.1 | Division admin: manage own area, role permissions | PARTIALLY | AR-01. But see GAP #4 |

### MISSING FROM BUILD PLAN (POC scenarios not covered)

| POC # | Scenario | Gap |
|---|---|---|
| **5.2** | Division admin: grant a user reporting access to ONE specific course/learning path in their area, without access to other reports | **GAP #5 - MISSING** |
| **5.3** | Division admin: show available data/analytics, distinguish between target groups | **GAP #6 - WEAK** |
| **6.1** | Agency admin (etatsadministrator): create area/page where selected group has read access and edit access | **GAP #7 - MISSING** |
| **6.2** | Agency admin: configure a custom role with specific permissions and assign to users | **GAP #8 - WEAK** |
| **6.3** | Agency admin: maintain landing pages (change image, add info text, add content) for internal and external users | **GAP #9 - MISSING** |

---

## 2. DETAILED GAP DESCRIPTIONS

### GAP #1: Leader registers activity completed OUTSIDE the platform
**POC 3.3**: "Vis hvordan en leder kan registrere eller godkjenne en laeringsaktivitet som en medarbeider har fullfort, bade nar aktiviteten ligger i laeringsplattformen og nar den er gjennomfort utenfor plattformen."

**Current build plan**: LM-09 covers approval workflow for in-platform activities. But there is NO story for a leader (or learner) to **manually register** a learning activity that happened outside the LMS (e.g., external course, conference, field inspection, hospitering). This is critical for SVV's 70/20/10 model -- 70% of learning happens on the job.

**Required addition**: New user story:
> "As a leader, I want to register a learning activity that a team member completed outside the platform (external course, field observation, mentoring session), so that all competency-relevant activities are documented in one place."

Acceptance criteria:
- Manual activity registration form: activity type, date, description, evidence upload (optional)
- Activity appears in learner's learning history
- Can be marked as "pending leader approval" or "self-registered"
- Links to competency framework (activity maps to competency)

**Phase**: MVP (this is tested in the POC demo)

---

### GAP #2: Learning path with registration form (allergies, accommodation, etc.)
**POC 4.2**: "Vis hvordan et laeringsloep opprettes med e-laeringskurs, arrangement med registreringsskjema (allergier, overnatting, etc) og test."

**Current build plan**: LM-06 covers learning paths and LM-08 covers classroom events. But there is NO story for **registration forms with custom fields** (allergies, dietary requirements, accommodation needs) attached to classroom events within a learning path.

**Required addition**: Extend LM-08 or create new story:
> "As a content manager, I want to add a custom registration form to classroom events (collecting dietary needs, allergies, accommodation, travel preferences), so that event logistics are handled within the platform."

Acceptance criteria:
- Custom form builder for events (text, dropdown, checkbox fields)
- Fields: allergies, dietary needs, accommodation yes/no, accessibility needs
- Registration data visible to event administrator
- Export registration data as CSV

**Phase**: MVP (this is specifically demonstrated in POC 4.2)

---

### GAP #3: Same course with DIFFERENT certification duration per target group
**POC 4.4**: "Vis et konkret eksempel der samme kurs er obligatorisk for en malgruppe og valgfritt for en annen, med ulik varighet pa sertifiseringen."

**Current build plan**: LM-10 covers mandatory vs optional per org unit. CF-05 covers certifications with expiry. But there is NO story for **different certification validity periods** for the same course depending on the target group (e.g., arbeidsvarsling cert valid 2 years for one group, 1 year for another).

**Required addition**: Extend CF-05 or create new story:
> "As a content manager, I want to configure different certification validity periods for the same course depending on the target group, so that recertification schedules match regulatory requirements per role."

Acceptance criteria:
- Certification rules are configurable per target group (org unit, role, user group)
- Same course can generate certifications with different expiry dates
- Example: Course X -> Group A: mandatory, cert valid 2 years. Group B: optional, cert valid 1 year.
- Learner sees correct expiry for their group

**Phase**: MVP (specifically tested in POC 4.4)

---

### GAP #4: Division admin role with scoped management capabilities
**POC 5.1**: "Vis hvordan divisjonsadministrator kan forvalte eget omrade og hvilke rettigheter denne rollen typisk kan ha."

**Current build plan**: AR-01 covers dashboard for division admin. But there is NO story for the **scoped management** aspect -- a division admin should be able to manage content, users, and settings within their division, but NOT across the entire organization.

**Required addition**:
> "As a division administrator, I want to manage courses, users, and settings within my division only, so that I have autonomy over my area without affecting other divisions."

Acceptance criteria:
- Division admin can: create/edit courses for their division, manage learner enrollments within their org unit, view reports scoped to their division
- Division admin CANNOT: access other divisions' data, modify global settings, manage users outside their org unit
- Clear scope boundaries in the UI

**Phase**: MVP (tested in POC 5.1)

---

### GAP #5: Granular report access per course/learning path (MISSING)
**POC 5.2**: "Vis hvordan divisjonsadministrator gir en bruker innsyn i rapportering for ett konkret kurs eller laeringsloep i sitt omrade, uten at brukeren far tilgang til rapporter for ovrige kurs eller andre deler av organisasjonen."

**Current build plan**: NO STORY covers this granular permission. This requires per-object permission delegation -- a division admin can grant a specific user reporting access to ONE course, without giving them access to all reports.

**Required addition**:
> "As a division administrator, I want to grant a specific user reporting access to a single course or learning path, without giving them broader report access, so that data privacy is maintained while enabling collaboration."

Acceptance criteria:
- Permission model: grant "view reports" on specific course/learning path to specific user
- Granted user sees ONLY the report for that specific course
- No access to other courses' data or other divisions
- Division admin can revoke this access

**Phase**: MVP (specifically tested in POC 5.2)

---

### GAP #6: Data and analytics for division admin with target group filtering
**POC 5.3**: "Vis hvilke data og analyser som kan vaere tilgjengelig for Divisjonsadministrator, og hvordan divisjonsadministrator kan skille mellom ulike malgrupper."

**Current build plan**: AR-01 and AR-02 cover basic dashboards and reports. But the explicit ability to **filter analytics by target group** (malgruppe) is not a distinct story.

**Required addition**: Extend AR-01/AR-02:
> "As a division administrator, I want to filter all analytics and reports by target group (malgruppe), so that I can compare training status across different user segments."

Acceptance criteria:
- Target group = definable user segment (by role, org unit, custom group)
- All reports filterable by target group
- Comparison view: side-by-side target groups

**Phase**: MVP (tested in POC 5.3)

---

### GAP #7: Agency admin creates areas/pages with access control (MISSING)
**POC 6.1**: "Vis hvordan etatsadministrator kan opprette et omrade eller fagside der en utvalgt malgruppe har lesetilgang og redigeringstilgang."

**Current build plan**: NO STORY covers content areas / pages (fagsider) with configurable access control. This is about creating **learning spaces or knowledge areas** within the LMS where selected groups have read or edit access -- think of them as mini-portals or topic hubs within the platform.

**Required addition**: New epic or expand E9:
> "As an agency administrator, I want to create learning areas (omrader/fagsider) with configurable read and edit access per user group, so that different divisions and competency areas have their own managed spaces."

Acceptance criteria:
- Create named area/page with description, branding
- Assign read access to user groups
- Assign edit/contribute access to user groups
- Areas contain courses, documents, links, announcements
- Navigation: areas visible to users with access

**Phase**: MVP (specifically tested in POC 6.1)

---

### GAP #8: Custom role configuration and assignment
**POC 6.2**: "Vis hvordan en rolle i systemet kan konfigureres og tildeles andre brukere i systemet. Rollen bor ha flere rettigheter enn en sluttbruker, og tilgang til et utvalg rapporter."

**Current build plan**: PF-02 covers predefined roles (Learner, Leader, Content Manager, Division Admin, System Admin). But there is NO story for **creating custom roles** with configurable permission sets. POC 6.2 asks to CONFIGURE a new role (not just assign a predefined one).

**Required addition**: Extend PF-02:
> "As an agency administrator, I want to create custom roles with specific permission sets, so that I can define access patterns that match our organizational structure."

Acceptance criteria:
- Custom role creation: name, description, select permissions from a checklist
- Permissions granularity: view courses, edit courses, view reports (per scope), manage users (per scope), manage content
- Assign custom role to users
- Custom roles sit between "Learner" and "System Admin"

**Phase**: MVP (specifically tested in POC 6.2)

---

### GAP #9: Landing page content management (CMS-like) (MISSING)
**POC 6.3**: "Vis hvordan landingssider for interne og eksterne sluttbrukere kan vedlikeholdes og beskriv hvilke andre roller som kan utfore vedlikehold pa landingssider, som for eksempel a bytte ut et bilde, legge inn informasjonstekst eller legge til nytt innhold."

**Current build plan**: NO STORY covers CMS-like landing page editing. This requires a lightweight content management system within the LMS where admins can edit landing page content (images, text, featured courses) without developer involvement.

**Required addition**:
> "As an agency administrator, I want to edit landing page content (hero image, welcome text, featured courses, announcements) for internal and external users, so that the platform feels current and managed without developer help."

Acceptance criteria:
- WYSIWYG editor for landing page sections
- Upload/replace hero image
- Add/edit information text (rich text)
- Pin/feature specific courses on landing page
- Different landing page content for internal vs external users
- Delegatable: agency admin can grant landing page editing to other roles

**Phase**: MVP (specifically tested in POC 6.3)

---

## 3. LEARNING ACTIVITIES: Coverage Check

Cross-referencing Bilag 1 vedlegg 5 (Laeringsaktiviteter) against build plan:

| Activity Type | Build Plan Coverage | Gap? |
|---|---|---|
| 1.1 Classroom courses (physical) | LM-08 | COVERED |
| 1.2 Internal events (physical) | LM-08 | COVERED |
| 1.3 Conferences | LM-08 (basic) | WEAK - no conference-specific features (external attendees via subdomain) |
| 2.1 Webinars | LM-08 + LM-11 (Phase 2) | MVP: basic event. Phase 2: Teams integration |
| 2.2 Digital courses (live instructor) | LM-08 | COVERED as event type |
| 3.1 E-learning courses (SCORM) | CE-01, CE-02 | COVERED |
| 3.2 Quiz/tests | CE-03 | COVERED |
| 3.3 Assignment submissions | **LM-13 (Phase 2 only!)** | **GAP #10** - should be MVP |
| 4. Learning paths | LM-06 | COVERED |
| 5.1 Befaring (field inspections) | **MISSING** | **GAP #11** - no workplace activity type |
| 5.2 Hospitering (internships) | **MISSING** | **GAP #11** |
| 5.3 Beredskapsovelser (emergency drills) | **MISSING** | **GAP #11** |
| 5.5 Simulatortrening | **MISSING** | **GAP #11** |
| 5.6 Teoretiske prover | CE-03 (quiz) | PARTIALLY - formal exam vs quiz |
| 5.7 Praktiske prover (with sensor) | **MISSING** | **GAP #11** |
| 5.8 Tester/kunnskapskontroller | CE-03 | COVERED |
| 5.9 Jobbobservasjon/kollegaveiledning | **MISSING** | **GAP #11** |
| 5.10 Re-autorisasjon | CF-05 (certification renewal) | COVERED |
| 5.11 Egentrening (self-training) | **MISSING** | **GAP #1** (same as external activity registration) |
| 5.12 Veiledning/coaching/mentoring | **MISSING** | **GAP #11** |

---

### GAP #10: Assignment/task submissions (oppgaveinnleveringer)
**Vedlegg 5, section 3.3**: "Deltakere leverer tekst, rapporter, video eller lydfiler som del av laeringstiltak."

**Current build plan**: LM-13 covers this but is deferred to Phase 2. SVV lists this as a core learning activity type. Should be in MVP.

**Required addition**: Move LM-13 to MVP:
> "As a learner, I want to submit assignments (text, files, video) for instructor review, so that project-based and experiential learning is documented."

Acceptance criteria:
- File upload (PDF, DOCX, MP4, MP3)
- Text submission (rich text)
- Instructor review workflow (submit > review > grade/approve)
- Grade/result visible to learner and leader
- Status on dashboard and in reports

---

### GAP #11: Workplace-based activity types (arbeidsplassbaserte aktiviteter)
**Vedlegg 5, section 5**: SVV lists 12 types of workplace-based activities. The build plan has NO dedicated epic or stories for these.

This is **the biggest gap** and directly undermines the 70/20/10 model that is central to SVV's vision. The 70% (experiential learning) is entirely missing from the build plan's MVP.

**Required addition**: New epic or major expansion of E4:
> "As a learner/leader, I want to register, document, and track workplace-based learning activities (inspections, internships, drills, simulator training, practical exams, job observation, coaching, self-training), so that the 70% of learning that happens on the job is captured in the competency framework."

Key features needed:
1. **Activity type registry**: Configurable activity types (befaring, hospitering, beredskapsovelse, simulatortrening, praktisk prove, jobbobservasjon, egentrening, veiledning)
2. **Activity registration**: Learner or leader can register a completed workplace activity (type, date, description, evidence upload, observer/sensor name)
3. **Checklists/assessment forms**: For job observation and practical exams, support structured checklists that an observer fills out
4. **Sensor confirmation**: For practical exams, a designated sensor must confirm/approve the result
5. **Gyldighetsperioder**: Validity periods and automatic expiry for time-limited competencies
6. **Self-registration**: Learner registers own activities (egentrening, external courses), optionally approved by leader
7. **Integration with competency framework**: Workplace activities contribute to competency profile
8. **Reporting**: All workplace activities appear in reports alongside formal learning

**Phase**: MVP (this is what makes SVV's 70/20/10 vision real -- and what differentiates us from every other LMS)

---

## 4. REFERENCED FUNCTIONAL REQUIREMENTS (Krav IDs)

The POC document references specific requirement IDs. Cross-check:

| Krav ID | Referenced in POC | Build Plan Coverage |
|---|---|---|
| 002 | 6.3 (landing page maintenance) | **GAP #9** - No CMS-like page editing |
| 003 | 5.2 (granular report access) | **GAP #5** - No per-object report permissions |
| 004 | 5.2, 6.2 (reporting + role config) | **GAP #5, #8** - Partially covered |
| 009 | 5.1, 6.1, 6.3 (area management) | **GAP #4, #7** - No learning areas/fagsider |
| 020 | 5.3 (data/analytics by target group) | **GAP #6** - Weak target group filtering |
| 021 | 3.1 (leader: team learning status) | COVERED (LM-05) |
| 024 | 2.1 (external user registration) | COVERED (AU-03, EP-01) |
| 025 | 2.1 (external user profile fields) | COVERED (EP-01) |
| 029 | 1.1 (search and navigation) | COVERED (LM-02, AI-02) |
| 038 | 4.1 (course creation from SCORM) | COVERED (CE-01, CE-02) |
| 039 | 4.4 (mandatory/optional per group) | **GAP #3** - Different cert duration per group |
| 040 | 4.3 (update and archive courses) | COVERED (CE-05) |
| 043 | 4.1 (validate, preview, publish) | COVERED (CE-02) |
| 044 | 1.2, 4.2 (learning path) | COVERED (LM-06) |
| 055 | 3.3 (register/approve external activity) | **GAP #1** - External activity registration missing |
| 056 | 4.4 (certification with different validity) | **GAP #3** - Per-group cert validity |
| 057 | 1.1 (notifications and reminders) | COVERED (PF-04) |
| 058 | 4.2 (learning path with event + test + cert) | **GAP #2** - Event registration forms |
| 059 | 3.2 (leader assigns courses) | COVERED (LM-07) |

---

## 5. SUMMARY: Required Changes to Build Plan

### Must-Add to MVP (11 gaps)

| Gap # | Description | Priority | New Story Needed |
|---|---|---|---|
| **#1** | Register activities completed OUTSIDE the platform | P0 (POC 3.3) | Yes |
| **#2** | Event registration forms (allergies, accommodation) | P0 (POC 4.2) | Extend LM-08 |
| **#3** | Per-group certification validity duration | P0 (POC 4.4) | Extend CF-05 |
| **#4** | Division admin scoped management | P0 (POC 5.1) | Yes |
| **#5** | Granular per-course report access delegation | P0 (POC 5.2) | Yes |
| **#6** | Target group filtering in analytics | P1 (POC 5.3) | Extend AR-01/AR-02 |
| **#7** | Learning areas/fagsider with access control | P0 (POC 6.1) | Yes - new feature |
| **#8** | Custom role configuration | P0 (POC 6.2) | Extend PF-02 |
| **#9** | Landing page CMS editing | P0 (POC 6.3) | Yes - new feature |
| **#10** | Assignment submissions (move to MVP) | P1 | Move LM-13 to MVP |
| **#11** | Workplace-based activity types (70% of 70/20/10) | P0 (SVV vision) | Yes - major new feature set |

### Impact Assessment

- **11 gaps identified**, of which **9 are P0** (directly tested in POC or central to SVV's vision)
- Gaps #7, #9, #11 require entirely new feature areas not in the current plan
- Gap #11 (workplace activities) is the single biggest miss -- it's the 70% of SVV's learning model
- Adding these gaps adds approximately **8-12 additional user stories** to MVP
- Estimated additional effort: **4-6 sprints** (8-12 weeks)
- This extends the MVP timeline from 26 weeks to approximately 34-38 weeks

### Revised Effort Estimate

| | Original | With Gaps Fixed |
|---|---|---|
| MVP stories | 54 | 66-70 |
| MVP sprints | 13 | 17-19 |
| MVP duration | 26 weeks | 34-38 weeks |
| MVP cost | ~4.2 MNOK | ~5.5-6.0 MNOK |

---

*This gap analysis should be incorporated into the BUILD_DELIVERY_PLAN.md as a revision. The gaps are significant but addressable -- they represent real client requirements that were missed in the initial analysis, not fundamental architectural issues.*
