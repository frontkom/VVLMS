# Product UX Design -- LMS for Norwegian Public Sector

## Build Plan Document 02 | ArchitectUX

---

## 1. Design System Approach

### 1.1 Foundation: Designsystemet (Norwegian Government Design System)

The platform builds on Norway's official government design system (Designsystemet, formerly Altinn Design System) as the accessibility and compliance foundation, extended with a custom component layer for LMS-specific interactions.

**Rationale**: Norwegian public sector evaluators expect interfaces that feel consistent with government digital services (Altinn, Nav.no, Helsenorge). Using Designsystemet tokens and patterns signals compliance maturity and reduces cognitive friction for government users.

**Architecture**:

```
Layer 1: Designsystemet Tokens (colors, spacing, typography, shadows)
Layer 2: Radix UI Primitives (accessible headless components)
Layer 3: Custom LMS Components (course player, learning path, dashboards)
Layer 4: Theme Layer (light/dark/system + high-contrast)
```

### 1.2 CSS Design System Variables

```css
:root {
  /* ---- Norwegian Government Base Colors ---- */
  /* Derived from Designsystemet blue palette */
  --color-brand-1: #0062BA;       /* Primary action blue */
  --color-brand-2: #004A8E;       /* Primary hover */
  --color-brand-3: #00315C;       /* Primary active/pressed */

  /* Semantic Surface Colors -- Light Theme */
  --bg-default: #FEFEFE;
  --bg-subtle: #F4F5F6;
  --bg-surface: #FFFFFF;
  --bg-surface-hover: #E9EAEC;
  --bg-surface-active: #DBDDE0;
  --bg-accent: #E6F0FA;
  --bg-accent-hover: #CCE1F5;

  /* Text Colors */
  --text-default: #1E2B3C;
  --text-subtle: #4B5563;
  --text-on-action: #FFFFFF;
  --text-on-inverted: #F4F5F6;
  --text-danger: #BA1A1A;
  --text-success: #166534;
  --text-warning: #92400E;

  /* Border Colors */
  --border-default: #D1D5DB;
  --border-subtle: #E5E7EB;
  --border-strong: #6B7280;
  --border-focus: #0062BA;
  --border-danger: #DC2626;

  /* Status / Traffic Light Colors (leader dashboard) */
  --status-green: #16A34A;
  --status-green-bg: #DCFCE7;
  --status-yellow: #CA8A04;
  --status-yellow-bg: #FEF9C3;
  --status-red: #DC2626;
  --status-red-bg: #FEE2E2;
  --status-neutral: #6B7280;
  --status-neutral-bg: #F3F4F6;

  /* ---- Typography Scale ---- */
  --font-family-body: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  --font-family-heading: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  --font-family-mono: 'JetBrains Mono', 'Fira Code', monospace;

  --text-xs: 0.75rem;      /* 12px */
  --text-sm: 0.875rem;     /* 14px */
  --text-base: 1rem;       /* 16px */
  --text-lg: 1.125rem;     /* 18px */
  --text-xl: 1.25rem;      /* 20px */
  --text-2xl: 1.5rem;      /* 24px */
  --text-3xl: 1.875rem;    /* 30px */
  --text-4xl: 2.25rem;     /* 36px */

  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.75;

  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;

  /* ---- Spacing System (4px grid) ---- */
  --space-0: 0;
  --space-1: 0.25rem;    /* 4px */
  --space-2: 0.5rem;     /* 8px */
  --space-3: 0.75rem;    /* 12px */
  --space-4: 1rem;       /* 16px */
  --space-5: 1.25rem;    /* 20px */
  --space-6: 1.5rem;     /* 24px */
  --space-8: 2rem;       /* 32px */
  --space-10: 2.5rem;    /* 40px */
  --space-12: 3rem;      /* 48px */
  --space-16: 4rem;      /* 64px */
  --space-20: 5rem;      /* 80px */
  --space-24: 6rem;      /* 96px */

  /* ---- Layout ---- */
  --container-sm: 640px;
  --container-md: 768px;
  --container-lg: 1024px;
  --container-xl: 1280px;
  --container-2xl: 1440px;

  --sidebar-width: 260px;
  --sidebar-collapsed: 64px;
  --header-height: 64px;
  --bottom-nav-height: 56px;

  /* ---- Radius ---- */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;

  /* ---- Shadows ---- */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-md: 0 4px 6px -1px rgba(0,0,0,0.07), 0 2px 4px -2px rgba(0,0,0,0.05);
  --shadow-lg: 0 10px 15px -3px rgba(0,0,0,0.08), 0 4px 6px -4px rgba(0,0,0,0.04);
  --shadow-focus: 0 0 0 3px rgba(0,98,186,0.3);

  /* ---- Transitions ---- */
  --transition-fast: 100ms ease;
  --transition-base: 200ms ease;
  --transition-slow: 300ms ease;
}

/* ---- Dark Theme ---- */
[data-theme="dark"] {
  --bg-default: #111827;
  --bg-subtle: #1F2937;
  --bg-surface: #1F2937;
  --bg-surface-hover: #374151;
  --bg-surface-active: #4B5563;
  --bg-accent: #1E3A5F;
  --bg-accent-hover: #254B7A;

  --text-default: #F3F4F6;
  --text-subtle: #9CA3AF;
  --text-on-action: #FFFFFF;
  --text-danger: #FCA5A5;
  --text-success: #86EFAC;
  --text-warning: #FCD34D;

  --border-default: #374151;
  --border-subtle: #2D3748;
  --border-strong: #9CA3AF;

  --status-green-bg: #14532D;
  --status-yellow-bg: #713F12;
  --status-red-bg: #7F1D1D;
  --status-neutral-bg: #374151;

  --shadow-sm: 0 1px 2px rgba(0,0,0,0.3);
  --shadow-md: 0 4px 6px -1px rgba(0,0,0,0.4);
  --shadow-lg: 0 10px 15px -3px rgba(0,0,0,0.5);
}

/* ---- System Theme Preference ---- */
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --bg-default: #111827;
    --bg-subtle: #1F2937;
    --bg-surface: #1F2937;
    --bg-surface-hover: #374151;
    --bg-surface-active: #4B5563;
    --bg-accent: #1E3A5F;
    --bg-accent-hover: #254B7A;

    --text-default: #F3F4F6;
    --text-subtle: #9CA3AF;
    --text-danger: #FCA5A5;
    --text-success: #86EFAC;
    --text-warning: #FCD34D;

    --border-default: #374151;
    --border-subtle: #2D3748;
    --border-strong: #9CA3AF;

    --status-green-bg: #14532D;
    --status-yellow-bg: #713F12;
    --status-red-bg: #7F1D1D;
    --status-neutral-bg: #374151;
  }
}

/* ---- High Contrast Mode (WCAG) ---- */
@media (forced-colors: active) {
  :root {
    --border-focus: Highlight;
    --text-default: CanvasText;
    --bg-default: Canvas;
    --color-brand-1: LinkText;
  }
}
```

### 1.3 Component Library Strategy

**Build on Radix UI primitives + custom styling** (not shadcn/ui directly, since we need Designsystemet token alignment).

| Layer | Technology | Purpose |
|---|---|---|
| Primitives | Radix UI | Accessible dialog, dropdown, tabs, tooltip, popover, accordion |
| Forms | React Hook Form + Zod | Validation, Norwegian error messages |
| Tables | TanStack Table | Sortable, filterable data tables for admin/leader views |
| Charts | Recharts | Dashboard visualizations, progress charts |
| Date/Time | date-fns with `nb` locale | Norwegian date formatting throughout |
| Icons | Lucide React | Consistent, accessible icon set |
| Drag & Drop | dnd-kit | Learning path builder, dashboard customization |
| SCORM Player | Custom wrapper | SCORM 1.2/2004 runtime API |

### 1.4 Iconography

Lucide icon set with consistent 24px stroke-width icons. Supplemented with custom icons for LMS-specific concepts:

| Concept | Icon | Usage |
|---|---|---|
| Kurs (Course) | `book-open` | Course cards, navigation |
| Laeringssti (Learning Path) | `route` | Path visualization |
| Sertifikat (Certificate) | `award` | Certificate wallet, expiry alerts |
| Obligatorisk (Mandatory) | `shield-alert` | Mandatory course badge |
| Valgfritt (Optional) | `bookmark` | Optional course badge |
| Fullfort (Completed) | `check-circle` | Completion status |
| Pagar (In Progress) | `clock` | Progress status |
| Ikke startet (Not Started) | `circle` | Not-started status |
| Leder (Leader) | `users` | Leader dashboard |
| Rapport (Report) | `bar-chart-3` | Analytics/reports |
| Varsling (Notification) | `bell` | Notification bell |
| Innstillinger (Settings) | `settings` | Configuration |

### 1.5 Norwegian-First Design Language

**Language handling**:
- Default `lang="nb"` (Bokmal). User can switch to `lang="nn"` (Nynorsk)
- All UI strings in Norwegian, managed via i18n with `nb` and `nn` locale files
- No English fallbacks visible to users. Administrative/developer screens may use English for technical terms only where Norwegian equivalents do not exist
- Date format: `dd.MM.yyyy` (e.g., `22.03.2026`)
- Time format: `HH:mm` (24-hour, e.g., `14:30`)
- Number format: `1 234,56` (space as thousands separator, comma as decimal)
- Currency: `kr 1 234,56`

**Terminology alignment**:

| English | Norwegian (Bokmal) | Context |
|---|---|---|
| Dashboard | Oversikt / Startside | Learner landing |
| Course | Kurs | All contexts |
| Learning Path | Laeringssti | Path visualization |
| Certificate | Kompetansebevis / Sertifikat | Certificate wallet |
| Assignment | Tildeling | Leader assigning courses |
| Approval | Godkjenning | Leader approving activities |
| Report | Rapport | Analytics |
| Mandatory | Obligatorisk | Course flag |
| Optional | Valgfritt | Course flag |
| Expired | Utlopt | Certificate status |
| My Learning | Min laering | Navigation label |
| My Team | Mitt team | Leader navigation |
| Search | Sok | Search bar |
| Notifications | Varsler | Notification center |
| Settings | Innstillinger | User settings |
| Profile | Min profil | User profile |
| Content Library | Innholdsbibliotek | Content manager area |

---

## 2. User Flows for Each Role

### 2.1 Learner (Internal Employee) -- ~5,300 users

**Persona**: Kari Nordmann, 42, road inspector in Drift og vedlikehold, Bergen. Uses a tablet in the field and a desktop in the office. Moderate digital skills. Needs to keep safety certifications current.

#### Flow 1: Daily Login and Dashboard

```
1. Opens browser / PWA on tablet
2. Automatic SSO via ForgeRock (no password prompt)
3. Lands on personalized "Startside" (dashboard)
   |
   +-- Top banner: "Velkommen tilbake, Kari" with greeting
   |   (Shows time-appropriate greeting: God morgen/God ettermiddag)
   |
   +-- Alert section (if any):
   |   [!] "2 obligatoriske kurs forfaller innen 30 dager"
   |   One-click: "Ga til kurs" button on each alert
   |
   +-- "Fortsett der du slapp" section:
   |   Card: "HMS Grunnkurs 2026" -- 65% fullfort -- [Fortsett]-button
   |   Card: "Arbeidsvarsling nivaa 2" -- 40% fullfort -- [Fortsett]-button
   |
   +-- "Anbefalt for deg" section:
   |   3 cards based on role + history + competency gaps
   |   Each card: thumbnail, title, type badge, duration, [Se mer]
   |
   +-- "Min laeringssti" section:
   |   Visual progress: "Sikkerhet for inspektorer" -- 4/7 moduler fullfort
   |   Next module highlighted with arrow
   |
   +-- "Nyheter og oppdateringer" section:
       Recent announcements, new courses relevant to role
```

#### Flow 2: Course Discovery and Enrollment

```
1. Clicks "Sok" in top navigation (or Cmd+K shortcut)
2. Types "arbeidsvarsling"
   |
   +-- Instant results as user types (debounced 200ms)
   +-- Results grouped: Kurs (3), Laeringssti (1), Ressurser (2)
   +-- Each result: title, type icon, duration, format badge
   |
3. Clicks faceted filters panel:
   |   Kurstype: [E-laering] [Klasserom] [Webinar] [Blandet]
   |   Varighet: [< 30 min] [30-60 min] [1-3 timer] [> 3 timer]
   |   Status: [Obligatorisk] [Valgfritt] [Fullfort] [Ikke startet]
   |   Fagomrade: [Dropdown with ServiceNow categories]
   |
4. Clicks on course card -> Course detail page:
   |
   +-- Hero: Course image/thumbnail, title, short description
   +-- Metadata bar: Type, Duration, Format, Difficulty, Language
   +-- Tabs: [Oversikt] [Innhold] [Vurderinger] [Relatert]
   |   Oversikt: Full description, learning objectives, prerequisites
   |   Innhold: Module list with durations
   |   Vurderinger: Star rating + comments from other learners
   |   Relatert: Suggested follow-up courses
   +-- Sidebar: [Meld deg pa]-button (primary action)
   |   Prerequisite check: "Krever: HMS Grunnkurs (fullfort)"
   |   Validity: "Gyldig i 2 ar etter fullfort"
   |
5. Clicks [Meld deg pa]
   |
   +-- Confirmation toast: "Du er pameldt. Kurset er lagt til i Min laering."
   +-- Option: "Start na" or "Legg til i kalender"
```

#### Flow 3: Course Player (SCORM)

```
1. Opens course from dashboard or "Min laering"
2. Course player loads in full-width view:
   |
   +-- Top bar: Course title, progress indicator (e.g., "Modul 3 av 7")
   +-- Close button (X) with "Fremgangen din er lagret"
   +-- SCORM content iframe (full remaining viewport)
   +-- Bottom bar (optional): Previous | Next module navigation
   |
3. SCORM runtime API handles:
   |   - cmi.core.lesson_status
   |   - cmi.core.score
   |   - cmi.suspend_data (resume position)
   |   - cmi.core.session_time
   |
4. On module completion:
   |   +-- Slide-in panel: "Modul fullfort!" with checkmark animation
   |   +-- Auto-advance to next module after 3 seconds
   |   +-- Or click "Neste modul" immediately
   |
5. On course completion:
   |   +-- Full-screen completion view:
   |   +-- "Gratulerer, Kari!" with completion badge
   |   +-- Score display (if applicable)
   |   +-- [Last ned kompetansebevis] button (PDF)
   |   +-- [Tilbake til oversikt] button
   |   +-- Certificate automatically added to digital wallet
```

#### Flow 4: Certificate Wallet and Progress Tracking

```
1. Navigates to "Min laering" via sidebar/bottom nav
2. Tab view: [Aktive kurs] [Fullforte] [Kompetansebevis] [Laeringssti]
   |
   +-- Aktive kurs: Cards with progress bars, resume buttons
   +-- Fullforte: Completed courses with dates, scores, re-take option
   +-- Kompetansebevis:
   |   Grid of certificate cards:
   |   +-- Certificate title
   |   +-- Utstedt: 15.03.2025
   |   +-- Gyldig til: 15.03.2027
   |   +-- Status badge: [Gyldig] (green) / [Utloper snart] (yellow) / [Utlopt] (red)
   |   +-- Actions: [Last ned PDF] [Del] [Forny]
   |
   +-- Laeringssti:
       Visual path with nodes:
       [Fullfort] -> [Fullfort] -> [Pagar] -> [Last] -> [Last] -> [Last] -> [Avslutning]
       Click any node to see module details or start
```

---

### 2.2 External User -- ~16,100 users

**Persona**: Erik Hansen, 35, traffic controller for Mesta AS. Needs arbeidsvarsling certification. Uses a smartphone primarily. Visits the platform 1-2 times per year.

#### Flow 1: Registration via ID-porten

```
1. Receives link from employer or finds SVV competency portal via web search
2. Lands on clean registration/login page:
   |
   +-- SVV logo + "Kompetanseportalen"
   +-- Brief tagline: "Kurs og kompetansebevis for transportbransjen"
   +-- [Logg inn med ID-porten] (primary button, large)
   +-- "Har du allerede en konto? Du logges inn automatisk."
   |
3. Clicks [Logg inn med ID-porten]
   |
   +-- Redirect to ID-porten
   +-- Chooses BankID / MinID
   +-- Authenticates with personal eID
   +-- Redirect back to platform
   |
4. First-time user detection -> Onboarding wizard:
   |
   +-- Step 1/3: "Bekreft dine opplysninger"
   |   Pre-filled from Folkeregisteret: Name, date of birth
   |   Editable: Email, phone number
   |   [Neste]
   |
   +-- Step 2/3: "Din arbeidsgiver"
   |   Search field: Type company name
   |   Auto-complete from Bronnoysund register
   |   Or: "Jeg er selvstendig naeringsdrivende"
   |   [Neste]
   |
   +-- Step 3/3: "Kom i gang"
   |   Show 2-3 most relevant courses for their employer category
   |   "Du kan ogsa bla gjennom alle tilgjengelige kurs"
   |   [Ga til kurskatalogen]
   |
5. Lands on external user dashboard (simplified version)
```

#### Flow 2: Course Completion and Certificate

```
1. Dashboard shows:
   |
   +-- "Mine kurs" section: Active enrollments
   +-- "Mine kompetansebevis" section: Current + expiring certificates
   +-- "Tilgjengelige kurs" section: Relevant courses for their profile
   |
2. Enrolls in "Arbeidsvarsling grunnkurs"
   |
   +-- Course detail: Description, requirements, duration, exam info
   +-- [Start kurs] (if self-paced) or [Meld deg pa] (if scheduled)
   |
3. Completes course modules (same player as internal users)
4. Takes exam (if required):
   |
   +-- Exam instructions page with timer info
   +-- Quiz interface with clear question numbering
   +-- Progress: "Sporsmal 12 av 25"
   +-- Submit: "Er du sikker? Du kan ikke endre svarene etter innsending."
   |
5. Receives certificate:
   |
   +-- Result page: "Bestatt! Score: 88%"
   +-- Certificate preview (PDF/A format)
   +-- [Last ned kompetansebevis] (PDF download)
   +-- QR code on certificate for third-party verification
   +-- Employer notified automatically
   +-- Certificate stored in digital wallet
   |
6. Return visit experience:
   |
   +-- SSO via ID-porten (remembers device)
   +-- Dashboard immediately shows: active certificates, expiry dates
   +-- Proactive renewal alerts: 90/60/30 days before expiry
   +-- One-click re-enrollment for renewal courses
```

---

### 2.3 Leader -- ~400 users

**Persona**: Per Olsen, 50, section head in Trafikant og kjoretoey. 25 direct reports across 3 locations. Checks the platform once a week, usually Monday mornings.

#### Flow 1: Team Compliance Dashboard

```
1. SSO login -> Lands on leader dashboard (role-based view)
   |
   +-- Top metrics bar:
   |   [23/25 i samsvar] [2 utlopte sertifikater] [3 godkjenninger venter]
   |
   +-- "Teamoversikt" widget (traffic light grid):
   |   +----------------------------------------------+
   |   | Medarbeider        | Status | Forfaller snart |
   |   |---------------------|--------|-----------------|
   |   | Ola Hansen          | GREEN  |                 |
   |   | Lise Berge          | GREEN  |                 |
   |   | Tor Eriksen         | YELLOW | HMS: 15.04.2026 |
   |   | Marte Johansen      | RED    | Arbv: utlopt    |
   |   | ...                 | GREEN  |                 |
   |   +----------------------------------------------+
   |   Filter: [Alle] [Gronn] [Gul] [Rod]
   |   Sort: [Navn] [Status] [Forfallsdato]
   |
   +-- "Kommende frister" widget:
   |   Timeline: Next 90 days of certification expiries
   |   Sorted by urgency (red first)
   |
   +-- "Ventende godkjenninger" widget:
       List of activities awaiting approval
       Each with [Godkjenn] [Avvis] quick actions
```

#### Flow 2: Course Assignment

```
1. Clicks [Tildel kurs] from dashboard or navigates to Tildeling
2. Assignment wizard:
   |
   +-- Step 1: "Velg kurs"
   |   Search/browse course catalog
   |   Select: "HMS Grunnkurs 2026"
   |   Course details shown inline
   |
   +-- Step 2: "Velg medarbeidere"
   |   Checkboxes: Individual employees
   |   Or: Select by team/group
   |   Or: "Alle i min enhet" (all in my unit)
   |   Selected count shown: "5 medarbeidere valgt"
   |
   +-- Step 3: "Sett frist"
   |   Date picker: "Frist for fullforelse"
   |   Optional: Priority (Normal / Hoy / Kritisk)
   |   Optional: Message to employees
   |
   +-- Step 4: "Bekreft og send"
   |   Summary: Course name, 5 employees listed, deadline
   |   [Tildel kurs] (primary action)
   |
3. Confirmation:
   +-- "Kurs tildelt 5 medarbeidere. Varsler sendt."
   +-- Link: "Se tildelingshistorikk"
```

#### Flow 3: Activity Approval

```
1. Notification badge on dashboard: "3 godkjenninger venter"
2. Opens approval queue:
   |
   +-- List view:
   |   | Medarbeider     | Aktivitet                  | Dato       | [Handling]     |
   |   |-----------------|----------------------------|------------|----------------|
   |   | Ola Hansen      | Befaring E39 Svegatjorn    | 18.03.2026 | [Se] [Godkj.]  |
   |   | Lise Berge      | HMS Inspeksjon, bru 42-016 | 19.03.2026 | [Se] [Godkj.]  |
   |   | Tor Eriksen     | Vintervedlikehold praksis  | 20.03.2026 | [Se] [Godkj.]  |
   |
3. Clicks [Se] to review:
   |
   +-- Submission detail:
   |   Activity type, date, description
   |   Any attachments (photos, documents)
   |   Employee's self-assessment (if applicable)
   |
4. Actions:
   |   [Godkjenn] -> Toast: "Aktivitet godkjent. Medarbeider er varslet."
   |   [Avvis] -> Opens comment field: "Skriv en begrunnelse" -> [Send avslag]
   |   [Godkjenn alle] -> Batch approval for routine items
```

#### Flow 4: Delegation

```
1. Navigates to Innstillinger > Delegering
2. Sets deputy:
   |   "Velg stedfortreder": Search for colleague with leader role
   |   "Periode": From date -> To date
   |   "Tillatelser": [Godkjenne aktiviteter] [Tildele kurs] [Se rapporter]
   |   [Aktiver delegering]
3. Deputy receives notification and sees delegated team in their dashboard
```

---

### 2.4 Content Manager -- subset of ~400 admins

**Persona**: Lise Berg, 38, instructional designer in Fellesfunksjoner og HR. Creates and manages digital courses. Expert in Articulate Storyline. Manages 50+ courses.

#### Flow 1: Course Creation (SCORM Upload)

```
1. Navigates to "Innhold" section via admin sidebar
2. Dashboard shows:
   |
   +-- Stats bar: [52 publiserte] [3 utkast] [1 til gjennomgang] [148 aktive laerende]
   +-- Quick actions: [+ Nytt kurs] [+ Ny laeringssti] [Importer innhold]
   +-- Tabs: [Mine kurs] [Alle kurs] [Maler] [Arkivert]
   |
3. Clicks [+ Nytt kurs]
4. Course type selector:
   |
   +-- [SCORM-pakke]        Upload existing e-learning package
   +-- [Klasseromskurs]     Instructor-led with scheduling
   +-- [Webinar]            Virtual classroom session
   +-- [Innebygd editor]    Create simple content natively
   +-- [Blandet laering]    Combination of multiple types
   |
5. Selects [SCORM-pakke]
6. Upload interface:
   |
   +-- Drag-and-drop zone: "Dra SCORM-pakken hit, eller klikk for a velge fil"
   +-- Supported: .zip (SCORM 1.2, SCORM 2004, xAPI, cmi5)
   +-- File selected -> Automatic validation:
   |   [OK] imsmanifest.xml funnet
   |   [OK] SCORM 2004 4th Edition
   |   [OK] 7 SCO-er identifisert
   |   [OK] Totalt 45 MB
   +-- [Forhandsvis] button -> Opens SCORM in preview player
   |
7. Metadata form (smart defaults pre-populated from SCORM manifest):
   |
   +-- Grunnleggende:
   |   Tittel: [Auto-filled from manifest, editable]
   |   Beskrivelse: [Rich text editor]
   |   Kort beskrivelse: [Auto-generated, editable]
   |   Kursbilde: [Upload or select from library]
   |
   +-- Klassifisering:
   |   Fagomrade: [Dropdown - synced from ServiceNow categories]
   |   Underkategori: [Dependent dropdown]
   |   Stikkord: [Tag input with auto-suggest]
   |   Vanskelighetsnivaa: [Grunnleggende / Videregaende / Ekspert]
   |
   +-- Malgruppe og tilgang:
   |   Malgruppe: [Multi-select from org units / job roles]
   |   Synlighet: [Alle ansatte] [Valgte enheter] [Kun eksterne]
   |   Obligatorisk: [Toggle] -> If on: applies to selected target audience
   |
   +-- Varighet og gyldighet:
   |   Estimert varighet: [Auto from SCORM, editable]
   |   Gyldighetsperiode: [Ingen] [6 mnd] [1 ar] [2 ar] [3 ar] [Egendefinert]
   |   Fornyelseskurs: [Link to renewal course if exists]
   |
   +-- Fullforelseskriterier:
   |   [Alle moduler fullfort] [Bestatt eksamen] [Minimum score: ___%]
   |   Sertifikatmal: [Select certificate template]
   |
8. [Lagre som utkast] or [Send til gjennomgang]
```

#### Flow 2: Learning Path Builder

```
1. Clicks [+ Ny laeringssti]
2. Learning path editor opens:
   |
   +-- Left panel: Course library (search + filter)
   +-- Center: Visual path canvas
   +-- Right panel: Properties of selected node
   |
3. Canvas interactions:
   |
   +-- Drag course from library onto canvas -> Creates node
   +-- Connect nodes by dragging between them -> Creates dependency arrow
   +-- Node types:
   |   [Kurs] Standard course module
   |   [Vurdering] Assessment/exam checkpoint
   |   [Aktivitet] Practical activity (needs leader approval)
   |   [Ressurs] Supplementary material (optional)
   |   [Sertifikat] Certificate issuance point
   |
   +-- Example visual layout:
   |
   |   [HMS Intro] --> [Sikkerhet teori] --> [Praktisk ovelse]
   |                                              |
   |                                              v
   |                                        [Godkjenning] --> [Sertifikat]
   |
   +-- Branching: Parallel tracks for different specializations
   +-- Prerequisites: Minimum score gates between nodes
   |
4. Path properties (right panel):
   |   Tittel: "Sikkerhet for inspektorer"
   |   Beskrivelse: [Rich text]
   |   Estimert total varighet: [Auto-calculated from courses]
   |   Sertifikatmal: [Select template for path completion]
   |
5. [Forhandsvis som laerende] -> Shows learner view of the path
6. [Publiser laeringssti]
```

#### Flow 3: Course Lifecycle Management

```
1. From course list, status badges show lifecycle state:
   |
   |   [Utkast] -> [Til gjennomgang] -> [Publisert] -> [Oppdatert] -> [Arkivert]
   |     gray        yellow              green           blue           gray
   |
2. Update existing course:
   |   Opens course -> [Last opp ny versjon]
   |   Upload new SCORM package
   |   Version comparison: "v2.1 -> v2.2"
   |   Change log: [Required text field]
   |   Impact notice: "47 aktive laerende pavirkes. Eksisterende fullforeiser beholdes."
   |   [Publiser oppdatering]
   |
3. Archive course:
   |   [Arkiver kurs]
   |   Confirmation: "Kurset fjernes fra katalogen. Historiske data beholdes."
   |   Option: "Erstattet av:" [Select replacement course]
   |   Active learners notified and redirected
```

---

### 2.5 Admin (Division Administrator) -- subset of ~400 admins

**Persona**: Anders Haugen, 45, HR/competency administrator in Utbygging. Produces monthly compliance reports for division director. Comfortable with data and PowerBI.

#### Flow 1: Analytics Dashboard

```
1. SSO login -> Admin dashboard:
   |
   +-- Division KPI cards (top row):
   |   [92% samsvar] [1 240 fullforte kurs Q1] [3 450 laeringstimer] [98% fornoyelse]
   |   Each card clickable for drill-down
   |
   +-- Trendgraf (center):
   |   Line chart: Completion rates last 12 months by team
   |   Toggle: [Fullforelse %] [Timer] [Pameldte]
   |
   +-- Samsvarsstatus (left):
   |   Stacked bar chart by team
   |   Green (compliant) / Yellow (expiring) / Red (expired)
   |   Click bar segment -> drill to individual employees
   |
   +-- Kommende utlop (right):
   |   Timeline: Next 90 days
   |   Grouped by certification type
   |   Count badges on each group
```

#### Flow 2: Report Builder

```
1. Navigates to Rapporter > Ny rapport
2. Report builder interface:
   |
   +-- Left panel: Available fields (drag & drop)
   |   Kategorier:
   |   +-- Bruker (navn, enhet, rolle, divisjon)
   |   +-- Kurs (tittel, type, fagomrade, varighet)
   |   +-- Fullforelse (dato, score, status, tid brukt)
   |   +-- Sertifikat (type, utstedt, utloper, status)
   |
   +-- Center: Report canvas
   |   Drop zones: [Rader] [Kolonner] [Verdier] [Filter]
   |
   +-- Right panel: Visualization selector
   |   [Tabell] [Stolpediagram] [Linjediagram] [Kakediagram]
   |
3. Example: "Samsvarsrapport per team"
   |   Rader: Enhet > Medarbeider
   |   Kolonner: Sertifikattype
   |   Verdier: Status (traffic light)
   |   Filter: Divisjon = Utbygging, Periode = Q1 2026
   |
4. Actions:
   |   [Forhandsvis] -> Live preview of report
   |   [Lagre mal] -> Name and save for reuse
   |   [Eksporter] -> [Excel] [CSV] [PDF]
   |   [Planlegg] -> Schedule recurring delivery
   |       Frekvens: [Daglig] [Ukentlig] [Manedlig]
   |       Mottakere: [Select users/email addresses]
   |       Format: [Excel] [PDF]
```

#### Flow 3: Data Export and PowerBI

```
1. Navigates to Rapporter > Dataeksport
2. Export options:
   |
   +-- "Hurtigeksport" (one-click):
   |   [Fullforelsesdata - Excel]
   |   [Samsvarsrapport - Excel]
   |   [Sertifikatoversikt - Excel]
   |   Each generates formatted Excel with SVV header and metadata
   |
   +-- "PowerBI-integrasjon":
   |   Status: "Tilkoblet" (green indicator)
   |   Last sync: "22.03.2026 kl. 06:00"
   |   Data sources configured:
   |   +-- Fullforelsesdata (daglig oppdatering)
   |   +-- Brukerdata (daglig oppdatering)
   |   +-- Kursmetadata (ved endring)
   |   +-- Sertifikatdata (daglig oppdatering)
   |   [Konfigurer] -> API endpoint settings, authentication
   |   [Test tilkobling] -> Verifies PowerBI can reach data
   |
   +-- "Revisjonslogg":
       Filter by user, action, date range
       Shows: who changed what, when
       Exportable for compliance audits
```

---

### 2.6 Super Admin (Multi-tenant) -- system operator

**Persona**: System operator managing multiple government agency tenants.

#### Flow 1: Tenant Management

```
1. Super admin portal (separate URL: admin.platform.no)
2. Tenant list:
   |
   +-- Tenant cards:
   |   [Statens vegvesen] 5 300 users | Active | 2.1 TB storage
   |   [Tenant B]         2 100 users | Active | 0.8 TB storage
   |   [Tenant C]         800 users   | Trial  | 0.1 TB storage
   |
3. Click tenant -> Tenant detail:
   |
   +-- Overview: Usage metrics, storage, API calls, active users
   +-- Configuration: Branding, feature toggles, integration settings
   +-- Users: Admin user management for tenant
   +-- Billing: License usage, invoicing
   +-- Audit log: Tenant-level audit trail
   |
4. Global configuration:
   |   Platform version and update management
   |   Feature flag rollout (gradual per tenant)
   |   Security policies (password requirements, session timeouts)
   |   Notification templates
   |   System health and monitoring dashboard
```

---

## 3. Key Screen Concepts

### 3.1 Learner Dashboard (Startside)

```
+------------------------------------------------------------------+
| [=] SVV Laeringsplattformen           [Sok...]     [Bell] [Avatar]|
+--------+---------------------------------------------------------+
|        |                                                         |
| NAV    |  God morgen, Kari                                       |
|        |                                                         |
| Home * |  +-- ALERT BAR (red) --------------------------------+  |
| Min    |  | [!] 2 obligatoriske kurs forfaller innen 30 dager  |  |
| laering|  | HMS Grunnkurs (15.04.2026) | Arbv. (22.04.2026)   |  |
| Sok    |  +----------------------------------------------------+  |
| Varsler|                                                         |
| Profil |  FORTSETT DER DU SLAPP                                  |
|        |  +------------------+ +------------------+              |
|        |  | [img]            | | [img]            |              |
|        |  | HMS Grunnkurs    | | Arbeidsvarsling  |              |
|        |  | 2026             | | nivaa 2          |              |
|        |  | ===65%========   | | ==40%==========  |              |
|        |  | [Fortsett]       | | [Fortsett]       |              |
|        |  +------------------+ +------------------+              |
|        |                                                         |
|        |  ANBEFALT FOR DEG                                       |
|        |  +---------------+ +---------------+ +---------------+  |
|        |  | Brusikkerhet  | | Tunneldrift   | | Forstehjelp   |  |
|        |  | E-laering     | | Klasserom     | | E-laering     |  |
|        |  | 45 min        | | 1 dag         | | 30 min        |  |
|        |  +---------------+ +---------------+ +---------------+  |
|        |                                                         |
|        |  MIN LAERINGSSTI: Sikkerhet for inspektorer             |
|        |  [*]--[*]--[*]--[*]--[o]--[ ]--[ ]   4/7 fullfort     |
|        |                   ^current                              |
|        |  Neste: Praktisk ovelse i felt  [Start]                 |
|        |                                                         |
+--------+---------------------------------------------------------+
         | (* = fullfort, o = pagar, [ ] = ikke startet)           |
```

**Mobile version** (320px):
```
+-------------------------------+
| [=] SVV          [Bell] [Av] |
+-------------------------------+
| God morgen, Kari              |
|                               |
| [!] 2 obligatoriske kurs      |
|     forfaller snart  [Se]     |
|                               |
| FORTSETT                      |
| +---------------------------+ |
| | HMS Grunnkurs 2026        | |
| | ====65%====               | |
| | [Fortsett]                | |
| +---------------------------+ |
| +---------------------------+ |
| | Arbeidsvarsling nivaa 2   | |
| | ==40%==                   | |
| | [Fortsett]                | |
| +---------------------------+ |
|                               |
| ANBEFALT FOR DEG              |
| (horizontal scroll cards)     |
|                               |
+-------------------------------+
| [Home] [Laering] [Sok] [Mer] |
+-------------------------------+
```

### 3.2 Course Player (SCORM Wrapper)

```
+------------------------------------------------------------------+
| [<- Tilbake]    HMS Grunnkurs 2026    Modul 3/7   [65%]   [X]   |
+------------------------------------------------------------------+
|                                                                  |
|                                                                  |
|                                                                  |
|                    +------------------------------+              |
|                    |                              |              |
|                    |       SCORM CONTENT           |              |
|                    |       (iframe)                |              |
|                    |                              |              |
|                    |       Full viewport           |              |
|                    |       rendering               |              |
|                    |                              |              |
|                    |                              |              |
|                    +------------------------------+              |
|                                                                  |
|                                                                  |
+------------------------------------------------------------------+
| [<- Forrige modul]                         [Neste modul ->]      |
+------------------------------------------------------------------+
```

**Key behaviors**:
- Progress auto-saves every 30 seconds and on page unload
- SCORM API wrapper translates between SCORM 1.2/2004 and platform data model
- Fullscreen mode available (F11 or button)
- Exit button always visible with "progress saved" confirmation
- Offline: If PWA and course is downloaded, player works without connectivity
- Accessibility: SCORM content in iframe gets `title` attribute for screen readers

### 3.3 Learning Path Visualization

```
+------------------------------------------------------------------+
| Laeringssti: Sikkerhet for inspektorer                           |
| Estimert tid: 12 timer | 4 av 7 moduler fullfort | 57%          |
+------------------------------------------------------------------+
|                                                                  |
|  [1]--------[2]--------[3]--------[4]--------[5]--------[6]--[7]|
|  HMS        Risiko-     Personlig   Arbeids-   Praktisk   Eksamen |
|  Intro      vurdering   verneutstyr varsling   ovelse     Cert.  |
|  Fullfort   Fullfort    Fullfort    Pagar      Last       Last   |
|  45 min     1t 30m      30 min      2 timer    4 timer    1 time |
|  Score:92%  Score:88%   Score:95%   40%        -          -      |
|                                     ^                            |
|                                [Fortsett]                        |
|                                                                  |
+------------------------------------------------------------------+
| Detaljer for valgt modul:                                        |
| Arbeidsvarsling (Modul 4)                                        |
| Status: Pagar (40% fullfort)                                     |
| Sist apnet: 20.03.2026                                           |
| Forutsetning: Personlig verneutstyr (fullfort)                   |
| [Fortsett kurs]                                                  |
+------------------------------------------------------------------+
```

**Mobile**: Vertical timeline layout with nodes stacked vertically. Current module expanded, others collapsed.

### 3.4 Leader Competency Dashboard (Traffic Light)

```
+------------------------------------------------------------------+
| Mitt team: Trafikant og kjoretoey              Per Olsen, leder  |
+------------------------------------------------------------------+
| [23/25 samsvar]    [2 utlopte]    [3 godkjenninger]    [5 snart] |
|   92% GREEN          RED            YELLOW               YELLOW  |
+------------------------------------------------------------------+
|                                                                  |
| TEAMOVERSIKT                                    [Eksporter Excel]|
| +--------------------------------------------------------------+|
| | Navn              | Samsvar | Utloper snart   | Handling      ||
| |-------------------+---------+-----------------+---------------||
| | Ola Hansen        | [GRN]   |                 |               ||
| | Lise Berge        | [GRN]   | HMS: 15.05.2026 | [Tildel forny]||
| | Tor Eriksen       | [YLW]   | Arbv: 01.04.26  | [Tildel forny]||
| | Marte Johansen    | [RED]   | HMS: UTLOPT     | [Tildel na!]  ||
| | Knut Vik          | [GRN]   |                 |               ||
| | ...20 more        | [GRN]   |                 |               ||
| +--------------------------------------------------------------+|
|                                                                  |
| FRISTER NESTE 90 DAGER                                           |
| +--------------------------------------------------------------+|
| | Apr 2026                                                      ||
| | [!] 01.04 Tor Eriksen - Arbeidsvarsling utloper               ||
| | [!] 15.04 Kari Vik - HMS Grunnkurs utloper                    ||
| |                                                                ||
| | Mai 2026                                                       ||
| | [i] 15.05 Lise Berge - HMS forfaller                          ||
| | [i] 22.05 3 medarbeidere - Forstehjelp forfaller              ||
| +--------------------------------------------------------------+|
|                                                                  |
| VENTENDE GODKJENNINGER                            [Godkjenn alle]|
| +--------------------------------------------------------------+|
| | Ola Hansen - Befaring E39 (18.03.2026)  [Se] [Godkjenn]      ||
| | Lise Berge - HMS Inspeksjon (19.03.2026) [Se] [Godkjenn]     ||
| +--------------------------------------------------------------+|
+------------------------------------------------------------------+
```

### 3.5 Content Creation Wizard

```
+------------------------------------------------------------------+
| Nytt kurs                                          [Lagre utkast]|
+---+--------------------------------------------------------------+
|   |                                                              |
| 1 | TRINN 1: Kurstype                                            |
| * |                                                              |
| 2 | +------------------+ +------------------+ +----------------+ |
| . | | [icon]           | | [icon]           | | [icon]         | |
| 3 | | SCORM-pakke      | | Klasseromskurs   | | Webinar        | |
| . | | Last opp en      | | Instruktorledet  | | Virtuelt       | |
| 4 | | ferdig pakke      | | med timeplan     | | klasserom      | |
| . | +------ VALGT -----+ +------------------+ +----------------+ |
| 5 |                                                              |
|   | +------------------+ +------------------+                    |
|   | | [icon]           | | [icon]           |                    |
|   | | Innebygd editor  | | Blandet laering  |                    |
|   | | Enkel innholds-  | | Kombiner flere   |                    |
|   | | oppbygging       | | typer             |                    |
|   | +------------------+ +------------------+                    |
|   |                                                              |
|   | [Neste ->]                                                    |
+---+--------------------------------------------------------------+

Step 2: Upload
+---+--------------------------------------------------------------+
|   |                                                              |
| 1 | TRINN 2: Last opp SCORM-pakke                                |
| . |                                                              |
| 2 | +--------------------------------------------------------+  |
| * | |                                                        |  |
| 3 | |   Dra SCORM-pakken hit                                 |  |
| . | |   eller klikk for a velge fil                          |  |
| 4 | |                                                        |  |
| . | |   Stotter: SCORM 1.2, SCORM 2004, xAPI, cmi5          |  |
| 5 | |   Maks filstorrelse: 500 MB                            |  |
|   | +--------------------------------------------------------+  |
|   |                                                              |
|   | Validering:                                                  |
|   | [OK] imsmanifest.xml funnet                                  |
|   | [OK] SCORM 2004 4th Edition                                  |
|   | [OK] 7 SCO-er identifisert                                   |
|   | [OK] Totalt 45 MB                                             |
|   |                                                              |
|   | [<- Forrige]    [Forhandsvis]    [Neste ->]                   |
+---+--------------------------------------------------------------+
```

### 3.6 Admin Analytics Dashboard

```
+------------------------------------------------------------------+
| Analyse: Utbygging                    Periode: [Q1 2026 v]       |
+------------------------------------------------------------------+
|                                                                  |
| +-------------+ +-------------+ +-------------+ +-------------+ |
| | 92%         | | 1 240       | | 3 450       | | 4.2/5       | |
| | Samsvar     | | Fullforte   | | Laerings-   | | Tilfredshet | |
| | +2% fra Q4  | | kurs        | | timer       | |             | |
| +-------------+ +-------------+ +-------------+ +-------------+ |
|                                                                  |
| +-------------------------------+ +----------------------------+ |
| | FULLFORELSESRATE OVER TID     | | SAMSVAR PER TEAM           | |
| |                               | |                            | |
| |  100%|        ___----"""      | | Team A  [========90%==]    | |
| |   80%|   __--"               | | Team B  [=======85%=]     | |
| |   60%| _-"                    | | Team C  [==========97%]   | |
| |   40%|                        | | Team D  [====72%===]      | |
| |      +---+---+---+---+--     | |                            | |
| |       Nov Des Jan Feb Mar     | | [Gronn: >90%] [Gul: 70-90]| |
| +-------------------------------+ +----------------------------+ |
|                                                                  |
| +--------------------------------------------------------------+ |
| | DETALJERT RAPPORT                        [Eksporter] [Filter] | |
| |------------------------------------------------------------| |
| | Enhet      | Ansatte | Fullfort | Pagar | Forsinket | Rate  | |
| |------------|---------|----------|-------|-----------|-------| |
| | Team A     | 45      | 210      | 15    | 3         | 90%   | |
| | Team B     | 32      | 148      | 22    | 8         | 85%   | |
| | Team C     | 28      | 165      | 4     | 1         | 97%   | |
| | Team D     | 55      | 198      | 38    | 22        | 72%   | |
| |            |         |          |       |           |       | |
| | TOTALT     | 160     | 721      | 79    | 34        | 88%   | |
| +--------------------------------------------------------------+ |
+------------------------------------------------------------------+
```

---

## 4. Responsive / Mobile Strategy

### 4.1 Mobile-First Breakpoints

| Breakpoint | Target Device | Layout Strategy |
|---|---|---|
| 320-479px | Mobile portrait | Single column, bottom tab nav, stacked cards |
| 480-767px | Mobile landscape / small tablet | Single column, larger touch targets |
| 768-1023px | Tablet | Two-column where appropriate, collapsible sidebar |
| 1024-1279px | Desktop | Full sidebar + main content area |
| 1280px+ | Large desktop | Max-width container, expanded dashboard widgets |

### 4.2 Mobile Navigation Pattern

Bottom tab bar for primary actions (matches native app patterns Norwegian users know from Vipps, Ruter, yr.no):

```
+-------------------------------+
|                               |
|        [Page Content]         |
|                               |
+-------------------------------+
| [Hjem] [Laering] [Sok] [Mer] |
+-------------------------------+
```

- **Hjem**: Dashboard / Startside
- **Laering**: Min laering (courses, paths, certificates)
- **Sok**: Search with filters
- **Mer**: Notifications, profile, settings, help

Leader/admin roles get additional tab: **[Team]** or **[Admin]**.

### 4.3 Offline Considerations for Field Workers

SVV has road inspectors, maintenance crews, and tunnel workers who often lack connectivity.

**PWA Strategy**:
- Service worker caches critical app shell and navigation
- Downloaded courses stored in IndexedDB for offline playback
- SCORM progress queued locally and synced when connectivity returns
- Offline indicator banner: "Du er frakoblet. Fremgangen lagres lokalt."
- Course download UI: "Last ned for bruk uten nett" button on course detail
- Maximum offline course size: 200 MB per course (configurable)

**Sync behavior**:
- Auto-sync when connectivity detected
- Conflict resolution: last-write-wins for progress data
- Notification when sync completes: "Fremgangen din er synkronisert"

### 4.4 Responsive Component Behavior

| Component | Mobile (< 768px) | Tablet (768-1023px) | Desktop (1024px+) |
|---|---|---|---|
| Navigation | Bottom tab bar | Collapsed sidebar (icon-only) | Full sidebar with labels |
| Dashboard widgets | Stacked vertically, full-width | 2-column grid | 3-4 column grid |
| Course catalog | List view, horizontal scroll filters | Grid (2 columns), side panel filters | Grid (3-4 columns), sidebar filters |
| Learning path | Vertical timeline | Vertical timeline with detail panel | Horizontal timeline |
| Data tables | Card view (one card per row) | Condensed table, horizontal scroll | Full table with sort/filter |
| Course player | Full screen, no chrome | Full screen with minimal top bar | Embedded with sidebar navigation |
| Forms | Full-width inputs, stacked | Two-column layout where logical | Multi-column with sidebar help |
| Modals | Full screen sheets (bottom-up) | Centered modal (480px max) | Centered modal (560px max) |
| Search | Full screen overlay | Inline with dropdown results | Inline with dropdown results |
| Charts | Single chart, swipe between | Side-by-side (2) | Dashboard grid layout |

---

## 5. WCAG 2.1 AA Compliance

### 5.1 Legal Requirement

Norway mandates WCAG 2.1 AA compliance through:
- **Likestillings- og diskrimineringsloven, sections 17-18**: Universal design of ICT
- **Forskrift om universell utforming av IKT-losninger**: Explicitly requires WCAG 2.1 AA
- **EU Web Accessibility Directive** (transposed to Norwegian law)

Non-compliance carries legal risk and would fail the tender evaluation. Accessibility is not optional -- it is a hard requirement scored under quality criteria.

### 5.2 Implementation by WCAG Principle

#### Perceivable

| Guideline | Implementation |
|---|---|
| 1.1 Text alternatives | All images, icons, and charts have descriptive `alt` text in Norwegian. Decorative images use `alt=""`. Chart data available as accessible table. |
| 1.2 Time-based media | All video content has Norwegian captions (WebVTT). Audio content has transcripts. Live webinars have real-time captioning option. |
| 1.3 Adaptable | Semantic HTML5 throughout: proper `<h1>`-`<h6>` hierarchy, `<nav>`, `<main>`, `<aside>`, `<section>`. ARIA landmarks for dynamic content. Logical reading order independent of visual layout. |
| 1.4 Distinguishable | Minimum 4.5:1 contrast ratio for text. 3:1 for large text and UI components. Text resizable to 200% without loss of content. No information conveyed by color alone (traffic lights use icons + labels). |

#### Operable

| Guideline | Implementation |
|---|---|
| 2.1 Keyboard accessible | All functionality accessible via keyboard. Logical tab order. No keyboard traps. Skip-to-content link on every page. Focus management in modals (trap focus, return focus on close). |
| 2.2 Enough time | Session timeout: 30-minute warning with option to extend. Exam timers: adjustable with documented request process. Auto-play content: user-initiated only. |
| 2.3 Seizures | No content flashes > 3 times/second. All animations respect `prefers-reduced-motion`. Video content reviewed for seizure-inducing patterns. |
| 2.4 Navigable | Skip links on every page. Descriptive page titles in Norwegian. Breadcrumb navigation. Visible focus indicator (3px blue outline, `var(--shadow-focus)`). Consistent heading hierarchy. |
| 2.5 Input modalities | Touch targets minimum 44x44px. No complex gestures required (pinch/drag alternatives provided). Motion-based inputs have non-motion alternatives. |

#### Understandable

| Guideline | Implementation |
|---|---|
| 3.1 Readable | `lang="nb"` on HTML root. Language changes within content marked with `lang` attribute. Reading level appropriate for target audience. |
| 3.2 Predictable | Consistent navigation across all pages. No unexpected context changes on focus or input. Form submission requires explicit user action. |
| 3.3 Input assistance | Required fields clearly marked. Error messages identify the field and suggest correction (in Norwegian). Input validation on blur, not just on submit. Error summary at top of form with anchor links to fields. |

#### Robust

| Guideline | Implementation |
|---|---|
| 4.1 Compatible | Valid HTML5. ARIA roles, states, and properties for custom components (Radix UI handles this). Tested with NVDA (Windows), VoiceOver (macOS/iOS), TalkBack (Android). Regular accessibility audits with axe-core in CI/CD pipeline. |

### 5.3 Accessibility-Specific UI Patterns

**Traffic light without color dependency**:
```
Instead of just colored dots:
  [GREEN dot] Ola Hansen     -> [checkmark icon] Ola Hansen - "I samsvar"
  [YELLOW dot] Tor Eriksen   -> [warning icon] Tor Eriksen - "Utloper snart"
  [RED dot] Marte Johansen   -> [alert icon] Marte Johansen - "Utlopt"
```

**Focus indicator**: 3px solid blue outline with 2px offset, visible on all interactive elements. Custom `:focus-visible` styling that does not appear on mouse click but does on keyboard navigation.

**Screen reader announcements**: Dynamic content changes (toast notifications, status updates, loading states) announced via `aria-live` regions. SCORM player progress communicated via `aria-valuenow` on progress bar.

### 5.4 Testing Strategy

| Method | Tool | When |
|---|---|---|
| Automated scanning | axe-core (via @axe-core/react) | Every component render in development + CI |
| Lighthouse audit | Lighthouse CI | Every PR, must score > 95 accessibility |
| Manual keyboard testing | Human tester | Every sprint, full flow walkthrough |
| Screen reader testing | NVDA, VoiceOver | Every sprint on critical flows |
| Color contrast | Colour Contrast Analyser | During design review |
| Zoom testing | Browser zoom to 200% | Every sprint on key screens |
| Motion sensitivity | `prefers-reduced-motion` toggle | During design review |

---

## 6. Norwegian-First Design

### 6.1 Language Strategy

**Dual language support**: Bokmal (nb) as default, Nynorsk (nn) as alternate. User can choose in profile settings. Organization can set default per tenant.

**i18n architecture**:
- All UI strings externalized to JSON locale files
- No hardcoded Norwegian or English strings in components
- ICU MessageFormat for pluralization and gender
- Date, time, and number formatting via `Intl` API with `nb-NO` locale

**Translation quality**:
- Native Norwegian copywriter reviews all UI text
- Terminology consistency with Norwegian government digital services
- Avoid direct English translations -- use idiomatic Norwegian
- Formal but approachable tone (not bureaucratic)

### 6.2 Date, Time, and Number Formatting

| Type | Format | Example |
|---|---|---|
| Full date | `dd. MMMM yyyy` | `22. mars 2026` |
| Short date | `dd.MM.yyyy` | `22.03.2026` |
| Relative date | `i dag`, `i gar`, `for 3 dager siden` | Dynamic |
| Time | `HH:mm` | `14:30` |
| Date + time | `dd.MM.yyyy kl. HH:mm` | `22.03.2026 kl. 14:30` |
| Duration | `X t Y min` | `2 t 30 min` |
| Numbers | Space thousand separator, comma decimal | `1 234,56` |
| Percentage | `XX,X %` | `92,5 %` |
| File size | `XX MB` | `45 MB` |

### 6.3 Norwegian Cultural UX Patterns

**Trust signals**: Norwegian users trust government-affiliated design language. Use:
- Statens vegvesen logo placement consistent with government guidelines
- Muted, professional color palette (not flashy startup aesthetic)
- Clear data privacy indicators ("Dine data lagres i Norge/EU")
- ID-porten integration signals government-level security

**Interaction patterns familiar from Norwegian digital services**:
- Altinn: Stepped wizard pattern for multi-step processes
- Helsenorge: Clean, white-space-heavy layouts with card-based content
- Vipps: Bottom navigation, smooth transitions, immediate feedback
- NAV: Clear status indicators, timeline-based history views

**Tone of voice**:
- Direct and clear, not verbose
- Helpful but not patronizing
- Professional but human
- Use "du" (informal you), consistent with modern Norwegian government communication

### 6.4 Theme Toggle

Located in the header navigation bar, accessible on all pages:

```html
<!-- Theme Toggle Component -->
<div class="theme-toggle" role="radiogroup" aria-label="Velg tema">
  <button class="theme-toggle-option" data-theme="light"
          role="radio" aria-checked="false">
    Lyst
  </button>
  <button class="theme-toggle-option" data-theme="dark"
          role="radio" aria-checked="false">
    Morkt
  </button>
  <button class="theme-toggle-option" data-theme="system"
          role="radio" aria-checked="true">
    System
  </button>
</div>
```

**Behavior**:
- Defaults to "System" (follows OS preference)
- User choice persisted in localStorage
- Transition: 200ms ease on `background-color` and `color`
- All components and charts adapt to theme
- SCORM iframe content: cannot be theme-controlled (external content), but wrapper respects theme

---

## 7. Layout Framework

### 7.1 Application Shell

```
+------------------------------------------------------------------+
| HEADER (64px fixed)                                               |
| Logo | Navigation breadcrumb | Search | Theme | Notifications    |
+--------+---------------------------------------------------------+
|        |                                                         |
| SIDE-  |  MAIN CONTENT AREA                                      |
| BAR    |                                                         |
| (260px)|  Max-width: 1280px                                      |
| fixed  |  Padding: 24px (desktop), 16px (mobile)                 |
|        |                                                         |
| Collapses to 64px icons on tablet                                |
| Becomes bottom nav on mobile                                     |
|        |                                                         |
+--------+---------------------------------------------------------+
```

### 7.2 Grid System

```css
/* Container */
.container {
  width: 100%;
  max-width: var(--container-xl);  /* 1280px */
  margin: 0 auto;
  padding: 0 var(--space-6);      /* 24px */
}

@media (max-width: 768px) {
  .container {
    padding: 0 var(--space-4);    /* 16px */
  }
}

/* Dashboard Grid */
.dashboard-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: var(--space-6);
}

/* KPI Row */
.kpi-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: var(--space-4);
}

@media (max-width: 768px) {
  .kpi-row {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Course Catalog Grid */
.course-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: var(--space-6);
}

/* Two-column layout (content + sidebar) */
.content-with-sidebar {
  display: grid;
  grid-template-columns: 1fr 320px;
  gap: var(--space-8);
}

@media (max-width: 1024px) {
  .content-with-sidebar {
    grid-template-columns: 1fr;
  }
}
```

### 7.3 Component Hierarchy

```
1. Layout Components (shell, containers, grids)
   |
   2. Page Components (dashboard, catalog, player, admin)
      |
      3. Section Components (widget, card group, table section)
         |
         4. Content Components (course card, user row, chart)
            |
            5. Interactive Components (button, input, dropdown, toggle)
               |
               6. Utility Components (badge, icon, tooltip, skeleton)
```

### 7.4 File Structure

```
src/
+-- styles/
|   +-- tokens.css           # Design system variables (colors, spacing, typography)
|   +-- reset.css             # CSS reset / normalize
|   +-- layout.css            # Grid system, containers, responsive utilities
|   +-- components.css        # Component-specific styles
|   +-- utilities.css         # Helper classes (sr-only, spacing, text alignment)
|   +-- themes.css            # Light/dark/high-contrast theme definitions
|
+-- components/
|   +-- layout/
|   |   +-- AppShell.tsx      # Header + sidebar + main layout
|   |   +-- Sidebar.tsx       # Navigation sidebar (desktop/tablet)
|   |   +-- BottomNav.tsx     # Bottom navigation (mobile)
|   |   +-- Header.tsx        # Top header with search, notifications
|   |   +-- ThemeToggle.tsx   # Light/dark/system theme switcher
|   |
|   +-- common/
|   |   +-- Button.tsx
|   |   +-- Input.tsx
|   |   +-- Card.tsx
|   |   +-- Badge.tsx
|   |   +-- Modal.tsx         # Radix Dialog wrapper
|   |   +-- Dropdown.tsx      # Radix DropdownMenu wrapper
|   |   +-- Tabs.tsx          # Radix Tabs wrapper
|   |   +-- Toast.tsx         # Notification toasts
|   |   +-- ProgressBar.tsx
|   |   +-- StatusIndicator.tsx  # Traffic light with icon+label
|   |   +-- Skeleton.tsx      # Loading skeleton
|   |
|   +-- courses/
|   |   +-- CourseCard.tsx
|   |   +-- CourseDetail.tsx
|   |   +-- CoursePlayer.tsx   # SCORM wrapper
|   |   +-- CourseCatalog.tsx
|   |   +-- CourseSearch.tsx
|   |
|   +-- learning-paths/
|   |   +-- PathVisualization.tsx
|   |   +-- PathBuilder.tsx    # Drag-and-drop editor (content manager)
|   |   +-- PathNode.tsx
|   |
|   +-- certificates/
|   |   +-- CertificateCard.tsx
|   |   +-- CertificateWallet.tsx
|   |   +-- CertificateViewer.tsx
|   |
|   +-- dashboard/
|   |   +-- LearnerDashboard.tsx
|   |   +-- LeaderDashboard.tsx
|   |   +-- AdminDashboard.tsx
|   |   +-- KPICard.tsx
|   |   +-- ComplianceChart.tsx
|   |   +-- ExpiryTimeline.tsx
|   |
|   +-- admin/
|   |   +-- UserManagement.tsx
|   |   +-- ReportBuilder.tsx
|   |   +-- CourseCreationWizard.tsx
|   |   +-- ApprovalQueue.tsx
|   |   +-- AuditLog.tsx
|   |
|   +-- i18n/
|       +-- nb.json            # Bokmal translations
|       +-- nn.json            # Nynorsk translations
```

---

## 8. Implementation Priority

### 8.1 MVP UX Scope (for 01.01.2027 target)

| Priority | Component | Rationale |
|---|---|---|
| P0 | Application shell (header, nav, responsive layout) | Foundation for everything |
| P0 | SSO login flow | Required for any access |
| P0 | Learner dashboard | Primary user landing page |
| P0 | Course catalog with search | Core discovery experience |
| P0 | SCORM course player | Core learning experience |
| P0 | Course detail page | Enrollment flow |
| P0 | Progress tracking (Min laering) | Core learner value |
| P1 | Leader dashboard (traffic light) | High-impact for evaluation |
| P1 | Certificate wallet | Compliance value |
| P1 | Course assignment flow | Leader workflow |
| P1 | Content upload (SCORM) | Content manager workflow |
| P1 | Basic reporting with export | Admin value |
| P2 | Learning path visualization | Enhanced navigation |
| P2 | Learning path builder | Content manager tool |
| P2 | Report builder (drag & drop) | Advanced analytics |
| P2 | AI recommendations | Differentiator for bid |
| P2 | External user onboarding (ID-porten) | KKP portal scope |
| P3 | Offline PWA capabilities | Field worker support |
| P3 | Gamification elements | Engagement features |
| P3 | Advanced dashboard customization | Admin power users |

### 8.2 POC Demo UX Focus

For the 80-minute demonstration (11-12.05.2026), the UX must show polish in these areas above all:

1. **Learner dashboard** -- first impression, personalization, Norwegian language quality
2. **Course player** -- smooth SCORM integration, progress tracking
3. **Leader traffic light** -- solves the Excel pain point, immediate comprehension
4. **Content creation** -- SCORM upload + validation, learning path builder
5. **Reporting** -- one-click Excel export, PowerBI integration

Every screen shown in the demo must have realistic Norwegian data, zero English strings visible, and sub-2-second load times.

---

## 9. Developer Handoff Notes

### 9.1 CSS Methodology

**Approach**: CSS Modules (component-scoped) with global design tokens. Utility classes for spacing and typography only.

- Component styles: CSS Modules (`*.module.css`) for encapsulation
- Global tokens: CSS custom properties in `tokens.css`
- No Tailwind -- keeps bundle size predictable and avoids class soup in government-accessible HTML
- BEM naming within modules for sub-element clarity

### 9.2 Accessibility Checklist (per component)

Before any component ships:
- [ ] Keyboard navigable (tab, enter, escape, arrow keys where applicable)
- [ ] Screen reader tested (NVDA or VoiceOver)
- [ ] Color contrast verified (4.5:1 minimum)
- [ ] Focus indicator visible
- [ ] ARIA attributes correct (roles, states, labels)
- [ ] Norwegian labels and error messages
- [ ] Responsive at all breakpoints
- [ ] Theme-aware (light/dark/high-contrast)
- [ ] axe-core passes (zero violations)

### 9.3 Performance Budget

| Metric | Target |
|---|---|
| First Contentful Paint | < 1.2s |
| Largest Contentful Paint | < 2.0s |
| Cumulative Layout Shift | < 0.1 |
| First Input Delay | < 100ms |
| Time to Interactive | < 3.0s |
| JavaScript bundle (initial) | < 200KB gzipped |
| CSS bundle (initial) | < 50KB gzipped |

---

**Document**: 02_ux_design.md
**Author**: ArchitectUX Agent
**Date**: 2026-03-22
**Status**: Complete
**Handoff**: Ready for development implementation
**Dependencies**: Technical architecture (01), AI strategy (03) for recommendation engine UX
