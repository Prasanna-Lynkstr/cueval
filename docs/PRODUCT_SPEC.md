# Cueval by Lynkstr Labs — Claude Code Spec
## Full Prototype: Single HTML File, Browser-Only

---

## 1. PRODUCT OVERVIEW

**Cueval** is an AI dataset curation and evaluation platform for teams building and fine-tuning language models. It covers the full pipeline: ingest raw data → clean and score → human review → experiment tracking → eval → release approval.

**Brand:** Cueval by Lynkstr Labs
**Tagline:** From raw data to release-ready.
**Stack:** Single self-contained HTML file. Vanilla JS + CSS. No frameworks. No backend. No external dependencies except Google Fonts (loaded via CDN). All data lives in browser memory (JS objects/arrays). Mock logic simulates all AI scoring, eval, and LLM calls.

---

## 2. DESIGN SYSTEM

### Palette
```
--bg-base:       #0A0A0F   /* near-black, deep space */
--bg-surface:    #111118   /* card/panel background */
--bg-elevated:   #1A1A24   /* modals, dropdowns */
--bg-hover:      #1F1F2E   /* hover state */
--border:        #2A2A3A   /* subtle borders */
--border-active: #3D3D5C   /* active/focus borders */

--coral:         #E85D26   /* Lynkstr primary — CTAs, active states */
--coral-dim:     #C4410C   /* coral hover */
--coral-glow:    rgba(232,93,38,0.15) /* ambient glow */

--pink:          #D63384   /* secondary accent — scores, highlights */
--pink-dim:      rgba(214,51,132,0.15)

--text-primary:  #F0F0F8   /* headlines */
--text-secondary:#9898B0   /* labels, metadata */
--text-muted:    #5A5A72   /* placeholders, disabled */

--green:         #22C55E   /* pass/approved */
--yellow:        #EAB308   /* warning/review needed */
--red:           #EF4444   /* fail/rejected */
--blue:          #3B82F6   /* info/neutral */

--gradient-brand: linear-gradient(135deg, #E85D26, #D63384)
```

### Typography
```
Display:  'Space Grotesk' (Google Fonts) — weight 500/600/700. Used for product name, big numbers only. (Superseded 'Syne', which read as too extended/stretched.)
Body:     'Inter' (Google Fonts) — weight 400/500/600. All UI text.
Mono:     'JetBrains Mono' (Google Fonts) — code, JSON, dataset rows, scores.
```

### Signature Design Element
**Orbital dock navigation** — a vertical icon dock on the left edge with circular glowing indicators showing live counts (pending reviews, running evals). Each icon, when active, pulses with a coral glow ring. No labels visible by default — tooltip appears on hover. The dock floats over content with a frosted glass effect. This is the single most memorable UI element — nothing like a traditional sidebar.

### Layout
- Full viewport. No scroll on outer shell.
- Left dock: 64px wide, fixed.
- Top bar: 48px, fixed. Project + Release selector + user avatar only.
- Main canvas: fills remaining space. Internal sections scroll independently.
- Right panel: slides in from right (380px) for detail views — never navigates away from current context.
- Command palette: `Cmd+K` / `Ctrl+K` — searches projects, datasets, rows, runs globally.

### Motion
- Page transitions: 200ms fade + 8px upward translate.
- Right panel: 250ms ease-out slide.
- Dock icons: 150ms scale on hover (1.0 → 1.12).
- Score bars: animate from 0 on mount (600ms ease-out).
- Glow pulses on dock: 2s infinite breathing animation for pending items.
- Reduce motion: respect `prefers-reduced-motion`.

---

## 3. AUTHENTICATION

### Login Screen
- Full viewport dark background with subtle animated mesh gradient (coral + pink, very low opacity).
- Center card: Cueval logo + "by Lynkstr Labs" + email/password fields.
- No registration flow in prototype.

### Demo Credentials (hardcoded)
```
admin@cueval.ai        / demo123   → Admin
pm@cueval.ai           / demo123   → Project Manager
architect@cueval.ai    / demo123   → Architect
mlengineer@cueval.ai   / demo123   → ML Engineer
reviewer@cueval.ai     / demo123   → Reviewer
annotator@cueval.ai    / demo123   → Annotator
```

### Session
Store current user in JS variable. On logout, clear and return to login screen. No localStorage.

---

## 4. DATA MODEL (in-memory JS)

```javascript
// Organisation
org = {
  name: "IITM Pravartak AI Lab",
  plan: "Enterprise"
}

// Users — 6 demo users as above

// Roles & Permissions Matrix (fully configurable by Admin)
roles = {
  Admin: { all permissions: true },
  ProjectManager: { ... },
  Architect: { ... },
  MLEngineer: { ... },
  Reviewer: { ... },
  Annotator: { limited to own queue }
}

// Projects
projects = [
  {
    id, name, description, status,
    members: [{ userId, roleId }],
    releases: [...],
    createdAt
  }
]

// Releases (versions under a project)
releases = [
  {
    id, projectId, name,          // e.g. "v1.0 — Base Model"
    status,                        // draft | in-review | approved | training | deployed
    dataset: datasetId,
    trainingConfig: { ... },
    checkpoint: checkpointId,
    evalRun: evalRunId,
    createdBy, createdAt
  }
]

// Datasets
datasets = [
  {
    id, projectId, releaseId,
    name, format,                  // jsonl | csv | parquet | tsv | txt
    totalRows, flaggedRows,
    overallScore,                  // 0-100
    languages: [...],              // detected languages with %
    uploadedAt, uploadedBy,
    rows: [
      {
        id, instruction, input, output,
        qualityScore,              // 0-100
        flags: [...],              // 'duplicate' | 'near-duplicate' | 'short-response' |
                                   // 'format-violation' | 'pii-detected' | 'toxic' | 'low-clarity'
        language,                  // en | hi | ta | te | kn | ml | mixed
        reviewStatus,              // pending | approved | rejected | edited
        reviewedBy, reviewNote,
        originalOutput             // if edited, keep original
      }
    ]
  }
]

// Eval Runs
evalRuns = [
  {
    id, projectId, releaseId,
    checkpointA, checkpointB,      // for comparison
    rubric: [
      { dimension, weight },       // Factuality | Instruction Following | Tone |
                                   // Task Completion | Refusal Behaviour
    ],
    results: {
      checkpointA: { dimension: score, ... },
      checkpointB: { dimension: score, ... }
    },
    humanReviewQueue: [...],
    status, runAt, runBy
  }
]

// Experiments
experiments = [
  {
    id, projectId, releaseId, name,
    datasetId, datasetVersion,
    trainingConfig: {
      baseModel, epochs, learningRate,
      batchSize, loraRank, optimizer
    },
    checkpointId,
    evalRunId,
    costEstimate,
    status,
    createdAt, createdBy,
    notes
  }
]
```

---

## 5. MOCK DATA

### Project 1: Legal AI Assistant
- Release v1.0 (approved), v1.1 (in-review), v1.2 (draft)
- Dataset: 847 rows, English + Hindi, overall score 73
- 12 rows pending review
- 2 eval runs completed

### Project 2: Customer Support Bot — Tamil
- Release v1.0 (training)
- Dataset: 1,243 rows, Tamil + English mixed
- 34 rows flagged, 8 pending review
- 1 eval run in progress

### Project 3: Medical QA — Multilingual
- Release v0.9 (draft)
- Dataset: 412 rows, Hindi + Telugu + Kannada
- High flag rate (41%) — PII detected, short responses
- No eval run yet

### Sample Dataset Rows (minimum 50 rows per dataset, mixed quality)
Include realistic instruction/input/output pairs with:
- 8–10 clean rows (score 85–95)
- 6–8 near-duplicates (score 30–45)
- 5–6 PII flagged rows (score 20–35) — fake names/phone numbers in output
- 4–5 short responses (score 40–55)
- 3–4 format violations (score 25–40)
- 5–6 Indic language rows (Tamil, Hindi, Telugu)
- 3–4 toxic/biased rows (score 10–25)

---

## 6. NAVIGATION (Orbital Dock)

Left dock, 64px wide, fixed, frosted glass background.

Icons (top to bottom):
```
1. 🏠  Dashboard      — org-level overview
2. 📁  Projects       — project list + switcher
3. 🗄️  Datasets       — all datasets for current project/release
4. ✏️  Review Queue   — annotation and review (badge: pending count)
5. ⚡  Eval Harness   — run and view evals
6. 🧪  Experiments    — experiment tracker
7. 🚀  Releases       — release management and approval
8. ⚙️  Settings       — org, roles, permissions (Admin only)
```

Bottom of dock:
```
9. 👤  User avatar + name (click → profile dropdown)
10. ⌘  Command palette hint
```

Active state: coral glow ring around icon.
Pending badge: coral dot with count for Review Queue.
Running badge: animated pulse for Eval Harness when run is in progress.
Role-based visibility: Settings only visible to Admin.

---

## 7. TOP BAR

```
[Cueval logo]  [Project dropdown ▾]  [Release dropdown ▾]  ............  [🔔 Notifications]  [Avatar]
```

- Project dropdown: lists all projects user has access to. Switching project resets release to latest.
- Release dropdown: lists all releases under selected project. Color-coded by status.
- Notifications: bell icon, dropdown shows recent activity (row reviewed, eval completed, release approved).

---

## 8. SCREENS

---

### 8.1 DASHBOARD

**Role-aware CTAs — highest priority items surfaced immediately.**

Layout: 3-column grid.

**Column 1 — My Work Today (role-specific)**
- Annotator: "12 rows pending your review" → [Start Reviewing]
- Reviewer: "3 batches awaiting approval" → [Review Batches]
- ML Engineer: "Eval run completed on v1.1" → [View Results]
- Architect: "v1.2 dataset score dropped to 61" → [Investigate]
- PM: "v1.1 release blocked — 8 rows unreviewed" → [Check Status]
- Admin: "2 users pending role assignment" → [Manage Users]

**Column 2 — Project Health**
- Cards per active project showing:
  - Dataset quality score (large number, color-coded)
  - Review completion % (progress bar)
  - Eval status (last run score vs previous)
  - Release status pill

**Column 3 — Activity Feed**
- Chronological feed: who did what, when
- "Ravi approved 23 rows in Legal AI v1.1"
- "Eval run completed — Factuality improved +4.2"
- "Neha flagged 3 PII rows in Medical QA"

**Top strip — org-level metrics:**
```
Total Rows Curated  |  Avg Quality Score  |  Active Eval Runs  |  Releases This Month
     14,832         |        74.3          |         2          |          3
```

---

### 8.2 PROJECTS

**List view:**
- Cards in a 3-column grid
- Each card: project name, description, member count, active release, dataset score, status pill
- [+ New Project] button (PM/Admin only)

**New Project modal:**
- Name, description, assign members (with role per member), tags

**Project detail (click a card):**
- Header: name, description, members, status
- Tabs: Releases | Datasets | Activity
- Releases tab: timeline view of all releases with status

---

### 8.3 DATASETS

**Left: dataset list** (for current project)
- File name, format badge, row count, score, upload date
- [Upload Dataset] button

**Upload flow:**
- Drag and drop or file picker
- Accepts: .jsonl, .csv, .tsv, .parquet (mock parse), .txt
- After upload → mock curation runs (animated progress: "Detecting language... Scoring rows... Flagging issues... Done")
- Shows curation summary report

**Right: Dataset detail**
- Header: name, format, total rows, flagged rows, overall score (large, color-coded)
- Language breakdown: horizontal bar showing % per language (English 67%, Tamil 23%, Hindi 10%)
- Quality score distribution: histogram showing rows by score bucket
- Flag summary: count per flag type with icons

**Row table:**
- Columns: #, Instruction (truncated), Response (truncated), Score, Flags, Language, Status
- Row click → right panel slides in with full row detail
- Filters: by flag type, language, score range, review status
- Sort: by score (asc/desc), by flag count
- Bulk select → bulk approve/reject

**Right panel — Row Detail:**
- Full instruction text
- Full input (if present)
- Full output
- Quality score with breakdown (clarity score, completeness score, relevance score)
- Flags list with explanation per flag
- Language detected
- Review action: [Approve] [Reject] [Edit Output] [Add Note]
- If edited: show diff between original and edited output

**Export:**
- [Export Cleaned Dataset] → downloads .jsonl with only approved rows
- Format selector: JSONL | CSV | Axolotl | LLaMA-Factory

---

### 8.4 REVIEW QUEUE

**Annotator/Reviewer's primary screen.**

Split layout:
- Left (60%): current row being reviewed
- Right (40%): queue list

**Current row panel:**
- Large readable instruction + output
- Flag badges with explanations ("Near-duplicate of row #234")
- Language badge
- Quality score
- [Approve ✓] [Reject ✗] [Edit ✎] [Skip →] — keyboard shortcuts shown
- Edit mode: inline textarea for output, shows character count, [Save Edit]

**Queue panel:**
- List of pending rows with score and primary flag
- Progress: "8 of 20 reviewed"
- Filter: show only flagged | show all
- Sorted by: score ascending (worst first by default)

**Batch approval (Reviewer role):**
- After annotators complete, Reviewer sees batch summary
- "Annotator Ravi reviewed 23 rows: 18 approved, 5 rejected"
- [Approve Batch] [Send Back for Re-review]

---

### 8.5 EVAL HARNESS

**Top: Run Configuration**
- Select release (auto-populated from current)
- Select checkpoints to compare (Checkpoint A vs Checkpoint B)
- Rubric builder: 5 default dimensions shown, each with weight slider (total must = 100%)
- Dimensions: Factuality | Instruction Following | Tone | Task Completion | Refusal Behaviour
- [+ Add Custom Dimension]
- [Run Eval] → triggers mock run (animated progress, 3-4 seconds)

**Results view:**

*Radar chart:* Two overlapping polygons (Checkpoint A in coral, Checkpoint B in pink) across 5 dimensions. Animated draw on mount.

*Score table:*
```
Dimension            | Checkpoint A | Checkpoint B | Delta
Factuality           |    72.4      |    78.1      | +5.7 ▲
Instruction Follow   |    81.2      |    79.8      | -1.4 ▼
Tone                 |    88.0      |    91.2      | +3.2 ▲
Task Completion      |    69.5      |    74.3      | +4.8 ▲
Refusal Behaviour    |    94.1      |    93.7      | -0.4 ▼
Overall              |    81.0      |    83.4      | +2.4 ▲
```

*Human review queue:* Rows where judge confidence < 70% flagged for human review. Same approve/reject UI as Review Queue.

*Cost tracker:* "This eval run consumed ~4,200 tokens. Estimated cost: ₹0.84"

*History:* List of all past eval runs with date, checkpoints, overall score. Click to reload results.

---

### 8.6 EXPERIMENT TRACKER

**Timeline view** — vertical timeline, each node is an experiment.

Each experiment card shows:
```
[Experiment Name]
Dataset v2.1 (score: 74) → Training Config → Checkpoint v1.1 → Eval Score: 81.0
Status: Completed | Cost: ₹2,340 | Run by: Arjun ML | Date: 12 Jun 2025
```

**New Experiment:**
- Link: select dataset version
- Training config: base model (dropdown), epochs, learning rate, batch size, LoRA rank, optimizer
- Notes field
- [Save Experiment] — adds to timeline

**Experiment detail (right panel):**
- Full config display
- Dataset quality score at time of experiment
- Eval results linked
- Reproducibility block: "To recreate this run, use dataset snapshot v2.1.3 with config..."
- [Clone Experiment] — pre-fills new experiment form with same config

**Comparison view:**
- Select 2 experiments → side-by-side config diff + eval score comparison

---

### 8.7 RELEASES

**Release pipeline view** — horizontal kanban-style flow:
```
Draft → In Review → Approved → Training → Deployed
```

Each release card shows: name, dataset score, eval score, blocking issues count.

**Release detail:**
- Checklist of gates:
  - ✅ Dataset uploaded
  - ✅ Curation score > 70
  - ⚠️ Human review < 100% complete (8 rows pending)
  - ✅ Eval run completed
  - ❌ Architect approval pending
  - ❌ PM sign-off pending
- [Approve Release] button (role-gated — Architect/PM/Admin only)
- [Request Changes] with comment

**Version comparison:**
- Select two releases → diff view showing dataset score change, eval score change, row count change

---

### 8.8 SETTINGS (Admin only)

**Tabs: Organisation | Users | Roles & Permissions | Integrations**

**Roles & Permissions tab:**
Full permissions matrix as a table.
- Rows: permission groups (Org, Projects, Releases, Datasets, Annotation, Eval, Experiments, Reports)
- Columns: each role
- Each cell: toggle (on/off)
- [+ Create Custom Role] → name it, configure from scratch
- [Save Changes]

**Users tab:**
- List of all users with role, last active, project count
- [Invite User] → email + default role
- [Edit Role] per user

---

## 9. COMMAND PALETTE (Cmd+K)

Triggered by keyboard shortcut or clicking ⌘ hint in dock.

Full-screen overlay, search input at top.

Searches across:
- Projects (→ navigate to project)
- Datasets (→ open dataset)
- Rows by instruction text (→ open row in right panel)
- Eval runs (→ open results)
- Experiments (→ open detail)
- Actions: "Upload dataset", "Run eval", "New project", "New experiment"

Groups results by type. Keyboard navigable. Esc to close.

---

## 10. MOCK LOGIC

### Quality Scoring
```javascript
function scoreRow(row) {
  let score = 100;
  if (row.output.length < 20) score -= 40;           // short response
  if (row.output.length < 50) score -= 20;           // marginal response
  if (!row.instruction || row.instruction.length < 10) score -= 30; // no instruction
  if (isDuplicate(row)) score -= 50;                  // exact duplicate
  if (isNearDuplicate(row)) score -= 30;             // near duplicate (Jaccard < 0.3)
  if (hasPII(row)) score -= 35;                      // phone/email/aadhaar pattern
  if (hasFormatViolation(row)) score -= 25;          // missing required fields
  if (hasToxic(row)) score -= 60;                    // toxic keyword list
  return Math.max(0, Math.min(100, score));
}
```

### Language Detection (mock)
```javascript
function detectLanguage(text) {
  // Check Unicode ranges
  if (/[\u0B80-\u0BFF]/.test(text)) return 'ta';   // Tamil
  if (/[\u0900-\u097F]/.test(text)) return 'hi';   // Hindi/Devanagari
  if (/[\u0C00-\u0C7F]/.test(text)) return 'te';   // Telugu
  if (/[\u0C80-\u0CFF]/.test(text)) return 'kn';   // Kannada
  if (/[\u0D00-\u0D7F]/.test(text)) return 'ml';   // Malayalam
  return 'en';
}
```

### Eval Scoring (mock)
```javascript
function runEval(checkpointId, rubric) {
  // Generate plausible scores with small random variance
  // Checkpoint B always slightly better than A to show improvement
  return rubric.map(dim => ({
    dimension: dim.name,
    scoreA: 65 + Math.random() * 25,
    scoreB: 68 + Math.random() * 25
  }));
}
```

### PII Detection (mock)
```javascript
function hasPII(row) {
  const patterns = [
    /\b\d{10}\b/,                    // phone
    /[a-z0-9._%+-]+@[a-z0-9.-]+/i,  // email
    /\b\d{4}\s?\d{4}\s?\d{4}\b/,    // aadhaar-like
    /\bpan\s?[a-z]{5}\d{4}[a-z]\b/i // PAN-like
  ];
  return patterns.some(p => p.test(row.output) || p.test(row.input || ''));
}
```

---

## 11. NOTIFICATIONS

In-app notification system (no backend).

Trigger notifications for:
- Row review completed by annotator
- Batch approved by reviewer
- Eval run completed
- Release status changed
- Dataset upload completed
- User assigned to project

Bell icon in top bar shows unread count. Dropdown shows last 10 notifications. Mark all read.

---

## 12. RESPONSIVE & ACCESSIBILITY

- Desktop-first (optimised for 1280px+ viewport)
- Minimum supported: 1024px width
- Keyboard navigable: all interactive elements focusable
- Focus rings: coral outline, 2px
- ARIA labels on all icon-only buttons
- Color not used as sole indicator (always paired with icon or text)
- Reduced motion: disable animations when `prefers-reduced-motion: reduce`

---

## 13. EMPTY STATES

Every empty state includes:
- A simple icon illustration (SVG inline)
- One-line explanation of what goes here
- A primary CTA to create/upload

Examples:
- No projects: "No projects yet. Start by creating your first project." [Create Project]
- No datasets: "Upload a dataset to begin curation." [Upload Dataset]
- Empty queue: "You're all caught up. No rows pending review." ✅
- No eval runs: "Run your first eval to compare checkpoints." [Run Eval]

---

## 14. IMPLEMENTATION NOTES FOR CLAUDE CODE

1. Build as a single `.html` file. All CSS in `<style>`, all JS in `<script>`. No external files.
2. Load Google Fonts via `<link>` in `<head>`: Space Grotesk, Inter, JetBrains Mono.
3. No frameworks. Pure DOM manipulation with vanilla JS.
4. Use SVG icons inline (Heroicons style) — do not import icon libraries.
5. Routing: simple JS state variable `currentView`. Re-render main canvas on change.
6. Right panel: absolute positioned div, transform translateX(100%) when closed, 0 when open.
7. Mock data: define all data as JS const arrays at top of script. Minimum dataset size as specified.
8. File upload: use FileReader API to read uploaded file. Parse based on extension. Mock curation runs after parse with setTimeout chain showing animated steps.
9. Chart (radar): draw using HTML5 Canvas API. No chart library.
10. Export: use Blob + URL.createObjectURL to trigger real file download.
11. Command palette: keydown listener on document for Cmd+K / Ctrl+K.
12. Animations: CSS transitions and keyframes only. No JS animation libraries.
13. All views must be fully functional with mock data — no placeholder "coming soon" sections.
14. Build the entire file in one complete output. Do not truncate or summarise any section.
15. Test every role login — each must show a meaningfully different dashboard and navigation.

---

## 15. MULTI-TENANCY

### Architecture Model
Database-per-tenant (simulated in prototype via isolated JS objects scoped to `tenantId`). Every data access function receives and filters by `currentTenant.id`. No cross-tenant data leakage is possible by design — tenant context is resolved at login and never changes mid-session.

### Tenant Data Structure
```javascript
const tenants = [
  {
    id: "tenant_pravartak",
    name: "IITM Pravartak AI Lab",
    slug: "pravartak",
    plan: "Enterprise",           // Starter | Pro | Enterprise
    planLimits: {
      rowsPerMonth: Infinity,
      evalRunsPerMonth: Infinity,
      maxUsers: Infinity,
      maxProjects: Infinity,
      sovereignDeployment: true
    },
    usage: {
      rowsThisMonth: 14832,
      evalRunsThisMonth: 7,
      activeUsers: 6,
      activeProjects: 3
    },
    createdAt: "2025-01-15",
    status: "active"
  },
  {
    id: "tenant_legalai",
    name: "Legal AI Corp",
    slug: "legalai",
    plan: "Pro",
    planLimits: {
      rowsPerMonth: 500000,
      evalRunsPerMonth: 50,
      maxUsers: 20,
      maxProjects: 10,
      sovereignDeployment: false
    },
    usage: {
      rowsThisMonth: 312400,
      evalRunsThisMonth: 31,
      activeUsers: 8,
      activeProjects: 3
    },
    createdAt: "2025-03-02",
    status: "active"
  },
  {
    id: "tenant_healthbot",
    name: "HealthBot India",
    slug: "healthbot",
    plan: "Starter",
    planLimits: {
      rowsPerMonth: 50000,
      evalRunsPerMonth: 10,
      maxUsers: 5,
      maxProjects: 3,
      sovereignDeployment: false
    },
    usage: {
      rowsThisMonth: 41200,
      evalRunsThisMonth: 8,
      activeUsers: 3,
      activeProjects: 1
    },
    createdAt: "2025-05-10",
    status: "active"             // active | suspended | trial
  }
];
```

### Tenant Resolution at Login
```javascript
// Each user belongs to exactly one tenant
const users = [
  // Pravartak users
  { email: "admin@cueval.ai",       tenantId: "tenant_pravartak", role: "Admin" },
  { email: "pm@cueval.ai",          tenantId: "tenant_pravartak", role: "ProjectManager" },
  { email: "architect@cueval.ai",   tenantId: "tenant_pravartak", role: "Architect" },
  { email: "mlengineer@cueval.ai",  tenantId: "tenant_pravartak", role: "MLEngineer" },
  { email: "reviewer@cueval.ai",    tenantId: "tenant_pravartak", role: "Reviewer" },
  { email: "annotator@cueval.ai",   tenantId: "tenant_pravartak", role: "Annotator" },

  // Legal AI Corp users
  { email: "admin@legalai.com",     tenantId: "tenant_legalai",   role: "Admin" },
  { email: "ml@legalai.com",        tenantId: "tenant_legalai",   role: "MLEngineer" },

  // HealthBot India users
  { email: "admin@healthbot.in",    tenantId: "tenant_healthbot",  role: "Admin" },

  // Super Admin — Lynkstr Labs (sees ALL tenants)
  { email: "superadmin@cueval.ai",  tenantId: null,               role: "SuperAdmin" }
];

// All passwords: demo123
// On login: resolve user → set currentUser + currentTenant
// All subsequent data queries filter by currentTenant.id
// SuperAdmin: no tenant filter — sees cross-tenant view only
```

### Data Isolation Rule
```javascript
// Every data accessor enforces this:
function getProjects() {
  return projects.filter(p => p.tenantId === currentTenant.id);
}
function getDatasets() {
  return datasets.filter(d => d.tenantId === currentTenant.id);
}
// Same pattern for releases, experiments, evalRuns, users
// SuperAdmin bypasses filter — accesses all tenants
```

### Mock Data Per Tenant

**Tenant: Legal AI Corp**
- Project: Contract Analysis AI (v1.0 deployed, v1.1 in-review)
- Project: Due Diligence Bot (v0.8 draft)
- Project: Compliance Checker (v1.0 approved)
- Dataset quality: avg 79 (better than Pravartak — more curated English data)
- 8 users, 31 eval runs this month
- Usage bar: 312K / 500K rows (62% of plan limit — show warning at 80%)

**Tenant: HealthBot India**
- Project: Symptom Triage Assistant (v1.0 training)
- Dataset: 41,200 rows, Hindi + English
- Usage bar: 41.2K / 50K rows (82% — show warning, approaching limit)
- 3 users, 8 eval runs
- Status: approaching plan limit — show upgrade CTA prominently

### Usage Limit Warnings (per tenant)
- < 70% used: green indicator
- 70–85% used: yellow warning banner in settings
- > 85% used: orange warning in top bar + settings
- > 95% used: red banner, block new uploads, show [Upgrade Plan] CTA
- HealthBot India is at 82% — must show yellow warning

### Plan Limits Display
In Settings → Organisation tab, show usage meters:
```
Rows this month:      41,200 / 50,000   ████████░░  82%  ⚠️
Eval runs:            8 / 10            ████████░░  80%  ⚠️
Active users:         3 / 5             ██████░░░░  60%
Projects:             1 / 3             ███░░░░░░░  33%
Sovereign deploy:     Not available on Starter — [Upgrade to Enterprise]
```

---

## 16. SUPER ADMIN SCREEN (superadmin@cueval.ai)

Super Admin is the Lynkstr Labs operator view. Completely separate from tenant views. Login resolves to a different shell — no project/release selector in top bar, different dock.

### Super Admin Dock
```
1. 🏢  Tenants         — all org list
2. 📊  Platform Usage  — cross-tenant metrics
3. 💳  Billing         — plan, status per tenant
4. 🔔  Alerts          — approaching limits, errors, suspicious activity
5. ⚙️  Platform Config — feature flags, plan definitions
```

### Tenant List View
Table of all tenants:
```
Tenant Name          | Plan       | Users | Projects | Rows/mo      | Eval Runs | Status   | Actions
IITM Pravartak       | Enterprise | 6     | 3        | 14,832 / ∞   | 7 / ∞     | Active   | [View] [Manage]
Legal AI Corp        | Pro        | 8     | 3        | 312K / 500K  | 31 / 50   | Active   | [View] [Manage]
HealthBot India      | Starter    | 3     | 1        | 41.2K / 50K  | 8 / 10    | ⚠️ Limit  | [View] [Upgrade]
```

### Tenant Detail (read-only drill-down)
- Click [View] on any tenant → see their projects, datasets, users (read-only, no edit)
- Clearly watermarked "Super Admin View — Read Only"
- [Impersonate Admin] button — logs in as that tenant's admin (show banner: "Impersonating Legal AI Corp — [Exit]")

### Platform Usage Dashboard
Cross-tenant aggregate metrics:
```
Total Tenants:  3         Active This Month: 3
Total Rows:     368K      Total Eval Runs:   46
MRR:            ₹2,40,000 Churn:             0
```

Charts:
- Monthly row ingestion trend (line chart, per tenant stacked)
- Eval run frequency per tenant (bar chart)
- Plan distribution (donut: 1 Enterprise, 1 Pro, 1 Starter)

### Billing View
Per tenant:
- Plan, monthly fee, next renewal, payment status
- [Change Plan] [Suspend] [Send Invoice]

### Alerts View
- "HealthBot India at 82% row limit — 5 days left in billing cycle"
- "Legal AI Corp eval runs at 62% with 12 days remaining"
- All alerts sorted by severity (red → yellow → blue)

### Platform Config
- Feature flags per plan (toggle sovereign deployment, custom roles, etc.)
- Plan definitions: edit row limits, eval limits, seat limits per plan tier
- [+ Create New Tenant] — provision a new org

---

## 17. UPDATED DEMO CREDENTIALS (COMPLETE LIST)

```
// Lynkstr Labs — Super Admin
superadmin@cueval.ai     / demo123   → Super Admin (cross-tenant)

// IITM Pravartak AI Lab — Enterprise
admin@cueval.ai          / demo123   → Admin
pm@cueval.ai             / demo123   → Project Manager
architect@cueval.ai      / demo123   → Architect
mlengineer@cueval.ai     / demo123   → ML Engineer
reviewer@cueval.ai       / demo123   → Reviewer
annotator@cueval.ai      / demo123   → Annotator

// Legal AI Corp — Pro
admin@legalai.com        / demo123   → Admin
ml@legalai.com           / demo123   → ML Engineer

// HealthBot India — Starter
admin@healthbot.in       / demo123   → Admin (sees upgrade CTAs)
```

Show all available logins on the login screen as a collapsible "Demo accounts" panel — makes it easy to switch roles during demos.

---

## 18. UPDATED IMPLEMENTATION NOTES

16. All data objects must include `tenantId` field. All query functions must filter by `currentTenant.id` unless `currentUser.role === 'SuperAdmin'`.
17. SuperAdmin login renders a completely different shell — no project/release bar, different dock, tenant-management UI.
18. Tenant context set at login, stored in `currentTenant` JS variable, never changes until logout.
19. Usage meters recalculate from mock data on each settings page render.
20. HealthBot India must visually show limit warnings throughout — top bar badge, settings meters, upload blocked state.
21. Impersonation mode: SuperAdmin can enter a tenant's context (read-only) — show persistent banner "Impersonating [Tenant Name]" with [Exit Impersonation] button.
22. Login screen: show "Demo accounts ▾" collapsible section listing all 10 credentials grouped by tenant.

---

## 19. DOCUMENT INGESTION MODULE

### Supported Input Formats
PDF (digital), PDF (scanned/image-based), DOCX, DOC, PPT, PPTX, JPG, PNG, TIFF

### New Navigation Dock Icon
Add between Datasets and Review Queue:
```
📄  Documents  — document upload and ingestion pipeline
```

### Document Ingestion Screen

**Upload panel:**
- Drag and drop zone accepting all document formats listed above
- Auto-detects format on drop
- Shows document type badge: "Digital PDF" | "Scanned PDF" | "DOCX" | "Image"
- Bulk upload supported (up to 20 files per batch)

**Per-document pipeline — mock progress steps (animated, sequential):**
```
Digital PDF:
→ Extracting text... → Detecting structure... → Chunking... → Done

Scanned PDF / Image:
→ Pre-processing (deskew, denoise)...
→ Detecting layout (columns, tables, headings)...
→ Detecting script per region (English / Tamil / Hindi / Telugu)...
→ Running OCR...
→ Scoring confidence per region...
→ Post-processing (Unicode normalise, ligature fix)...
→ Done — X regions extracted, Y flagged for human correction
```

**Document list view (after processing):**
Table showing:
- Document name
- Type badge (Digital / Scanned)
- Pages
- Scripts detected (language badges: EN, HI, TA, TE, KN, ML)
- Regions extracted
- Avg OCR confidence (color-coded: green >85%, yellow 60-85%, red <60%)
- Status: Processed | Needs Review | Ready for Chunking

### OCR Correction Queue (new task type in Review Queue)

When OCR confidence < 60% on any region, creates a correction task:

Split view:
- Left: image of the scanned region (original scan)
- Right: editable OCR text output
- Confidence score shown per region
- Annotator corrects errors directly in the text field
- [Accept] [Edit & Accept] [Flag for Expert]
- Language badge showing detected script for that region

Separate tab in Review Queue: **Datasets** | **OCR Corrections**

Badge on dock icon shows combined pending count.

### Chunking Screen (after OCR/extraction complete)

User selects a processed document and configures chunking:

**Strategy selector (radio buttons with preview):**
- Structural — split at headings/sections (best for contracts, circulars with clear structure)
- Semantic — split when topic shifts (best for research papers)
- Paragraph — split at paragraph boundaries (safe default for unstructured scans)
- Fixed size — split every N tokens (fallback)

**Live preview panel:**
- Left: extracted text with chunk boundaries highlighted
- Right: resulting chunks listed with token count per chunk
- Slider to adjust chunk size / overlap
- [Apply Chunking] button

### Instruction Pair Synthesis Screen

After chunking, user can optionally generate instruction pairs:

**Configuration:**
- Pair types (multi-select): QA Pairs | Summarisation | Information Extraction | Instruction Following
- Pairs per chunk: 1 / 2 / 3 (slider)
- LLM to use: Mock (prototype) | Claude API | Local Model
- [Generate Pairs] → animated progress per chunk

**Results:**
- Generated pairs shown in table (instruction + output per row)
- Source chunk visible on click (right panel shows chunk text + page reference)
- Each pair has: confidence score, source chunk link, pair type badge
- [Send to Curation Pipeline] → routes all pairs into existing dataset flow with quality scoring

### Mock Logic for Document Module

**OCR confidence simulation:**
```
Digital PDF: confidence always 95–99% (text extracted directly)
Scanned English: confidence 75–92% randomly distributed
Scanned Indic: confidence 45–80% with ~30% of regions below 60%
Tables: confidence 50–75%
```

**Script detection simulation:**
Assign scripts based on document name:
- Files with "tamil" / "tn" in name → Tamil regions detected
- Files with "hindi" / "central" → Hindi regions
- Otherwise → English default

**Instruction pair synthesis simulation:**
Generate 2–3 plausible QA pairs per chunk using template:
```
Q: "What does this section describe?"
A: [first 2 sentences of chunk]

Q: "Summarise the following: [chunk title]"
A: [condensed version of chunk]
```

### Mock Document Data

Pre-load 3 sample documents per tenant:

**IITM Pravartak:**
- "Audit_Observation_Report_2024.pdf" — Scanned, Hindi+English, 24 pages, 6 low-confidence regions
- "Legal_Framework_AI_Guidelines.pdf" — Digital, English, 18 pages, clean
- "Tamil_Heritage_Research_Paper.pdf" — Scanned, Tamil+English, 12 pages, 14 low-confidence regions

**Legal AI Corp:**
- "Contract_Template_v3.pdf" — Digital, English, 8 pages
- "Supreme_Court_Judgment_2023.pdf" — Scanned, English+Hindi, 32 pages, 9 low-confidence regions

**HealthBot India:**
- "Clinical_Guidelines_Hindi.pdf" — Scanned, Hindi, 16 pages, 22 low-confidence regions (most flagged — drives upgrade CTA)

### Updated Dashboard CTAs

Add document-aware CTAs:

- ML Engineer: "14 regions pending OCR correction in Tamil Heritage paper" → [Correct OCR]
- Architect: "Tamil Heritage paper ready for chunking" → [Configure Chunking]
- PM: "3 documents processed, 2 awaiting OCR review" → [View Documents]
- Annotator: "8 OCR corrections assigned to you" → [Start Corrections]

### Updated Data Model

```javascript
documents = [
  {
    id, tenantId, projectId,
    name, format,              // pdf-digital | pdf-scanned | docx | image
    pages, status,
    scriptsDetected: [],       // ['en', 'ta', 'hi']
    avgOcrConfidence,          // null for digital PDFs
    regionsTotal,
    regionsFlagged,            // confidence < 60%
    chunks: [...],
    synthesisedPairs: [...],   // links to dataset rows after synthesis
    uploadedBy, uploadedAt
  }
]

ocr_regions = [
  {
    id, documentId, tenantId,
    pageNumber, regionIndex,
    regionType,                // text | table | heading | figure
    script,                    // en | ta | hi | te | kn | ml
    rawText,                   // OCR output before correction
    correctedText,             // after human review
    confidence,                // 0-100
    correctionStatus,          // pending | corrected | accepted | flagged
    correctedBy, correctedAt
  }
]

chunks = [
  {
    id, documentId, tenantId,
    chunkIndex,
    text, tokenCount,
    strategy,                  // structural | semantic | paragraph | fixed
    sourcePages: [],           // which pages this chunk spans
    headingContext,            // nearest heading above this chunk
    synthesisedPairIds: []     // linked dataset rows
  }
]
```

### Updated Command Palette

Add searchable actions:
- "Upload document"
- "Correct OCR regions"
- "Configure chunking for [document name]"
- "Generate pairs from [document name]"
- Search documents by name

---

## 20. BULK INGESTION MODULE (CORPUS SCALE)

### Overview
Separate mode from standard document upload. Designed for 10K–500K document corpus ingestion. Accessible via UI (this prototype) and eventually via API/CLI. Demonstrates scale-aware architecture visually even in the mock.

---

### New Navigation Item
Add under Documents dock icon — secondary tab:
```
📄  Documents
    ├── My Documents    (existing — single/small batch upload)
    └── Bulk Ingestion  (new — corpus scale)
```

---

### Bulk Ingestion Screen

**Entry point:**
Large card on Documents screen:
```
┌─────────────────────────────────────────┐
│  📦  Bulk Corpus Ingestion              │
│  For large document sets (1K–500K PDFs) │
│  API, folder path, or S3 bucket         │
│  [Start Bulk Job]                       │
└─────────────────────────────────────────┘
```

---

### New Bulk Job Wizard (3 steps)

**Step 1 — Source Configuration**
```
Source type (radio):
○ Upload folder (zip)
○ S3 / Object Storage bucket path
○ API push (show endpoint + token)

Bucket path:  [s3://pravartak-corpus/audit-docs/          ]
Credentials:  [Auto-detected from server config            ]

Document types to include (checkboxes):
☑ PDF (digital)   ☑ PDF (scanned)   ☑ DOCX   ☐ Images only

Estimated count: [Auto-detected after path scan: 51,240 files]
Estimated size:  [Auto-detected: 284 GB                       ]
```

**Step 2 — Processing Configuration**
```
OCR Language Pack:
☑ English   ☑ Hindi   ☑ Tamil   ☐ Telugu   ☐ Kannada   ☐ Malayalam

Chunking strategy (applied to all):
○ Auto-detect per document (recommended)
○ Structural   ○ Semantic   ○ Paragraph   ○ Fixed size

Instruction pair synthesis:
○ Skip (extract text + chunks only)
○ Generate pairs after chunking
  Pair types: ☑ QA   ☑ Summarisation   ☐ Extraction

Deduplication:
☑ File-level (SHA256 — instant, always on)
☑ Content near-duplicate (MinHash — runs after extraction)
  Similarity threshold: [0.92 ▾]

Worker allocation:
CPU workers:  [20 ▾]   (parallel document processors)
GPU workers:  [2  ▾]   (scanned OCR acceleration, if available)

Estimated processing time: ~2.4 days
Estimated compute cost:    ~₹11,200 (one-time burst)
```

**Step 3 — Review Queue Configuration**
```
OCR confidence threshold for human review:
Auto-route regions below [60%▾] confidence to correction queue

Annotation team assignment:
Language    Assigned to            Target SLA
English  →  [Batch Auto-assign ▾]  [3 days  ▾]
Hindi    →  [Ravi Kumar        ▾]  [5 days  ▾]
Tamil    →  [Priya S           ▾]  [7 days  ▾]

Skip correction for documents where:
○ Avg confidence > 85% (accept automatically)
○ Never skip — review everything
● Skip if avg confidence > 75% (recommended)

[Start Ingestion Job]
```

---

### Bulk Job Dashboard (main view after job starts)

Full-screen live status view. Mock animates counters incrementing in real time.

**Top strip — key numbers:**
```
Total Files    Queued      Processing   Complete    Failed    Skipped (dup)
  51,240       31,840        200         18,942       258        1,847
```
Progress bar across full width: 37% complete. Estimated completion: 1d 14h remaining.

**Pipeline stage funnel (animated):**
```
Detected      →  Deduped    →  Extracted  →  OCR'd     →  Chunked   →  Ready
51,240           49,393        41,200        31,840        18,942       12,105
                 -1,847 dups   -8,193        -9,360        -12,898
                               (failed/      (queued)      (queued)
                                queued)
```
Each stage shows count + drop-off explanation.

**Live worker status panel:**
```
Worker          Status          Current Doc              Speed
cpu-worker-01   Processing      audit_report_4521.pdf    12 docs/min
cpu-worker-02   Processing      circular_TN_2019.pdf     8 docs/min
cpu-worker-03   Idle            —                        —
...
gpu-worker-01   OCR Running     scanned_order_8821.pdf   4 docs/min
gpu-worker-02   OCR Running     gazette_hindi_003.pdf    3 docs/min
```

**Failure log (collapsible panel):**
```
Doc ID      Filename                    Stage       Reason              Action
doc_4821    corrupted_scan_991.pdf      Pre-process Unreadable file      [Skip] [Retry]
doc_7103    arabic_contract.pdf         OCR         Unsupported script   [Skip]
doc_9341    empty_document.pdf          Extract     No text found        [Skip]
```
[Retry All Failed] [Export Failure Log]

**Deduplication report:**
```
1,847 duplicates detected and skipped
  - 1,203 exact duplicates (SHA256 match)
  - 644 near-duplicates (content similarity > 0.92)

Top duplicate clusters:
  "Budget_Circular_2023.pdf" — 47 identical copies
  "Standard_Operating_Procedure.pdf" — 31 copies
  [View All Clusters]
```

---

### Annotation Project View

When bulk job generates a large OCR correction backlog, it creates an **Annotation Project** — not just a queue.

Accessed from: Review Queue → OCR Corrections → [View as Project]

**Project header:**
```
Annotation Project: Audit Corpus OCR Review
Documents: 51,240   Regions flagged: 14,832   Corrected: 3,241 (21%)
Team: 4 annotators   Target: 7 days   Status: In Progress
```

**Assignment table:**
```
Annotator       Language    Assigned    Complete    Remaining   Pace        ETA
Ravi Kumar      Hindi       4,200       1,840       2,360       420/day     5.6 days
Priya S         Tamil       6,100       890         5,210       180/day     28 days ⚠️
Auto-assign     English     3,200       510         2,690       —           —
Unassigned      Telugu      1,332       0           1,332       —           —  ⚠️
```

Warnings surface automatically:
- Priya S at current pace won't meet deadline → [Reassign some load]
- Telugu has no annotator assigned → [Assign Now]

**Document-level progress (table):**
```
Document                        Pages  Regions  Corrected  Status
audit_report_TN_2021.pdf         48     312       312      ✅ Complete
gazette_notification_2019.pdf    12     84        41       🔄 In Progress
scanned_circular_hindi_88.pdf    6      156       0        ⏳ Not Started
```

Click any document → see its regions, correction status, assigned annotator.

---

### Cost Tracker (shown throughout bulk job)

Persistent panel on bulk ingestion dashboard:

```
Compute Cost Breakdown
──────────────────────────────────
CPU workers (20×, 2.4 days)    ₹8,400
GPU workers (2×, 2.4 days)     ₹2,800
Storage (284GB raw + outputs)  ₹1,200
LLM pair synthesis (est.)      ₹3,600
──────────────────────────────────
Total estimated                ₹16,000
Spent so far                   ₹5,940
```

---

### Mock Data for Bulk Ingestion

Pre-load one completed bulk job per tenant to show history:

**IITM Pravartak:**
```javascript
{
  name: "Audit Corpus 2015–2024",
  totalFiles: 51240,
  status: "in_progress",
  complete: 18942,
  failed: 258,
  duplicatesSkipped: 1847,
  ocrRegionsFlagged: 14832,
  ocrRegionsCorrected: 3241,
  workersActive: 22,
  startedAt: "2025-06-28T09:00:00",
  estimatedCompletion: "2025-07-01T11:30:00",
  computeCostSoFar: 5940
}
```

**Legal AI Corp:**
```javascript
{
  name: "Legal Document Corpus v1",
  totalFiles: 12400,
  status: "complete",
  complete: 12400,
  failed: 43,
  duplicatesSkipped: 892,
  ocrRegionsFlagged: 2841,
  ocrRegionsCorrected: 2841,
  startedAt: "2025-06-01T08:00:00",
  completedAt: "2025-06-03T14:22:00",
  computeCostTotal: 4200
}
```

**HealthBot India:**
```javascript
{
  name: "Clinical Guidelines Hindi",
  totalFiles: 3200,
  status: "complete",
  complete: 3200,
  failed: 12,
  duplicatesSkipped: 341,
  ocrRegionsFlagged: 8920,
  ocrRegionsCorrected: 4100,   // only 46% corrected — at plan limit
  startedAt: "2025-06-10T10:00:00",
  completedAt: "2025-06-12T16:00:00",
  computeCostTotal: 2800,
  note: "OCR correction paused — row limit reached. Upgrade to continue."
}
```

HealthBot India shows upgrade CTA prominently in the annotation project view — 4,820 regions uncorrected because they hit their Starter plan row limit.

---

### Updated Dashboard CTAs for Bulk Jobs

```
Admin:       "Bulk job 37% complete — 1d 14h remaining" → [View Job]
PM:          "Tamil annotation team behind schedule — 28 days at current pace" → [Reassign]
ML Engineer: "18,942 documents ready for chunking" → [Configure Chunking]
Architect:   "1,847 duplicates removed — view dedup report" → [View Report]
Annotator:   "You have 2,360 Hindi corrections remaining" → [Continue]
```

---

### Updated Command Palette Actions

```
"Start bulk ingestion job"
"View bulk job status"
"View deduplication report"
"Reassign annotation workload"
"Export failure log"
"Retry failed documents"
"View annotation project"
```

---

### Implementation Notes for Claude Code

23. Bulk job dashboard counters should animate — increment mock numbers every 2 seconds to simulate live processing. Use setInterval with realistic increments (10–15 docs/second across all workers).
24. Pipeline funnel numbers must stay mathematically consistent as they animate — complete + processing + queued must always sum to total minus duplicates minus failed.
25. Worker status table updates every 3 seconds — rotate which worker is processing which document name from a pre-defined list.
26. Annotation project ETA calculates from pace — if pace changes, ETA updates. Show ⚠️ if ETA exceeds target.
27. Cost tracker increments in real time proportional to job progress.
28. HealthBot India bulk job must show upgrade CTA blocking further OCR correction — annotation project view shows a locked state with [Upgrade Plan] button.
29. All bulk job history persists in mock data — switching tenants shows that tenant's jobs only.

---

## 21. AUDIO & VIDEO INGESTION MODULE

### Overview
Third ingestion modality alongside text datasets and documents. Same unified curation, review, and export layer underneath. Modality-specific processing upstream. Designed to show the concept clearly in the prototype — mock media, real UI.

---

### Supported Formats
```
Audio: MP3, WAV, FLAC, OGG, M4A
Video: MP4, MOV, AVI, MKV
```

---

### Navigation
Documents dock icon expands to three tabs:
```
📄  Documents
    ├── Text & Docs     (PDF, DOCX — existing)
    ├── Audio           (new)
    └── Video           (new)
```

---

### Audio Screen

**Upload panel:**
- Drag and drop zone accepting audio formats
- Shows duration, file size, format badge on drop
- Bulk upload supported
- Estimated processing time shown: "~4 min per hour of audio on GPU"

**Audio pipeline — mock progress steps (animated):**
```
→ Normalising format (16kHz WAV)...
→ Checking audio quality (SNR, clipping, silence)...
→ Detecting language / speaker count...
→ Transcribing (Whisper Indic)...
→ Scoring transcript confidence per word...
→ Segmenting into training clips...
→ Done — 847 segments created, 312 flagged for review
```

**Audio file list view:**
```
Filename              Duration   Language   Segments   Avg SNR   Transcript Conf   Status
court_hearing_01.mp3  1h 24m     Hindi+En   203        18dB      73%               ⚠️ Review
field_interview_ta.wav 42m       Tamil      98         12dB      54%               🔴 Needs Work
customer_call_02.mp3  18m        English    41         24dB      91%               ✅ Ready
```

Color coding:
- Transcript confidence > 85%: green
- 60–85%: yellow
- < 60%: red — human correction mandatory

**Audio detail view (click a file → right panel):**
```
┌─────────────────────────────────────────┐
│  court_hearing_01.mp3                   │
│  Duration: 1h 24m  |  Language: Hi+En  │
│  Segments: 203     |  Flagged: 67      │
│                                         │
│  Quality Metrics                        │
│  SNR:              18dB   ████░░  Good  │
│  Transcript Conf:  73%    ███░░░  Fair  │
│  Speaker count:    3      (multi)       │
│  Clipping:         None   ✅            │
│  Silence > 3s:     12 instances ⚠️      │
│                                         │
│  Language breakdown                     │
│  Hindi    62%  ████████░░               │
│  English  38%  █████░░░░░               │
│                                         │
│  [View Segments] [Start Review]         │
└─────────────────────────────────────────┘
```

**Segment table (inside audio detail):**
```
Seg    Timestamp      Duration   Transcript (truncated)         Conf   Status
001    00:00–00:08    8s         "न्यायालय की कार्यवाही..."      89%    ✅
002    00:08–00:21    13s        "The defendant claims that..."  94%    ✅
003    00:21–00:29    8s         "aur isliye hamara [unclear]"   41%    🔴
004    00:29–00:45    16s        "Section 302 ke antargat..."    71%    ⚠️
```

---

### Audio Correction Queue (new task type in Review Queue)

New tab: **Datasets** | **OCR Corrections** | **Audio Corrections** | **Video Corrections**

**Audio correction task UI:**

```
┌──────────────────────────────────────────────────────────┐
│  Audio Correction    Segment 003 of 203   [◄] [►]        │
│  court_hearing_01.mp3 | Hindi | 00:21–00:29              │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  [Waveform — static SVG visualisation]                   │
│  ▐▌▐▐▌▌▐▐▌▐▌▐▐▌▌▐▐▌▐▌▐▐▌▌▐▐▌▐▌▐▐▌▌▐▐▌▐▌▐▐▌▌▐▐▌▐▌       │
│  ◄ 00:21 ──────────────────────────── 00:29 ►           │
│                                                          │
│  [▶ Play] [⟳ Replay last 3s] [0.75x] [1x] [1.5x]       │
│                                                          │
├──────────────────────────────────────────────────────────┤
│  Auto Transcript (Confidence: 41%)                       │
│  ┌────────────────────────────────────────────────────┐  │
│  │ aur isliye hamara [unclear] ke saath milkar        │  │
│  │ [inaudible] kar sakte hain                         │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  Your Correction                                         │
│  ┌────────────────────────────────────────────────────┐  │
│  │ aur isliye hamara paksh ke saath milkar            │  │
│  │ nirnay kar sakte hain                              │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  Flag:  ○ Background noise  ○ Wrong speaker             │
│         ○ Unclear speech    ○ Domain term error         │
│         ○ Unusable — discard this segment               │
│                                                          │
│  [Accept Auto] [Save Correction] [Discard] [Skip →]     │
│  Keyboard: A=Accept  S=Save  D=Discard  →=Skip          │
└──────────────────────────────────────────────────────────┘
```

Note: In prototype, waveform is a static SVG illustration. Play button shows a toast "Audio playback simulated". The correction workflow is fully functional with mock data.

---

### Video Screen

**Upload panel:**
- Drag and drop zone accepting video formats
- Shows duration, resolution, file size on drop
- Estimated processing time: "~12 min per hour of video"

**Video pipeline — mock progress steps:**
```
→ Extracting audio track...
→ Transcribing audio (Whisper)...
→ Extracting keyframes (1 per scene change)...
→ Filtering frames (blur, dark, duplicates)...
→ Generating captions per frame...
→ Detecting on-screen text (OCR on frames)...
→ Scoring visual quality per frame...
→ Synthesising VideoQA pairs...
→ Done — 1,240 frames extracted, 284 captioned, 156 flagged
```

**Video file list view:**
```
Filename               Duration  Resolution  Frames  Captions  Visual Score  Status
surgery_demo_01.mp4    22m       1080p       440     440       82%           ✅ Ready
field_audit_tn.mp4     1h 8m     480p        820     612       54%           ⚠️ Review
lecture_hindi_03.mp4   48m       720p        960     960       91%           ✅ Ready
```

**Video detail view (right panel):**
```
┌─────────────────────────────────────────┐
│  field_audit_tn.mp4                     │
│  Duration: 1h 8m  |  Language: Ta+En   │
│  Frames extracted: 820                  │
│  Flagged: 208                           │
│                                         │
│  Quality Metrics                        │
│  Visual clarity:   54%   ███░░░  Fair  │
│  Audio quality:    71%   ████░░  Good  │
│  Transcript conf:  61%   ███░░░  Fair  │
│  Blurry frames:    124 removed          │
│  Dark frames:      43 removed           │
│  Scene changes:    38 detected          │
│                                         │
│  Content type (detected):               │
│  ● Documentary / Field recording        │
│                                         │
│  [View Frames] [View Transcript]        │
│  [Start Review] [Generate QA Pairs]     │
└─────────────────────────────────────────┘
```

**Frame gallery view:**
Grid of extracted keyframes (mock thumbnail images — use placeholder colored rectangles with frame number overlay in prototype):

```
┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
│ 001 │ │ 002 │ │ 003 │ │ 004 │
│ ✅  │ │ ⚠️  │ │ ✅  │ │ 🔴  │
└─────┘ └─────┘ └─────┘ └─────┘
00:00   00:12   01:04   01:45
```

Click frame → right panel shows full frame detail with caption editor.

---

### Video Correction Queue

**Video correction task UI:**

```
┌──────────────────────────────────────────────────────────┐
│  Video Correction   Frame 156 of 820   [◄] [►]           │
│  field_audit_tn.mp4 | Tamil+English | 01:45              │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────┐               │
│  │                                      │               │
│  │   [Video frame placeholder —         │               │
│  │    grey rectangle with timestamp     │               │
│  │    and frame number overlay]         │               │
│  │                                      │               │
│  │   Frame 156 | 01:45                  │               │
│  └──────────────────────────────────────┘               │
│  [◄ Prev frame] [► Next frame] [▶ Play clip ±5s]        │
│                                                          │
├──────────────────────────────────────────────────────────┤
│  Auto Caption (Visual Score: 41%)                        │
│  "A person is standing near some equipment outdoors"     │
│                                                          │
│  On-screen text detected:                               │
│  "Tamil Nadu PWD Inspection — March 2024"               │
│                                                          │
│  Your Caption                                            │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Field engineer inspecting irrigation canal         │  │
│  │ infrastructure, Tamil Nadu PWD site visit          │  │
│  │ March 2024                                         │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  Generate QA pair from this frame:                       │
│  Q: [What is the engineer inspecting?              ]     │
│  A: [Irrigation canal infrastructure               ]     │
│  [+ Add QA pair]                                        │
│                                                          │
│  Flag:  ○ Blurry/unclear  ○ Wrong scene boundary       │
│         ○ Sensitive content  ○ Discard frame            │
│                                                          │
│  [Accept Auto] [Save Caption] [Discard] [Skip →]        │
└──────────────────────────────────────────────────────────┘
```

---

### Quality Metrics — Audio & Video Scoring

Add to existing quality scorer mock logic:

**Audio quality score:**
```
score = 100
if SNR < 10dB:          score -= 40   // very noisy
if SNR < 15dB:          score -= 20   // noisy
if transcript_conf < 50: score -= 35  // low confidence
if transcript_conf < 70: score -= 15
if speaker_count > 2:   score -= 10   // multiple speakers
if silence_ratio > 0.3: score -= 20   // too much silence
if duration < 3:        score -= 30   // too short
if duration > 30:       score -= 10   // consider splitting
```

**Video quality score:**
```
score = 100
if blur_score < 40:     score -= 35   // blurry frame
if darkness < 30:       score -= 30   // dark frame
if caption_conf < 50:   score -= 25   // poor auto-caption
if is_duplicate_frame:  score -= 50   // near-identical to adjacent frame
if has_faces and no_consent_flag: score -= 20  // flag for review
```

---

### Mock Data — Audio & Video

**IITM Pravartak:**
```javascript
audioFiles = [
  {
    name: "court_hearing_01.mp3",
    duration: "1h 24m", language: "hi+en",
    segments: 203, flagged: 67,
    avgSnr: 18, transcriptConf: 73,
    status: "in_review"
  },
  {
    name: "field_interview_tamil.wav",
    duration: "42m", language: "ta",
    segments: 98, flagged: 52,
    avgSnr: 12, transcriptConf: 54,
    status: "needs_work"
  }
]

videoFiles = [
  {
    name: "heritage_site_scan_hampi.mp4",
    duration: "38m", resolution: "4K",
    frames: 760, captioned: 612, flagged: 148,
    visualScore: 78, status: "in_review"
  }
]
```

**Legal AI Corp:**
```javascript
audioFiles = [
  {
    name: "deposition_recording_2024.mp3",
    duration: "2h 12m", language: "en",
    segments: 312, flagged: 28,
    avgSnr: 26, transcriptConf: 91,
    status: "ready"
  }
]
```

**HealthBot India:**
```javascript
audioFiles = [
  {
    name: "doctor_patient_hindi_01.wav",
    duration: "18m", language: "hi",
    segments: 41, flagged: 19,
    avgSnr: 14, transcriptConf: 68,
    status: "in_review"
  }
]
videoFiles = [
  {
    name: "symptom_explainer_hindi.mp4",
    duration: "12m", resolution: "720p",
    frames: 240, captioned: 240, flagged: 31,
    visualScore: 84, status: "ready"
  }
]
```

---

### Export Formats — Audio & Video

**Audio export:**
```
ASR Training (Whisper format):
{ "audio": "segment_001.wav", "text": "transcript...", "language": "hi" }

Mozilla DeepSpeech format:
wav_filename, wav_filesize, transcript

Common Voice format:
client_id, path, sentence, up_votes, down_votes, age, gender, accent

Custom JSONL:
{ "audio_id": "...", "transcript": "...", "language": "...",
  "duration": 8.2, "speaker_id": "spk_01", "confidence": 0.89 }
```

**Video export:**
```
VideoQA JSONL:
{ "video_id": "...", "frame_timestamp": 105.3,
  "question": "...", "answer": "...", "caption": "..." }

Frame + Caption:
{ "frame_id": "...", "image_path": "...",
  "caption": "...", "on_screen_text": "...", "timestamp": 105.3 }
```

Format selector shown in export modal — same pattern as text dataset export.

---

### Updated Dashboard CTAs

```
ML Engineer:  "312 audio segments ready for ASR export" → [Export Dataset]
Annotator:    "52 Tamil audio corrections assigned to you" → [Start Corrections]
Architect:    "148 video frames pending caption review" → [Review Frames]
PM:           "field_interview_tamil.wav at 54% transcript confidence — needs attention" → [View]
```

---

### Updated Bulk Ingestion (Section 20 extension)

Add audio and video to bulk job configuration:

```
Source types (bulk):
☑ PDF (digital)   ☑ PDF (scanned)   ☑ DOCX
☑ Audio files     ☑ Video files

Audio processing:
Language packs: ☑ English  ☑ Hindi  ☑ Tamil  ☐ Telugu
Transcription model: ○ Whisper Base  ● Whisper Large  ○ IndicWhisper

Video processing:
☑ Extract keyframes      Frame interval: [Auto ▾]
☑ Generate captions      Caption model: [Mock (prototype) ▾]
☑ Extract audio track    → runs audio pipeline on extracted audio
☐ OCR on frames          (computationally expensive — opt-in)

Estimated processing (for 10K hours of audio):
With 2 GPU workers: ~83 hours
Without GPU: ~830 hours ⚠️ Not recommended at this scale
```

Bulk job pipeline funnel adds audio/video stages:
```
Detected → Deduped → Extracted/Transcribed → Segmented → Scored → Flagged → Ready
```

---

### Implementation Notes for Claude Code

30. Waveform visualisation: draw as static SVG path using sine wave approximation — no real audio processing needed. Different shape per segment to look unique.
31. Video frame thumbnails: use CSS gradient rectangles with timestamp overlay — no real video needed. Vary colours per frame to look like different scenes.
32. Play button for audio/video: show a toast notification "Playback simulated in prototype" — do not attempt real media playback.
33. Audio correction keyboard shortcuts must work: A = accept, S = save, D = discard, arrow keys = next/prev segment.
34. Video frame grid: render 12 thumbnails per page, paginate through mock frame list.
35. Export format selector for audio/video: show format options specific to modality — different from text export options.
36. Quality score colour coding consistent across all three modalities: green >85%, yellow 60–85%, red <60%.
37. Review queue tabs update badge counts independently: Datasets | OCR Corrections | Audio Corrections | Video Corrections — each shows its own pending count.
