# UX Strategy & POC Demonstration Plan

## Statens vegvesen LMS Tender (25/223727)

---

## 1. Executive Summary

Usability carries 12.5% of the 80% quality weight (effectively 10% of total score). Evaluation is based on three pillars: (1) UI/UX descriptions and user stories in Bilag 2, (2) live POC demonstration of 5 role-based scenarios in 80 minutes, and (3) hands-on evaluator testing in a demo environment. This strategy optimizes for all three using ISO 9241-11:2018 and Nielsen's 10 heuristics as scoring frameworks.

**Core UX thesis**: SVV employees range from office workers to field personnel on highways, bridges, and tunnels. The platform must feel as intuitive as the consumer apps they use daily (Netflix-like recommendations, mobile-first interactions) while delivering the administrative rigor a 5,000-person government organization requires.

---

## 2. User Journey Maps

### 2.1 Internal User (ansatt) -- ~5,300 users

**Profile**: Road inspectors, engineers, administrative staff, consultants. Varying technical skill. Many work in the field with tablets/phones. Age range 25-65.

**Journey: Daily Learning Interaction**

```
Login (SSO/ForgeRock) --> Personalized Landing Page --> Quick Actions
     |                        |                           |
     |                   [Recommendations]           [Resume course]
     |                   [Mandatory alerts]           [View path]
     |                   [News/updates]              [Notifications]
     |                        |
     |                   Search/Browse --> Course Detail --> Enroll --> Learn --> Complete
     |                                                                            |
     |                                                              [Certificate generated]
     |                                                              [Progress updated]
     |                                                              [Next in path unlocked]
```

**Key UX moments**:
- **First 5 seconds**: Landing page must show personalized, relevant content. No generic dashboards.
- **Mandatory course alert**: Red badge/banner with deadline. One-click access to overdue courses.
- **Search**: Faceted, fast, tolerant of Norwegian typos. Filter by type, duration, category, mandatory/optional.
- **Learning path**: Visual progress bar, clear next step, estimated time remaining.
- **Mobile field use**: Offline-capable viewing, large touch targets, minimal data usage.
- **Notifications**: Configurable (browser push, email, in-app). Never overwhelming.

**Pain points to solve**:
- Current Kilden has no personalization -- everyone sees the same content
- No mobile-optimized experience for field workers
- Search is basic, no recommendations

---

### 2.2 External User (ekstern bruker) -- ~16,100 users

**Profile**: Contractors, transport companies, arbeidsvarsling personnel. Many are non-technical workers who interact with the system a few times per year for certification. Norwegian-speaking, varying digital literacy.

**Journey: Registration to First Certification**

```
SVV website/email link --> Registration page --> ID-porten auth (BankID/MinID)
     |                                               |
     |                                    [Account created automatically]
     |                                    [Profile pre-filled from Folkeregisteret]
     |                                               |
     |                                    Onboarding wizard (3 steps max)
     |                                    1. Confirm personal details
     |                                    2. Select employer/organization
     |                                    3. View available courses/certifications
     |                                               |
     |                                    Course catalog (filtered for external) --> Enroll
     |                                               |
     |                                    Complete course --> Exam --> Certificate issued
     |                                               |
     |                                    [Certificate PDF downloadable]
     |                                    [Certificate in digital wallet]
     |                                    [Employer notified]
```

**Key UX moments**:
- **Registration**: Must be frictionless. ID-porten handles identity. System pre-fills everything possible.
- **Onboarding**: Maximum 3 steps. Progressive disclosure -- don't overwhelm.
- **Return visits**: Remember user, show their certificates and expiry dates prominently.
- **Certificate renewal**: Proactive notification 90/60/30 days before expiry.
- **KKP portal continuity**: External users familiar with current KKP must find familiar workflows.

**Pain points to solve**:
- Current KKP is a separate system with separate login
- No proactive certificate expiry notifications
- Manual certificate issuance process

---

### 2.3 Leader (leder med personalansvar) -- ~400 users

**Profile**: Section heads, team leaders, division directors. Need oversight without micromanagement. Time-poor, need information at a glance.

**Journey: Team Competency Management**

```
Login (SSO) --> Leader Dashboard
     |              |
     |         [Team overview widget]     [Alerts: 3 expiring certs]
     |         [Compliance status]        [Pending approvals: 2]
     |         [Quick actions]            [Upcoming deadlines]
     |              |
     |         Team Status View --> Individual Employee --> Competency Profile
     |              |                                          |
     |         [Filter by: division, role, status]        [Learning history]
     |         [Traffic light: green/yellow/red]          [Certifications]
     |         [Export to Excel]                          [Assign course]
     |              |
     |         Assign Course --> Select employees --> Set deadline --> Confirm
     |              |
     |         Approve Activity --> Review submission --> Approve/Reject with comment
```

**Key UX moments**:
- **Dashboard**: Traffic light system (green/yellow/red) for instant team compliance overview.
- **Certification expiry**: Automatic alerts 90/60/30 days before. Sorted by urgency.
- **Bulk assignment**: Assign courses to groups/teams, not individual-by-individual.
- **Approval workflow**: Quick approve/reject with minimal clicks. Batch approval for routine items.
- **Delegation**: Ability to delegate approval authority during absence.

**Pain points to solve**:
- Leaders currently track competency in Excel spreadsheets
- No proactive alerts -- they discover expired certificates reactively
- Assignment is manual and time-consuming

---

### 2.4 Content Manager (innholdsansvarlig) -- subset of ~400 admins

**Profile**: Instructional designers, HR competency specialists, subject matter experts. Technically proficient. Need efficient content workflows.

**Journey: Course Creation to Publication**

```
Login (SSO) --> Content Management Area
     |              |
     |         [My courses]              [Drafts]
     |         [Templates]               [Under review]
     |         [Content library]         [Published]
     |              |
     |         Create New Course --> Select type (SCORM upload / built-in editor / classroom)
     |              |
     |         [SCORM path]:
     |         Upload SCORM package --> Preview --> Set metadata --> Set target audience
     |              |                                    |
     |              |                              [Title, description]
     |              |                              [Category, tags]
     |              |                              [Duration, difficulty]
     |              |                              [Mandatory/optional]
     |              |                              [Validity period]
     |              |
     |         Create Learning Path --> Add courses in sequence --> Set dependencies
     |              |                                                    |
     |              |                                              [Prerequisites]
     |              |                                              [Completion criteria]
     |              |                                              [Certificate template]
     |              |
     |         Course Lifecycle:
     |         Draft --> Under Review --> Published --> Update --> Archive
     |                                      |
     |                                [Version control]
     |                                [Enrollment stats]
     |                                [Completion rates]
```

**Key UX moments**:
- **SCORM upload**: Drag-and-drop, automatic validation, instant preview. Support SCORM 1.2 and 2004.
- **Metadata**: Smart defaults, auto-suggest tags, template-based creation.
- **Learning path builder**: Visual drag-and-drop interface. Clear dependency arrows.
- **Course lifecycle**: Status badges, version history, one-click archive with data retention.
- **Preview**: WYSIWYG preview as different user roles before publishing.

**Pain points to solve**:
- Current system lacks visual learning path builder
- No version control for courses
- Limited metadata and categorization options

---

### 2.5 Division Administrator (divisjonsadministrator) -- subset of ~400 admins

**Profile**: Division-level HR/competency administrators. Need reporting and analytics for strategic decisions. Data-oriented, familiar with PowerBI.

**Journey: Reporting and Analytics**

```
Login (SSO) --> Admin Dashboard
     |              |
     |         [Division KPIs]           [Quick reports]
     |         [Compliance overview]     [Scheduled exports]
     |         [Trend charts]           [Data freshness indicator]
     |              |
     |         Reports Section --> Pre-built reports / Custom report builder
     |              |                        |
     |         [Completion rates]       [Drag-and-drop fields]
     |         [Compliance status]      [Filter by: period, division, role]
     |         [Training hours]         [Visualization options]
     |         [Certification expiry]   [Save as template]
     |              |
     |         Dashboard Builder --> Widget selection --> Layout customization
     |              |
     |         Data Export --> Select format (Excel/CSV/PowerBI) --> Schedule or download
     |              |
     |         [PowerBI integration: auto-push to SVV reporting platform]
```

**Key UX moments**:
- **Dashboard**: Pre-configured with most-requested KPIs. Customizable widget layout.
- **Reports**: Both pre-built templates and custom builder. No SQL knowledge required.
- **Data export**: One-click Excel export. Scheduled PowerBI data feeds.
- **Drill-down**: Click any number to see underlying data. From division to team to individual.
- **Comparison**: Compare divisions, time periods, course types side-by-side.

**Pain points to solve**:
- Current reporting is manual Excel compilation
- No real-time data -- reports are always stale
- No standardized KPIs across divisions

---

## 3. POC Demonstration Strategy

### 3.1 Format and Timing

The POC is **40 + 40 minutes** (two sessions). This is extremely tight for 5 scenarios. Every second counts.

**Recommended time allocation**:

| Scenario | Role | Time | Priority |
|---|---|---|---|
| 1. Internal user | Internal employee | 18 min | HIGH -- most users, most visible |
| 2. External user | External contractor | 10 min | MEDIUM -- distinct user journey |
| 3. Leader | Team leader | 16 min | HIGH -- solves real pain (Excel tracking) |
| 4. Content manager | Content admin | 18 min | HIGH -- core platform value |
| 5. Division admin | Division admin | 18 min | MEDIUM -- reporting differentiator |

**Session split**:
- **Session 1 (40 min)**: Scenarios 1 + 2 + 3 (internal user, external user, leader)
- **Session 2 (40 min)**: Scenarios 4 + 5 (content manager, division admin)

### 3.2 Demonstration Principles

1. **Start strong**: Open with the most impressive screen (personalized landing page). First impressions anchor the entire evaluation.
2. **Tell a story, not a feature list**: Each scenario is a user story. "Maria is a road inspector in Trondheim who needs to renew her safety certification..."
3. **Show, don't explain**: Minimize talking over screens. Let the evaluators see the interaction flow naturally.
4. **Anticipate questions**: Pre-load realistic Norwegian data (course names in Norwegian, real SVV division names, realistic employee names).
5. **Demonstrate speed**: Fast page loads, instant search results, quick interactions. Evaluators notice latency.
6. **Show error recovery**: Deliberately show one validation message or edge case -- it demonstrates robustness and builds trust.
7. **End each scenario with the "wow" moment**: AI recommendation, one-click dashboard, visual learning path builder.

### 3.3 Scenario Scripts

#### Scenario 1: Internal User (18 min)

**Character**: Kari Nordmann, road inspector, Drift og vedlikehold division, based in Bergen.

| Time | Action | What evaluators see | UX principle |
|---|---|---|---|
| 0:00 | SSO login | Seamless, no password prompt | Efficiency |
| 0:30 | Landing page loads | Personalized: Kari's name, her courses, her path progress | Recognition > recall |
| 1:30 | Show recommendations | "Based on your role and learning history" -- 3-4 relevant courses | Match real world |
| 3:00 | Show mandatory alert | Red badge: "2 mandatory courses due within 30 days" | Visibility of status |
| 4:00 | Click notification bell | Dropdown with recent notifications, mark as read | Consistency |
| 5:00 | Search for "arbeidsvarsling" | Instant results, faceted filters (type, duration, format) | Flexibility & efficiency |
| 7:00 | Open course detail | Rich preview: description, duration, format, reviews, prerequisites | Help & documentation |
| 8:00 | Enroll in course | One-click enrollment confirmation | User control |
| 9:00 | Navigate to learning path | Visual progress: 4/7 modules complete, next module highlighted | Visibility of status |
| 11:00 | Open next module | Course player loads, progress auto-saved | Error prevention |
| 13:00 | Complete a quiz | Immediate feedback, score shown, certificate preview | Feedback |
| 15:00 | View certificate | Professional PDF, downloadable, in certificate wallet | Satisfaction |
| 16:00 | Mobile view | Switch to tablet/phone view -- responsive, touch-friendly | Flexibility |
| 17:00 | Show accessibility | Keyboard navigation, screen reader compatible, contrast modes | Universal design |

#### Scenario 2: External User (10 min)

**Character**: Erik Hansen, traffic controller for Mesta AS, needs arbeidsvarsling certification.

| Time | Action | What evaluators see | UX principle |
|---|---|---|---|
| 0:00 | Access registration link | Clean, branded SVV registration page | Aesthetic design |
| 0:30 | Click "Logg inn med ID-porten" | ID-porten redirect, BankID authentication | Consistency with Norwegian standards |
| 1:30 | Account created | Profile pre-filled from Folkeregisteret | Efficiency |
| 2:00 | Onboarding wizard | 3 steps: confirm info, select employer, see available courses | Progressive disclosure |
| 4:00 | First time landing page | Tailored for external users, relevant courses prominent | Match real world |
| 5:00 | Find certification course | Filtered catalog, clear pricing (if applicable), prerequisites shown | Help & documentation |
| 6:30 | Enroll and start | Immediate access, progress tracking visible | User control |
| 8:00 | Show certificate history | Previous certifications, expiry dates, renewal options | Visibility of status |
| 9:00 | Show mobile experience | Full functionality on phone -- field workers' primary device | Flexibility |

#### Scenario 3: Leader (16 min)

**Character**: Per Olsen, section head, Trafikant og kjoretoey division, 25 direct reports.

| Time | Action | What evaluators see | UX principle |
|---|---|---|---|
| 0:00 | SSO login to leader view | Dashboard with team overview widget | Recognition > recall |
| 1:00 | Team compliance overview | Traffic light grid: 20 green, 3 yellow, 2 red | Visibility of status |
| 2:30 | Click red status employee | Detail: "Sikkerhetskurs expired 2026-02-15" with one-click assign renewal | Error prevention |
| 4:00 | Certification expiry alerts | List sorted by urgency, 90/60/30 day warnings | Feedback |
| 6:00 | Bulk course assignment | Select 5 employees, assign "HMS grunnkurs", set deadline | Flexibility & efficiency |
| 8:00 | Confirmation and notification | Employees notified automatically, calendar entry option | Consistency |
| 9:00 | Pending approvals | 2 activity completions awaiting approval | User control |
| 10:30 | Approve activity | Review submission, approve with one click, add comment | Efficiency |
| 12:00 | Team training report | This quarter vs last quarter, completion rates, hours | Visibility of status |
| 14:00 | Export to Excel | One-click download, formatted and ready for management meeting | Match real world |
| 15:00 | Show delegation | Set deputy approver during planned absence | Error prevention |

#### Scenario 4: Content Manager (18 min)

**Character**: Lise Berg, instructional designer, Fellesfunksjoner og HR.

| Time | Action | What evaluators see | UX principle |
|---|---|---|---|
| 0:00 | Content management area | Dashboard: my courses, drafts, published, analytics | Recognition > recall |
| 1:00 | Create new course | Select "SCORM upload" from course type options | User control |
| 2:00 | Drag-and-drop SCORM | Upload, automatic validation, package contents shown | Error prevention |
| 3:30 | SCORM preview | Course plays in preview mode, all interactions work | Visibility of status |
| 5:00 | Set metadata | Title, description, category, tags (auto-suggest), target audience | Flexibility |
| 7:00 | Configure settings | Mandatory/optional toggle, validity period, completion criteria | Consistency |
| 8:30 | Create learning path | Visual builder: drag courses into sequence, set dependencies | Direct manipulation |
| 11:00 | Add prerequisites | Draw connection between courses, set minimum score | Visibility of status |
| 12:30 | Preview as user | Switch to learner view, see how path appears | Help & documentation |
| 14:00 | Publish course | Status change from draft to published, notification to target audience | Feedback |
| 15:00 | Update existing course | Upload new version, old completions preserved, changelog | Error prevention |
| 16:30 | Archive course | Archive with data retention, replaced by newer version | User control |
| 17:30 | View course analytics | Completion rates, average scores, drop-off points | Visibility of status |

#### Scenario 5: Division Administrator (18 min)

**Character**: Anders Haugen, HR/competency administrator, Utbygging division.

| Time | Action | What evaluators see | UX principle |
|---|---|---|---|
| 0:00 | Admin dashboard | Division KPIs: compliance %, training hours, certifications issued | Recognition > recall |
| 1:30 | Pre-built report | "Compliance status by team" -- instant results, color-coded | Visibility of status |
| 3:00 | Drill down | Click team row, see individual status, click individual, see history | Consistency |
| 5:00 | Custom report builder | Drag fields, set filters (date range, division, course type) | Flexibility & efficiency |
| 7:00 | Visualization | Switch between table/bar chart/trend line | User control |
| 8:30 | Save report template | Name it, schedule it, set recipients | Error prevention |
| 10:00 | Division comparison | Side-by-side: Utbygging vs Drift og vedlikehold completion rates | Match real world |
| 12:00 | Certification dashboard | All certifications expiring next 90 days across division | Visibility of status |
| 13:30 | Data export - Excel | One-click, formatted with SVV header, ready for management | Efficiency |
| 15:00 | Data export - PowerBI | Show PowerBI integration, live data feed configuration | Match real world |
| 16:30 | Scheduled reports | Set up weekly compliance email to division director | Flexibility |
| 17:30 | Audit trail | Show who changed what, when, for compliance purposes | Help & documentation |

### 3.4 Demo Environment Preparation

**Data requirements** (must be loaded before demonstration):

| Data type | Volume | Details |
|---|---|---|
| Users | 50+ | Realistic Norwegian names, spread across all 6 divisions |
| Courses | 30+ | Mix of e-learning, classroom, webinars. Norwegian titles and descriptions |
| SCORM packages | 3-5 | Pre-loaded, verified working. Include one for live upload demo |
| Learning paths | 3+ | With 4-7 courses each, varying completion states |
| Certifications | 20+ | Mix of valid, expiring soon, and expired |
| Completion records | 200+ | Varied dates, scores, statuses for realistic reporting |
| Notifications | 10+ | Mix of types: reminders, assignments, completions |

**Environment checklist**:
- [ ] Norwegian language fully configured (no English strings visible)
- [ ] SVV logo and color scheme applied (brand identity)
- [ ] Realistic organizational structure (6 divisions, teams, roles)
- [ ] ID-porten test environment connected and verified
- [ ] SCORM test package prepared and validated
- [ ] Mobile/tablet responsive views tested on actual devices
- [ ] All 5 user accounts pre-configured with appropriate data
- [ ] Performance verified: all pages load in under 2 seconds
- [ ] Accessibility verified: keyboard navigation, screen reader, contrast
- [ ] Backup plan: offline screenshots/video if connectivity fails
- [ ] Two browsers ready: one for presentation, one for quick recovery

---

## 4. Accessibility Compliance Plan

### 4.1 Legal Framework

Norway requires compliance with:
- **Likestillings- og diskrimineringsloven sections 17-18**: Universal design of ICT
- **Forskrift om universell utforming av IKT-losninger**: Mandates WCAG 2.1 AA for public-sector solutions
- **EU Web Accessibility Directive** (transposed into Norwegian law)

### 4.2 WCAG 2.1 AA Compliance Matrix

| WCAG Principle | Key Requirements | LMS Implementation |
|---|---|---|
| **Perceivable** | | |
| 1.1 Text alternatives | Alt text for all images | All icons, images, charts have descriptive alt text |
| 1.2 Time-based media | Captions, transcripts | Video content has Norwegian captions. Audio has transcripts |
| 1.3 Adaptable | Semantic HTML, logical structure | Proper heading hierarchy (h1-h6), landmark regions, ARIA labels |
| 1.4 Distinguishable | Color contrast, text resize | 4.5:1 contrast ratio minimum. Text scalable to 200% without loss |
| **Operable** | | |
| 2.1 Keyboard accessible | All functionality via keyboard | Tab order logical, focus visible, no keyboard traps |
| 2.2 Enough time | Adjustable time limits | Exam timers adjustable, session timeout warnings with extension |
| 2.3 Seizures | No flashing content | No content flashes more than 3 times per second |
| 2.4 Navigable | Skip links, page titles, focus order | Skip-to-content link, breadcrumbs, consistent navigation |
| **Understandable** | | |
| 3.1 Readable | Language declaration | `lang="nb"` set, language changes marked |
| 3.2 Predictable | Consistent navigation | Navigation identical across pages, no unexpected context changes |
| 3.3 Input assistance | Error identification, labels | Form errors identified in text (not just color), suggestions provided |
| **Robust** | | |
| 4.1 Compatible | Valid HTML, ARIA | Semantic HTML5, ARIA roles/states for dynamic content, tested with NVDA/JAWS |

### 4.3 Accessibility Testing Plan

- **Automated**: axe-core, Lighthouse, WAVE integrated into CI/CD
- **Manual**: Keyboard-only navigation testing for all workflows
- **Screen reader**: Testing with NVDA (Windows), VoiceOver (macOS/iOS), TalkBack (Android)
- **Color**: Contrast checker, colorblind simulation (deuteranopia, protanopia, tritanopia)
- **Zoom**: All content usable at 200% browser zoom
- **Motion**: `prefers-reduced-motion` respected, animations disable-able

### 4.4 POC Accessibility Demonstration Points

During the POC, explicitly demonstrate:
1. **Keyboard navigation**: Tab through landing page, enroll in course, complete quiz -- all without mouse
2. **Screen reader**: Show how NVDA/VoiceOver announces page structure, form labels, notifications
3. **High contrast mode**: Toggle high contrast, show all content remains readable
4. **Text scaling**: Zoom to 200%, show layout adapts without horizontal scrolling
5. **Focus management**: Show visible focus indicators, logical tab order in modals and dropdowns

---

## 5. Responsive Design Approach

### 5.1 Breakpoint Strategy (Mobile-First)

| Breakpoint | Target | Layout Adjustments |
|---|---|---|
| 320-479px | Mobile (portrait) | Single column, hamburger nav, stacked cards, bottom navigation |
| 480-767px | Mobile (landscape) / small tablet | Single column, slightly larger touch targets |
| 768-1023px | Tablet | Two-column where appropriate, sidebar navigation option |
| 1024-1279px | Desktop | Full layout, sidebar + main content, multi-column dashboards |
| 1280px+ | Large desktop | Maximum width container, expanded dashboard widgets |

### 5.2 Mobile-Specific UX Considerations

SVV has field workers (road inspectors, maintenance crews) who will primarily use mobile devices:

- **Touch targets**: Minimum 44x44px (WCAG 2.5.5)
- **Navigation**: Bottom tab bar for primary actions (Home, My Learning, Search, Notifications, Profile)
- **Offline capability**: Downloaded courses accessible without connectivity (tunnels, remote roads)
- **Data efficiency**: Lazy loading images, compressed video, progressive enhancement
- **Orientation**: Support both portrait and landscape, no content loss on rotation
- **Native-like**: PWA with home screen installation, push notifications

### 5.3 Responsive Component Patterns

| Component | Mobile | Tablet | Desktop |
|---|---|---|---|
| Navigation | Bottom bar + hamburger | Side rail | Full sidebar |
| Dashboard | Stacked cards, swipeable | 2-column grid | Multi-column grid |
| Course catalog | List view, swipe filters | Grid view, side filters | Grid + sidebar filters |
| Learning path | Vertical timeline | Vertical timeline + details | Horizontal timeline |
| Reports/tables | Card view per row | Condensed table | Full table with sorting |
| Course player | Full screen | Full screen with sidebar | Embedded with navigation |

---

## 6. Nielsen's 10 Heuristics Mapping

### 6.1 Heuristic-to-Scenario Matrix

| Heuristic | S1: Internal | S2: External | S3: Leader | S4: Content Mgr | S5: Admin |
|---|---|---|---|---|---|
| **1. Visibility of system status** | Progress bars, loading states, completion % | Registration steps indicator | Traffic light team status | Course status badges (draft/review/published) | Real-time data freshness indicator |
| **2. Match real world** | Norwegian language, SVV terminology, familiar metaphors | ID-porten (known auth), familiar form patterns | "Mine medarbeidere" not "user management" | Familiar content management metaphors | Division/team hierarchy matching SVV org chart |
| **3. User control & freedom** | Undo enrollment, bookmark courses, customize dashboard | Back navigation clear, exit onboarding option | Undo course assignment, modify deadlines | Revert course version, unpublish | Reset report filters, modify saved reports |
| **4. Consistency & standards** | Consistent icons, button placement, interaction patterns | Same patterns as internal UI, just scoped | Same interaction patterns as employee view | Content tools follow platform conventions | Consistent filter/sort/export patterns |
| **5. Error prevention** | Confirm before dropping course, validate prerequisites | Pre-filled fields reduce input errors | Warn before assigning expired course | SCORM validation before upload, preview before publish | Confirm before large data exports |
| **6. Recognition > recall** | Sidebar with recent courses, visual course thumbnails | Previously viewed courses remembered | Team members with photos, recent actions | Recently edited content, template gallery | Saved report templates, recent exports |
| **7. Flexibility & efficiency** | Keyboard shortcuts, quick actions, search-first | Simplified flow, minimal required fields | Bulk operations, filtered views | Templates, batch metadata editing | Custom reports, scheduled exports |
| **8. Aesthetic & minimal design** | Clean cards, whitespace, no clutter | Minimal registration, calm onboarding | Dashboard widgets, not data tables | Content-focused editor, clean toolbars | Data visualization over raw numbers |
| **9. Help users recognize/recover errors** | Clear error messages in Norwegian, suggestion to fix | "Fant ikke kontoen din? Prosv igjen eller kontakt..." | "Kurset er utlopst. Vil du tildele fornyelse?" | SCORM error: "Pakken mangler imsmanifest.xml" | "Ingen data for valgt periode. Prosv et annet datointervall" |
| **10. Help & documentation** | Contextual help tooltips, guided tours for new users | Onboarding wizard, FAQ, chat support link | Leader guide, tooltip on dashboard widgets | Content creation guides, SCORM spec help | Report field descriptions, export format help |

### 6.2 Heuristic Demonstration Priorities

During the 80-minute POC, the following heuristics have the highest visibility and should be demonstrated most prominently:

1. **Visibility of system status** -- show in every scenario (easiest to demonstrate)
2. **Match between system and real world** -- Norwegian language, SVV terminology, realistic data
3. **Flexibility and efficiency of use** -- shortcuts, bulk operations, search
4. **Aesthetic and minimalist design** -- clean, modern UI speaks for itself
5. **Error prevention** -- one deliberate validation demo builds trust

---

## 7. ISO 9241-11:2018 Optimization Strategy

ISO 9241-11 defines usability through three dimensions: **Efficacy**, **Efficiency**, and **Satisfaction**. Each must be demonstrated.

### 7.1 Efficacy (Effectiveness)

*Can users achieve their goals completely and accurately?*

| Scenario | Efficacy demonstration |
|---|---|
| Internal user | Complete full learning path module, receive certificate -- goal fully achieved |
| External user | From zero to enrolled in certification course in under 3 minutes |
| Leader | Identify all team members with expiring certifications and assign renewals |
| Content manager | Upload SCORM, configure metadata, publish -- course live and accessible |
| Division admin | Generate compliance report, export to Excel -- data accurate and complete |

**How to maximize efficacy scores**:
- Ensure every POC task reaches a clear, successful end state
- No dead ends or "this feature is coming soon" moments
- Show confirmation of completed actions (success messages, updated status)

### 7.2 Efficiency

*Can users achieve goals with minimum time and effort?*

| Metric | Target | How to demonstrate |
|---|---|---|
| Task completion time | Below industry average | Time key tasks during rehearsal, show speed in demo |
| Click count | Minimal for common tasks | Count clicks for each scenario, optimize the most frequent |
| Navigation depth | Max 3 clicks to any content | Flat navigation, search-first, quick actions |
| Cognitive load | Low -- no training needed | Evaluators should understand the UI without explanation |
| Error rate | Near zero for standard tasks | Pre-validated data, smart defaults, clear affordances |

**Specific efficiency targets for POC**:
- SSO login to landing page: < 2 seconds
- Search to course detail: < 3 clicks
- Course enrollment: 1 click from detail page
- Leader: identify at-risk employee: visible on dashboard without clicking
- Report generation: < 5 clicks from dashboard to exported Excel

### 7.3 Satisfaction

*Do users feel positive about the experience?*

Satisfaction is the hardest to demonstrate in a POC but the most differentiating. Strategies:

| Satisfaction driver | Implementation |
|---|---|
| **Visual appeal** | Modern, clean design. Not a "government system" feel. Inspire confidence |
| **Personalization** | "Welcome back, Kari" -- content tailored to role and history |
| **Responsiveness** | Instant feedback on every interaction. No waiting, no spinners |
| **Delight moments** | Subtle animations on completion, confetti-free but satisfying progress updates |
| **Trust signals** | Secure badge, data privacy indicator, clear consent management |
| **Accessibility** | "Everyone can use this" feeling -- inclusive design is satisfying for evaluators |
| **Norwegian-first** | Perfect Norwegian language, not translated English. Idiomatically correct |

**Critical**: Evaluators will form satisfaction impressions in the first 60 seconds. The landing page IS the satisfaction statement.

---

## 8. UI/UX Differentiators to Win the Bid

These are features and design choices that go beyond "meeting requirements" and create competitive advantage:

### 8.1 High-Impact Differentiators

| Differentiator | Why it wins | Risk level |
|---|---|---|
| **AI-powered recommendations** | "Netflix for learning" -- evaluators see innovation, aligns with SVV's AI objective (#9) | Medium (must work flawlessly in demo) |
| **Visual learning path builder** | Drag-and-drop course sequencing -- dramatically better than list-based competitors | Low |
| **Leader traffic light dashboard** | Instant "how's my team doing?" -- solves the Excel pain point they described | Low |
| **Mobile-first field worker mode** | Competitors often have weak mobile. SVV has 5,000+ field workers. This is relevant | Low |
| **Proactive certification management** | Auto-alerts, renewal workflows, expiry forecasting -- safety-critical for SVV | Low |
| **Norwegian-first design** | Not a translated English product. Terminology, date formats, cultural patterns are Norwegian | Low |
| **One-click PowerBI integration** | SVV uses PowerBI. Showing native integration is powerful | Medium |
| **Accessibility excellence** | Go beyond WCAG AA minimum. Show it as a first-class feature, not a checkbox | Low |
| **Gamification elements** | Badges, streaks, leaderboards (optional) for engagement -- "70/20/10" friendly | Medium (can seem frivolous if overdone) |

### 8.2 Differentiation Through Design Language

- **Design system**: Consistent, component-based UI that signals "enterprise-grade but human"
- **Micro-interactions**: Subtle feedback animations (button presses, state changes, progress updates)
- **Information density**: More data per screen than competitors without feeling cluttered
- **Empty states**: When there's no data, show helpful guidance (not blank screens)
- **Loading states**: Skeleton screens, not spinners. Feels faster and more polished

### 8.3 Things to Avoid (Common Competitor Mistakes)

- **English leaking through**: Any English string in the UI will be noticed negatively
- **Generic dashboard**: "Welcome to the LMS" without personalization signals laziness
- **Deep navigation**: If evaluators can't find a feature within 10 seconds, it doesn't exist
- **Slow performance**: Any visible loading delay during demo will be remembered
- **Dated UI**: Evaluators compare against consumer apps they use daily (Netflix, Vipps, Altinn)
- **Over-engineering the demo**: Don't show 50 features; show 15 features that work perfectly

---

## 9. Demo Environment Preparation

### 9.1 Technical Setup

**Primary setup**:
- Dedicated demo tenant with performance tuning
- Pre-warmed caches for all pages shown in demo
- Dedicated bandwidth or local hosting to eliminate network variability
- Two monitors: one for presentation, one for notes/fallback
- Screen resolution matching projector/screen in demo room (confirm with SVV)

**Fallback plan**:
- Pre-recorded video of each scenario (screen capture with narration)
- Static screenshots of key screens as PDF backup
- Mobile device with local demo accessible without internet
- Spare laptop with identical setup

### 9.2 Data Preparation

All demo data must be:
- **Realistic**: Norwegian names, SVV division structure, real course titles
- **Consistent**: User stories connect (Kari's courses show in Per's team view)
- **Pre-tested**: Every click path rehearsed. No surprises
- **Reset-able**: Can restore to initial state between sessions if needed

**Sample data matrix**:

| Name | Role | Division | State |
|---|---|---|---|
| Kari Nordmann | Road inspector | Drift og vedlikehold | 4/7 learning path, 2 mandatory due |
| Erik Hansen | External traffic controller | Mesta AS (external) | New registration |
| Per Olsen | Section head | Trafikant og kjoretoey | 25 reports, 3 yellow, 2 red |
| Lise Berg | Instructional designer | Fellesfunksjoner og HR | 12 published courses, 3 drafts |
| Anders Haugen | Division HR admin | Utbygging | Full report access |

### 9.3 Rehearsal Plan

| Rehearsal | When | Focus |
|---|---|---|
| Technical rehearsal 1 | 2 weeks before (week of 2026-04-27) | All scenarios work end-to-end, data loaded |
| Technical rehearsal 2 | 1 week before (week of 2026-05-04) | Timing rehearsal, each scenario within time budget |
| Dress rehearsal | Day before (2026-05-10) | Full 80-minute run-through with internal audience |
| Morning check | Demo day (2026-05-11/12) | Connectivity, data state, screen resolution, audio |

---

## 10. Hands-On Testing Preparation

After the live POC demonstration, evaluators will test the demo environment themselves. This is separate from the scripted demonstration and harder to control.

### 10.1 Evaluator Testing Considerations

- **Evaluators will try unexpected paths**: Ensure all navigation leads somewhere sensible
- **They will test edge cases**: What happens with empty search? Wrong password? No courses?
- **They will check mobile**: Have the responsive version polished, not just functional
- **They will check Norwegian**: Every tooltip, every error message, every button label
- **They will test accessibility**: Expect keyboard-only navigation attempts

### 10.2 Environment Hardening

- **Error pages**: Custom 404 and error pages in Norwegian, with helpful navigation
- **Empty states**: All empty states show guidance text, not blank areas
- **Performance**: Pre-warm all pages. No cold starts
- **Data boundaries**: Evaluator accounts can explore freely but can't break demo data for other roles
- **Help content**: Contextual help available on every page (tooltip or sidebar)
- **Breadcrumbs**: Evaluators should always know where they are in the application

### 10.3 Evaluator Account Setup

Provide 3-5 test accounts with different roles:
- Generic internal employee account
- Generic leader account
- Generic admin account
- All with pre-loaded, rich data that makes the system feel alive

---

## 11. Risk Assessment: What Could Go Wrong

| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| **Network failure during demo** | Critical | Low | Local/cached demo environment, pre-recorded video backup |
| **SCORM upload fails** | High | Medium | Pre-test exact SCORM file day before. Have backup SCORM ready |
| **ID-porten test env down** | High | Medium | Pre-recorded ID-porten flow, or mock the redirect |
| **Slow page loads** | High | Medium | Pre-warm caches, dedicated bandwidth, reduce demo data volume if needed |
| **English strings visible** | Medium | High | Systematic language audit 1 week before. Screenshot every page |
| **Evaluator finds bug during hands-on** | High | Medium | Thorough testing of all paths, not just demo script. Have support engineer on standby |
| **Time overrun on scenarios** | High | High | Strict time discipline. Practice with stopwatch. Have planned cuts for each scenario |
| **Evaluator asks about missing feature** | Medium | High | Prepare "planned for implementation" responses with timeline. Never say "no" |
| **Accessibility failure** | High | Medium | Automated + manual testing pre-demo. Test with actual screen readers |
| **Wrong data / inconsistent state** | Medium | Medium | Data validation script run morning of demo. Frozen demo environment |
| **Projector/screen resolution mismatch** | Medium | Medium | Ask SVV for room specs in advance. Test on external monitor beforehand |
| **Demo persona confusion** | Low | Low | Clear printed script with role names, character names, login credentials |

### 11.1 Recovery Strategies

- **The 10-second rule**: If something breaks, acknowledge it, switch to the next topic, circle back. Never debug live.
- **The backup browser**: Second browser window with each scenario pre-loaded at a key point.
- **The narration pivot**: If a feature doesn't load, narrate what it would show while navigating elsewhere. "As you can see from the architecture, this loads the compliance dashboard which shows..."
- **The honest moment**: If something genuinely fails, say "This is a test environment -- in production this is handled by [X]. Let me show you [alternative path]." Evaluators respect honesty over cover-ups.

---

## 12. ISO 9241-11 Scoring Optimization Checklist

Use this checklist to verify all three ISO dimensions are addressed in every scenario:

### Per-Scenario Verification

For each of the 5 scenarios, confirm:

**Efficacy**:
- [ ] User achieves the stated goal completely
- [ ] Result is accurate and correct
- [ ] No dead ends or incomplete workflows
- [ ] Success is clearly confirmed to the user

**Efficiency**:
- [ ] Task completed in minimal steps
- [ ] No unnecessary data entry or clicks
- [ ] Smart defaults reduce decision fatigue
- [ ] Power user shortcuts available (but not required)

**Satisfaction**:
- [ ] Interface is visually appealing and modern
- [ ] Interaction feels responsive and smooth
- [ ] Language is natural Norwegian (not translated)
- [ ] User feels in control at all times
- [ ] The system feels trustworthy and professional

---

## 13. Implementation Priorities for Development

Based on the UX strategy, these are the implementation priorities ordered by impact on POC scoring:

### Priority 1: Must-Have for POC (Weeks 1-3)

1. Personalized landing page with role-based content
2. Search with faceted filtering (Norwegian, fast, tolerant)
3. Learning path visualization with progress tracking
4. Leader dashboard with traffic light compliance view
5. SCORM upload with drag-and-drop and preview
6. Basic reporting with pre-built templates and Excel export
7. Norwegian language throughout -- zero English strings
8. Responsive design verified on mobile, tablet, desktop
9. WCAG 2.1 AA compliance on all demo pages
10. Light/dark/system theme support

### Priority 2: Should-Have for POC (Weeks 3-5)

11. AI-powered course recommendations
12. Visual learning path builder (drag-and-drop)
13. Notification system (in-app + email)
14. Certificate generation and wallet
15. PowerBI data export integration
16. Custom report builder
17. Bulk course assignment for leaders
18. Course lifecycle management (draft/review/publish/archive)

### Priority 3: Nice-to-Have for POC (Week 5-6)

19. Onboarding wizard for external users
20. ID-porten integration in demo environment
21. Scheduled reports
22. Division comparison analytics
23. Delegation/deputy approval workflow
24. Contextual help and guided tours

---

*Document prepared by ArchitectUX*
*Date: 2026-03-22*
*Tender: Statens vegvesen LMS (25/223727)*
*Next: Handoff to development team for POC implementation*
