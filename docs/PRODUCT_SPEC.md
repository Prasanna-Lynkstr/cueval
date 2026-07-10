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

---

## 22. UPDATED POSITIONING & TAGLINE

Replace all instances of tagline "From raw data to release-ready" with:

**"The continuous intelligence layer for AI model quality."**

Subline (shown on login screen and dashboard header):
**"From raw data and documents to training-ready datasets, production monitoring, and back. Every inference makes the next model smarter."**

Update product overview in Section 1 to reflect:
Cueval covers the full AI model quality loop:
Ingest → Curate → Annotate → Train → Evaluate → Deploy → Monitor → Improve → Repeat

---

## 23. INFERENCE MONITOR

### New Dock Icon
Add between Eval Harness and Experiments:
```
📡  Monitor  — live production inference monitoring
```

### Inference Monitor Screen

**Top strip — live metrics (animate every 5 seconds in mock):**
```
Inferences Today   Flag Rate   Avg Quality Score   Human Review Queue   Feedback (👍/👎)
    14,832           8.3%           81.4              47 pending           94% / 6%
```

**Flag rate trend chart (line chart, Canvas API):**
- X axis: last 14 days
- Y axis: flag rate %
- Two lines: current model version (coral) vs previous version (pink)
- Annotation on chart: "v1.2 deployed" marker
- Shows improvement or regression visually

**Live inference feed (table, mock updates every 3 seconds):**
```
Time      Model   Instruction (truncated)              Score   Flags              Action
14:32:01  v1.2    "What is the penalty under Sec 12?"  91%     None               ✅
14:32:04  v1.2    "Summarise this audit observation"   43%     Hallucination ⚠️   [Review]
14:32:07  v1.2    "List all parties in this contract"  78%     Low confidence     [Review]
14:32:09  v1.2    "क्या यह अनुबंध वैध है?"              34%     Policy violation 🔴 [Review]
14:32:12  v1.2    "Extract vendor details from..."     88%     None               ✅
```

Color code rows by score: green >80%, yellow 60–80%, red <60%.

**Failure category breakdown (donut chart, Canvas API):**
```
Hallucination:      34%  ████████░░░░
Wrong citation:     22%  █████░░░░░░░
Policy violation:   18%  ████░░░░░░░░
Low confidence:     15%  ███░░░░░░░░░
Incomplete answer:  11%  ██░░░░░░░░░░
```

**Model version comparison panel:**
```
Metric              v1.1      v1.2      Change
Flag rate           11.2%     8.3%      -2.9% ▼ ✅
Avg quality score   77.1      81.4      +4.3  ▲ ✅
Hallucination rate  5.8%      3.1%      -2.7% ▼ ✅
User feedback 👎    9.2%      6.1%      -3.1% ▼ ✅
P95 latency         420ms     380ms     -40ms ▼ ✅
```

**Inference detail (right panel on row click):**
```
┌─────────────────────────────────────────┐
│  Inference Detail                       │
│  ID: inf_8821  |  14:32:04  |  v1.2    │
├─────────────────────────────────────────┤
│  Instruction:                           │
│  "Summarise this audit observation      │
│   regarding procurement irregularity"   │
│                                         │
│  Context provided: 2 document chunks    │
│  [View chunks ▾]                        │
│                                         │
│  Model Response:                        │
│  "The procurement showed irregularities │
│   totalling ₹45 crore in FY2022..."    │
│                                         │
│  ⚠️ Hallucination detected              │
│  "₹45 crore" not found in source docs  │
│                                         │
│  Scores:                                │
│  Factuality:       31%   🔴             │
│  Completeness:     78%   🟡             │
│  Policy:           92%   ✅             │
│  Overall:          43%   🔴             │
│                                         │
│  User feedback: Not yet received        │
│                                         │
│  [Add to Training Set] [Dismiss]        │
│  [Request Human Review]                 │
└─────────────────────────────────────────┘
```

[Add to Training Set] creates a new row in the current project's dataset with:
- instruction = the original query
- output = blank (annotator will write corrected response)
- flags = ['from_production', 'hallucination_detected']
- review_status = 'pending'

This is the loop closing — production failure becomes training example.

**Feedback Ingestion Panel (collapsible):**
```
Connected Applications:  2
└── Legal AI Assistant (prod)   → 847 inferences today, 6.1% negative
└── Audit QA Bot (staging)      → 124 inferences today, 12.3% negative

Negative feedback clusters:
"Wrong section cited"     → 34 instances  [Send to Review Queue]
"Incomplete answer"       → 22 instances  [Send to Review Queue]
"Hallucinated figure"     → 18 instances  [Send to Review Queue]
```

### Mock Inference Data

Pre-load 50 mock inferences per tenant. Mix of scores:
- 60% high quality (score >80, no flags)
- 25% medium quality (score 60–80, minor flags)
- 15% low quality (score <60, hallucination or policy flags)

Animate 3–4 new rows appearing every 5 seconds in the live feed.

---

## 24. PREFERENCE RANKING (RLHF ANNOTATION)

### New Task Type in Review Queue

New tab: **Datasets** | **OCR Corrections** | **Audio Corrections** | **Video Corrections** | **Preference Ranking**

### Preference Ranking Task UI

```
┌──────────────────────────────────────────────────────────────┐
│  Preference Ranking    Pair 23 of 150    [◄] [►]             │
│  Legal AI RLHF Dataset v1  |  English                        │
├──────────────────────────────────────────────────────────────┤
│  Instruction:                                                │
│  "Explain the force majeure clause in simple terms"          │
│  Context: [Contract Section 12.3 — view ▾]                  │
├─────────────────────┬────────────────────────────────────────┤
│  Response A         │  Response B                            │
│                     │                                        │
│  "Force majeure     │  "This clause means neither side       │
│  refers to          │  is responsible for delays caused      │
│  unforeseeable      │  by events completely outside          │
│  circumstances      │  their control — like natural          │
│  beyond the         │  disasters, government actions,        │
│  parties' control   │  or other emergencies. If something    │
│  as defined under   │  like this happens, neither party      │
│  Section 56 of      │  can be held liable for the           │
│  the Contract Act   │  resulting delay."                     │
│  1872..."           │                                        │
│                     │                                        │
│  Confidence: 71%    │  Confidence: 88%                       │
├─────────────────────┴────────────────────────────────────────┤
│  Which response is better?                                   │
│                                                              │
│  ○ Response A is better                                      │
│  ● Response B is better                                      │
│  ○ Both are equally good                                     │
│  ○ Both are bad — neither acceptable                         │
│                                                              │
│  Why? (optional)                                             │
│  [B is clearer and avoids unnecessary legal jargon    ]      │
│                                                              │
│  Dimensions (optional detail):                               │
│  Clarity:      ○A  ●B    Accuracy:   ●A  ○B                 │
│  Completeness: ○A  ●B    Tone:       ○A  ●B                 │
│                                                              │
│  [Save & Next →]  [Skip]  [Flag as Ambiguous]               │
│  Keyboard: A=Pick A  B=Pick B  T=Tie  X=Both bad  →=Skip    │
└──────────────────────────────────────────────────────────────┘
```

**Export format (RLHF preference dataset):**
```json
{
  "instruction": "Explain force majeure in simple terms",
  "chosen": "This clause means neither side...",
  "rejected": "Force majeure refers to unforeseeable...",
  "chosen_score": 0.88,
  "rejected_score": 0.71,
  "annotator_reasoning": "B is clearer and avoids unnecessary legal jargon"
}
```
Compatible with: TRL (HuggingFace), OpenRLHF, trlX.

**Inter-annotator agreement for preference:**
When 3+ annotators rank the same pair, show agreement score:
- All agree: ✅ High confidence
- 2/3 agree: ⚠️ Review recommended
- No agreement: 🔴 Ambiguous — escalate to architect

**Preference Ranking configuration (Admin/Architect):**
- Number of annotators per pair: 1 / 3 / 5 (more = higher confidence)
- Blind mode: annotators don't see which model generated which response
- Dimension weights: configure which quality dimensions matter most

### Mock Preference Data

Pre-load 150 pairs per project. Mix:
- 60% clear winner (B better — shows improvement)
- 25% close call (annotator reasoning required)
- 15% both bad (feeds back as "needs better response" task)

---

## 25. ENTITY LABELLING (STRUCTURED ANNOTATION)

### Overview
NER / information extraction annotation. Annotators highlight text spans and assign entity labels. Used to train document understanding, information extraction, and classification models.

### New Project Type

When creating a project, add project type selector:
```
Project Type:
○ LLM Fine-tuning     (existing — instruction pairs)
○ Document Ingestion  (existing — PDF/OCR pipeline)
● Entity Labelling    (new — span annotation for NER)
○ Audio / ASR         (existing)
○ Multimodal / Video  (existing)
```

Entity Labelling projects have a different primary workflow.

### Label Schema Configuration (Architect/Admin)

Before annotation begins, define the label schema:

```
Label Schema: Resume Parsing
┌─────────────────────────────────────────────────┐
│  Label       Color     Keyboard    Description   │
│  NAME        🟦 Blue   N           Person name   │
│  EMAIL       🟩 Green  E           Email address │
│  PHONE       🟨 Yellow P           Phone number  │
│  ADDRESS     🟧 Orange A           Location      │
│  COMPANY     🟥 Red    C           Employer name │
│  DESIGNATION 🟪 Purple D           Job title     │
│  SKILL       🩵 Cyan   S           Technical skill│
│  EDUCATION   🟫 Brown  U           Degree/school │
│  DATE        ⬜ White  T           Any date      │
│  [+ Add Label]                                   │
└─────────────────────────────────────────────────┘

Annotation rules:
☑ Allow overlapping spans
☑ Require minimum 1 label per document
☐ Allow blank documents (no entities)
Minimum annotators per document: [3 ▾]
```

### Entity Labelling Task UI

```
┌──────────────────────────────────────────────────────────────┐
│  Entity Labelling    Doc 47 of 100    [◄] [►]                │
│  Resume_Batch_June2025 | English | 3 annotators required     │
├─────────────────────────────────────────────────────────────-┤
│  Label Palette (click after selecting text):                 │
│  [N NAME] [E EMAIL] [P PHONE] [A ADDRESS] [C COMPANY]        │
│  [D DESIGNATION] [S SKILL] [U EDUCATION] [T DATE]           │
├──────────────────────────────────────────────────────────────┤
│  Document:                                                   │
│                                                              │
│  ╔══════════════════════════════════════════════════════╗   │
│  ║  [Rahul Sharma]NAME [rahul.s@gmail.com]EMAIL         ║   │
│  ║  [+91 98765 43210]PHONE                              ║   │
│  ║  [Mumbai, Maharashtra]ADDRESS                        ║   │
│  ║                                                      ║   │
│  ║  Senior [Software Engineer]DESIGNATION at            ║   │
│  ║  [Tata Consultancy Services]COMPANY (2019–present)   ║   │
│  ║                                                      ║   │
│  ║  Skills: [Java]SKILL, [Python]SKILL, [React]SKILL    ║   │
│  ║                                                      ║   │
│  ║  [B.Tech Computer Science]EDUCATION                  ║   │
│  ║  [IIT Bombay]EDUCATION, [2015]DATE                   ║   │
│  ╚══════════════════════════════════════════════════════╝   │
│                                                              │
│  Labelled entities (12):                                     │
│  NAME: "Rahul Sharma"     EMAIL: "rahul.s@gmail.com"        │
│  PHONE: "+91 98765..."    [View all ▾]                      │
│                                                              │
│  [Clear All] [Undo Last] [Save & Next →] [Flag] [Skip]      │
│  Select text then press label keyboard shortcut              │
└──────────────────────────────────────────────────────────────┘
```

**Interaction model:**
- Highlight text with mouse → label palette activates → click label or press shortcut
- Labelled spans highlighted in label colour with label badge overlay
- Click existing label → remove it
- Overlapping spans allowed (same text can be multiple entity types if schema permits)

### Conflict Resolution (Reviewer role)

When 3 annotators label the same document, conflicts surface:

```
┌──────────────────────────────────────────────────────────────┐
│  Conflict Review    Doc 47    3 annotators                   │
├──────────────────────────────────────────────────────────────┤
│  3 conflicts found:                                          │
│                                                              │
│  Span: "IIT Bombay"                                          │
│  Annotator 1: EDUCATION ✓                                    │
│  Annotator 2: COMPANY ✗                                      │
│  Annotator 3: EDUCATION ✓                                    │
│  → Majority: EDUCATION  [Accept] [Override]                  │
│                                                              │
│  Span: "2015"                                                │
│  Annotator 1: DATE ✓                                         │
│  Annotator 2: (not labelled)                                 │
│  Annotator 3: DATE ✓                                         │
│  → Majority: DATE  [Accept] [Override]                       │
│                                                              │
│  Span: "Mumbai, Maharashtra"                                  │
│  Annotator 1: ADDRESS ✓                                      │
│  Annotator 2: ADDRESS ✓                                      │
│  Annotator 3: (not labelled)                                 │
│  → Majority: ADDRESS  [Accept] [Override]                    │
│                                                              │
│  [Accept All Majority] [Review Each]                         │
└──────────────────────────────────────────────────────────────┘
```

### Export Formats (Entity Labelling)

```
spaCy format (JSON):
{"text": "Rahul Sharma...", "entities": [[0,12,"NAME"],[13,31,"EMAIL"]...]}

CoNLL format:
Rahul   B-NAME
Sharma  I-NAME
rahul   B-EMAIL
...

HuggingFace token classification:
{"tokens": [...], "ner_tags": [0,1,2,0,3...]}

IOB2 format (standard NER):
Rahul B-PER
Sharma I-PER
```

### Mock Entity Labelling Data

Pre-load 3 entity labelling projects across tenants:

**IITM Pravartak:**
- "Audit Report Entity Extraction" — 200 documents, labels: AMOUNT, DEPARTMENT, DATE, VIOLATION_TYPE, OFFICER_NAME
- 67% complete, 12 conflicts pending review

**Legal AI Corp:**
- "Contract NER Dataset" — 150 contracts, labels: PARTY, CLAUSE_TYPE, OBLIGATION, DATE, JURISDICTION, AMOUNT
- 34% complete

**HealthBot India:**
- "Medical Record Extraction" — 100 records, labels: PATIENT, DIAGNOSIS, MEDICATION, DOSAGE, DATE, DOCTOR
- 12% complete (just started)

### Dashboard CTAs for Entity Labelling

```
Annotator:    "38 resume documents pending entity labelling" → [Start Labelling]
Reviewer:     "12 conflict documents awaiting resolution" → [Resolve Conflicts]
Architect:    "Contract NER dataset 34% complete — on track" → [View Progress]
PM:           "Medical record project behind schedule" → [Reassign Team]
```

---

## 26. DATASET LINEAGE VIEW

### Where It Lives
Inside Experiment Tracker → click any experiment → new tab: **Lineage**

### Lineage View

Full traceability from data origin to model deployment and back:

```
┌─────────────────────────────────────────────────────────────┐
│  Dataset Lineage    Legal AI v1.2                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SOURCE ORIGINS (3 types)                                   │
│  ┌──────────────┐  ┌─────────────┐  ┌──────────────────┐   │
│  │ 📄 Documents │  │ 💬 Manual   │  │ 📡 Production    │   │
│  │ 4,200 rows   │  │ 1,840 rows  │  │ 892 rows         │   │
│  │ (PDF corpus) │  │ (annotated) │  │ (inference       │   │
│  │              │  │             │  │  corrections)    │   │
│  └──────┬───────┘  └──────┬──────┘  └────────┬─────────┘   │
│         └─────────────────┴─────────────────-┘             │
│                           │                                 │
│                    ┌──────▼──────┐                          │
│                    │  Curation   │                          │
│                    │  Pipeline   │                          │
│                    │  6,932 rows │                          │
│                    │  in → 5,841 │                          │
│                    │  out (-16%) │                          │
│                    └──────┬──────┘                          │
│                           │                                 │
│                    ┌──────▼──────┐                          │
│                    │  Human      │                          │
│                    │  Review     │                          │
│                    │  5,841 in   │                          │
│                    │  5,203 out  │                          │
│                    │  (-11%)     │                          │
│                    └──────┬──────┘                          │
│                           │                                 │
│                    ┌──────▼──────┐                          │
│                    │  Training   │                          │
│                    │  Snapshot   │                          │
│                    │  v1.2.snap  │                          │
│                    │  5,203 rows │                          │
│                    │  SHA: a3f8c │                          │
│                    └──────┬──────┘                          │
│                           │                                 │
│                    ┌──────▼──────┐                          │
│                    │  Model v1.2 │                          │
│                    │  Eval: 83.4 │                          │
│                    │  Deployed   │                          │
│                    │  Jun 2025   │                          │
│                    └──────┬──────┘                          │
│                           │                                 │
│                    ┌──────▼──────┐                          │
│                    │  Production │                          │
│                    │  Monitor    │                          │
│                    │  892 bad    │                          │
│                    │  inferences │                          │
│                    │  → corrected│                          │
│                    └──────┬──────┘                          │
│                           │                                 │
│                    Feeds into v1.3 dataset ↺               │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  Row-level traceability: search any row                     │
│  [row_id: 4821  ▸  "What penalty applies under Sec 12?"]   │
│                                                             │
│  Origin: Production inference inf_8821 (14 Jun 2025)       │
│  Flagged: Hallucination detected (score 43%)               │
│  Corrected by: Ravi Kumar (15 Jun 2025)                    │
│  Added to: Dataset v1.3 (batch B-2025-06-15)              │
│  Curation score: 87% ✅                                     │
│  Review status: Approved                                    │
│  In training snapshot: v1.3.snap (SHA: b7d2a)             │
└─────────────────────────────────────────────────────────────┘
```

---

## 27. UPDATED REVIEW QUEUE TABS

Final tab structure for Review Queue dock item:

```
Review Queue (dock badge = total across all tabs)
├── Text Review          (instruction pair approval)
├── OCR Corrections      (scanned document text fixes)
├── Audio Corrections    (ASR transcript fixes)
├── Video Corrections    (frame caption review)
├── Preference Ranking   (RLHF — pick better response)
└── Entity Labelling     (NER span annotation)
```

Each tab shows:
- Pending count (badge)
- My assigned count
- Completion % for current batch
- [Start] button routing to correct task UI

---

## 28. UPDATED IMPLEMENTATION NOTES FOR CLAUDE CODE

38. Inference monitor live feed: use setInterval (3000ms) to prepend new mock rows. Keep max 50 rows visible, remove oldest. Animate new rows sliding in from top.
39. Flag rate trend chart: draw 14-day line chart on Canvas. Two lines coral + pink. Add vertical dashed line marker for "v1.2 deployed" annotation.
40. Entity labelling span selection: use mouseup event on document text container. Get window.getSelection() to capture highlighted text + offsets. Show label palette as floating toolbar near selection.
41. Labelled spans render as inline highlighted elements with label badge. Use mark elements or span with background-color from label colour.
42. Conflict resolution: show side-by-side comparison of what each annotator labelled for conflicting spans. Majority vote auto-suggested, reviewer can override.
43. Preference ranking: A/B keyboard shortcuts must work globally while on this task. Prevent accidental navigation.
44. Dataset lineage: render as vertical flowchart using SVG. Nodes are boxes, edges are lines with arrows. Animate draw on mount (paths draw themselves top to bottom over 800ms).
45. Row-level lineage search: filter mock rows by ID or instruction text. Show full provenance chain for matching row in panel below.
46. All five annotation task types must be accessible from Review Queue. Tab switching animates (200ms fade). Badge counts update independently.
47. Inference monitor [Add to Training Set] button: adds row to current project's active dataset in mock data. Shows toast "Added to [project name] dataset". Row appears in Datasets view.

---

## 29. MONITOR CONFIGURATION SCREEN

### Where It Lives
Inference Monitor screen → top right → [Configure Monitor] button
Also accessible from: Project Settings → Monitor tab

### Monitor Configuration UI

```
┌─────────────────────────────────────────────────────────────┐
│  Monitor Configuration    Legal AI Assistant                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  MONITORING MODE                                            │
│  ○ Off           No monitoring                             │
│  ○ Safety Gate   Flag before/alongside response (high-     │
│                  stakes — medical, legal, audit)            │
│  ● Quality Loop  Collect + batch for periodic review       │
│                  (standard production model)                │
│  ○ Regression    Activate only on new deployments          │
│                  (post-deployment validation only)          │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  SCORING SIGNALS                                            │
│                                                             │
│  Signal              Enabled   Sample Rate   Weight        │
│  Factuality (NLI)    ☑         10%           [35% ▾]      │
│  Hallucination       ☑         10%           [25% ▾]      │
│  Policy Compliance   ☑         100%          [20% ▾]      │
│  Confidence Calib.   ☐         —             [0%  ▾]      │
│  Semantic Drift      ☑         10%           [10% ▾]      │
│  User Feedback       ☑         100%          [10% ▾]      │
│                                                             │
│  Weights must total 100%  Current total: 100% ✅           │
│                                                             │
│  ⚠️ Confidence Calibration requires logprob access         │
│  Not available for current model endpoint                  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  ROUTING THRESHOLDS                                         │
│                                                             │
│  Score band          Action                                 │
│  > 85%               Log only                              │
│  60% – 85%           Add to spot-check sample              │
│  < 60%               Auto-create review task               │
│  User 👎             Always create review task             │
│                                                             │
│  Repeated failure trigger:                                  │
│  Same instruction type fails [3 ▾] times → Alert Architect │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  REVIEW CADENCE                                             │
│                                                             │
│  Review mode:                                              │
│  ○ Real-time    Flag surfaces immediately in queue         │
│  ● Daily digest Batch flagged inferences, reviewed daily   │
│  ○ Weekly batch Accumulate for weekly review session       │
│                                                             │
│  Daily digest sent to: [mlengineer@cueval.ai         ]     │
│  Alert channel: ○ Email  ● In-app  ○ Webhook               │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  SAMPLING CONFIGURATION                                     │
│                                                             │
│  Traffic volume (estimated): 10,000 inferences/day         │
│                                                             │
│  Tier 1 (policy — rule based):   100% of traffic  Free    │
│  Tier 2 (NLI + hallucination):   [10% ▾] of traffic       │
│  Tier 3 (LLM judge — flagged):   Flagged only              │
│                                                             │
│  Estimated monthly cost:   ₹1,800 – ₹2,400                │
│  (based on current signal config + traffic estimate)       │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  CONNECTED APPLICATIONS                                     │
│                                                             │
│  App Name              SDK Version  Inferences/day  Status │
│  Legal AI Assistant    v1.2.1       8,420            ✅     │
│  Audit QA Bot          v1.1.0       1,240            ✅     │
│  [+ Connect New Application]                               │
│                                                             │
│  Integration snippet (copy to your application):          │
│  ┌────────────────────────────────────────────────────┐   │
│  │ from cueval import monitor                         │   │
│  │ response = monitor.generate(                       │   │
│  │   model=model,                                     │   │
│  │   instruction=instruction,                         │   │
│  │   context=context,                                 │   │
│  │   project_id="legal-ai-prod",                      │   │
│  │   model_version="v1.2"                             │   │
│  │ )                                                  │   │
│  └────────────────────────────────────────────────────┘   │
│  [Copy] [View Full SDK Docs]                               │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  USE CASE PRESETS                                           │
│                                                             │
│  Quick-configure signal weights for common use cases:      │
│                                                             │
│  [Legal / Compliance]  [Medical / Clinical]                │
│  [Customer Support]    [Government / Audit]                │
│  [Internal Knowledge]  [Custom →]                          │
│                                                             │
│  Selecting a preset updates signal weights above.          │
│  You can customise after applying preset.                  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                        [Cancel]  [Save Configuration]      │
└─────────────────────────────────────────────────────────────┘
```

### Preset Configurations (mock weight values)

**Legal / Compliance:**
Factuality 35%, Hallucination 25%, Policy 20%, Confidence 10%, Drift 10%

**Medical / Clinical:**
Factuality 40%, Hallucination 35%, Policy 15%, Confidence 10%, Drift 0%

**Customer Support:**
Factuality 20%, Hallucination 15%, Policy 30%, Confidence 5%, Drift 30%

**Government / Audit:**
Factuality 35%, Hallucination 30%, Policy 20%, Confidence 10%, Drift 5%

**Internal Knowledge:**
Factuality 15%, Hallucination 10%, Policy 25%, Confidence 5%, Drift 45%

### Score Breakdown in Inference Detail Panel

Update the inference detail right panel (Section 23) to show per-signal scores:

```
Score Breakdown:
Signal                Score    Sample?   Notes
Factuality (NLI)      31%  🔴  ✅ scored  2 claims contradicted
Hallucination         28%  🔴  ✅ scored  "₹45 crore" not in context
Policy Compliance     92%  ✅  ✅ scored  All rules passed
Confidence Calib.     —        ⬜ skipped Logprobs unavailable
Semantic Drift        71%  🟡  ✅ scored  Moderate drift from cluster
User Feedback         —        ⏳ pending No feedback yet

Weighted Overall:     43%  🔴
```

This makes clear to the user what drove the low score — not a black box number.

### Scoring Transparency Note (shown below score)

```
ℹ️ Scores approximate quality based on 5 signals.
A high score does not guarantee correctness.
A low score does not guarantee the response is wrong.
Human review makes the final determination.
```

Always visible in inference detail. Sets correct expectations.

### Mock Data — Monitor Configuration Per Tenant

**IITM Pravartak:**
- Mode: Safety Gate
- Preset: Government / Audit
- Tier 2 sample rate: 100% (all inferences scored — high stakes)
- Review cadence: Real-time
- Connected apps: 2

**Legal AI Corp:**
- Mode: Quality Loop
- Preset: Legal / Compliance
- Tier 2 sample rate: 10%
- Review cadence: Daily digest
- Connected apps: 1

**HealthBot India:**
- Mode: Regression (only active post-deployment)
- Preset: Medical / Clinical
- Tier 2 sample rate: 25%
- Review cadence: Weekly batch
- Connected apps: 1
- Note: Starter plan — monitor limited to 50K inferences/month

### Implementation Notes for Claude Code

48. Preset buttons update all weight sliders simultaneously with smooth animation (300ms transition).
49. Weight total validator: recalculates on every slider change. Shows red warning if total ≠ 100%. Save button disabled until total = 100%.
50. Confidence Calibration signal: always shown as disabled with explanation tooltip "Requires logprob access from model endpoint."
51. Cost estimator updates in real time as sample rate slider moves.
52. Integration snippet: [Copy] button copies code to clipboard, shows "Copied!" toast for 2 seconds.
53. Score breakdown in inference detail: show signal rows even when signal was not sampled — mark as "skipped" with reason.
54. Scoring transparency note: always visible, non-dismissable. Cannot be hidden by any user role.
55. Mode selector: changing to Safety Gate shows a confirmation modal — "This mode flags responses in real-time. Ensure your application handles flag callbacks. Continue?"

---

## 31. DELTA — COLD START STATE & CORPUS-FREE INFERENCE SCORING

### Corrections to Earlier Sections

**Section 23 (Inference Monitor) and Section 29 (Monitor Configuration):**
Remove any implication that Cueval stores full production inference content.

Replace with: Cueval receives embeddings and quality signals only. Raw instruction and response text stays on the client's infrastructure unless explicitly flagged for human review and the client chooses to transfer it. Sovereign clients run all scoring workers on-premise — only scores and embeddings flow, never content.

---

### Cold Start Monitor State

Triggered when: project connected to monitor AND corpus embeddings < 500.

Replaces the standard coverage indicator with a "Building" state panel.

**Coverage indicator — cold start variant:**

```
┌─────────────────────────────────────────────────────────────┐
│  Monitor: Building    Connected 4 days ago                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Signal              Status      Detail                     │
│  Policy Compliance   ✅ Active   12 rules configured        │
│  User Feedback       ✅ Active   Collecting                 │
│  Factuality (RAG)    ✅ Active   Context docs at inference  │
│  Hallucination (RAG) ✅ Active   Context docs at inference  │
│  Semantic Drift      ⏳ Warming  47 / 500 embeddings        │
│                      ████░░░░░░  ~12 days to activate       │
│  Confidence Calib.   ❌ Inactive Logprobs unavailable       │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  Already catching issues:                                   │
│  Policy failures today:        3   [Review →]              │
│  User 👎 received:             7   [Review →]              │
│  RAG hallucinations flagged:   2   [Review →]              │
│                                                             │
│  Semantic drift active in ~12 days at current pace.        │
│  Non-RAG factuality active after 500 corpus entries.       │
└─────────────────────────────────────────────────────────────┘
```

**Key design decisions:**
- Never shows "0% coverage" or broken state — always shows what IS working
- Estimated activation date calculated from daily embedding accumulation rate
- RAG factuality and hallucination explicitly shown as active — these work from day one regardless of corpus size because they check response against inference-time context documents, not Cueval's stored corpus
- Non-RAG factuality stays disabled until corpus builds — shown clearly with reason

---

### Signal Activation Rules

```javascript
// What activates at connection — no corpus needed
ALWAYS_ACTIVE = [
  'policy_compliance',   // rule-based, zero corpus dependency
  'user_feedback',       // collected passively from application
]

// Active from day one IF client provides context docs at inference time
RAG_ACTIVE = [
  'factuality',          // checks response vs inference-time context chunks
  'hallucination',       // checks entities vs inference-time context chunks
]
// If no context provided: disabled with note "Enable by providing context at inference"

// Activates when corpus embeddings >= 500
CORPUS_DEPENDENT = [
  'semantic_drift',      // needs cluster to compare against
]

// Activates only if model API exposes logprobs
API_DEPENDENT = [
  'confidence_calibration',
]
```

---

### What Cueval Actually Stores Per Inference

Corrects earlier spec sections. Cueval never stores full inference content by default.

```javascript
// What SDK sends to Cueval per inference:
{
  inference_id:     "inf_8821",        // client-generated ID
  embedding:        [0.23, -0.41, ...], // 384 floats ~1.5KB — response embedding only
  policy_signals:   {                  // computed client-side before sending
    refusal_correct: true,
    format_valid:    true,
    prohibited_terms: false,
    length_ok:       true
  },
  rag_signals:      {                  // null if no context provided
    factuality_score:    0.31,         // computed client-side
    hallucination_flags: ['₹45 crore'] // flagged entities only, not full text
  },
  model_version:    "v1.2",
  timestamp:        "2025-06-14T14:32:04Z",
  user_feedback:    null               // populated later via feedback API
}
// Raw instruction and response text: NOT sent. Stays on client.
```

Full content transferred ONLY when:
- Inference is flagged AND
- Client has configured content transfer as permitted AND
- Client is not in sovereign mode

In sovereign mode: all scoring workers run on-premise. Only scores sent to Cueval dashboard. Content never leaves client network under any configuration.

---

### Corpus Seeding from User Feedback

When 50+ thumbs-up inferences accumulated with no corpus:

**Notification (bell icon + in-app):**
```
💡 Corpus Seed Available
53 positively-rated inferences ready for corpus review.
Approving these will activate semantic drift scoring.
[Review Batch →]  — ML Engineer / Architect only
```

**Batch review UI (in Review Queue → new tab: Corpus Seeds):**
```
┌─────────────────────────────────────────────────────────────┐
│  Corpus Seeds    53 proposed entries    Batch #1            │
│  Source: User 👍 feedback (inference embeddings only)       │
├─────────────────────────────────────────────────────────────┤
│  Entry   Instruction type        Feedback   Policy score    │
│  001     Contract clause query   👍 ×3      94%             │
│  002     Section lookup          👍 ×1      91%             │
│  003     Summarisation request   👍 ×2      88%             │
│  ...                                                        │
│                                                             │
│  ⚠️ You are approving embeddings, not response content.    │
│  These entries will anchor the semantic drift cluster.     │
│  Review instruction types — ensure they represent          │
│  typical production usage before approving.                │
│                                                             │
│  [Approve All] [Review Each] [Reject Batch]                │
└─────────────────────────────────────────────────────────────┘
```

On approval: embeddings enter the corpus cluster. Semantic drift signal activates. Coverage percentage updates immediately.

Note: instruction type labels shown (e.g. "Contract clause query") are derived from embedding clustering — not from raw text. No content transferred.

---

### Updated Mock Data — Cold Start Project

Add one cold start project to IITM Pravartak tenant:

```javascript
{
  name: "Prison Welfare AI (New)",
  monitorState: "cold_start",
  connectedDaysAgo: 4,
  corpus_embeddings: 47,
  daily_embedding_rate: 38,
  estimated_drift_activation: "12 days",
  signals: {
    policy: { active: true, flags_today: 3 },
    user_feedback: { active: true, negative_today: 7 },
    factuality: { active: true, mode: "rag_only", flags_today: 2 },
    hallucination: { active: true, mode: "rag_only", flags_today: 0 },
    semantic_drift: { active: false, reason: "warming", progress: 47, target: 500 },
    confidence: { active: false, reason: "no_logprobs" }
  },
  corpus_seeds_available: 0  // below 50 threshold — notification not yet triggered
}
```

---

### Implementation Notes for Claude Code (delta)

64. Cold start state: check corpus_embeddings < 500 on project load. Render Building panel instead of standard coverage indicator. Never show a "broken" or "0%" state.
65. Embedding accumulation progress bar: animate fill proportional to 47/500. Show estimated days remaining calculated from daily_embedding_rate.
66. "Already catching issues" counts in cold start panel: pull from mock signal data. Each count is a link to the relevant review queue tab.
67. Corpus Seeds tab in Review Queue: only visible to ML Engineer and Architect roles. Hidden from Annotator, Reviewer, PM.
68. Corpus Seeds notification: show in bell dropdown when seeds >= 50. Badge on Review Queue dock icon does NOT include seed count — seeds are not review tasks, they are approval decisions.
69. What Cueval stores disclaimer: show as a one-line note under the inference detail panel: "Content stored only if flagged and transfer permitted. Embeddings and scores only by default."
70. Sovereign mode indicator: if project is configured as sovereign, show a "🔒 Sovereign" badge on the Monitor screen header. Tooltip: "All scoring runs on your infrastructure. No content leaves your network."

---

## 32. DELTA — INTEGRATION GUIDE SCREEN

### Where It Lives
Monitor Configuration → bottom of screen → [View Integration Guide →] button
Also accessible from: empty Monitor screen when no app connected → [Connect Your Application]

---

### Integration Guide Screen

Full-screen modal or dedicated view. Three tabs across top.

---

### Tab 1 — SDK Wrapper

```
┌─────────────────────────────────────────────────────────────┐
│  SDK Wrapper                                                │
│  Simplest integration. One code change. Recommended for     │
│  applications where adding a Python/JS dependency is fine.  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SETUP CHECKLIST                                            │
│  ☑ Install Cueval SDK (pip install cueval-sdk)             │
│  ☑ Add project ID and API key to environment               │
│  ☐ Wrap model call with monitor.generate()                 │
│  ☐ Pass context documents if using RAG                     │
│  ☐ Notify Cueval on model version change                   │
│  ☐ Add feedback widget to application UI                   │
│                                                             │
│  INTEGRATION SNIPPET                                        │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ from cueval import monitor                          │   │
│  │                                                     │   │
│  │ # Initialise once at app startup                    │   │
│  │ monitor.init(                                       │   │
│  │   project_id  = "legal-ai-prod",                   │   │
│  │   api_key     = os.environ["CUEVAL_KEY"],           │   │
│  │   model_ver   = "v1.2",                             │   │
│  │   sovereign   = False                               │   │
│  │ )                                                   │   │
│  │                                                     │   │
│  │ # Replace your existing model call                  │   │
│  │ # Before:                                           │   │
│  │ # response = model.generate(instruction, context)   │   │
│  │                                                     │   │
│  │ # After:                                            │   │
│  │ response = monitor.generate(                        │   │
│  │   model       = model,                              │   │
│  │   instruction = instruction,                        │   │
│  │   context     = context,   # pass if RAG            │   │
│  │ )                                                   │   │
│  │ # response is identical — no application change     │   │
│  └─────────────────────────────────────────────────────┘   │
│  [Copy]                                                     │
│                                                             │
│  FEEDBACK WIDGET SNIPPET                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ # After showing response to user:                   │   │
│  │ monitor.feedback(                                   │   │
│  │   inference_id = response.inference_id,             │   │
│  │   rating       = "positive" | "negative",           │   │
│  │   comment      = user_comment  # optional           │   │
│  │ )                                                   │   │
│  └─────────────────────────────────────────────────────┘   │
│  [Copy]                                                     │
│                                                             │
│  ⚠️ KNOWN LIMITATIONS — SDK WRAPPER                        │
│  • Context docs must be passed explicitly — not captured   │
│    automatically. Without them, factuality scoring off.    │
│  • Background embedding adds 10–50ms in separate thread.   │
│    First inference after cold start may not be embedded.   │
│  • Logprobs only captured if model returns them.           │
│    Most hosted APIs do not. Confidence signal likely off.  │
│  • Multi-turn: pass last 2 turns as context for better     │
│    drift scoring. Single-turn scoring only by default.     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Tab 2 — Sidecar Service

```
┌─────────────────────────────────────────────────────────────┐
│  Sidecar Service                                            │
│  No application code change. Proxy intercepts model calls. │
│  Best for legacy systems or multiple calling services.      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SETUP CHECKLIST                                            │
│  ☑ Deploy Cueval sidecar container                         │
│  ☐ Point application model endpoint to sidecar URL         │
│  ☐ Set original model endpoint in sidecar config           │
│  ☐ Configure project ID and API key in sidecar env         │
│  ☐ Add feedback endpoint call to application UI            │
│  ☐ Notify sidecar on model version change via API          │
│                                                             │
│  DOCKER COMPOSE SNIPPET                                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ cueval-sidecar:                                     │   │
│  │   image: cueval/sidecar:latest                      │   │
│  │   ports: ["8080:8080"]                              │   │
│  │   environment:                                      │   │
│  │     CUEVAL_PROJECT_ID: "legal-ai-prod"              │   │
│  │     CUEVAL_API_KEY:    "${CUEVAL_KEY}"              │   │
│  │     MODEL_ENDPOINT:    "http://your-model:8000"     │   │
│  │     MODEL_VERSION:     "v1.2"                       │   │
│  │     SOVEREIGN:         "false"                      │   │
│  └─────────────────────────────────────────────────────┘   │
│  [Copy]                                                     │
│                                                             │
│  APPLICATION CHANGE                                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ # Only change: point to sidecar instead of model   │   │
│  │ MODEL_URL = "http://cueval-sidecar:8080/generate"   │   │
│  │ # API signature identical to original endpoint      │   │
│  └─────────────────────────────────────────────────────┘   │
│  [Copy]                                                     │
│                                                             │
│  ⚠️ KNOWN LIMITATIONS — SIDECAR                            │
│  • Context documents not automatically captured from        │
│    RAG retrieval. Must be passed in request headers.       │
│    Requires application-side instrumentation.              │
│  • Adds one network hop — typically 2–5ms additional       │
│    latency. Negligible for most applications.              │
│  • Sidecar must be kept up to date — outdated sidecar      │
│    version may produce inconsistent scoring signals.       │
│  • Streaming responses: partial support. Scoring runs      │
│    after stream completes — not on partial tokens.         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Tab 3 — On-Premise Agent

```
┌─────────────────────────────────────────────────────────────┐
│  On-Premise Agent    🔒 Sovereign                           │
│  All scoring on your infrastructure. Nothing leaves your   │
│  network. Required for sovereign / air-gapped deployments. │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SETUP CHECKLIST                                            │
│  ☑ Deploy Cueval on-premise stack (Docker Compose)         │
│  ☐ Configure local scoring models (MiniLM, FastText)       │
│  ☐ Configure local LLM judge (Ollama / vLLM)              │
│  ☐ Point SDK or sidecar to local Cueval endpoint           │
│  ☐ Configure outbound: scores only, no content             │
│  ☐ Verify air-gap: run network isolation test              │
│                                                             │
│  WHAT STAYS ON YOUR NETWORK                                 │
│  ✅ Raw instructions and responses — never transmitted      │
│  ✅ Context documents — never transmitted                   │
│  ✅ All scoring computation — runs locally                  │
│  ✅ Embeddings — generated and stored locally               │
│  ✅ Review queue — accessed via local Cueval UI             │
│                                                             │
│  WHAT IS TRANSMITTED TO CUEVAL CLOUD (if enabled)          │
│  → Aggregated score metrics only (no content)              │
│  → Model version change notifications                      │
│  → Plan usage counts (row counts, eval counts)             │
│  → Software update checks (version numbers only)           │
│                                                             │
│  FULLY AIR-GAPPED OPTION                                    │
│  Disable all outbound. Cueval runs entirely offline.        │
│  Software updates delivered as signed package files.        │
│  Plan management handled via offline licence key.           │
│  [Configure Air-Gap Mode →]                                 │
│                                                             │
│  DOCKER COMPOSE SNIPPET                                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ # Full on-premise stack                             │   │
│  │ services:                                           │   │
│  │   cueval-api:                                       │   │
│  │   cueval-frontend:                                  │   │
│  │   cueval-worker:                                    │   │
│  │   cueval-scoring:    # local scoring models         │   │
│  │   ollama:            # local LLM judge              │   │
│  │   postgres:                                         │   │
│  │   redis:                                            │   │
│  │   minio:                                            │   │
│  └─────────────────────────────────────────────────────┘   │
│  [Copy] [Download Full Compose File]                        │
│                                                             │
│  ⚠️ KNOWN LIMITATIONS — ON-PREMISE                         │
│  • Local LLM judge (7B model) less accurate than cloud     │
│    judge. Calibration required — see Section 29.           │
│  • GPU recommended for scoring at > 1,000 inferences/day. │
│    CPU-only viable below this threshold.                   │
│  • Software updates manual — Lynkstr ships signed          │
│    packages. Auto-update not available in air-gap mode.    │
│  • Cueval dashboard accessible on local network only.      │
│    Remote access requires client's own VPN/tunnelling.     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Accuracy Expectations Panel

Below the three tabs — always visible regardless of active tab.

```
┌─────────────────────────────────────────────────────────────┐
│  What to Expect — Scoring Accuracy                         │
├──────────────────────────────┬──────────────────────────────┤
│  Scenario                    │  Realistic Accuracy          │
├──────────────────────────────┼──────────────────────────────┤
│  RAG · English · corpus >2K  │  75–82%  ✅ Most reliable   │
│  RAG · Indic · corpus >2K    │  58–68%  🟡 Improving       │
│  Non-RAG · any language      │  45–55%  🟡 Policy + feedback│
│  Cold start · corpus <500    │  30–40%  ⚠️ Building        │
├──────────────────────────────┴──────────────────────────────┤
│                                                             │
│  False positive rate (flagged but actually fine):  18–25%  │
│  False negative rate (bad but not flagged):        8–15%   │
│                                                             │
│  These figures improve as corpus grows and reviewers       │
│  provide feedback on flags. A dismissed flag teaches the   │
│  system what acceptable looks like for your domain.        │
└─────────────────────────────────────────────────────────────┘
```

---

### Disclaimers Panel

Below accuracy panel. Collapsible. Expanded by default on first visit, collapsed on return.

```
┌─────────────────────────────────────────────────────────────┐
│  Disclaimers  [▾ collapse]                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ACCURACY                                                   │
│  Cueval's inference scoring is probabilistic triage, not   │
│  verification. A high score does not certify correctness.  │
│  A low score does not certify a response is wrong. Human   │
│  review makes the final determination in all cases.        │
│                                                             │
│  COVERAGE                                                   │
│  Scoring accuracy depends on corpus size, context docs at  │
│  inference time, and policy rule configuration. A newly    │
│  connected project will have lower coverage than a mature  │
│  project. Coverage improves with annotation activity.      │
│                                                             │
│  RAG DEPENDENCY                                            │
│  Factuality and hallucination scoring only apply to RAG    │
│  inferences where context documents are provided. For      │
│  models answering from training knowledge alone, these     │
│  signals are disabled.                                     │
│                                                             │
│  INDIC LANGUAGES                                           │
│  Scoring accuracy for Indic languages is 10–20% lower than │
│  English due to limitations in available NLI and embedding │
│  models. Cueval is actively improving Indic accuracy.      │
│                                                             │
│  NOT A COMPLIANCE TOOL                                     │
│  Cueval does not certify outputs for regulated industries  │
│  (medical, legal, financial, defence). Clients are         │
│  responsible for their own compliance frameworks.          │
│                                                             │
│  DATA                                                      │
│  Cueval stores embeddings and quality signals by default,  │
│  not raw content. Content transfer only when explicitly    │
│  configured. Sovereign mode: all processing on-premise,    │
│  no data leaves client network under any configuration.    │
│                                                             │
│  MODEL INTERNALS                                           │
│  Cueval assesses observed inputs and outputs only. It      │
│  cannot explain why a model produced a response, only      │
│  whether it meets configured quality criteria.             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Version Change Notification API

Referenced in all three integration tabs. Show as a standalone callout:

```
┌─────────────────────────────────────────────────────────────┐
│  ⚠️  Always notify Cueval when you deploy a new model       │
│  version. Without this, scoring baselines drift silently.  │
│                                                             │
│  monitor.notify_deployment(                                 │
│    model_version = "v1.3",                                  │
│    deployed_at   = datetime.utcnow(),                       │
│    changelog     = "Improved factuality on legal queries"  │
│  )                                                          │
│  [Copy]                                                     │
│                                                             │
│  This call:                                                 │
│  • Marks a deployment event on the Monitor timeline        │
│  • Resets regression detection baseline to current scores  │
│  • Creates a version comparison checkpoint automatically   │
└─────────────────────────────────────────────────────────────┘
```

---

### Implementation Notes for Claude Code (delta)

71. Integration Guide opens as a full-screen overlay (z-index above everything). Close button top right. Escape key closes.
72. Tab switching in Integration Guide: instant — no animation needed. Content is dense, tab switch should feel snappy.
73. All code snippets: syntax highlighted using CSS classes (keyword, string, comment colours from design system). [Copy] button copies raw text without HTML tags.
74. Setup checklist items: first two pre-checked (☑) as "done by Cueval" steps. Remaining unchecked (☐) as "your steps". Not interactive — informational only.
75. Accuracy table: render as a styled HTML table. False positive / negative rates shown below table in plain text, not in table rows — different nature of metric.
76. Disclaimers panel: remember collapsed/expanded state in JS variable per session. Default expanded on first render of the screen.
77. Version change notification callout: shown as a distinct amber-bordered box. Not part of any tab — appears below all three tabs, always visible.
78. [Download Full Compose File] button in On-Premise tab: triggers download of a mock docker-compose.yml file with all services listed. Use Blob + createObjectURL same as dataset export.
79. Known limitations in each tab: rendered as amber warning box with ⚠️ icon. Each limitation is a separate line. Not collapsible.
80. On-Premise tab: 🔒 Sovereign badge in tab label itself — visible before clicking the tab.

---

## 30. SCORING TRANSPARENCY & COVERAGE INDICATOR

### Overview
Two additions to the Inference Monitor that set accurate expectations:
1. "Checked against" source line per signal in inference detail
2. Coverage indicator on the Monitor screen showing how reliably Cueval can score this project's inferences

---

### Updated Inference Detail Score Breakdown

Replace the existing score breakdown table in the inference detail right panel with this version:

```
Score Breakdown
──────────────────────────────────────────────────────────────
Signal            Score    Checked Against                    
──────────────────────────────────────────────────────────────
Factuality        31% 🔴   3 context chunks at inference time
                           (2 claims contradicted, 1 neutral) 

Hallucination     28% 🔴   Context chunks + domain entity list
                           "₹45 crore" absent from all sources

Policy            92% ✅   12 configured rules
                           All passed except tone threshold

Confidence        —   ⬜   Logprobs not available from endpoint
                           Cannot score — signal disabled

Semantic Drift    71% 🟡   4,203 approved responses in corpus
                           Moderate distance from cluster centre

User Feedback     —   ⏳   No feedback received yet
──────────────────────────────────────────────────────────────
Weighted Overall  43% 🔴
──────────────────────────────────────────────────────────────

ℹ️  Scores reflect what Cueval can verify against known data.
    Responses may be correct even when flagged, and incorrect
    even when not flagged. Human review makes the final call.
```

"Checked against" text is greyed out secondary text under each signal row. Not prominent — informational only. Reviewers who want to understand a flag can read it. Those who just want to act don't need to.

---

### Coverage Indicator

New panel on the Inference Monitor main screen. Positioned below the top strip metrics, above the live feed. Collapsible — collapsed by default, expandable with one click.

**Collapsed state (always visible):**
```
📊 Monitor Coverage: 68%  — Factuality + Hallucination checks limited by corpus size  [▾ Details]
```

Color coded: green >80%, yellow 60–80%, orange 40–60%, red <40%.

**Expanded state:**

```
┌─────────────────────────────────────────────────────────────┐
│  Monitor Coverage    What Cueval can reliably score         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  INFERENCE TYPE BREAKDOWN                                   │
│                                                             │
│  Type                    % of traffic   Coverage   Basis   │
│  RAG (context provided)  74%            89% ✅     Context │
│  Fine-tune only          18%            51% 🟡     Corpus  │
│  Open generation         8%             31% 🔴     Policy  │
│                                                             │
│  Overall weighted coverage: 68%                            │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  SIGNAL COVERAGE                                            │
│                                                             │
│  Signal              Coverage   Limiting factor            │
│  Policy compliance   100% ✅     Rules configured           │
│  User feedback       100% ✅     Always collected           │
│  Hallucination       71%  🟡     RAG inferences only        │
│  Factuality          71%  🟡     RAG inferences only        │
│  Semantic drift      61%  🟡     Corpus has 4,203 responses │
│                                 Improves as corpus grows   │
│  Confidence calib.   0%   🔴     Logprobs unavailable       │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  HOW TO IMPROVE COVERAGE                                    │
│                                                             │
│  ✦ Provide context documents at inference time             │
│    → Factuality + Hallucination coverage: 71% → 89%        │
│                                                             │
│  ✦ Add 2,000 more approved responses to corpus             │
│    → Semantic drift coverage: 61% → 78%                    │
│    Current corpus: 4,203  Recommended minimum: 10,000      │
│    [Go to Datasets →]                                       │
│                                                             │
│  ✦ Enable logprob access on model endpoint                 │
│    → Confidence calibration: 0% → full coverage            │
│    [View endpoint configuration →]                         │
│                                                             │
│  ✦ Collect more user feedback                              │
│    → Add feedback widget to your application               │
│    Current feedback rate: 3.2% of inferences               │
│    Recommended: > 8%                                        │
│    [Copy feedback widget snippet →]                         │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  WHAT COVERAGE MEANS                                        │
│                                                             │
│  Coverage is the % of your inferences where Cueval has     │
│  enough reference data to produce a meaningful score.      │
│                                                             │
│  Low coverage does not mean monitoring is off — policy     │
│  and user feedback are always collected. It means the      │
│  automated quality signals have limited reference data     │
│  for that portion of your inference traffic.               │
│                                                             │
│  The monitor becomes more accurate as your approved        │
│  dataset grows and your team provides feedback on flags.   │
└─────────────────────────────────────────────────────────────┘
```

---

### Coverage Calculation Logic (mock)

```javascript
function calculateCoverage(project) {
  const ragPct = project.inferenceTypes.rag / project.totalInferences
  const ftPct  = project.inferenceTypes.finetuneOnly / project.totalInferences
  const genPct = project.inferenceTypes.openGeneration / project.totalInferences

  const ragCoverage = 0.89     // high — context always available
  const ftCoverage  = 0.51     // medium — corpus-dependent
  const genCoverage = 0.31     // low — policy + feedback only

  return (ragPct * ragCoverage)
       + (ftPct  * ftCoverage)
       + (genPct * genCoverage)
}

// Semantic drift coverage scales with corpus size:
function driftCoverage(corpusSize) {
  if (corpusSize < 1000)  return 0.30
  if (corpusSize < 5000)  return 0.61
  if (corpusSize < 10000) return 0.78
  return 0.91
}
```

---

### Mock Coverage Data Per Tenant

**IITM Pravartak:**
```javascript
{
  overallCoverage: 84,
  inferenceTypes: { rag: 91, finetuneOnly: 7, openGeneration: 2 },
  corpusSize: 5203,
  feedbackRate: 6.8,
  logprobsAvailable: false,
  improvements: [
    "Enable logprob access → +8% coverage",
    "Corpus growing well — continue annotation"
  ]
}
```

**Legal AI Corp:**
```javascript
{
  overallCoverage: 68,
  inferenceTypes: { rag: 74, finetuneOnly: 18, openGeneration: 8 },
  corpusSize: 4203,
  feedbackRate: 3.2,
  logprobsAvailable: false,
  improvements: [
    "Provide context at inference time for 18% fine-tune traffic",
    "Add 2,000 more approved responses to corpus",
    "Increase feedback widget visibility — 3.2% rate is low"
  ]
}
```

**HealthBot India:**
```javascript
{
  overallCoverage: 41,
  inferenceTypes: { rag: 45, finetuneOnly: 40, openGeneration: 15 },
  corpusSize: 890,
  feedbackRate: 1.8,
  logprobsAvailable: false,
  improvements: [
    "Corpus too small — minimum 5,000 responses recommended",
    "Most inferences are fine-tune only — context not provided",
    "Upgrade plan to access full corpus size limits"
  ]
}
```

HealthBot India at 41% coverage with upgrade CTA visible in the improvements section.

---

### Corpus Growth Tracker (inside Coverage panel)

Small progress bar showing corpus growth trajectory:

```
Approved responses in corpus:
Legal AI Corp    ████████░░░░░░░  4,203 / 10,000 recommended
                 +312 this month  At current rate: full coverage in 18 months
                 [Accelerate → Run annotation sprint]
```

[Accelerate] button opens annotation queue filtered to this project — direct path to growing the corpus faster.

---

### Implementation Notes for Claude Code

56. Coverage indicator collapsed by default. Click [▾ Details] expands with smooth height animation (300ms).
57. Overall coverage number in collapsed state updates color dynamically: >80% green, 60–80% yellow, 40–60% orange, <40% red.
58. "How to improve coverage" items each have a specific CTA that navigates to the relevant screen. Navigation uses existing routing — no new screens needed.
59. Corpus growth progress bar animates fill on expand. Show recommended target (10,000) as a vertical marker on the bar.
60. "Checked against" text in inference detail is secondary grey text — font size 11px, not competing with the score itself.
61. HealthBot India coverage panel prominently shows upgrade CTA in the improvements list — styled differently (coral background) from regular improvement suggestions.
62. Coverage percentage in collapsed state recalculates when switching between projects in top bar — animates number change.
63. "What Coverage Means" section uses plain language — no technical jargon. Targeted at PM and Admin roles who may not have ML background.

---


---

## 33. DELTA — AGENTIC LAYER (PHASE 3+)

### Overview
Agentic Cueval adds an orchestration layer above existing services. The platform monitors conditions, takes permitted actions autonomously, and surfaces humans only when genuine judgment is required. Every agent action is logged, bounded, reversible where possible, and auditable. This section specifies what to show in the prototype — the UI concept — not the production implementation.

---

### New Dock Icon
Add between Monitor and Experiments:
```
🤖  Agents  — autonomous actions and orchestration log
```

---

### Agents Screen

Two panels: left = active agents and their status, right = live action log.

**Left panel — Agent roster:**

```
┌─────────────────────────────────────────────────────────────┐
│  Active Agents                                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🟢  Curation Agent         Running    Legal AI v1.3       │
│       Last action: 4 min ago                               │
│       Auto-approved 847 rows. Routed 23 to review.         │
│                                                             │
│  🟢  Annotation Router      Running    All projects        │
│       Last action: 12 min ago                              │
│       Reassigned 34 rows from Priya S (SLA risk).         │
│                                                             │
│  🟡  Eval Trigger Agent     Waiting    Legal AI v1.3       │
│       Trigger: dataset approval (pending)                  │
│       Will run eval on checkpoint v1.3 when approved.     │
│                                                             │
│  🟢  Inference Monitor Agent Running   All projects        │
│       Last action: 2 min ago                              │
│       Hallucination cluster detected → created sprint.    │
│                                                             │
│  🔴  Release Agent          Blocked    Legal AI v1.2       │
│       Blocked: Architect approval pending 2 days          │
│       Sent reminder to architect@cueval.ai                │
│                                                             │
│  ⚪  Dataset Improvement     Idle       Medical QA         │
│       Agent                                                │
│       Next scheduled analysis: 06:00 tomorrow             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Status colours:
- 🟢 Running — actively monitoring or just acted
- 🟡 Waiting — trigger condition not yet met
- 🔴 Blocked — escalation required, cannot proceed
- ⚪ Idle — scheduled, no current activity

Each agent row expandable → shows full config and permission set.

---

**Right panel — Live action log:**

```
┌─────────────────────────────────────────────────────────────┐
│  Agent Action Log    [Filter by agent ▾] [Filter by project ▾]│
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  14:32  Curation Agent  ✅ AUTO-APPROVED                    │
│  Legal AI v1.3 dataset — 847 rows (score > 85%, flags < 5%)│
│  Reasoning: threshold met, no PII detected, no toxicity    │
│  [View rows] [Undo — 8 min remaining]                      │
│  ─────────────────────────────────────────────────────────  │
│  14:28  Annotation Router  ✅ REASSIGNED                    │
│  34 rows moved from Priya S → Auto-assign pool             │
│  Reasoning: Priya S at 12% daily pace vs 28% required.    │
│  SLA breach in 1.4 days at current rate.                   │
│  [View batch] [Undo — 4 min remaining]                     │
│  ─────────────────────────────────────────────────────────  │
│  14:21  Inference Monitor Agent  ⚠️ ESCALATED              │
│  Hallucination cluster: 7 similar inferences in 90 min     │
│  All cite "Section 44(b)" — not present in corpus          │
│  Action taken: Created annotation sprint #12               │
│  Requires: ML Engineer to confirm sprint scope             │
│  [Review sprint] [Dismiss]                                 │
│  ─────────────────────────────────────────────────────────  │
│  13:58  Release Agent  📧 REMINDER SENT                    │
│  Architect approval pending 2 days for Legal AI v1.2       │
│  Sent reminder to architect@cueval.ai                      │
│  Escalates to Admin if no response in 24 hours             │
│  [View release] [Suppress reminders]                       │
│  ─────────────────────────────────────────────────────────  │
│  06:00  Dataset Improvement Agent  💡 INSIGHT              │
│  Medical QA: 15% of production query types have < 10       │
│  corpus entries. Top gap: "drug interaction queries"       │
│  Recommendation: Add 150 examples to close coverage gap    │
│  Estimated eval improvement: +6–9% on factuality          │
│  [Create annotation task] [Dismiss insight]                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Key UI patterns:
- Every action shows: what was done, reasoning, what triggered it
- [Undo] available for reversible actions, with countdown timer
- Escalations show clearly what human action is needed
- Insights are suggestions, not actions — human must choose to act

---

### Agent Configuration (per agent, per project)

Accessible from Agents screen → click any agent → [Configure]

```
┌─────────────────────────────────────────────────────────────┐
│  Configure: Curation Agent    Legal AI project              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PERMITTED ACTIONS                                          │
│  ☑ Auto-approve rows where score > [85%▾] and no flags    │
│  ☑ Route flagged rows to review queue                      │
│  ☑ Remove exact duplicates automatically                   │
│  ☐ Remove near-duplicates automatically   ← human only     │
│  ☐ Redact PII automatically               ← human only     │
│  ☑ Notify team on completion                               │
│                                                             │
│  ESCALATION RULES                                          │
│  Escalate to: [ML Engineer ▾]                              │
│  When:                                                      │
│  ☑ PII detected in > [5%▾] of rows                        │
│  ☑ Toxicity detected in any row                            │
│  ☑ Overall score < [60%▾] — dataset quality too low        │
│  ☑ Near-duplicate rate > [40%▾]                            │
│                                                             │
│  CONFIDENCE THRESHOLD                                       │
│  Below [70%▾] confidence on any decision → escalate        │
│  Agent explains reasoning in every escalation              │
│                                                             │
│  AUDIT                                                      │
│  ☑ Log every action with full reasoning                    │
│  ☑ Require human confirmation for bulk deletions > [100▾]  │
│  ☑ Daily summary sent to: [pm@cueval.ai              ]     │
│                                                             │
│  SOVEREIGN MODE                                             │
│  🔒 All agent reasoning runs on-premise                    │
│     No agent decisions sent to Cueval cloud                │
│                                                             │
│  [Cancel]  [Save Configuration]                            │
└─────────────────────────────────────────────────────────────┘
```

Hard-coded non-configurable limits (shown greyed out, non-toggleable):
- Final release approval: always human
- PII redaction above 5% of dataset: always human  
- Bulk deletion > 500 rows: always human
- Any action in sovereign deployment that isn't logged: blocked by design

---

### Dashboard Integration

Agent-aware CTAs replace standard CTAs when agents are active:

```
ML Engineer:  "Curation Agent auto-approved 847 rows — 23 need your review"
              [Review 23 rows] [View agent reasoning]

PM:           "Release Agent blocked — Architect approval 2 days overdue"
              [Send escalation] [View release gate]

Architect:    "Dataset Improvement Agent found 3 coverage gaps"
              [Review insights] [Create sprint]

Admin:        "Annotation Router reassigned 34 rows due to SLA risk"
              [View reassignments] [Undo all]

Annotator:    No agent CTAs — annotators don't see agent activity
```

---

### What Agents Cannot Do — Always Visible

Shown as a static callout on the Agents screen:

```
┌─────────────────────────────────────────────────────────────┐
│  Agent Boundaries — hardcoded, non-configurable             │
├─────────────────────────────────────────────────────────────┤
│  ✗  Agents cannot approve a release for production         │
│  ✗  Agents cannot redact or delete PII without approval    │
│  ✗  Agents cannot resolve high-conflict annotations        │
│  ✗  Agents cannot bulk-delete > 500 rows without approval  │
│  ✗  Agents cannot take action without logging reasoning    │
│  ✗  In sovereign mode: no agent reasoning leaves network   │
│                                                             │
│  Every agent action is reversible for at least 10 minutes. │
│  Every escalation includes full reasoning for human review. │
└─────────────────────────────────────────────────────────────┘
```

---

### Mock Agent Data

```javascript
agents = [
  {
    id: "curation_agent",
    name: "Curation Agent",
    status: "running",
    project: "Legal AI Assistant",
    lastAction: "4 min ago",
    lastActionSummary: "Auto-approved 847 rows. Routed 23 to review.",
    actionsToday: 3,
    escalationsToday: 0
  },
  {
    id: "annotation_router",
    name: "Annotation Router",
    status: "running",
    project: "All projects",
    lastAction: "12 min ago",
    lastActionSummary: "Reassigned 34 rows from Priya S (SLA risk).",
    actionsToday: 7,
    escalationsToday: 1
  },
  {
    id: "eval_trigger",
    name: "Eval Trigger Agent",
    status: "waiting",
    project: "Legal AI Assistant",
    waitingOn: "Dataset v1.3 approval",
    scheduledAction: "Run eval on checkpoint v1.3"
  },
  {
    id: "inference_monitor_agent",
    name: "Inference Monitor Agent",
    status: "running",
    project: "All projects",
    lastAction: "2 min ago",
    lastActionSummary: "Hallucination cluster detected → created sprint #12.",
    actionsToday: 12,
    escalationsToday: 1
  },
  {
    id: "release_agent",
    name: "Release Agent",
    status: "blocked",
    project: "Legal AI v1.2",
    blockedReason: "Architect approval pending 2 days",
    actionTaken: "Sent reminder to architect@cueval.ai",
    escalatesIn: "24 hours to Admin"
  },
  {
    id: "dataset_improvement",
    name: "Dataset Improvement Agent",
    status: "idle",
    project: "Medical QA",
    nextRun: "Tomorrow 06:00",
    lastInsight: "15% of production query types have < 10 corpus entries"
  }
]

// Pre-load 12 mock action log entries mixing all agent types
// Mix: auto-approvals, reassignments, escalations, insights, reminders
// Each with realistic reasoning text
```

---

### Implementation Notes for Claude Code (delta)

81. Agents screen: two-panel layout. Left panel fixed width 340px. Right panel fills remainder and scrolls independently.
82. Agent status dot: animate 🟢 with subtle pulse (2s infinite) when status is "running". Static for other states.
83. Action log: prepend new mock entry every 45 seconds (setInterval). Animate slide-in from top. Max 20 entries visible, remove oldest.
84. [Undo] button: countdown timer counts down from 10:00 in real time using setInterval. When reaches 0:00, button disappears and entry shows "Action permanent".
85. Agent config modal: greyed-out non-configurable items rendered with opacity 0.4 and a 🔒 icon. Not interactive. Tooltip on hover: "This limit is hardcoded and cannot be changed."
86. "Agent Boundaries" callout: always visible on agents screen, cannot be dismissed or hidden. Coral left border, dark background.
87. Dashboard CTAs: when mock data shows active agents, replace standard CTAs with agent-aware versions. Role-based as specified.
88. Escalation entries in log: amber background tint to distinguish from standard action entries.
89. Insight entries in log: blue-tinted background. [Create annotation task] CTA navigates to new task form pre-filled with agent's suggestion.
90. Sovereign mode indicator: 🔒 badge on agent config modal header and on each agent row in the roster when project is configured as sovereign.

---

## 34. DELTA — REMOVE ENTITY LABELLING, ADD ANNOTATION IMPORT CONNECTOR

### Removed from Spec
Section 25 (Entity Labelling) is withdrawn. Do not build the span annotation UI, label schema configuration, conflict resolution, or NER export formats.

Remove from:
- Review Queue tabs — remove "Entity Labelling" tab entirely
- Dashboard CTAs — remove entity labelling CTAs
- Project type selector — remove "Entity Labelling" project type
- Mock data — remove all entity labelling projects from all tenants
- Section 27 — update Review Queue tabs to 5 tabs (remove Entity Labelling)

Updated Review Queue tab structure:
```
Review Queue
├── Text Review
├── OCR Corrections
├── Audio Corrections
├── Video Corrections
└── Preference Ranking
```

---

### New: Annotation Import Connector

**Where it lives:**
Datasets screen → [Upload Dataset ▾] dropdown now has two options:
```
▾ Add Data
  ├── Upload File          (existing — JSONL, CSV, PDF, audio, video)
  └── Import from Annotation Tool  (new)
```

---

### Import Connector Screen

```
┌─────────────────────────────────────────────────────────────┐
│  Import from Annotation Tool                                │
│  Bring externally labelled data into Cueval's curation     │
│  pipeline. Cueval does not replace annotation tools —      │
│  it starts where they finish.                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SELECT SOURCE                                              │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Label Studio │  │   Argilla    │  │   Prodigy    │     │
│  │  ✅ Connected │  │  + Connect   │  │  + Connect   │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Labelbox   │  │   Scale AI   │  │  Custom JSON  │     │
│  │  + Connect   │  │  + Connect   │  │  Upload file  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  LABEL STUDIO — Connected                                   │
│                                                             │
│  Project:  [Contract NER — Legal AI Corp        ▾]         │
│  Tasks:    1,240 total  |  1,198 completed  |  42 pending  │
│                                                             │
│  Import:   ○ Completed tasks only (1,198)                  │
│            ● All tasks with completion filter              │
│            ○ Tasks annotated by specific annotators        │
│                                                             │
│  Data type detected: NER (entity spans)                    │
│                                                             │
│  CONVERSION TO CUEVAL FORMAT                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Label Studio output → Cueval instruction pair      │   │
│  │                                                     │   │
│  │  Input text + entity spans                          │   │
│  │         ↓                                           │   │
│  │  Instruction: "Extract all named entities from      │   │
│  │               the following text"                   │   │
│  │  Output: JSON of labelled entities                  │   │
│  │                                                     │   │
│  │  Conversion template: [NER → Instruction Pair ▾]   │   │
│  │  [Preview conversion (5 samples)]                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  TARGET                                                     │
│  Project:  [Legal AI Assistant               ▾]            │
│  Release:  [v1.3 — NER Training Data         ▾]            │
│                                                             │
│  After import: run full curation pipeline automatically    │
│  ☑ Quality scoring   ☑ PII detection   ☑ Near-dup check   │
│                                                             │
│  Estimated import: 1,198 rows → ~940 after curation       │
│                                                             │
│  [Cancel]  [Preview Import]  [Import Now]                  │
└─────────────────────────────────────────────────────────────┘
```

---

### Conversion Templates

When importing labelled data, Cueval converts it to instruction pairs using configurable templates. Pre-built templates for common annotation output types:

```
NER → Instruction Pair:
  Instruction: "Extract all [ENTITY_TYPES] from the following text"
  Input: [source_text]
  Output: JSON array of {text, label, start, end}

Classification → Instruction Pair:
  Instruction: "Classify the sentiment of the following text"
  Input: [source_text]
  Output: [label] — [confidence if available]

Preference Ranking → RLHF Pair:
  Directly maps to Cueval's preference format
  chosen: [higher-ranked response]
  rejected: [lower-ranked response]

Span QA → Instruction Pair:
  Instruction: [question]
  Input: [context_document]
  Output: [extracted_answer_span]

Custom template:
  User defines instruction template with variable placeholders
  Maps annotation fields to instruction/input/output
```

---

### Import History

In Datasets screen, show import provenance on dataset cards:

```
Contract NER Dataset
1,198 rows  |  Source: Label Studio (imported 14 Jun 2025)
Quality score: 79%  |  42 rows filtered by curation
[View import log]
```

Import log shows:
- Source tool and project
- Import timestamp and who triggered it
- Rows imported, rows filtered by curation and why
- Conversion template used
- Link to original Label Studio project (if accessible)

This maintains full provenance — auditors can trace a training row back to its original annotation task in the external tool.

---

### Positioning Note (visible on connector screen)

```
┌─────────────────────────────────────────────────────────────┐
│  Cueval does not replace your annotation tool.             │
│                                                             │
│  Use Label Studio, Argilla, or any tool your team prefers  │
│  for raw data labelling. Cueval starts where they finish — │
│  converting labelled data to training-ready instruction    │
│  pairs, running quality checks, and closing the loop       │
│  back from production to your next annotation sprint.      │
└─────────────────────────────────────────────────────────────┘
```

Shown at top of connector screen. Sets expectation immediately.

---

### Mock Data for Import Connector

**Label Studio — pre-configured as connected for Legal AI Corp:**
```javascript
{
  tool: "label_studio",
  status: "connected",
  project: "Contract NER — Legal AI Corp",
  tasks_total: 1240,
  tasks_complete: 1198,
  tasks_pending: 42,
  data_type: "ner",
  last_sync: "2025-06-14T09:30:00Z"
}
```

Pre-load one completed import in Legal AI Corp dataset list:
```javascript
{
  name: "Contract NER Dataset",
  rows: 1198,
  source: "label_studio",
  source_project: "Contract NER — Legal AI Corp",
  imported_at: "2025-06-14T10:12:00Z",
  rows_filtered: 42,
  filter_reasons: { low_quality: 28, near_duplicate: 11, pii: 3 },
  quality_score: 79
}
```

All other tools shown as unconnected (+ Connect state).

---

### Implementation Notes for Claude Code (delta)

91. Remove entity labelling tab from Review Queue. Update tab badge count to cover 5 tabs only.
92. Remove entity labelling CTAs from all role dashboards.
93. Remove entity labelling project type from project creation modal.
94. Remove entity labelling mock data from all three tenants.
95. [Upload Dataset ▾] becomes a dropdown with two options. On mobile-width viewports, renders as a full-width bottom sheet.
96. Import connector: Label Studio shown as connected (green badge). All others show "+ Connect" in muted style. Clicking "+ Connect" on any other tool shows a toast: "Coming soon — email prasanna@lynkstr.com to request early access."
97. Conversion template preview: clicking [Preview conversion] shows a modal with 5 mock rows showing Label Studio format on left → Cueval instruction pair on right. Static mock — no actual conversion logic needed.
98. [Import Now] triggers mock import animation: progress bar + step labels (Fetching from Label Studio → Converting format → Running curation pipeline → Done). Completes in 4 seconds. Then routes to Datasets screen showing new dataset card with "Source: Label Studio" badge.
99. Import history on dataset cards: show "Source: Label Studio" badge in muted text below row count. Clicking [View import log] opens right panel with import provenance details.
100. Positioning note callout: always visible at top of connector screen. Same styling as Agent Boundaries callout — non-dismissable, coral left border.

---

## 36. DELTA — REMOVE AUDIO/VIDEO, ADD COMING SOON PLACEHOLDERS

### Removed from Prototype Build
- Section 21 (Audio & Video Ingestion Module) — do not build
- Audio Corrections tab from Review Queue
- Video Corrections tab from Review Queue
- Audio/video mock data from all three tenants
- Whisper status panel UI (Section 35 UI components only — architecture notes retained for reference)
- Audio/video configuration from bulk ingestion wizard (Section 20)
- Audio/video CTAs from all role dashboards

---

### Updated Review Queue — 4 Tabs Only
```
Review Queue
├── Text Review
├── OCR Corrections
└── Preference Ranking
```
Remove audio/video tab badge counts. Dock icon badge = sum of above 3 tabs only.

---

### Updated Documents Screen — Coming Soon Cards

On the Documents screen, after the existing upload zone and document list, add a "More formats coming soon" section:

```
┌─────────────────────────────────────────────────────────────┐
│  More Data Types — Coming Soon                              │
├──────────────────────┬──────────────────────────────────────┤
│  🎙 Audio            │  🎬 Video                            │
│                      │                                      │
│  ASR transcription + │  Keyframe extraction +               │
│  correction workflow │  captioning for video                │
│  for speech and      │  understanding and                   │
│  voice training data │  multimodal models                   │
│                      │                                      │
│  [Notify me →]       │  [Notify me →]                       │
└──────────────────────┴──────────────────────────────────────┘
```

Styling: muted border, dark background, greyed text. Clearly not active. [Notify me →] shows a toast: "We'll reach out when audio support is available."

---

### Updated Bulk Ingestion Wizard — Section 20

Remove audio and video checkboxes from source type selector. Replace with:

```
Document types to include:
☑ PDF (digital)   ☑ PDF (scanned)   ☑ DOCX
🔒 Audio files    Coming soon
🔒 Video files    Coming soon
```

🔒 items non-interactive, muted, with tooltip "Available in a future release."

---

### Implementation Notes for Claude Code (delta)

93. Coming soon cards: render as a 2-column grid below the active documents section. Not collapsible. Greyed palette — use C.muted for text, C.border for card border.
94. [Notify me →] button: shows toast "Thanks — we'll be in touch when this is ready." No form, no email capture. Static interaction.
95. Bulk ingestion audio/video rows: render with 🔒 icon and "Coming soon" label. cursor: not-allowed. No click handler.
96. Remove all audio/video references from dashboard CTAs across all roles and tenants.

---

## §38 — Eval Harness Updates (Judge Configuration, Calibration, DLP)

### 38.1 Judge configuration panel
The Eval Harness screen leads with a **Judge configuration** panel. The eval judge is selectable between three modes (radio):
- **Local (Mistral 7B INT8)** — on-prem/in-tenant judge, no data leaves the tenant boundary. Default for privacy-sensitive tenants.
- **Cloud API** — hosted frontier judge. Selecting Cloud immediately opens the DLP acknowledgement modal (§38.4); the selection does **not** change to Cloud until the modal is confirmed.
- **Human eval** — routes rows to human reviewers; hides the reference-grounded toggle.

### 38.2 Calibration status
Each tenant has its own eval configuration (per-tenant isolation). The calibration panel shows the human↔judge agreement (Cohen's κ):
- Overall κ is color-coded: **green** > 0.70, **yellow** 0.60–0.70, **red** < 0.60 (`kappaColor`).
- A **Calibrated / Not calibrated** badge, who calibrated it, and when.
- The weakest dimension is surfaced with a "focus calibration here" CTA.
- Per-dimension κ is expandable (`kappaOpen` toggle → per-dimension table: Factuality, Instruction Following, etc.).
- Seed data: Pravartak calibrated (κ=0.74, weakest Factuality 0.61); Legal AI calibrated (κ=0.79, by Compliance Officer); HealthBot **uncalibrated**.

### 38.3 Reference-grounded toggle
For LLM-judge modes, a toggle chooses what factuality is checked against:
- **Ingested corpus** (reference-grounded) — claims verified against the tenant's ingested documents.
- **Model knowledge** — judge uses its own parametric knowledge.
HealthBot defaults to model-knowledge (referenceGrounded:false). Hidden entirely in Human eval mode.

### 38.4 DLP acknowledgement modal
Switching the judge to **Cloud API** triggers a mandatory Data-Privacy (DLP) acknowledgement modal:
- Full-screen scrim; **not dismissable** by outside click or Escape — only Cancel or Confirm.
- The **Confirm** button is disabled until the acknowledgement checkbox is ticked.
- **Cancel** reverts the judge to its prior value (stays Local).
- **Confirm** enables the Cloud judge, sets `dlpConfirmed=true`, writes an audit event (`cloud_judge_dlp_confirmed` with user_id/project_id), pushes an activity + notification, and shows a success toast.

### 38.5 Run gating (`evalRunnable`)
Run Eval is blocked when:
- The tenant is **uncalibrated** (must run calibration first — "Start Calibration Run" CTA shown), or
- Cloud judge is selected but DLP has not been confirmed.
When blocked, the Run Eval button is disabled with an explanatory tooltip, and `runEvalFlow` refuses (guards even palette/keyboard triggers) with the reason toast.

### Implementation notes
104. Per-tenant `evalConfigs` keyed by tenantId; `getEvalConfig()` scoped to current tenant; SuperAdmin has none (panel renders empty). Isolation: each tenant's judge mode / calibration / DLP state is independent.
105. `kappaColor(k)`: >0.7 green, ≥0.6 yellow, else (incl null) red.
106. Switching to Cloud immediately triggers the DLP modal — do not change the selection until the modal is confirmed. Judge stays on the prior mode meanwhile.
107. DLP modal is Escape-locked via the global keydown handler (`if(el('dlp-modal')) return;` before `closeModal()`).
108. Reference-grounded state persists on the eval config and reflects in the eval result view (`refGroundNote`).

---

## §39 — NBFC Case Study Project (Pre-seeded Demo Tenant)

A fourth tenant, **Finserv NBFC (Case Study)** (`tenant_nbfc`, `isCaseStudy:true`, Pro plan), is pre-seeded to walk the full 8-stage Cueval pipeline end-to-end without manual setup. Standard tenant isolation (§5) applies — its data is invisible to the other three tenants and vice-versa.

### Seed data
- **Users** (all `demo123`): `ml@finserv.com` (Arjun Mehta, ML Engineer), `annotator@finserv.com` (Priya Nair, Annotator), `pm@finserv.com` (Rohan Desai, PM). Added as a login demo-credentials group.
- **Project** `proj_nbfc_chatbot` — Customer Support Chatbot (loan eligibility, EMI, KYC).
- **Releases** — v1.0 (deployed, historically *blocked* on eval gate then deployed after the v1.1 fix), v1.1 (deployed, full approval audit trail), v1.2 (in-review, interest-rate update).
- **Datasets** — v1.0 (2,800 rows, score 71, three source badges JSONL/PDF/CSV, curation summary 2,310 auto-approved / 490 flagged), v1.1 (2,398 rows, score 84), v1.2 (2,450 rows, score 93). Each carries explicit representative rows covering every flag type: clean, near-duplicate (rejected against `row_001`), PII-redacted (edit diff on the row panel), short/low-quality (expanded), and **hallucination** (wrong interest rates, pending) — plus corrected interest-rate rows in v1.1/v1.2.
- **Experiments** — Baseline Mistral 7B (`ckpt_v1_0`) and Factuality Fix / structural chunking (`ckpt_v1_1`), with snapshot hashes and training config.
- **Eval runs** — `eval_v1_0` Overall **74.2** (Factuality 71, below the 80 gate) and `eval_v1_1` two-checkpoint comparison **74.2 → 81.6** (Factuality 71 → 79). Local judge, ₹0, κ=0.74.
- **Inference monitor** — NBFC live feed seeded with a **23-strong hallucination cluster** on interest-rate queries (the trigger for v1.2) plus healthy traffic.
- **Activity feed** — the full chronological 8-stage story (curation → annotation → eval → blocked → fix → deploy → monitor cluster → agent sprint #7 → v1.2 in review).

### Case-study chrome
- **Banner** (`renderCaseBanner`): a persistent, non-dismissable amber banner rendered below the top bar for NBFC tenant users only (never for SuperAdmin's platform view or other tenants). Carries a **"How to demo this"** button.
- **New flag type** `hallucination` — red pill, with a per-row explanation drawn from the row's reference-mismatch note.
- **Dataset source badges** (note 116) and a **curation summary** panel on the dataset detail screen.
- **Demo-flow guide modal** (`demoGuideModal`, note 123) — nine numbered stages, each with a "Go →" button (`demo-goto`) that selects the NBFC project and jumps straight to the relevant screen / dataset / eval.
- **Release detail** shows the v1.0 blocked → deployed **history** (with a "view what changed" jump to the v1.1 dataset) and the v1.1 **approval audit trail**.

### Implementation notes
- NBFC datasets ship explicit rows (`buildNbfcRows`) and fixed narrative rollups; `initMockData` skips `buildRows`/`finaliseDataset`/`computeEval` for them so seeded scores/results are preserved verbatim.
- `screenEval` only honours a pinned `appState.evalResult` if it belongs to the current tenant's runs (isolation hardening).
- Represented through existing screens rather than bespoke widgets: the two-checkpoint radar (already two-polygon), the flag-rate trend chart, and the PII edit diff (existing row-panel diff). The agent sprint #7 is told through the activity trail; the NBFC Agents screen uses the standard empty state.

---

## §40 — Eval Harness: Judge-Mode Differentiation, Accuracy Monitoring & Guidance

Builds on §38. The three judge modes now behave distinctly, and the Eval Harness surfaces interpretation guidance and an ongoing judge-accuracy monitor.

### 40.1 Judge-mode-aware runs (`runEvalFlow` / `stampJudge` / `humanEvalFlow`)
- **Local (sovereign):** animated run with an on-prem scoring step; cost stamped **₹0** with the label "Local judge (on-prem, nothing left your network)".
- **Cloud API:** animated run whose scoring step reads "Transmitting rows to external API (Claude / GPT-4)…"; cost is a **real per-token ₹ bill** (label shows tokens sent externally). Still gated behind the DLP acknowledgement (§38).
- **Human eval:** does **not** auto-score. Run diverts to `humanEvalFlow` — a modal that samples up to 5 held-out rows and lets the reviewer score each (Poor/Fair/Good/Excellent), showing the judge's score alongside. On finish it aggregates the human average, computes a mock **human↔judge agreement κ**, produces an eval run (`judgeMode:'human'`, radar/table relabelled **Judge vs Human**), and feeds the accuracy monitor. If κ falls below the configured threshold it raises a recalibration alert (notification + activity + toast) and sets the monitor outcome to "Recalibration triggered".
- Every produced run is stamped with the `judgeMode` and `referenceGrounded` it used; the result view shows a judge-mode badge and the mode-specific cost.

### 40.2 Interpretation note (`evalInterpretationNote`)
Shown under every result: *"Eval scores reflect one judge's assessment against the configured rubric. Individual row scores may be inaccurate — aggregate trends across 200+ rows are more reliable. For regulated releases, supplement with human eval on a sample."*

### 40.3 Ongoing judge-accuracy monitoring (`accuracyMonitorPanel`)
Per-tenant config (`ec.accuracyMonitor`): **every [5/10/20 ▾] eval runs, sample [1/5/10% ▾] of rows** for human review; **if agreement (κ) drops below [0.6/0.65/0.7 ▾] → alert Architect + trigger recalibration.** Shows the last spot-check (date · κ · outcome, red when below threshold) and "Next spot-check: After next eval run". Dropdowns persist to the tenant's config (`eval-am-set`). Seeded: Pravartak/NBFC last spot-check 14 Jun 2025 · κ 0.71 · No action required.

### 40.4 Rubric-writing tip (`rubricTipBox`)
A tip card in the run configuration: specific rubric instructions reduce judge hallucination, with a before/after example ("Score factuality based on accuracy" → "Score factuality 1 if the response cites any financial figure … not present in the reference document.").

### Implementation notes
- Added an `evalConfigs` entry for `tenant_nbfc` (previously missing) — calibrated κ=0.74 — plus an `accuracyMonitor` block on every tenant.
- Human-eval modal uses the locked `.ig-scrim` (no outside-click close); the Next button is disabled until the current row is scored.
- Isolation: each tenant's judge mode, calibration, and accuracy-monitor settings remain independent.

---

## §41 — Research-Backed Quality Improvements (Priority 1)

First tier of the §40/§41 research-backed changes (spec ships its own build order; implemented tier by tier).

### 41.1 Eval confidence intervals (Change 2 / Patch 1)
Every eval dimension is reported as **mean ± standard deviation** (e.g. "Factuality 79 ± 5") across two judge passes. `computeEval` generates per-dimension variance (Factuality least stable, Tone most; the newer checkpoint B scores tighter). Cells with variance > 7 render on an amber background with a "high variance — consider human eval" tooltip. A **✓ Confidence intervals** badge sits on the result header. NBFC evals carry explicit variance (v1.0 Factuality ±8 → v1.1 ±5; Overall ±5.1 → ±3.8).

### 41.2 Krippendorff's Alpha (Change 8 / Patch 4, 8)
All user-facing agreement metrics renamed from Cohen's/Fleiss' κ to **Krippendorff's α**. Interpretation bands (`alphaBand`): <0.40 poor, 0.40–0.67 moderate, 0.67–0.80 good, >0.80 excellent. The calibration band now reads "agreement (α) = 0.74 (Good — Krippendorff's Alpha)" with a tooltip, surfaces the **weakest dimension** with guidance ("reviewers disagree on what counts here; add rubric examples"), and the per-dimension table gains a Band column. The accuracy-monitor and human-eval agreement displays use α. (Internal field names remain `kappa`/`overallKappa` for stability — display-only rename per note 140.)

### 41.3 Typed hallucination badges (Change 4 / Patch 7 badge-only)
Hallucinations are classified: **Contradiction** (amber — conflicts with a source doc), **Fabrication** (red — not in any document), **Conflation** (purple — merges two chunks). `halTypeBadge` renders a coloured pill in the dataset row panel, the monitor inference panel, and the live-feed flags column. NBFC: the 6 wrong-rate dataset rows and the 23-row monitor cluster are Contradictions; a standalone "Q2 2025 rate revision" monitor inference is a Fabrication.

Priority 2 (AI-suggested edits, RLTHF preference routing, dataset diversity, swap-augmented animation) and Priority 3 (claim-level hallucination, fatigue detection, model-change warning, bulk diversity alert) to follow.

---

## §41 — Research-Backed Quality Improvements (Priority 2)

### 41.4 Swap-augmented evaluation (Change 1 / Patch 1)
LLM-judge pairwise runs are now two-pass: the run animation shows "Pass 1 of 2 — scoring responses…" then "Pass 2 of 2 — positions swapped…" then "Aggregating + confidence intervals…". Runs are stamped `swapAugmented` and the result header shows a **✓ Swap-augmented** badge (tooltip cites the ~30%→~8% position-bias reduction, Wang et al. 2024). NBFC seeded evals carry the flag.

### 41.5 Dataset diversity score (Change 7 / Patch 3)
Dataset detail gains a **Quality & diversity** panel: two gauges (quality + diversity, colour-banded >75 green / 60–75 amber / <60 red) plus an instruction-type distribution bar chart. Over-represented clusters (>30%) are flagged amber, gaps (<5% with production traffic) red. NBFC v1.0 scores 68 with an **interest-rate gap (9%)** that links to the Monitor findings (`View Monitor findings →`); v1.1 rises to 79, v1.2 to 84 as the gap closes. Diversity glossary entry added.

### 41.6 AI-suggested edits (Change 5 / Patch 2)
For rows flagged **short-response / low-clarity only** (never PII/toxic), the row panel shows a collapsible "💡 Suggested edit available" with the original, an AI suggestion (deliberately good-but-not-perfect), and **Use Suggestion / Edit Suggestion / Write My Own**. Actions are tracked (`suggestionStats`) and the PM board shows an **AI-suggestions used/edited** adoption stat. NBFC short-response rows carry seeded suggestions. `finaliseDataset` now preserves NBFC fixed rollups so edits don't clobber the narrative counts.

### 41.7 RLTHF preference routing (Change 6 / Patch 2)
Preference pairs are pre-scored and routed: score-diff > 1.5 → **auto-labelled** (obvious winner, human never sees it); 0.5–1.5 → needs review; < 0.5 → **contested** (needs 2 reviewers). The Preference Ranking pane shows a routing summary ("N total — M auto-labelled, K sent to reviewers"), a collapsed **Auto-labelled** section with an audit sample, and a red **Contested pair** banner on close pairs. Auto-label rate ~74%. NBFC now has preference pairs; `prefQueue` excludes auto-labelled so counts reflect human work only.

Priority 3 (claim-level hallucination highlighting, annotator fatigue detection, model-change warning, bulk diversity alert) remains.

---

## §41 — Research-Backed Quality Improvements (Priority 3)

### 41.8 Claim-level hallucination (Change 3 / Patch 7 full)
NBFC hallucination inferences carry a MiniCheck-style `hallucinationAnalysis` (per-sentence grounded/hallucinated with source attribution). The inference panel renders the response with **per-sentence `<mark>` highlighting** (green underline = grounded; amber/red/purple = Contradiction/Fabrication/Conflation) and a **Claim-level analysis (MiniCheck)** breakdown listing each sentence, its status, the mismatch detail, and the source chunk (e.g. "Section 3.2 Para 1", or "— not found in corpus —").

### 41.9 Annotator fatigue detection (Change 9 / Patch 5, 9)
`annotatorSessions` tracks per-annotator session health (time, α-start→now, fatigue flag). A **Session health** panel (PM/Admin board only, wellbeing-framed) shows each annotator's α drift and a Fatigue/Stable pill. A fatigued annotator sees a non-blocking, dismissible **rest prompt** (bottom-right) on the Review Queue ("You've been reviewing for 94 minutes…"). NBFC: Priya Nair fatigued (α 0.81→0.68, 94 min, recovers to 0.79 after break); the fatigue event is also in the activity trail.

### 41.10 Model-relative quality warning (Change 10 / Patch 6)
Creating an experiment whose base model differs from the project's most recent completed experiment triggers a **Base model changed** modal (cites Ye et al. 2025), recommending a 200–500-row calibration experiment, with **Proceed anyway** / **Create calibration experiment**. Same-model saves proceed silently. (NBFC stays on Mistral; switching to Llama/Gemma warns.)

### 41.11 Bulk low-diversity alert (Patch 10)
Completed bulk jobs carry a diversity score; when < 60 the job dashboard shows a **Low diversity detected** card (cites LIMA 2023) with the dominant cluster % and **View diversity breakdown** / **Create annotation sprint** actions (sprint pre-named for the under-represented cluster). Legal AI's "Legal Document Corpus v1" scores 48 and demos the alert.

### Notes on fidelity
- Claim-level uses an inline breakdown list for source attribution rather than a click-popover (note 142); highlighting is inline via `<mark>`.
- Fatigue sessions are seeded mock data (PM/Admin visibility), not live per-row tracking; the rest prompt has no 30s auto-dismiss timer (avoids leaked timers, CLAUDE.md §10).
- `finaliseDataset` preserves NBFC fixed rollups so review/edit actions never clobber the narrative counts (added in Priority 2).

---

## §42 — Domain-Aware Synthesis Templates

The instruction-pair synthesis step (Document Ingestion) gains a configurable domain-template system.

### 42.1 Synthesis template config
The synthesis sub-screen leads with a **Synthesis template** panel: a radio selector across six templates — **Generic Q&A, Audit & Compliance, Legal Reference, Financial Services, Medical / Clinical, Custom** (`SYNTH_TEMPLATES`). Selecting a template checks its recommended **pair types** (checkbox multi-select). A **Pairs per chunk** dropdown and **Preview prompt** button are shown; **Edit prompt** is Architect-only (edits persist per template in `appState.synthPromptEdits`). `synthesisePairs` now generates by the selected pair types and tags every pair with `pairType`, `synthesisTemplate`, `domainTag`, and `source:'synthesis_pipeline'`.

### 42.2 Pair-type tagging & badges
Each pair type has a coloured pill (`PAIR_TYPE` / `pairTypePill`): rule/norm=blue, violation=amber, evidence=teal, finding=coral, impact=purple. Badges appear in the synthesis result table, the dataset row table (with a gold ★ for expert-seeded rows), and the row panel (with a Synthesised / Expert-seed source pill).

### 42.3 Pair-type distribution report
The dataset detail screen shows a **Pair type distribution** bar chart (below diversity) with under-represented types (<10%) flagged and a recommendation to run a domain-expert seeding sprint. The dataset header shows a **Template: …** pill.

### 42.4 Domain-expert seeding
Architect / ML-Engineer only (hidden from Annotator/Reviewer): an **Add Expert Pairs** button opens a seeding modal that turns one pasted observation (rule / violation type / evidence / financial effect) into four typed pairs (violation detection, evidence checklist, finding formulation, impact quantification), tagged `source:'domain_expert_seed'`.

### 42.5 CAG Audit Intelligence mock project
A fifth Pravartak project ("CAG Audit Intelligence", `audit_compliance` template, 47 domain-seed rows, corpus 1,200) with a **GFR Audit Corpus v1** dataset (1,247 rows, score 81, pair-type distribution 38/31/14/10/7). Rows carry realistic GFR Rule 230 audit content across all five pair types, two of them domain-expert-seeded. `finaliseDataset` preserves the fixed rollups (`_cag`), so seeding/editing never clobbers the narrative counts.
