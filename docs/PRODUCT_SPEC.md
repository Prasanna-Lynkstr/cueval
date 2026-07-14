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
Display:  'Syne' (Google Fonts) — weight 700/800. Used for product name, big numbers only.
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
2. Load Google Fonts via `<link>` in `<head>`: Syne, Inter, JetBrains Mono.
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

## 35. DELTA — WHISPER ON-PREMISE / AIR-GAPPED DEPLOYMENT

### Updates to Section 21 (Audio/Video Module)

Add to audio pipeline description — sovereign mode behaviour:

**Audio transcription — two modes:**

**SaaS mode:**
Audio sent to Whisper API endpoint (OpenAI hosted or Cueval cloud). Faster, higher throughput. Not available for sovereign clients — data would leave the network.

**Sovereign / on-premise mode:**
`faster-whisper` container runs locally with quantised weights. No network call. Language auto-routed by FastText detection output:
- English → Whisper large-v3 (quantised INT8)
- Hindi/Devanagari → Whisper large-v3 multilingual
- Tamil/Telugu/Kannada/Malayalam → IndicWhisper (AI4Bharat fine-tuned weights)
- Mixed/unknown → multilingual model with language detection per segment

**Docker image variants shipped:**
```
cueval/whisper:en           — Whisper large-v3, English optimised, ~1.8GB
cueval/whisper:indic        — IndicWhisper, Indic languages, ~2.1GB
cueval/whisper:multilingual — Both models, auto-routes, ~3.2GB (recommended)
```

Client installs once via `docker compose pull`. Weights baked into image. Fully air-gapped after initial pull.

**Processing speed on-premise (faster-whisper INT8):**
```
Hardware          Speed (1 hr audio)   Recommended for
16-core CPU       12–15 min            < 50 hrs/month volume
GPU (T4/RTX3090)  2–4 min              > 50 hrs/month volume
GPU (A100)        < 1 min              Bulk corpus ingestion
```

**Known limitations — add to Ingest stage limitations box:**
- On-prem CPU: 1-hour audio takes 12–15 min. GPU recommended above 50 hours/month.
- IndicWhisper WER on legal/medical domain: 18–30%. Human correction mandatory for institutional use.
- Base Whisper Tamil/Telugu WER without IndicWhisper: 30–40% — use IndicWhisper variant.
- First Docker pull: ~3GB download. After that fully air-gapped, no connectivity needed.
- Fine-tuning Whisper on client domain audio not bundled — separate engagement.

---

### Updates to Section 18 (On-Premise Implementation Notes)

Add to Docker Compose service list:

```yaml
cueval-whisper:
  image: cueval/whisper:multilingual
  volumes:
    - whisper_models:/models      # weights on persistent volume
  environment:
    MODEL_SIZE: large-v3
    QUANTISATION: int8
    DEVICE: cpu                   # change to cuda if GPU available
    LANGUAGE_PACK: multilingual
  ports:
    - "9001:9001"                 # internal only, not exposed externally
  restart: unless-stopped
```

Celery audio worker routes transcription jobs to this container via internal HTTP. Port 9001 not exposed beyond the Docker network. Zero external traffic during transcription.

**First-time setup note (add to on-premise install README):**
"Run `docker compose pull` before air-gapping the server. This downloads the Whisper model weights (~3GB). After initial pull, the server can be fully isolated. Subsequent Cueval software updates ship as signed packages and do not require re-downloading model weights unless a model upgrade is included."

---

### Updates to Section 20 (Bulk Ingestion)

Add to bulk job configuration panel — audio processing section:

```
Audio Transcription Engine (Sovereign Mode)
Model:   ● faster-whisper large-v3 (recommended)
         ○ faster-whisper medium (faster, lower accuracy)
         ○ IndicWhisper (Indic languages only)

Language routing:   ● Auto (FastText detection per file)
                    ○ Force English
                    ○ Force Indic

Estimated throughput (current hardware):
  CPU only:   ~4 hrs/hr of audio
  With GPU:   ~0.5 hrs/hr of audio

For 10,000 hours of audio:
  CPU only:   40,000 hours (~4.5 years) ← show warning
  1x A100:    5,000 hours (~7 months)   ← show warning
  4x A100:    1,250 hours (~7 weeks)    ← show recommendation

⚠️ Bulk audio ingestion at scale requires GPU. Contact Lynkstr
   for infrastructure sizing guidance before starting.
```

---

### Mock UI Addition — Whisper Status Panel

Add to Audio screen (Section 21) — small status card visible when project is sovereign:

```
┌─────────────────────────────────────────────────────┐
│  🔒 Sovereign Transcription                         │
│  faster-whisper multilingual · INT8 · CPU mode     │
│  Model: large-v3 + IndicWhisper                    │
│  Status: ✅ Ready                                   │
│  Throughput: ~4x realtime on current hardware      │
│  [Switch to GPU mode] [View model details]         │
└─────────────────────────────────────────────────────┘
```

Show only when project has sovereign mode enabled. Hidden for SaaS clients (they use cloud transcription automatically).

---

### Implementation Note for Claude Code (delta)

91. Whisper status panel: render as a small card below the audio upload zone on the Audio screen. Only visible when current tenant has sovereign=true in mock data. IITM Pravartak = sovereign=true. Legal AI Corp and HealthBot India = sovereign=false (don't show panel).
92. Bulk ingestion audio throughput warning: if estimated processing time exceeds 30 days on CPU, show red warning and GPU recommendation. If 7–30 days, show amber warning. Under 7 days, show green estimate.

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

## 37. DELTA — EVAL HARNESS: JUDGE SELECTION, DATA SOVEREIGNTY, REFERENCE-GROUNDED SCORING

### Corrections to Section 5 (Evaluate) and Section 29 (Monitor Configuration)

---

### Judge Selection — Three Modes

Replace all references to "Claude API as judge" with configurable judge selection. Cloud judge is never the default — it requires explicit opt-in with DLP acknowledgement.

```
Judge Mode         Data leaves network?   Accuracy    When to use
─────────────────────────────────────────────────────────────────
Local (sovereign)  No                     Medium*     Default for all clients
Cloud API          Yes (DLP required)     High        Only with explicit approval
Human eval         No                     Ground truth Calibration + high-stakes dims
```

*Accuracy improves significantly after calibration against human reference set.

---

### Reference-Grounded Evaluation (Default for Regulated Industries)

For clients in regulated industries (NBFC, banking, healthcare, legal, government), factuality scoring must check responses against ingested source documents — not the judge's general training knowledge.

**How it works:**

```
Standard judge prompt (wrong for regulated use):
"Is this response factually accurate?"
→ Judge uses its own training knowledge as ground truth
→ Cannot verify against client-specific documents
→ May score hallucinated but plausible-sounding content as correct

Reference-grounded prompt (correct):
"Given this source document: [retrieved chunk from client corpus]
 And this customer question: [question]
 And this model response: [response]
 
 Score 1–5:
 1. Does the response contradict any fact in the source document?
 2. Does it answer the specific question asked?
 3. Does it cite or reference content not present in the source?
 
 Return JSON: {factuality: N, task_completion: N, reasoning: string}"
```

The source document chunk is retrieved from Cueval's ingested corpus at eval time — the same corpus used for RAG in production. The judge compares response against document, not against its own priors.

**What this means:** Factuality score = "does this response accurately reflect our documents" not "does this sound right to a general-purpose LLM." For an NBFC, this is the difference between catching a wrong interest rate and missing it.

---

### Cloud Judge — DLP Gate

When user selects cloud judge (Claude/GPT-4 API), show mandatory acknowledgement before any eval run proceeds:

```
┌─────────────────────────────────────────────────────────────┐
│  ⚠️  Data Privacy Acknowledgement Required                  │
├─────────────────────────────────────────────────────────────┤
│  Selecting a cloud judge will send the following data       │
│  to an external API:                                        │
│                                                             │
│  ✓ Eval dataset rows (instruction + model response)        │
│  ✓ Source document chunks (if reference-grounded eval)     │
│  ✗ Raw customer PII (stripped during curation)             │
│                                                             │
│  For regulated industries (NBFC, banking, healthcare,      │
│  government), verify this is permitted under your          │
│  data governance and DLP policies before proceeding.       │
│                                                             │
│  ☐ I confirm this eval dataset has received DLP           │
│    clearance for external API transmission                  │
│                                                             │
│  [Cancel — Use Local Judge]  [Confirm — Use Cloud Judge]   │
└─────────────────────────────────────────────────────────────┘
```

Checkbox must be ticked. Cannot proceed without confirmation. Decision logged in audit trail with user ID and timestamp.

---

### Calibration Run — Human Eval First

For sovereign mode, local judge accuracy depends on calibration. Calibration workflow:

**Step 1 — Sample calibration set (one-time per project):**
- Select 50–100 rows from eval dataset
- Route to domain expert reviewer (not an annotator — needs domain knowledge)
- Domain expert scores each row manually on all rubric dimensions
- This becomes the calibration reference set

**Step 2 — Calibrate local judge:**
- Run local judge on same 50–100 rows
- Compare local scores to human scores
- Fit calibration model (isotonic regression per dimension)
- Apply calibration to all future eval runs

**Step 3 — Ongoing validation:**
- Every 10th eval run, sample 5% of rows for human spot-check
- If human-judge agreement drops below 0.7 kappa → trigger recalibration

**Show in UI:**

```
┌─────────────────────────────────────────────────────────────┐
│  Eval Judge: Local (Mistral 7B INT8)     🔒 Sovereign      │
│                                                             │
│  Calibration status:                                        │
│  ✅ Calibrated on 50 rows by Compliance Officer            │
│     Calibrated: 14 Jun 2025                                 │
│     Human-judge agreement: κ = 0.74 (good)                 │
│                                                             │
│  ⚠️  Factuality calibration weakest (κ = 0.61)            │
│     Consider expanding calibration set for this dimension  │
│     [Add calibration rows →]                               │
└─────────────────────────────────────────────────────────────┘
```

---

### Known Limitations — Add to Eval Stage

Add to Stage 05 limitations box in prototype and deck:

- LLM-as-judge has known biases: verbosity bias (longer = higher score), position bias (first response in comparison preferred), self-preference (Claude judge may favour Claude-style responses)
- Cloud judge does not know client-specific documents unless reference-grounded eval is used — general factuality scoring is unreliable for domain-specific applications
- Local judge accuracy on nuanced dimensions (tone, refusal behaviour) lower than cloud judge — calibration partially compensates but gap remains
- Calibration set created by domain expert, not annotator — for regulated industries, this means compliance officer or subject matter expert time required upfront
- Eval accuracy figures (65–82%) apply to calibrated local judge — uncalibrated local judge accuracy is 45–60%

---

### Mock Data Updates for Prototype

**Eval configuration screen — update default:**
```javascript
// Before:
judgeMode: 'cloud',
judgeModel: 'claude-api'

// After:
judgeMode: 'local',
judgeModel: 'mistral-7b-int8',
calibrationStatus: {
  calibrated: true,
  calibratedBy: 'Compliance Officer',
  calibratedAt: '2025-06-14',
  humanJudgeKappa: 0.74,
  weakestDimension: 'factuality',
  weakestKappa: 0.61
}
```

**DLP acknowledgement modal:** show when user clicks "Switch to Cloud Judge" anywhere in the eval harness. Pre-populated checkbox unchecked. Cancel returns to local judge.

**Calibration status panel:** show on eval harness screen for all projects. HealthBot India shows uncalibrated state (no compliance officer has run the calibration yet) with [Start Calibration Run →] CTA.

---

### Implementation Notes for Claude Code (delta)

101. DLP acknowledgement modal: blocks eval run if cloud judge selected without confirmation. Rendered as full-screen overlay, not dismissable by clicking outside — must use Cancel or Confirm buttons.
102. Calibration status panel: shown above eval run configuration. Green if κ > 0.7, yellow if 0.6–0.7, red if < 0.6 or uncalibrated.
103. HealthBot India eval screen: show red "Uncalibrated" badge on judge panel. [Run Eval] button disabled with tooltip: "Complete calibration run first — assign 50 rows to a domain expert."
104. Reference-grounded eval: show toggle in eval config: "Factuality check against: ○ Judge knowledge  ● Ingested corpus (recommended for regulated industries)". Default to corpus for all projects where documents have been ingested.
105. Audit log entry on cloud judge use: log {user_id, timestamp, dlp_confirmed: true, judge: 'claude-api', eval_run_id} — visible to Admin in audit log.

---

## 38. DELTA — CLAUDE CODE SPEC: EVAL HARNESS UPDATES

### Updates to Section 8.5 (Eval Harness Screen)

Replace the existing eval configuration panel with the following:

---

**Judge Configuration Panel (add above rubric builder):**

```
┌─────────────────────────────────────────────────────────────┐
│  Eval Judge                                                 │
├─────────────────────────────────────────────────────────────┤
│  ● Local (Mistral 7B INT8)     🔒 Sovereign — recommended  │
│  ○ Cloud API (Claude / GPT-4)  Data leaves network         │
│  ○ Human eval                  Manual — for calibration    │
│                                                             │
│  [Calibration status panel — see below]                    │
│                                                             │
│  Factuality check against:                                  │
│  ○ Judge general knowledge                                  │
│  ● Ingested corpus (recommended for regulated industries)   │
│    Uses same document chunks ingested during Stage 01       │
└─────────────────────────────────────────────────────────────┘
```

**Calibration status panel (shown inside judge config, below judge selector):**

```
// When calibrated (Pravartak, Legal AI Corp):
┌─────────────────────────────────────────────────────────────┐
│  ✅ Calibrated  ·  Compliance Officer  ·  14 Jun 2025      │
│  Human-judge agreement: κ = 0.74 (good)                    │
│  Weakest: Factuality κ = 0.61  [Add calibration rows →]   │
└─────────────────────────────────────────────────────────────┘

// When uncalibrated (HealthBot India):
┌─────────────────────────────────────────────────────────────┐
│  🔴 Not calibrated — accuracy 45–60% only                  │
│  Complete a calibration run before relying on eval scores  │
│  [Start Calibration Run →]                                  │
└─────────────────────────────────────────────────────────────┘
```

**[Run Eval] button behaviour:**
- Calibrated project: enabled, runs mock eval
- Uncalibrated project (HealthBot): disabled, tooltip "Complete calibration run first"
- Cloud judge selected without DLP modal confirmed: disabled until modal confirmed

---

**Reference-grounded toggle behaviour:**
- Default: ● Ingested corpus — when project has documents ingested
- Default: ● Judge knowledge — when project has only structured datasets, no documents
- Pravartak and Legal AI Corp: corpus toggle active by default (both have documents)
- HealthBot India: judge knowledge default (structured datasets only, no documents ingested yet)

---

### New Component: DLP Acknowledgement Modal

Triggered when user selects "Cloud API" as judge in any eval configuration.

```
┌─────────────────────────────────────────────────────────────┐
│  ⚠️  Data Privacy Acknowledgement                          │
│                                                             │
│  Using a cloud judge will send the following data          │
│  to an external API (Claude / GPT-4):                      │
│                                                             │
│  ✓ Eval dataset rows (instruction + model response)       │
│  ✓ Source document chunks (reference-grounded eval)       │
│  ✗ Customer PII (stripped during curation pipeline)       │
│                                                             │
│  For regulated industries — NBFC, banking, healthcare,     │
│  government — verify this is permitted under your data     │
│  governance and DLP policies before proceeding.            │
│                                                             │
│  ☐ I confirm this eval dataset has DLP clearance          │
│    for external API transmission                           │
│                                                             │
│  [← Use Local Judge]    [Confirm — Use Cloud Judge]        │
└─────────────────────────────────────────────────────────────┘
```

Behaviour:
- Full-screen overlay, z-index above everything
- Not dismissable by clicking outside or pressing Escape — must use buttons
- Checkbox unchecked by default. [Confirm] button disabled until checked.
- On confirm: cloud judge selected, mock DLP confirmation logged to audit trail
- On cancel: returns to Local judge selection
- Audit log entry: `{ action: 'cloud_judge_dlp_confirmed', user_id, timestamp, project_id, eval_config_id }`

---

### Updated Mock Data — Eval Configs Per Tenant

```javascript
// IITM Pravartak
evalConfig = {
  judgeMode: 'local',
  judgeModel: 'mistral-7b-int8',
  sovereign: true,
  referenceGrounded: true,
  calibration: {
    status: 'calibrated',
    calibratedBy: 'Dr. Anand Krishnan (Domain Expert)',
    calibratedAt: '2025-06-14',
    rowsUsed: 50,
    overallKappa: 0.74,
    dimensionKappa: {
      factuality: 0.61,
      instructionFollowing: 0.78,
      tone: 0.82,
      taskCompletion: 0.75,
      refusalBehaviour: 0.79
    }
  }
}

// Legal AI Corp
evalConfig = {
  judgeMode: 'local',
  judgeModel: 'mistral-7b-int8',
  sovereign: false,
  referenceGrounded: true,
  calibration: {
    status: 'calibrated',
    calibratedBy: 'Compliance Officer',
    calibratedAt: '2025-05-22',
    rowsUsed: 75,
    overallKappa: 0.79,
    dimensionKappa: {
      factuality: 0.71,
      instructionFollowing: 0.82,
      tone: 0.85,
      taskCompletion: 0.78,
      refusalBehaviour: 0.81
    }
  }
}

// HealthBot India
evalConfig = {
  judgeMode: 'local',
  judgeModel: 'mistral-7b-int8',
  sovereign: false,
  referenceGrounded: false,   // no documents ingested
  calibration: {
    status: 'uncalibrated',   // blocks Run Eval button
    calibratedBy: null,
    calibratedAt: null,
    rowsUsed: 0,
    overallKappa: null
  }
}
```

---

### Implementation Notes for Claude Code (delta)

106. Judge selector: radio buttons, Local pre-selected. Switching to Cloud immediately triggers DLP modal — do not change selection until modal confirmed.
107. DLP modal [Confirm] button: disabled (opacity 0.4, cursor not-allowed) until checkbox ticked. Enable on tick.
108. Calibration status panel: colour-coded. κ > 0.7 = green. κ 0.6–0.7 = yellow with [Add calibration rows →] CTA. κ < 0.6 = red. Uncalibrated = red with [Start Calibration Run →].
109. HealthBot India [Run Eval] button: render as disabled grey with tooltip on hover: "Complete calibration run before running eval. Uncalibrated judge accuracy is 45–60% only."
110. Reference-grounded toggle: shown only when judge mode is Local or Cloud. Hidden for Human eval (human judges by definition check the content directly).
111. Kappa values shown per-dimension in an expandable section: "[▾ View per-dimension agreement]" collapsed by default, expands to show a small table of dimension → κ value.
112. Mock eval results: when reference-grounded is toggled ON, show a note below results: "Factuality scored against ingested corpus — not judge general knowledge." When toggled OFF: "Factuality scored against judge training knowledge — not recommended for domain-specific applications."
113. Audit log (Admin view): cloud judge DLP confirmations appear as a distinct entry type with a shield icon. Filterable separately from other audit entries.

---

## 39. DELTA — NBFC CASE STUDY PROJECT: SEED DATA & APP FLOW

### Overview

Add a fourth tenant and a dedicated project that mirrors the NBFC illustrative case study exactly. This tenant is pre-seeded to show the complete 8-stage flow — with the model visible at each stage so a demo can walk through the story end to end without manual setup.

---

### New Tenant — NBFC Case Study

```javascript
tenants.push({
  id:   "tenant_nbfc",
  name: "Finserv NBFC (Case Study)",
  slug: "finserv",
  plan: "Pro",
  planLimits: {
    rowsPerMonth: 500000,
    evalRunsPerMonth: 50,
    maxUsers: 20,
    maxProjects: 10,
    sovereignDeployment: false
  },
  usage: {
    rowsThisMonth: 2800,
    evalRunsThisMonth: 2,
    activeUsers: 3,
    activeProjects: 1
  },
  createdAt: "2025-04-01",
  status: "active",
  isCaseStudy: true   // flag — used to show case study banner in app
})
```

### New Users — NBFC Tenant

```javascript
// Add to users array:
{ email: "ml@finserv.com",         tenantId: "tenant_nbfc", role: "MLEngineer",     name: "Arjun Mehta" },
{ email: "annotator@finserv.com",  tenantId: "tenant_nbfc", role: "Annotator",      name: "Priya Nair" },
{ email: "pm@finserv.com",         tenantId: "tenant_nbfc", role: "ProjectManager", name: "Rohan Desai" },

// All passwords: demo123
```

Add to login screen demo credentials panel under new group "Finserv NBFC (Case Study)":
```
ml@finserv.com          / demo123   → ML Engineer (Arjun)
annotator@finserv.com   / demo123   → Annotator (Priya)
pm@finserv.com          / demo123   → Project Manager (Rohan)
```

---

### Case Study Banner

When any user from tenant_nbfc is logged in, show a persistent amber banner below the top bar:

```
┌─────────────────────────────────────────────────────────────┐
│  📖  Illustrative Case Study — Finserv NBFC Customer        │
│  Support Chatbot. This project is pre-seeded to demonstrate │
│  the full Cueval pipeline. Data is fictional.               │
└─────────────────────────────────────────────────────────────┘
```

---

### Project Seed Data

```javascript
const nbfcProject = {
  id:          "proj_nbfc_chatbot",
  tenantId:    "tenant_nbfc",
  name:        "Customer Support Chatbot",
  description: "NLP chatbot for loan eligibility, EMI calculations, repayment schedules, and KYC queries",
  status:      "active",
  members: [
    { userId: "ml@finserv.com",        role: "MLEngineer" },
    { userId: "annotator@finserv.com", role: "Annotator" },
    { userId: "pm@finserv.com",        role: "ProjectManager" },
  ],
  createdAt: "2025-04-01"
}
```

### Releases

```javascript
const nbfcReleases = [
  {
    id:        "rel_v1_0",
    projectId: "proj_nbfc_chatbot",
    tenantId:  "tenant_nbfc",
    name:      "v1.0 — Initial Build",
    status:    "deployed",          // completed the full loop
    createdAt: "2025-04-01",
    deployedAt: "2025-04-22"
  },
  {
    id:        "rel_v1_1",
    projectId: "proj_nbfc_chatbot",
    tenantId:  "tenant_nbfc",
    name:      "v1.1 — Factuality Fix",
    status:    "deployed",          // second iteration, now deployed
    createdAt: "2025-04-23",
    deployedAt: "2025-04-27"
  },
  {
    id:        "rel_v1_2",
    projectId: "proj_nbfc_chatbot",
    tenantId:  "tenant_nbfc",
    name:      "v1.2 — Interest Rate Update",
    status:    "in_review",         // current state — monitor improvement in progress
    createdAt: "2025-06-10"
  }
]
```

---

### Dataset Seed

#### Dataset 1 — Original Upload (v1.0)

```javascript
{
  id:          "ds_nbfc_v1",
  projectId:   "proj_nbfc_chatbot",
  releaseId:   "rel_v1_0",
  tenantId:    "tenant_nbfc",
  name:        "NBFC Support Dataset v1.0",
  format:      "mixed",             // 3 sources shown as tabs
  totalRows:   2800,
  flaggedRows: 490,
  scoredRows:  2800,
  overallScore: 71,
  status:      "approved",
  languages:   [
    { lang: "en", label: "English", pct: 78 },
    { lang: "hi", label: "Hindi",   pct: 22 }
  ],
  sources: [
    { label: "Chat logs (JSONL)",           rows: 2000, icon: "jsonl" },
    { label: "Product PDFs (15 docs)",      rows: 300,  icon: "pdf"  },
    { label: "Label Studio export (CSV)",   rows: 500,  icon: "csv"  }
  ],
  uploadedAt:  "2025-04-01T09:00:00Z",
  uploadedBy:  "ml@finserv.com"
}
```

#### Dataset Rows — 50 seeded rows covering all flag types

Generate the following mix. Each row has realistic NBFC content:

**10 clean rows (score 88–95, no flags, status: approved)**
```javascript
[
  {
    id: "row_001", instruction: "What is the minimum CIBIL score required for a personal loan?",
    input: "", output: "The minimum CIBIL score required for a personal loan at our NBFC is 700. A higher score improves your chances of approval and may qualify you for better interest rates.",
    qualityScore: 92, flags: [], language: "en", reviewStatus: "approved"
  },
  {
    id: "row_002", instruction: "How do I calculate my EMI for a home loan of ₹50 lakhs?",
    input: "", output: "Your EMI depends on the loan amount, interest rate, and tenure. For ₹50 lakhs at 8.5% p.a. for 20 years, your EMI would be approximately ₹43,391. Use our EMI calculator at our website for exact figures.",
    qualityScore: 91, flags: [], language: "en", reviewStatus: "approved"
  },
  {
    id: "row_003", instruction: "मुझे गृह ऋण के लिए कौन से दस्तावेज़ चाहिए?",
    input: "", output: "गृह ऋण के लिए आपको निम्नलिखित दस्तावेज़ चाहिए: आधार कार्ड, पैन कार्ड, आय प्रमाण, बैंक स्टेटमेंट (6 माह), और संपत्ति के दस्तावेज़।",
    qualityScore: 89, flags: [], language: "hi", reviewStatus: "approved"
  },
  {
    id: "row_004", instruction: "What documents are needed for KYC verification?",
    input: "", output: "For KYC, you need: Aadhaar card or passport (identity proof), utility bill or bank statement (address proof), and PAN card. Digital KYC is available through our app.",
    qualityScore: 93, flags: [], language: "en", reviewStatus: "approved"
  },
  {
    id: "row_005", instruction: "Can I prepay my personal loan without penalty?",
    input: "", output: "Yes, you can prepay your personal loan after 6 EMIs. A prepayment charge of 2% on the outstanding principal applies. After 12 months, prepayment is free.",
    qualityScore: 90, flags: [], language: "en", reviewStatus: "approved"
  },
  {
    id: "row_006", instruction: "What is the maximum loan tenure for a business loan?",
    input: "", output: "Business loans at our NBFC are available for tenures of 12 to 60 months. The maximum tenure depends on your business vintage and loan amount.",
    qualityScore: 88, flags: [], language: "en", reviewStatus: "approved"
  },
  {
    id: "row_007", instruction: "How long does loan approval take?",
    input: "", output: "Personal loans are approved within 24–48 hours of submitting complete documents. Home loans take 5–7 working days. Business loans take 3–5 working days.",
    qualityScore: 91, flags: [], language: "en", reviewStatus: "approved"
  },
  {
    id: "row_008", instruction: "What is the processing fee for a car loan?",
    input: "", output: "The processing fee for a car loan is 1–2% of the loan amount, with a minimum of ₹2,500 and maximum of ₹15,000. This is a one-time non-refundable fee.",
    qualityScore: 90, flags: [], language: "en", reviewStatus: "approved"
  },
  {
    id: "row_009", instruction: "क्या मैं अपनी EMI तारीख बदल सकता हूँ?",
    input: "", output: "हाँ, आप अपनी EMI तारीख बदल सकते हैं। इसके लिए आपको हमारी शाखा में जाकर या ऐप के माध्यम से अनुरोध करना होगा। तारीख परिवर्तन पर कोई शुल्क नहीं है।",
    qualityScore: 89, flags: [], language: "hi", reviewStatus: "approved"
  },
  {
    id: "row_010", instruction: "Is gold loan interest charged monthly or yearly?",
    input: "", output: "Gold loan interest is charged monthly on the outstanding principal. The interest rate ranges from 9–14% p.a. depending on the loan-to-value ratio and tenure.",
    qualityScore: 92, flags: [], language: "en", reviewStatus: "approved"
  }
]
```

**8 near-duplicate rows (score 32–44, flag: near_duplicate, status: rejected)**
```javascript
// 8 variations of the CIBIL score question — kept best (row_001), rejected rest
[
  { id: "row_011", instruction: "What CIBIL score is needed for a personal loan?", qualityScore: 38, flags: ["near_duplicate"], duplicateOf: "row_001", reviewStatus: "rejected" },
  { id: "row_012", instruction: "Minimum credit score for personal loan?", qualityScore: 35, flags: ["near_duplicate"], duplicateOf: "row_001", reviewStatus: "rejected" },
  { id: "row_013", instruction: "What credit score do I need for a loan?", qualityScore: 36, flags: ["near_duplicate"], duplicateOf: "row_001", reviewStatus: "rejected" },
  { id: "row_014", instruction: "Personal loan minimum CIBIL requirement?", qualityScore: 33, flags: ["near_duplicate"], duplicateOf: "row_001", reviewStatus: "rejected" },
  { id: "row_015", instruction: "Is 650 CIBIL score enough for personal loan?", qualityScore: 40, flags: ["near_duplicate"], duplicateOf: "row_001", reviewStatus: "rejected" },
  { id: "row_016", instruction: "What is the credit score requirement for loan approval?", qualityScore: 37, flags: ["near_duplicate"], duplicateOf: "row_001", reviewStatus: "rejected" },
  { id: "row_017", instruction: "How much CIBIL score is required?", qualityScore: 32, flags: ["near_duplicate"], duplicateOf: "row_001", reviewStatus: "rejected" },
  { id: "row_018", instruction: "My CIBIL is 720, can I get a personal loan?", qualityScore: 44, flags: ["near_duplicate"], duplicateOf: "row_001", reviewStatus: "rejected" },
]
```

**8 PII rows (score 18–28, flag: pii_detected, status: approved after edit)**
```javascript
// Customer phone numbers and Aadhaar-like numbers in outputs
[
  { id: "row_021", instruction: "I need help with my loan account",
    output_original: "Please call our customer care at 9876543210 or visit branch at MG Road.",
    output: "Please call our customer care helpline or visit your nearest branch. Contact details are on our website.",
    qualityScore: 22, flags: ["pii_detected"], piiType: "phone", reviewStatus: "edited" },
  { id: "row_022", instruction: "How do I update my Aadhaar in the system?",
    output_original: "Submit your Aadhaar 8765 4321 0987 at the nearest branch.",
    output: "Submit your Aadhaar card at the nearest branch or upload via our app's KYC section.",
    qualityScore: 20, flags: ["pii_detected"], piiType: "aadhaar", reviewStatus: "edited" },
  // ... 6 more similar
]
```

**8 low-quality rows (score 22–35, flag: short_response, status: approved after edit)**
```javascript
[
  { id: "row_031", instruction: "Can I get a loan if I'm self-employed?",
    output_original: "Yes.",
    output: "Yes, self-employed individuals are eligible for loans. You will need to provide 2 years of ITR, business bank statements for 12 months, and proof of business vintage of at least 2 years.",
    qualityScore: 26, flags: ["short_response"], reviewStatus: "edited" },
  { id: "row_032", instruction: "What happens if I miss an EMI?",
    output_original: "Please call us.",
    output: "Missing an EMI attracts a late payment fee of ₹500 or 2% of the EMI amount, whichever is higher. It also negatively impacts your CIBIL score. Please contact us immediately if you anticipate a payment delay.",
    qualityScore: 24, flags: ["short_response"], reviewStatus: "edited" },
  // ... 6 more
]
```

**6 factuality issue rows (score 28–42, flag: hallucination, from Monitor stage)**
```javascript
// These are the rows that triggered the v1.1 fix — wrong interest rates
[
  { id: "row_041", instruction: "What is the current home loan interest rate?",
    output: "Our home loan interest rate is 6.5% p.a. onwards.",
    qualityScore: 35, flags: ["hallucination"], hallucinationNote: "Rate cited (6.5%) not present in product documents. Actual rate: 8.5% p.a.", reviewStatus: "pending" },
  { id: "row_042", instruction: "What interest rate will I get on a gold loan?",
    output: "Gold loans start at 7% per annum at our NBFC.",
    qualityScore: 28, flags: ["hallucination"], hallucinationNote: "Rate cited (7%) contradicts product doc (9–14% p.a.)", reviewStatus: "pending" },
  // ... 4 more interest rate hallucinations
]
```

**10 clean rows for v1.2 dataset (from re-ingested PDFs — correct interest rates)**
```javascript
[
  { id: "row_051", instruction: "What is the current home loan interest rate?",
    output: "Our home loan interest rates start at 8.5% p.a. for salaried individuals and 9.0% p.a. for self-employed, subject to credit assessment. Rates are reviewed quarterly.",
    qualityScore: 94, flags: [], language: "en", reviewStatus: "approved" },
  { id: "row_052", instruction: "What is the gold loan interest rate?",
    output: "Gold loan interest rates range from 9% to 14% p.a. depending on the loan-to-value ratio. For LTV up to 65%, rate is 9% p.a. For LTV 65–75%, rate is 12% p.a.",
    qualityScore: 96, flags: [], language: "en", reviewStatus: "approved" },
  // ... 8 more corrected interest rate Q&As
]
```

---

### Curation Summary (shown on Dataset detail screen)

```javascript
curationSummary = {
  totalRows: 2800,
  autoApproved: 2310,
  flaggedForReview: 490,
  breakdown: {
    nearDuplicates:    142,   // badge: yellow
    piiDetected:        89,   // badge: red
    lowQuality:        201,   // badge: yellow
    formatViolations:   58,   // badge: yellow
  },
  afterReview: {
    approved: 2118,
    rejected:  182,   // mostly near-duplicates
    edited:    156,   // PII + low quality rows
  },
  overallScore: 71,
  languageBreakdown: [
    { lang: "en", pct: 78 },
    { lang: "hi", pct: 22 }
  ]
}
```

---

### Experiments

```javascript
const nbfcExperiments = [
  {
    id:          "exp_v1_0",
    projectId:   "proj_nbfc_chatbot",
    releaseId:   "rel_v1_0",
    name:        "Baseline Fine-tune — Mistral 7B",
    datasetId:   "ds_nbfc_v1",
    datasetRows: 2118,
    trainingConfig: {
      baseModel:    "Mistral-7B-Instruct-v0.2",
      epochs:        3,
      learningRate:  "2e-4",
      batchSize:     8,
      loraRank:      8,
      optimizer:     "AdamW"
    },
    checkpointId:  "ckpt_v1_0",
    snapshotHash:  "sha256:a3f8c2d9e1b4f7...",
    evalRunId:     "eval_v1_0",
    status:        "completed",
    createdAt:     "2025-04-08T10:00:00Z",
    completedAt:   "2025-04-08T14:32:00Z",
    notes:         "Baseline run on cleaned dataset v1.0"
  },
  {
    id:          "exp_v1_1",
    projectId:   "proj_nbfc_chatbot",
    releaseId:   "rel_v1_1",
    name:        "Factuality Fix — Structural Chunking",
    datasetId:   "ds_nbfc_v1_1",
    datasetRows: 2398,
    trainingConfig: {
      baseModel:    "Mistral-7B-Instruct-v0.2",
      epochs:        3,
      learningRate:  "2e-4",
      batchSize:     8,
      loraRank:      8,
      optimizer:     "AdamW"
    },
    checkpointId:  "ckpt_v1_1",
    snapshotHash:  "sha256:b7d2a1e4c9f3...",
    evalRunId:     "eval_v1_1",
    status:        "completed",
    createdAt:     "2025-04-24T09:00:00Z",
    completedAt:   "2025-04-24T13:18:00Z",
    notes:         "Re-ingested 15 product PDFs with structural chunking. +280 interest rate rows."
  }
]
```

---

### Eval Runs

```javascript
const nbfcEvalRuns = [
  {
    id:          "eval_v1_0",
    projectId:   "proj_nbfc_chatbot",
    releaseId:   "rel_v1_0",
    checkpointA: "ckpt_v1_0",
    checkpointB: null,
    judgeMode:   "local",
    judgeModel:  "mistral-7b-int8",
    referenceGrounded: true,
    calibration: { status: "calibrated", kappa: 0.74 },
    rubric: [
      { dimension: "Factuality",          weight: 40 },
      { dimension: "Task Completion",     weight: 30 },
      { dimension: "Tone",                weight: 20 },
      { dimension: "Refusal Behaviour",   weight: 10 },
    ],
    results: {
      "ckpt_v1_0": {
        Factuality:         71,
        "Task Completion":  78,
        Tone:               88,
        "Refusal Behaviour": 82,
        Overall:            74.2
      }
    },
    status:   "completed",
    runAt:    "2025-04-09T08:00:00Z",
    heldOutRows: 200,
    humanReviewQueue: 12,   // rows where judge confidence < 70%
    costEstimate: "₹0 (local judge)"
  },
  {
    id:          "eval_v1_1",
    projectId:   "proj_nbfc_chatbot",
    releaseId:   "rel_v1_1",
    checkpointA: "ckpt_v1_0",
    checkpointB: "ckpt_v1_1",
    judgeMode:   "local",
    judgeModel:  "mistral-7b-int8",
    referenceGrounded: true,
    calibration: { status: "calibrated", kappa: 0.74 },
    rubric: [
      { dimension: "Factuality",          weight: 40 },
      { dimension: "Task Completion",     weight: 30 },
      { dimension: "Tone",                weight: 20 },
      { dimension: "Refusal Behaviour",   weight: 10 },
    ],
    results: {
      "ckpt_v1_0": {
        Factuality:          71, "Task Completion": 78,
        Tone:                88, "Refusal Behaviour": 82, Overall: 74.2
      },
      "ckpt_v1_1": {
        Factuality:          79, "Task Completion": 83,
        Tone:                89, "Refusal Behaviour": 84, Overall: 81.6
      }
    },
    status:   "completed",
    runAt:    "2025-04-24T14:00:00Z",
    heldOutRows: 200,
    humanReviewQueue: 8,
    costEstimate: "₹0 (local judge)"
  }
]
```

---

### Release Gate States

**Release v1.0 — historical (blocked, then approved after v1.1 fix):**
Show in release history as "Blocked — Eval 74.2 < 80 threshold" with timestamp "Blocked: 09 Apr 2025" and "Deployed: 27 Apr 2025 after v1.1 fix".

**Release v1.1 — deployed:**
All gates green. Show approval audit trail:
```
Gate                    Status   Who             When
Dataset quality > 70    ✅ Pass   System          24 Apr 14:02
Annotation complete     ✅ Pass   System          24 Apr 14:02
Eval score ≥ 80         ✅ 81.6   System          24 Apr 14:04
ML Engineer approval    ✅ Pass   Arjun Mehta     24 Apr 14:30
PM sign-off             ✅ Pass   Rohan Desai     24 Apr 15:45
```

---

### Inference Monitor Seed Data

```javascript
const nbfcMonitor = {
  projectId:      "proj_nbfc_chatbot",
  tenantId:       "tenant_nbfc",
  modelVersion:   "v1.1",
  monitoringMode: "quality_loop",
  judgeConfig: {
    mode: "local",
    referenceGrounded: true,
    calibration: { status: "calibrated", kappa: 0.74 }
  },
  twoWeekStats: {
    totalInferences:  14200,
    flagRate:          6.2,
    negativeFeedback:  4.1,
    topFailureCategory: "hallucination",
    topFailureDetail:   "Incorrect interest rates — product docs not updated"
  },
  // 20 mock inferences in live feed
  liveInferences: [
    { id: "inf_001", instruction: "What is the home loan rate?",
      score: 34, flags: ["hallucination"],
      flagDetail: "'8.5%' in response but product doc shows '8.75% p.a. effective Q2 2025'",
      userFeedback: null, timestamp: "14:32:04" },
    { id: "inf_002", instruction: "How to foreclose my personal loan?",
      score: 88, flags: [], userFeedback: "positive", timestamp: "14:32:11" },
    { id: "inf_003", instruction: "ब्याज दर क्या है गृह ऋण पर?",
      score: 31, flags: ["hallucination"],
      flagDetail: "Rate cited in Hindi response doesn't match updated product doc",
      userFeedback: "negative", timestamp: "14:32:19" },
    // ... 17 more mixed score inferences
  ],
  // Hallucination cluster that triggered v1.2
  hallucinationCluster: {
    count: 23,
    pattern: "home loan interest rate queries",
    rootCause: "Product PDF updated with Q2 2025 rates not re-ingested",
    agentAction: "Created annotation sprint #7: 'Re-ingest updated rate PDFs + add 50 interest rate QA pairs'",
    sprintId: "sprint_007"
  }
}
```

---

### Activity Feed Seed Data (Dashboard)

```javascript
const nbfcActivity = [
  { text: "Curation completed — 2,310 rows auto-approved, 490 flagged", time: "04 Apr 09:48", actor: "System" },
  { text: "Priya Nair completed annotation batch — 156 rows edited, 182 rejected", time: "06 Apr 16:20", actor: "Priya Nair" },
  { text: "Eval run completed — Overall 74.2. Factuality flagged as weak point.", time: "09 Apr 08:45", actor: "System" },
  { text: "Release v1.0 blocked — Eval score 74.2 below threshold of 80", time: "09 Apr 08:46", actor: "System" },
  { text: "Root cause identified: PDF chunking issue. Re-ingestion started.", time: "22 Apr 11:00", actor: "Arjun Mehta" },
  { text: "Dataset v1.1 approved — 2,398 rows (280 new interest rate rows)", time: "23 Apr 17:30", actor: "System" },
  { text: "Eval v1.1 completed — Overall 81.6. Factuality improved 71→79.", time: "24 Apr 14:04", actor: "System" },
  { text: "Release v1.1 approved by Arjun Mehta", time: "24 Apr 14:30", actor: "Arjun Mehta" },
  { text: "Release v1.1 signed off by Rohan Desai. Deployed to production.", time: "24 Apr 15:45", actor: "Rohan Desai" },
  { text: "Monitor: 23 hallucination failures clustered around interest rate queries", time: "10 May 09:00", actor: "Agent" },
  { text: "Agent created sprint #7: Re-ingest updated rate PDFs", time: "10 May 09:01", actor: "Annotation Router Agent" },
  { text: "Dataset v1.2 in review — rate PDFs re-ingested", time: "12 Jun 14:00", actor: "Arjun Mehta" },
]
```

---

### Role-Specific Dashboard CTAs for NBFC Tenant

```javascript
// ML Engineer (Arjun) — logged in as ml@finserv.com
dashboardCTA_MLEngineer = {
  primaryAction: "v1.2 eval pending — run eval to compare ckpt_v1_2 vs ckpt_v1_1",
  cta:           "Run Eval →",
  navigateTo:    "eval_harness",
  secondaryInfo: "Monitor: flag rate 6.2% → 3.8% after v1.2. 47 inferences pending review."
}

// Annotator (Priya) — logged in as annotator@finserv.com
dashboardCTA_Annotator = {
  primaryAction: "Sprint #7: 12 interest rate QA pairs pending your review",
  cta:           "Start Review →",
  navigateTo:    "review_queue",
  secondaryInfo: "You reviewed 156 rows in v1.0. Agreement rate with reviewer: 94%."
}

// PM (Rohan) — logged in as pm@finserv.com
dashboardCTA_PM = {
  primaryAction: "v1.2 release gate: 3/5 gates green. Eval run needed.",
  cta:           "View Release Gate →",
  navigateTo:    "releases",
  secondaryInfo: "v1.1 deployed 24 Apr. Flag rate improved from 6.2% → 3.8% after v1.2 dataset update."
}
```

---

### How the Flow Appears in the App

The NBFC project is designed so a demo can walk through each stage sequentially:

**Stage 01 — Ingest:**
Navigate to Datasets. Show "NBFC Support Dataset v1.0" with three source badges (JSONL, PDF, CSV). Click dataset → see 2,800 rows ingested, language breakdown (78% EN, 22% HI).

**Stage 02 — Curate:**
On the same dataset view, show the curation summary panel: 2,310 auto-approved, 490 flagged. Expandable flag breakdown with counts. Quality score: 71 shown in amber.

**Stage 03 — Annotate:**
Navigate to Review Queue. Switch to annotator@finserv.com to show the queue (or show completed queue with history). Show the PII correction example (row_021/022) — original output with phone number vs edited clean output. Show the near-duplicate cluster (row_011–018) with "Rejected — near-duplicate of row_001" labels.

**Stage 04 — Train:**
Navigate to Experiment Tracker. Show exp_v1_0 card with Mistral 7B config, snapshot hash, training config. Lineage view shows: Dataset v1.0 → Experiment → Checkpoint v1.0.

**Stage 05 — Evaluate:**
Navigate to Eval Harness. Show eval_v1_0 results — radar with one polygon (no comparison yet). Factuality 71 in yellow. "Below 80% threshold" warning. Overall: 74.2.

**Stage 06 — Release (blocked):**
Navigate to Releases. Show v1.0 gate with ❌ on eval score gate. "Blocked: 09 Apr 2025." Root cause note visible.

**Stage 05b — Second eval:**
Show eval_v1_1 with two-polygon radar (ckpt_v1_0 coral, ckpt_v1_1 pink). Score comparison table showing 71→79 factuality, 74.2→81.6 overall.

**Stage 06b — Release approved:**
Show v1.1 gate all green with approval audit trail (Arjun + Rohan timestamps).

**Stage 07 — Monitor:**
Navigate to Monitor. Show live feed with hallucination flags on interest rate queries. Hallucination cluster panel: "23 similar failures — home loan interest rate queries."

**Stage 08 — Improve:**
Navigate to Agents. Show "Annotation Router Agent" created sprint #7. Navigate to Datasets → show v1.2 dataset with 50 corrected interest rate rows (score 94–96). Show monitor: flag rate 6.2% → 3.8% trend.

---

### Implementation Notes for Claude Code (delta)

114. NBFC tenant isCaseStudy flag: when true, show amber case study banner below top bar on all screens. Banner is non-dismissable.
115. All NBFC data pre-loaded in mock data arrays with tenantId: "tenant_nbfc". Standard tenant isolation applies — other tenants cannot see this data.
116. Dataset source badges (JSONL / PDF / CSV): show as three small pill badges on the dataset card and dataset header. Clicking each shows how many rows came from that source.
117. Near-duplicate cluster view: when viewing row_011–018, each shows "Near-duplicate of row_001" with a [View original →] link that scrolls to row_001. Rejected rows shown with a red strikethrough style on the instruction text.
118. PII correction diff: when viewing row_021 (edited status), show two-column diff — left: original output (phone number highlighted in red), right: corrected output. Same for all edited rows.
119. Release gate v1.0: show historical blocked state with a "View what changed →" link that navigates to the v1.1 dataset comparison (what 280 rows were added).
120. Eval radar chart: eval_v1_0 shows single polygon. eval_v1_1 shows two polygons (coral = v1.0, pink/green = v1.1). Label each polygon with checkpoint name.
121. Monitor hallucination cluster: clicking "23 similar failures" opens a right panel showing the cluster — all 23 inferences grouped, each with the flag detail showing which rate was cited vs what the product doc says.
122. Agent sprint creation: in the Agents action log, the sprint creation entry has a [View Sprint →] link that navigates to the Review Queue filtered to sprint #7 (the 12 interest rate rows pending Priya's review).
123. Demo flow guide: add a floating "?" button on the NBFC tenant dashboard that opens a modal: "How to demo this project" — a numbered checklist of the 9 stages with [Go to this stage →] links for each step.
124. Flag rate trend chart on Monitor: show two lines — v1.1 period (coral, flag rate ~6.2%) and v1.2 period (green, flag rate ~3.8%). Dashed vertical line at v1.2 deployment date. Annotation: "v1.2 deployed — rate PDFs updated."

---

## 40. DELTA — RESEARCH-BACKED QUALITY IMPROVEMENTS

### Overview
Ten design changes derived from peer-reviewed research. Each change is cited, justified, and specified precisely enough for Claude Code to implement in the prototype.

---

### Change 1 — Swap-Augmented Pairwise Evaluation
**Research basis:** Zheng et al. (2023) MT-Bench; Wang et al. (2024) "Large Language Models Are Not Fair Evaluators"
**Problem solved:** Position bias — LLM judges prefer whichever response appears first, regardless of quality. Error rate ~30% without mitigation.

**Implementation:**
In eval harness, when comparing two checkpoints (pairwise mode):
- Run evaluation twice per row: once with checkpoint A response first, once with checkpoint B first
- Average the two score sets
- If scores differ by >1.5 points between runs on any dimension: flag that row as "position-sensitive — low reliability"
- Show in results: "Swap-augmented evaluation used — position bias reduced"

**Mock behaviour:** Pre-compute both orderings in mock eval data. Show a small "±" variance indicator per dimension score.

**UI change:** Add badge on eval results: "✓ Swap-augmented" when this mode is active. Tooltip: "Each response evaluated twice with positions swapped and averaged — reduces position bias from ~30% to ~8% (Wang et al., 2024)."

---

### Change 2 — Eval Score Confidence Intervals
**Research basis:** Schroeder and Wood-Doughty (2024); CyclicJudge (2025) arxiv.org/abs/2603.01865
**Problem solved:** Single-judge evaluations yield insufficiently stable scores — the same judge may score the same response differently across runs.

**Implementation:**
Run each eval row through the judge twice with different temperature/seed settings. Report:
- Mean score per dimension
- Standard deviation across both runs
- Display as: "Factuality: 79 ± 6" not "Factuality: 79"

**UI change in eval results table:**
```
Dimension          | ckpt v1.0      | ckpt v1.1      | Delta
Factuality         | 71 ± 8         | 79 ± 5         | +8 ▲
Task Completion    | 78 ± 4         | 83 ± 3         | +5 ▲
Tone               | 88 ± 3         | 89 ± 3         | +1 ▲
Refusal Behaviour  | 82 ± 4         | 84 ± 4         | +2 ▲
Overall            | 74.2 ± 5.1     | 81.6 ± 3.8     | +7.4 ▲
```

High variance (±8 or more) shown in amber — tooltip: "High variance — consider human eval for this dimension."

**Mock data update:** Add variance fields to all eval run results. Factuality always highest variance. Overall variance decreases between v1.0 and v1.1 (better checkpoint = more consistent scores).

---

### Change 3 — MiniCheck Claim-Level Hallucination Detection
**Research basis:** Tang et al. (EMNLP 2024) "MiniCheck: Efficient Fact-Checking of LLMs on Grounding Documents"
**Problem solved:** Response-level hallucination flags are too coarse. "This response contains a hallucination" is less useful than "sentence 2 contains a hallucination."

**Implementation:**
When an inference is flagged for hallucination in the Monitor, show claim-level breakdown in the inference detail panel:

```
┌─────────────────────────────────────────────────────────────┐
│  Hallucination Analysis (MiniCheck)                         │
├─────────────────────────────────────────────────────────────┤
│  Sentence 1: "For a home loan, you need a CIBIL score"     │
│  ✅ Grounded — matches Section 3.2 of product doc          │
│                                                             │
│  Sentence 2: "of at least 720."                            │
│  🔴 Not grounded — document states 700, not 720            │
│  Source: Home Loan Product Guide, Section 3.2, Para 1      │
│                                                             │
│  Sentence 3: "Higher scores qualify for better rates."     │
│  ✅ Grounded — matches Section 3.4                         │
└─────────────────────────────────────────────────────────────┘
```

**Mock implementation:** Pre-tag sentences in mock inference rows as grounded/not-grounded with source attribution. Colour-highlight in the response text: green underline = grounded, red underline = hallucinated.

**UI note:** Hallucinated sentence shown with red underline inline in the response text. Click sentence → shows source document excerpt it was checked against (or "not found in corpus").

---

### Change 4 — Hallucination Type Classification
**Research basis:** Huang et al. (2025) "A Survey on Hallucination in LLMs" — ACM Transactions on Information Systems
**Problem solved:** Different hallucination types require different fixes. Treating all hallucinations as the same misses the root cause.

**Three types to classify in the monitor:**
- **Fabrication** — model stated a fact not present anywhere in corpus (entity invented)
- **Contradiction** — model stated something directly conflicting with source document
- **Conflation** — model merged information from two different chunks incorrectly

**UI change in inference detail:**
Replace single "hallucination" flag with type badge:
```
🔴 Contradiction — "720" contradicts "700" in Section 3.2
```
vs
```
🔴 Fabrication — "Q2 2025 rate revision" not found in any ingested document
```

**Mock data:** Tag each hallucination row in NBFC case study with type. Interest rate rows = Contradiction. Invented rate revision = Fabrication.

---

### Change 5 — AI-Suggested Edits in Review Queue
**Research basis:** CLEAR: Automated Data Curation (arxiv.org/abs/2403.12776) — auto-correction outperforms auto-rejection
**Problem solved:** Annotators currently write corrections from scratch. CLEAR shows suggesting a corrected output and asking for approval/modification is faster and produces better data.

**Implementation in Text Review queue:**
For flagged rows (low quality, short response, format violation), add below the output textarea:

```
┌─────────────────────────────────────────────────────────────┐
│  💡 Suggested Edit (AI-generated)                           │
│                                                             │
│  Original: "Yes, please call us."                           │
│                                                             │
│  Suggested: "Yes, self-employed individuals are eligible.   │
│  You will need 2 years of ITR, 12 months of business bank  │
│  statements, and proof of 2+ years business vintage."      │
│                                                             │
│  [Use Suggestion] [Edit Suggestion] [Write My Own]         │
└─────────────────────────────────────────────────────────────┘
```

[Use Suggestion] → copies to output field, annotator confirms
[Edit Suggestion] → copies to output field as editable text
[Write My Own] → clears and lets annotator type

**Mock data:** Pre-populate suggestions for all short-response flagged rows in NBFC case study. Make them realistic and slightly better than the original but not perfect — so annotators have something to edit.

**Tracking:** Log which action was taken (used/edited/own) per row. Surface in PM dashboard: "72% of suggestions used or edited — 28% written from scratch."

---

### Change 6 — RLTHF Routing for Preference Ranking
**Research basis:** RLTHF: Targeted Human Feedback (2025) — 6–7% annotation effort achieves full-human annotation quality
**Problem solved:** Most preference pairs are obvious — routing all of them to human annotators wastes time on easy cases.

**Implementation in Preference Ranking queue:**
Before showing a pair to a human reviewer, run LLM judge pre-score on both responses.

**Routing logic:**
```
Score difference > 1.5 points:  AUTO-LABEL (obvious winner)
                                 Mark as "Auto-labelled" in dataset
                                 Human never sees this pair

Score difference 0.5–1.5:       ROUTE TO HUMAN
                                 Show pre-scores AFTER annotator makes choice
                                 (avoid anchoring — show after, not before)

Score difference < 0.5:         ROUTE TO HUMAN + FLAG AS CONTESTED
                                 Show "This pair is close — take extra care"
                                 Require 2 annotators (not 1)
```

**UI change:**
Queue progress shows: "47 pairs total — 31 auto-labelled, 16 sent to reviewers"
Auto-labelled pairs visible in a separate "Auto-labelled" tab — reviewers can audit a sample.

**Mock data in NBFC:** Pre-tag preference pairs as auto-labelled (score diff > 1.5) or human-reviewed (score diff < 1.5). Show auto-label rate as ~70% — realistic based on RLTHF findings.

---

### Change 7 — Dataset Diversity Score
**Research basis:** LIMA (Zhou et al., 2023) — diversity of fine-tuning data matters as much as quality
**Problem solved:** A dataset of 2,000 clean rows may all be asking the same thing 2,000 ways. Homogeneous data produces models that can't generalise.

**Implementation:**
Add diversity analysis to the Dataset detail screen, shown alongside the quality score:

```
┌─────────────────────────────────────────────────────────────┐
│  Dataset Diversity Score: 68 / 100   ⚠️ Moderate           │
│                                                             │
│  Instruction type distribution:                            │
│  ██████████████░░░░  EMI / repayment queries    42%  ⚠️     │
│  ██████░░░░░░░░░░░░  Eligibility queries        18%         │
│  ████░░░░░░░░░░░░░░  KYC / documentation        12%         │
│  ███░░░░░░░░░░░░░░░  Interest rate queries       9%  ← gap  │
│  ██░░░░░░░░░░░░░░░░  Loan closure / prepay       7%         │
│  ██░░░░░░░░░░░░░░░░  Other                       12%        │
│                                                             │
│  ⚠️ EMI queries over-represented (42% vs recommended <30%) │
│  💡 Interest rate queries under-represented (9%)           │
│     This matches production failure pattern in v1.0        │
└─────────────────────────────────────────────────────────────┘
```

**Diversity score calculation (mock):**
- Cluster approved rows into instruction types using embedding similarity
- Measure distribution entropy — maximum diversity = equal distribution across types
- Flag any cluster >30% of total as over-represented
- Flag any cluster <5% where production queries exist as a gap

**NBFC case study connection:** The interest rate gap detected here directly foreshadows the Monitor finding in Stage 7. Show this link in the UI: "Interest rate queries (9%) — Monitor detected 23 production failures on this topic."

---

### Change 8 — Krippendorff's Alpha (Replace Fleiss' Kappa)
**Research basis:** "Selecting the Right Inter-Annotator Agreement Metric" (arxiv.org/abs/2603.06865)
**Problem solved:** Fleiss' Kappa requires the same annotators to label every item — impractical at enterprise scale where different annotators see different subsets. Krippendorff's Alpha handles missing data correctly.

**Implementation:**
Replace all Fleiss' Kappa calculations and displays with Krippendorff's Alpha.

**UI change:** Everywhere κ (kappa) is shown, change label to α (alpha). Update tooltip: "Krippendorff's Alpha — measures inter-annotator agreement across any number of annotators with partial overlap. Scale: <0.40 poor, 0.40–0.67 moderate, 0.67–0.80 good, >0.80 excellent (Krippendorff, 2011)."

**Interpretation guidance shown inline:**
```
Agreement (α): 0.74   ✅ Good
Weakest dimension: Factuality α = 0.61  ⚠️ Moderate
→ Domain expert disagreeing on what counts as a factual error
  Consider adding rubric examples for this dimension
```

---

### Change 9 — Annotator Fatigue Detection
**Research basis:** Snow et al. (2008); annotator performance degrades with session length and repetition; Temporal Simultaneity paper (2025) shows visible kappa decline across batches
**Problem solved:** Annotation quality degrades invisibly within long sessions. No current system tracks this.

**Implementation:**
Track per-annotator, per-session metrics invisibly (not shown to annotator):

```
Fatigue signals monitored:
- Time per row: if drops below 30s for 5+ consecutive rows → speed flag
- Agreement with consensus: if drops >15% from session start → quality flag
- Edit length: if edits getting shorter over time → effort flag

When 2+ signals fire simultaneously → show rest prompt:
┌──────────────────────────────────────────────────────┐
│  You've been reviewing for 94 minutes. Take a break? │
│  Your review quality is best maintained in sessions  │
│  under 90 minutes.                          [Resume] │
└──────────────────────────────────────────────────────┘
```

**PM dashboard addition:**
```
Annotator Reliability (This Session)
Priya Nair:    α = 0.81 → 0.68 over 2hrs  ⚠️  Fatigue signal
Ravi Kumar:    α = 0.79 → 0.77 stable     ✅
```

**Note:** Fatigue detection is PM/Admin visible only. Never shown to the annotator in a way that feels punitive. Frame as wellbeing, not surveillance.

---

### Change 10 — Model-Relative Quality Labels
**Research basis:** "Towards Understanding Valuable Preference Data" (arxiv.org/abs/2510.13212) — data quality is a property of the model, not just the data
**Problem solved:** A quality score assigned when training Mistral 7B may not transfer when the client switches to Llama 3. No current system warns about this.

**Implementation:**
Every dataset row's quality score is tagged with the base model it was calibrated against:

```
row_quality_score: {
  score: 87,
  calibrated_for: "Mistral-7B-Instruct-v0.2",
  calibrated_at: "2025-04-08",
  transferability: "unknown"  // until a cross-model eval is run
}
```

**Warning shown when experiment config changes base model:**

```
┌─────────────────────────────────────────────────────────────┐
│  ⚠️  Base model changed: Mistral 7B → Llama 3.1 8B        │
│                                                             │
│  Your dataset quality scores were calibrated for Mistral.  │
│  Research shows quality scores are model-relative —        │
│  rows that improved Mistral may behave differently with    │
│  Llama (Ye et al., 2025).                                  │
│                                                             │
│  Recommendation: Run a small calibration experiment        │
│  (200–500 rows) before full training.                      │
│                                                             │
│  [Proceed anyway]  [Run calibration experiment first]      │
└─────────────────────────────────────────────────────────────┘
```

---

### Implementation Notes for Claude Code (delta)

125. Swap-augmented eval: mock by storing two score sets per checkpoint in eval run data. Display averaged scores with ± variance. Run animation shows "Pass 1 of 2..." then "Pass 2 of 2 (positions swapped)..." before showing results.
126. Confidence intervals: show ± values in eval table. High variance (±8+) cells shown with amber background. Tooltip on variance: "High score variance — judge was inconsistent on this dimension. Consider human eval."
127. Claim-level hallucination: in inference detail panel, split response text into sentences. Each sentence has a status indicator (green dot = grounded, red dot = hallucinated). Click sentence → shows source excerpt in right sub-panel.
128. Hallucination type badge: replace generic "hallucination" flag label with typed badge: "Contradiction", "Fabrication", or "Conflation" with colour coding (Contradiction = amber, Fabrication = red, Conflation = purple).
129. AI-suggested edit: shown only for rows flagged as short_response or low_quality. Not shown for PII rows (too risky to auto-suggest). Pre-populated in mock data for NBFC low-quality rows.
130. RLTHF routing: preference ranking queue shows two sections: "Auto-labelled (31)" collapsed by default with [Audit sample] option, and "Needs review (16)" active. Auto-labelled badge on each auto-labelled row.
131. Diversity score: shown on dataset detail header alongside quality score. Two circular gauges side by side: Quality 71 and Diversity 68. Instruction type distribution as horizontal bar chart below.
132. Krippendorff's Alpha: global find-replace κ → α in all UI labels. Update all tooltips. Scale interpretation text updated per Krippendorff (2011) thresholds.
133. Fatigue detection: rest prompt appears as a gentle overlay (not blocking) after fatigue signals. Annotator can dismiss. Logged silently. PM dashboard shows session-level α trend chart per annotator.
134. Model-relative warning: fires only when experiment config base_model field changes between experiments in the same project. Does not fire on first experiment. Compare against most recent completed experiment's base_model.

---

## 41. DELTA — PATCHES TO EARLIER SECTIONS FROM SECTION 40 CHANGES

### Purpose
Section 40 introduced 10 research-backed changes. Several earlier sections now contradict or are incomplete relative to these changes. This section patches those conflicts precisely. Claude Code should treat Section 40 + Section 41 as overriding the corresponding earlier text.

---

### Patch 1 — Section 8.5 (Eval Harness): Add Swap-Augmented Mode + Confidence Intervals

**Replaces:** "Run Eval → animated progress 3–4 seconds → radar chart"

**Updated eval run behaviour:**
```
[Run Eval] triggers two-pass evaluation:

Pass 1 animation: "Evaluating checkpoint A responses..."  (1.5s)
Pass 2 animation: "Re-evaluating with positions swapped..." (1.5s)
Aggregation:       "Computing scores + confidence intervals..." (0.5s)

Results display:
- All dimension scores shown as "79 ± 5" format
- Variance > 8: cell background amber, tooltip explaining instability
- Badge on results header: "✓ Swap-augmented  ✓ Confidence intervals"
```

**Updated results table columns:**
```
Dimension | ckpt v1.0    | ckpt v1.1    | Delta | Variance flag
```

**Mock data update:** Add `variance` field to every eval dimension score in all three tenants. Factuality always highest variance (±7–9). Tone always lowest (±2–3). Overall variance decreases between older and newer checkpoints.

---

### Patch 2 — Section 8.4 (Review Queue): Add AI-Suggested Edits + RLTHF Routing

**Add to Text Review task UI** (after the output textarea, for short_response and low_quality flagged rows only):
```
Suggested Edit panel — visible only when flag is short_response OR low_quality
NOT shown for: pii_detected, toxic, near_duplicate, format_violation
```

**Add to Preference Ranking tab** — replace current single queue with two-section layout:
```
Tab: Preference Ranking (badge = human-review count only, not auto-labelled)

Section A: "Auto-labelled (31)"  [collapsed]  [Audit 5 samples ▾]
Section B: "Needs Review (16)"   [active]
  - Contested pairs (score diff < 0.5): red badge "Contested — 2 reviewers needed"
  - Regular pairs (score diff 0.5–1.5): standard queue
```

**Mock data:** Pre-tag all NBFC preference pairs. 70% auto-labelled (score diff > 1.5), 30% human-reviewed. Of human-reviewed, 20% marked as contested.

---

### Patch 3 — Section 8.3 (Datasets): Add Diversity Score Panel

**Add to Dataset detail screen** — below the quality score summary, above the row table:

```
Two gauges side by side:
[Quality Score: 71 🟡]    [Diversity Score: 68 ⚠️]

Below gauges: instruction type distribution horizontal bar chart
Flags: over-represented clusters (>30%) in amber, under-represented (<5% with prod traffic) in red
```

**Diversity score calculation (mock):**
```javascript
// Pre-compute for each dataset
diversityScore = {
  score: 68,
  clusters: [
    { label: 'EMI / repayment',   pct: 42, flag: 'over' },
    { label: 'Eligibility',        pct: 18, flag: null   },
    { label: 'KYC / documentation', pct: 12, flag: null  },
    { label: 'Interest rates',     pct: 9,  flag: 'under' },
    { label: 'Loan closure',       pct: 7,  flag: null   },
    { label: 'Other',              pct: 12, flag: null   },
  ],
  insight: "Interest rate queries under-represented — matches Monitor failure pattern"
}
```

**NBFC dataset connection:** For the NBFC case study dataset, the "Interest rates: 9% — under-represented" flag links to a callout: "This gap matches 23 production failures detected in Stage 07 Monitor." Show a [View Monitor findings →] link.

---

### Patch 4 — Section 8.4 (Review Queue): Replace Fleiss' κ with Krippendorff's α

**Global replacement throughout prototype:**
- All `κ` symbols → `α`
- All "Fleiss' Kappa" text → "Krippendorff's Alpha"
- All kappa field names in mock data → alpha
- Tooltip update: "Krippendorff's Alpha — correct for annotation pools where not all annotators label all items. <0.40 poor, 0.40–0.67 moderate, 0.67–0.80 good, >0.80 excellent."

**Mock data field rename (all tenants, all annotation objects):**
```javascript
// Before:
{ overallKappa: 0.74, dimensionKappa: { factuality: 0.61 } }

// After:
{ overallAlpha: 0.74, dimensionAlpha: { factuality: 0.61 } }
```

All calibration status panels, reviewer dashboards, and batch approval screens use α.

---

### Patch 5 — Review Queue: Add Annotator Fatigue Detection

**Add to annotator session (invisible to annotator, visible to PM/Admin):**

Per-session tracking object:
```javascript
annotatorSession = {
  userId: "annotator@finserv.com",
  startTime: "14:00",
  rowsReviewed: 47,
  avgTimePerRow: [45, 42, 38, 31, 28],  // trend — decreasing = speed flag
  alphaAtStart: 0.81,
  alphaNow: 0.68,                         // dropping = quality flag
  fatigueSignals: ["speed", "quality"],   // 2 signals = show rest prompt
  restPromptShown: false
}
```

**Rest prompt (shown to annotator when 2+ signals fire):**
```
Gentle overlay, not blocking:
"You've been reviewing for 94 minutes.
 Your best work happens in sessions under 90 minutes.
 Consider a short break?"
[Dismiss]  [Take a break — save progress]
```

**PM dashboard addition (new row in annotation project view):**
```
Session Health
Priya Nair:  α 0.81→0.68  ⚠️ Fatigue detected (94 min session)
Ravi Kumar:  α 0.79→0.77  ✅ Stable (67 min session)
```

**NBFC case study mock:** Priya Nair's session shows fatigue signal after 90 minutes. PM dashboard shows the α drop. Rest prompt shown. After break, α recovers to 0.79.

---

### Patch 6 — Section 6 (Experiment Tracker): Add Model-Relative Quality Warning

**Add to New Experiment form** — when user selects a base model different from the most recent completed experiment:

```javascript
// Trigger condition:
if (newExperiment.baseModel !== mostRecentExperiment.baseModel) {
  showModelChangeWarning()
}
```

**Warning modal:**
```
┌─────────────────────────────────────────────────────────────┐
│  ⚠️  Base Model Changed                                     │
│  Previous: Mistral-7B-Instruct-v0.2                        │
│  New:      Llama-3.1-8B-Instruct                           │
│                                                             │
│  Quality scores in your dataset were calibrated for        │
│  Mistral. Research shows data quality is model-relative    │
│  — rows that helped Mistral may behave differently         │
│  with Llama (Ye et al., 2025, arxiv:2510.13212).          │
│                                                             │
│  Recommendation: Run a calibration experiment on           │
│  200–500 rows before committing to full training.          │
│                                                             │
│  [Proceed anyway]  [Create calibration experiment]         │
└─────────────────────────────────────────────────────────────┘
```

**Mock data:** NBFC case study stays on Mistral — no warning fires. Legal AI Corp has a third experiment switching to Llama 3 — this triggers the warning. Show the modal in the Legal AI Corp demo flow.

---

### Patch 7 — Section 23 (Inference Monitor): Upgrade Hallucination to Claim-Level + Typed

**Replace** the current inference detail hallucination flag display with:

**Claim-level breakdown panel (shown when hallucination flag present):**
```javascript
// Each mock inference row with hallucination flag now has:
hallucinationAnalysis: {
  type: "Contradiction",  // or "Fabrication" or "Conflation"
  sentences: [
    { text: "For a home loan, you need a CIBIL score", status: "grounded",
      sourceChunk: "Section 3.2 Para 1" },
    { text: "of at least 720.", status: "hallucinated",
      type: "Contradiction",
      detail: "Document states 700, not 720",
      sourceChunk: "Section 3.2 Para 1" },
    { text: "Higher scores qualify for better rates.", status: "grounded",
      sourceChunk: "Section 3.4" },
  ]
}
```

**Typed badge colours:**
- Contradiction: amber `F59E0B` background
- Fabrication: red `EF4444` background
- Conflation: purple `8B5CF6` background

**Inline highlighting in response text:**
- Green underline = grounded sentence
- Red/amber/purple underline = hallucinated (colour matches type)
- Click any underlined sentence → shows source chunk excerpt in sub-panel

**NBFC mock inferences:** All 6 interest rate hallucination rows tagged as Contradiction type. The "Q2 rate revision" invented row tagged as Fabrication.

---

### Patch 8 — Section 38 (Eval Harness): Calibration α field rename

All kappa references in calibration status panel renamed to alpha:
```javascript
// Before:
calibration: { humanJudgeKappa: 0.74, dimensionKappa: { factuality: 0.61 } }

// After:
calibration: { humanJudgeAlpha: 0.74, dimensionAlpha: { factuality: 0.61 } }
```

Calibration status panel text:
```
// Before:
Human-judge agreement: κ = 0.74 (good)

// After:
Human-judge agreement: α = 0.74 (good — Krippendorff's Alpha)
```

---

### Patch 9 — Section 33 (Agents): Fatigue Agent Action in Log

**Add to Annotation Router Agent action log:**

```javascript
{
  type: "fatigue_detected",
  icon: "⏸️",
  text: "Fatigue signal: Priya Nair — α dropped 0.81→0.68 over 94 min session. Rest prompt shown.",
  action: "Rest prompt delivered. Session paused.",
  timestamp: "15:34",
  reversible: false,
  requiresHuman: false,
  note: "Annotator resumed at 15:52. α recovered to 0.79."
}
```

---

### Patch 10 — Section 20 (Bulk Ingestion): Diversity Warning on Large Datasets

After bulk ingestion completes, if diversity score < 60 on the resulting dataset, show alert in bulk job dashboard:

```
⚠️  Low Diversity Detected
Dataset diversity score: 48 / 100
Top cluster "loan eligibility queries" = 61% of all rows.
LIMA (2023) shows homogeneous datasets underperform diverse
ones even at larger sizes.
Recommendation: Enrich with under-represented query types
before training.
[View diversity breakdown]  [Create annotation sprint]
```

---

### Summary: Implementation Priority for Claude Code

Build in this order — each builds on the previous:

```
Priority 1 (visual, high impact, low complexity):
  - Confidence intervals on eval scores (Patch 1)
  - Krippendorff's α rename everywhere (Patch 4)
  - Typed hallucination badges (Patch 7 — badge only, no inline highlighting yet)

Priority 2 (interaction, medium complexity):
  - AI-suggested edits panel (Patch 2)
  - RLTHF two-section preference queue (Patch 2)
  - Dataset diversity score gauges (Patch 3)
  - Swap-augmented eval animation (Patch 1)

Priority 3 (data + logic, higher complexity):
  - Claim-level hallucination with inline highlighting (Patch 7 — full)
  - Fatigue detection + PM dashboard (Patch 5 + Patch 9)
  - Model change warning modal (Patch 6)
  - Bulk ingestion diversity alert (Patch 10)
```

---

### Implementation Notes for Claude Code (delta)

135. Confidence intervals: stored as `{ mean: 79, variance: 5 }` per dimension in mock eval data. Display as "79 ± 5". High variance threshold for amber: variance > 7.
136. Swap-augmented animation: two-phase progress bar. Label changes between phases. Total mock duration: 3.5 seconds (not 3–4 as previously specified).
137. AI-suggested edit panel: collapsed by default. Expands on click of "💡 Suggested edit available" link below the output. Not auto-expanded — annotator must opt in.
138. RLTHF auto-label badge: small grey "Auto" pill on each auto-labelled pair in the collapsed section. Clicking "Audit 5 samples" expands 5 random auto-labelled pairs for human spot-check.
139. Diversity gauge: same visual style as quality score gauge (circular, colour-coded). Place side-by-side. Score < 60 = red, 60–75 = amber, > 75 = green.
140. α rename: this is a global find-replace in all UI text strings. No logic changes — purely display. Ensure all tooltip text, table headers, and badge labels are updated.
141. Typed hallucination badge: rendered as a small coloured pill replacing the generic "hallucination" flag label. Three colours as specified. Badge click opens the claim-level breakdown panel (if available in mock data).
142. Inline sentence highlighting: implemented using `<mark>` elements with CSS class per type. Green = grounded, amber = contradiction, red = fabrication, purple = conflation. Click opens a tooltip-style popover with the source chunk text.
143. Fatigue rest prompt: z-index above content but below modals. Positioned bottom-right corner. Does not block annotation — annotator can continue without dismissing. Auto-dismisses after 30 seconds if no interaction.
144. Model change warning: fires once per experiment creation when base_model changes. Stores "warned" flag in session so it doesn't repeat if user dismisses and re-opens the form.
145. Bulk diversity alert: shown in the post-completion summary panel of bulk job dashboard. Only fires when diversity score < 60. [Create annotation sprint] pre-fills sprint name with "Diversity enrichment — [underrepresented cluster name]".

---

## 42. DELTA — DOMAIN-AWARE SYNTHESIS TEMPLATES

### Overview
The instruction pair synthesis pipeline (Section 19 — Document Ingestion) currently uses a generic prompt. This delta adds a configurable domain template system — pre-loaded templates per use case, editable by Architects, typed pair output.

---

### Synthesis Template Configuration Screen

**Where:** Document Ingestion → Chunking screen → after [Apply Chunking] → "Configure Synthesis" step

**New UI panel:**

```
┌─────────────────────────────────────────────────────────────┐
│  Synthesis Template                                         │
├─────────────────────────────────────────────────────────────┤
│  Select domain template:                                    │
│                                                             │
│  ○ Generic Q&A          General purpose — any domain       │
│  ● Audit & Compliance   Government audit, GFR, CAG-type    │
│  ○ Legal Reference      Contracts, case law, statutes      │
│  ○ Financial Services   NBFC, banking, RBI guidelines      │
│  ○ Medical / Clinical   Clinical guidelines, diagnostics   │
│  ○ Custom               Edit prompt directly               │
│                                                             │
│  Pair types to generate (multi-select):                    │
│  ☑ Rule / Norm explanation                                 │
│  ☑ Violation detection                                     │
│  ☑ Evidence checklist                                      │
│  ☑ Finding formulation                                     │
│  ☑ Recovery / impact quantification                        │
│                                                             │
│  Pairs per chunk: [2 ▾]                                    │
│  [Preview prompt] [Edit prompt ▾ — Architect only]         │
└─────────────────────────────────────────────────────────────┘
```

---

### Built-In Templates (mock — not real LLM calls)

**Template 1 — Generic Q&A (default)**
```
Instruction shown in UI:
"Generate factual Q&A pairs from the text. 
Focus on what the text says."

Sample output pair type: Rule explanation only
```

**Template 2 — Audit & Compliance**
```
Instruction shown in UI:
"Generate pairs that reflect how a government auditor 
would use this text — focused on compliance checking, 
violation detection, and finding formulation."

Five pair types generated:
1. Rule explanation:     "What does [rule] require?"
2. Violation detection:  "What constitutes non-compliance with [rule]?"
3. Evidence checklist:   "What documents should an auditor examine to 
                          verify compliance with [rule]?"
4. Finding formulation:  "How should an auditor frame a finding when 
                          [rule] has been violated?"
5. Impact quantification:"How is the financial impact of a [rule] 
                          violation calculated and reported?"
```

**Template 3 — Legal Reference**
```
Pair types:
1. Provision explanation
2. Applicability conditions
3. Exception and exclusion identification
4. Precedent and interpretation questions
5. Cross-reference to related provisions
```

**Template 4 — Financial Services**
```
Pair types:
1. Product/rule explanation
2. Eligibility and threshold questions
3. Compliance violation scenarios
4. Customer-facing response formulation
5. Regulatory cross-reference (RBI/SEBI/IRDAI)
```

**Template 5 — Medical / Clinical**
```
Pair types:
1. Clinical guideline explanation
2. Contraindication and risk identification
3. Diagnostic criteria questions
4. Treatment protocol steps
5. Differential diagnosis scenarios
```

---

### Pair Type Tagging in Dataset

Each synthesised row tagged with its pair type:

```javascript
{
  id: "row_201",
  instruction: "What constitutes non-compliance with GFR Rule 230 on contractor advances?",
  input: "",
  output: "Non-compliance with GFR Rule 230 occurs when: (1) advance paid without contractual provision, (2) advance not recovered from first running bill, (3) recovery deferred beyond scheduled instalments. Auditor should verify contract terms, payment voucher, and running bill dates.",
  qualityScore: 88,
  flags: [],
  language: "en",
  reviewStatus: "approved",
  pairType: "violation_detection",         // ← new field
  sourceChunk: "chunk_gfr_rule230_para1",  // ← traceability
  synthesisTemplate: "audit_compliance",   // ← which template generated this
  domainTag: "audit"                       // ← domain
}
```

---

### Domain Expert Seeding Queue (New Feature)

Separate from the annotation queue. Accessible via: Datasets → [+ Add Seed Annotations]

**Purpose:** Allow domain experts (not general annotators) to contribute high-value institutional knowledge pairs that synthesis cannot generate.

**UI:**

```
┌─────────────────────────────────────────────────────────────┐
│  Domain Expert Seed Annotation                              │
│  Contribute expert knowledge pairs directly                 │
├─────────────────────────────────────────────────────────────┤
│  Add from: ○ Scratch  ● Existing audit observation          │
│                                                             │
│  Source observation (paste):                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ "Audit of PWD revealed that advances of ₹2.3 Cr    │   │
│  │  were paid to contractors without contractual       │   │
│  │  provision during 2021-22, in violation of GFR      │   │
│  │  Rule 230. Recovery was not initiated as of date    │   │
│  │  of audit. Financial effect: ₹2.3 Cr."             │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Tag this observation:                                      │
│  Rule violated:    [GFR Rule 230                      ]    │
│  Violation type:   [Advance without contractual basis ]    │
│  Evidence needed:  [Contract, payment voucher, bills  ]    │
│  Financial effect: [₹2.3 Cr outstanding advance      ]    │
│  Finding type:     [Compliance — financial irregularity]   │
│                                                             │
│  This generates:                                           │
│  → 1 violation detection pair                              │
│  → 1 evidence checklist pair                               │
│  → 1 finding formulation pair                              │
│  → 1 impact quantification pair                            │
│                                                             │
│  [Generate pairs] [Preview] [Save to dataset]              │
└─────────────────────────────────────────────────────────────┘
```

**Who can use this:** Architect and ML Engineer roles only (not Annotator). Reflects that domain seeding requires expertise, not just task completion.

---

### Pair Type Distribution Report

Add to Dataset detail screen — below diversity score:

```
Pair Type Distribution
──────────────────────────────────────────────
Rule explanation        ████████████  38%
Violation detection     ██████████    31%   ← good for audit use case
Evidence checklist      ████          14%
Finding formulation     ███           10%   ← under-represented
Impact quantification   ██             7%   ← under-represented

⚠️  Finding formulation and impact quantification under-represented.
   For an audit chatbot, these are the highest-value pair types.
   Recommendation: Run domain expert seeding sprint focused on 
   these two types.
```

---

### Mock Data — CAG / Audit Project

Add a fifth project to IITM Pravartak tenant (or as a separate demo): "CAG Audit Intelligence"

```javascript
{
  id:          "proj_cag_audit",
  tenantId:    "tenant_pravartak",
  name:        "CAG Audit Intelligence",
  description: "NLP chatbot for government auditors — GFR compliance, audit observation drafting, financial rules reference",
  status:      "active",
  synthesisTemplate: "audit_compliance",
  pairTypeDistribution: {
    rule_explanation:      38,
    violation_detection:   31,
    evidence_checklist:    14,
    finding_formulation:   10,
    impact_quantification:  7
  },
  domainSeedRows: 47,   // contributed by senior auditors
  corpusSize: 1200,     // chunks from ingested GFR + circulars
  diversityScore: 72,
  qualityScore: 81
}
```

**Mock synthesised pairs (pre-loaded in dataset):**

```javascript
[
  {
    id: "cag_row_001",
    pairType: "violation_detection",
    instruction: "What conditions would cause advance payment to a contractor to constitute an audit irregularity under GFR?",
    output: "An advance payment constitutes an audit irregularity under GFR Rule 230 when: (1) paid without contractual provision, (2) not recovered from the first running bill, or (3) recovery deferred beyond scheduled instalments. Examine the contract terms, payment voucher date, and first running bill to establish timing of recovery.",
    qualityScore: 91, flags: [], synthesisTemplate: "audit_compliance"
  },
  {
    id: "cag_row_002",
    pairType: "evidence_checklist",
    instruction: "What documents should an auditor examine to verify compliance with GFR Rule 230 on contractor advances?",
    output: "Documents to examine: (1) Original contract — verify advance clause and quantum, (2) Sanction order for advance — check authorisation level, (3) Payment voucher — verify date and amount, (4) First running bill — verify recovery deduction, (5) Subsequent bills if recovery spread — verify instalment schedule, (6) Correspondence if recovery disputed.",
    qualityScore: 89, flags: [], synthesisTemplate: "audit_compliance"
  },
  {
    id: "cag_row_003",
    pairType: "finding_formulation",
    instruction: "How should an auditor frame an audit finding when GFR Rule 230 has been violated through unrecovered advance?",
    output: "Structure the finding as: Criteria: GFR Rule 230 requires advances to contractors to be recovered from the first running bill. Condition: Advances totalling ₹X were paid to M/s [contractor] during [period] but recovery was not initiated as of [audit date]. Cause: [reason — lack of monitoring / oversight failure]. Effect: Government funds of ₹X remain at risk with no assurance of recovery. Recommendation: Initiate immediate recovery and fix accountability.",
    qualityScore: 94, flags: [], synthesisTemplate: "audit_compliance"
  },
  {
    id: "cag_row_004",
    pairType: "rule_explanation",
    instruction: "What is the purpose of GFR Rule 230 and when does it apply?",
    output: "GFR Rule 230 regulates advance payments to contractors in government procurement. It applies whenever a contract involves advance payment and requires: (a) advances be permitted only under explicit contract terms, (b) recovery commence from the first running bill. The rule protects government funds by ensuring advances are not converted into interest-free loans to contractors.",
    qualityScore: 88, flags: [], synthesisTemplate: "audit_compliance"
  }
]
```

---

### Implementation Notes for Claude Code (delta)

146. Synthesis template selector: shown as radio buttons in the synthesis configuration step. Selecting a template updates the pair type checkboxes to the recommended selection for that domain.
147. [Preview prompt] button: opens a read-only modal showing the full prompt text that will be sent to the judge for this template + pair type combination. Architect role can edit; others read-only.
148. Pair type badge: shown on each row card in the dataset table as a small coloured pill. Rule explanation = blue, Violation detection = amber, Evidence checklist = teal, Finding formulation = coral, Impact quantification = purple.
149. Pair type distribution chart: horizontal bar chart, same styling as instruction type diversity chart from Patch 3. Under-represented types shown with recommendation callout below chart.
150. Domain expert seeding form: Architect/ML Engineer only. Hidden from Annotator and Reviewer roles. Accessible from Dataset detail → [+ Add Expert Pairs] button (appears only for users with correct role).
151. CAG audit mock project: pre-load in Pravartak tenant. Pair type distribution pre-computed. 4 sample rows pre-loaded with realistic audit content as above.
152. Generated pairs from domain seeding tagged with `source: "domain_expert_seed"` vs `source: "synthesis_pipeline"` — visible in row detail panel. Expert seed rows shown with a gold star badge.
153. Synthesis template name shown in dataset header: "Template: Audit & Compliance" visible alongside quality and diversity scores.

---

## 43. DELTA — MERGE DATASETS + DOCUMENTS INTO UNIFIED "DATA" MODULE

### Navigation Change

Remove "Datasets" and "Documents" as separate dock icons.

Replace with a single dock icon:

```
📁  Data
```

Position: first item in the dock (above Review Queue).

---

### Data Screen — Three Tabs

```
┌─────────────────────────────────────────────────────────────┐
│  Data    [Structured ▾]  [Documents ▾]  [Import ▾]         │
│          (active tab underlined in coral)                   │
└─────────────────────────────────────────────────────────────┘
```

**Tab 1 — Structured**
Everything previously in the Datasets screen. Upload JSONL, CSV, TSV, Parquet. View curated rows, quality scores, flag breakdown, diversity score, pair type distribution. Manage snapshots. Export.

**Tab 2 — Documents**
Everything previously in the Documents screen. Upload PDFs (digital or scanned). Configure chunking strategy. Select synthesis template. View OCR confidence per document. Track generated pairs.

**Tab 3 — Import**
The annotation import connector (Section 34). Connect Label Studio, Argilla, Prodigy. Map annotation fields to instruction pairs. View import history.

---

### Project Setup Wizard (New — shown on first visit to Data when project has no data)

When a user opens Data on a new project with zero rows:

```
┌─────────────────────────────────────────────────────────────┐
│  Where does your training data come from?                   │
│                                                             │
│  ○ Structured files                                         │
│    CSV, JSONL, Parquet — already formatted as Q&A pairs    │
│    → Start in Structured tab                               │
│                                                             │
│  ○ Documents                                                │
│    PDFs, circulars, reports — need conversion to pairs     │
│    → Start in Documents tab                                │
│                                                             │
│  ○ Annotation tool export                                   │
│    Label Studio, Argilla, Prodigy                           │
│    → Start in Import tab                                    │
│                                                             │
│  ○ All of the above                                         │
│    → See recommended sequence below                        │
│                                                             │
│  [Continue →]                                               │
└─────────────────────────────────────────────────────────────┘
```

When "All of the above" is selected, show sequence guidance:

```
Recommended sequence for your project:

1. Documents tab    → ingest PDFs, generate pairs
2. Import tab       → bring in annotation tool exports
3. Structured tab   → upload any existing JSONL/CSV
4. Review Queue     → all rows from all sources appear here
                      for curation and annotation
```

Wizard shown once. Dismissed on Continue. Not shown again.

---

### How All Paths Converge

All three ingestion paths produce rows in the same dataset:

```
Documents tab  →  chunk → synthesise  →  rows enter Structured tab
Import tab     →  convert format      →  rows enter Structured tab
Structured tab →  upload directly     →  rows in Structured tab
```

The Structured tab is always the single view of all approved rows regardless of how they were ingested. The distinction between tabs is ingestion method only — not data type.

---

### Reference Updates Throughout Spec

All earlier sections that reference "Datasets screen" or "Documents screen" now point to:

```
"Datasets screen"  →  "Data → Structured tab"
"Documents screen" →  "Data → Documents tab"
```

Specific sections affected:
- Section 8.3 (Datasets screen spec) → now Data → Structured tab
- Section 8 (Documents) → now Data → Documents tab
- Section 19 (Document Ingestion) → entry point is Data → Documents tab
- Section 20 (Bulk Ingestion) → accessible from Data → Documents tab → [Bulk Upload]
- Section 34 (Import Connector) → now Data → Import tab
- Section 39 (NBFC mock data) → dataset navigation updated to Data → Structured
- All dashboard CTAs → update "Go to Datasets" → "Go to Data"
- All dock icon references → replace two icons with one

---

### Mock Data Update

NBFC case study project setup wizard response: "Documents" selected (PDFs were the primary source). Wizard shows: "Start in Documents tab → then review generated pairs in Structured tab."

Pravartak project: "All of the above" selected. Wizard shows the recommended sequence.

Legal AI Corp: "Import" selected (Label Studio is their primary annotation tool).

HealthBot India: "Structured" selected (they upload JSONL directly).

---

### Implementation Notes for Claude Code (delta)

154. Remove Datasets and Documents from dock. Add single Data icon. Badge on Data icon = total flagged rows across both Structured and Import sources (same count that was previously on Datasets badge).
155. Tab memory: remember last active tab per project. If user was on Documents tab last session, Data opens on Documents tab next visit.
156. Setup wizard: show only when project has zero rows in all three tabs. Dismiss permanently on Continue. Store dismissed state in project mock data.
157. "All of the above" sequence guidance: render as a numbered list with coloured step indicators. Not interactive — informational only. Clicking Continue takes user to Documents tab (first step in the sequence).
158. Structured tab header: show combined row count from all ingestion sources. Below count, show source breakdown as small pills: "2,000 from upload · 300 from documents · 500 from import."
159. Documents tab: retain all existing Documents screen functionality. No content changes — location change only.
160. Import tab: retain all existing Import Connector functionality from Section 34. No content changes — location change only.
161. Breadcrumb: when navigating to Data from a dashboard CTA, land on the correct tab. "View flagged rows" CTA → Data → Structured. "Review OCR corrections" → Data → Documents. "View import history" → Data → Import.

---

## 44. DELTA — CHECKPOINT INTEGRITY AND TAMPER DETECTION

### Overview
Three additions to checkpoint registration and eval harness to detect weight modifications, endpoint drift, and base model changes.

---

### Addition 1 — Hash Verification on Every Eval Run

Before calling the inference endpoint for any eval run, Cueval recomputes the checkpoint file hash and compares against the value stored at registration.

**Flow:**
```
[Run Eval] clicked
        ↓
Cueval worker reads checkpoint.location from DB
        ↓
Recomputes SHA256 of adapter_model.safetensors (or full model if not LoRA)
        ↓
Compares against stored checkpoint.weightsHash
        ↓
Match:    proceed with eval
Mismatch: block eval, log to audit trail, alert Admin
```

**Mismatch UI (shown instead of eval results):**
```
┌─────────────────────────────────────────────────────────────┐
│  ⛔  Checkpoint Integrity Check Failed                      │
│                                                             │
│  Checkpoint:    ckpt_cag_v1.1                              │
│  Registered:    08 Apr 2025 14:32  by Rohan Sinha          │
│  Checked:       15 Jun 2025 10:14                          │
│                                                             │
│  Stored hash:   e4f2a9b1c7d3e8f5...                        │
│  Current hash:  a1b2c3d4e5f6a7b8...                        │
│                                                             │
│  The weights file at /data/models/cag-llm/v1.1/ has been  │
│  modified since checkpoint registration. Eval blocked.     │
│                                                             │
│  This event has been logged to the audit trail.            │
│  Admin has been notified.                                  │
│                                                             │
│  If the model was intentionally updated, register a        │
│  new checkpoint before running eval.                       │
│                                                             │
│  [Register New Checkpoint]  [View Audit Log]               │
└─────────────────────────────────────────────────────────────┘
```

**Audit log entry created automatically:**
```javascript
{
  event:        "checkpoint_integrity_failure",
  checkpointId: "ckpt_cag_v1.1",
  storedHash:   "e4f2a9b1...",
  foundHash:    "a1b2c3d4...",
  detectedBy:   "eval_worker",
  detectedAt:   "2025-06-15T10:14:22Z",
  evalRunId:    "eval_003",
  projectId:    "proj_cag_audit",
  notified:     ["admin@cag.gov.in"]
}
```

---

### Addition 2 — Probe Fingerprint Check

At checkpoint registration, Cueval sends a fixed probe question to the inference endpoint and stores the response embedding. At eval time, resends the same probe and compares embeddings. Catches the case where the endpoint URL is the same but serves different weights.

**At registration — probe capture:**
```
Fixed probe question (hardcoded, same for all projects):
"Describe the capital city of France in one sentence."

→ sent to inference endpoint
→ response received
→ MiniLM embedding computed from response
→ embedding stored in checkpoint record
```

**At eval time — probe comparison:**
```
Same probe sent to same endpoint
→ new response received
→ new embedding computed
→ cosine similarity against stored embedding

similarity ≥ 0.95:  ✅ Endpoint fingerprint matches — proceed
similarity 0.85–0.95: ⚠️ Endpoint fingerprint shifted — warn but allow
similarity < 0.85:   ⛔ Endpoint fingerprint mismatch — block + alert
```

**Warning UI (similarity 0.85–0.95):**
```
⚠️  Endpoint fingerprint has shifted since checkpoint registration.
    The model at http://cag-llm-internal:8080 may have been updated.
    Similarity: 0.89 (threshold: 0.95)
    
    Proceed only if you are certain this endpoint still serves
    ckpt_cag_v1.1. Otherwise register a new checkpoint.
    
    [Proceed anyway — log this]  [Cancel — Register New Checkpoint]
```

**Mock data:**
```javascript
checkpoint.probeFingerprint = {
  probeQuestion: "Describe the capital city of France in one sentence.",
  responseEmbedding: [0.23, -0.41, 0.87, ...],  // 384 floats
  capturedAt: "2025-04-08T14:35:00Z",
  capturedBy: "registration_worker"
}
```

---

### Addition 3 — Base Model Version Stored and Checked

Checkpoint record stores base model ID and version. Warning shown when base model changes between experiments in the same project.

**Updated checkpoint record:**
```javascript
{
  id:               "ckpt_cag_v1.1",
  experimentId:     "exp_002",
  location:         "/data/models/cag-llm/v1.1/",
  inferenceEndpoint:"http://cag-llm-internal:8080/generate",
  weightsHash:      "e4f2a9b1c7d3e8f5...",    // adapter weights
  baseModel: {
    id:      "mistralai/Mistral-7B-Instruct-v0.2",
    version: "v0.2",
    source:  "huggingface"
  },
  adapterType:  "lora",
  loraRank:     8,
  createdAt:    "2025-04-08T14:32:00Z",
  probeFingerprint: { ... }
}
```

**Warning when experiment uses different base model from previous:**

Shown when creating a new experiment and the selected base model differs from the most recent completed experiment's checkpoint.baseModel.id:

```
┌─────────────────────────────────────────────────────────────┐
│  ⚠️  Base Model Changed                                     │
│                                                             │
│  Previous experiment:  Mistral-7B-Instruct-v0.2            │
│  This experiment:      Mistral-7B-Instruct-v0.3            │
│                                                             │
│  Dataset quality scores were calibrated for v0.2.          │
│  Quality scores may not transfer to v0.3 — rows that       │
│  helped v0.2 may behave differently with v0.3              │
│  (Ye et al., 2025, arxiv:2510.13212).                      │
│                                                             │
│  Also: eval scores from ckpt_v1.1 (v0.2) are not          │
│  directly comparable to new checkpoints on v0.3.           │
│  Run a fresh baseline eval after switching.                │
│                                                             │
│  [Proceed anyway]  [Run calibration experiment first]      │
└─────────────────────────────────────────────────────────────┘
```

---

### Updated Checkpoint Registration UI

```
┌─────────────────────────────────────────────────────────────┐
│  Register Checkpoint                                        │
├─────────────────────────────────────────────────────────────┤
│  Checkpoint name:   [ckpt_cag_v1.1                    ]    │
│                                                             │
│  Location type:                                             │
│  ● Local path    ○ S3 / MinIO    ○ HuggingFace             │
│                                                             │
│  Path: [/data/models/cag-llm/v1.1/               ]        │
│                                                             │
│  Inference endpoint:                                        │
│  [http://cag-llm-internal:8080/generate           ]        │
│                                                             │
│  Base model:                                                │
│  [mistralai/Mistral-7B-Instruct-v0.2              ]        │
│                                                             │
│  Adapter type:  ● LoRA  ○ Full fine-tune  ○ Other         │
│  LoRA rank:     [8]                                         │
│                                                             │
│  [Compute hash + capture probe fingerprint]                │
│                                                             │
│  Hash:        ✅ e4f2a9b1c7d3e8f5...  (computed)           │
│  Fingerprint: ✅ Captured — cosine self-check 1.00         │
│                                                             │
│  [Register Checkpoint]                                     │
└─────────────────────────────────────────────────────────────┘
```

[Compute hash + capture probe fingerprint] is a single action:
1. SHA256 of weights file
2. Sends probe to inference endpoint
3. Computes and stores embedding
4. Shows both results before registration is confirmed

---

### What Should Happen Operationally (shown as a callout in the UI)

Shown on the checkpoint registration screen as a permanent note:

```
📋  If the model weights are updated for any reason — hotfix,
    rate change, version bump — register a NEW checkpoint.
    Do not reuse an existing checkpoint record.
    Cueval will detect the hash mismatch and block eval anyway.
```

---

### Mock Data Updates

**NBFC case study — add integrity metadata to existing checkpoints:**
```javascript
ckpt_cag_v1.0.weightsHash = "a3f8c2d9e1b4f7c3..."
ckpt_cag_v1.0.baseModel = { id: "mistralai/Mistral-7B-Instruct-v0.2", version: "v0.2" }
ckpt_cag_v1.0.probeFingerprint = { capturedAt: "2025-04-08T14:35:00Z" }
ckpt_cag_v1.0.integrityStatus = "verified"  // hash matched on last eval run

ckpt_cag_v1.1.weightsHash = "b7d2a1e4c9f3..."
ckpt_cag_v1.1.integrityStatus = "verified"
```

**Add one mock integrity failure to Legal AI Corp for demo purposes:**
```javascript
{
  checkpointId: "ckpt_legal_v2.0",
  integrityStatus: "failed",
  failureDetectedAt: "2025-06-10T09:22:00Z",
  storedHash: "c4e5f6a7b8...",
  foundHash:  "d1e2f3a4b5...",
  evalRunBlocked: true
}
```

Legal AI Corp eval harness shows the blocked state with the integrity failure UI above. [Register New Checkpoint] CTA links to registration form pre-filled with the same endpoint.

---

### Implementation Notes for Claude Code (delta)

162. Hash computation: mock with a pre-stored hash value per checkpoint. [Compute hash] button shows a 2-second spinner then reveals the hash. Do not implement real SHA256 computation — store mock values.
163. Probe fingerprint: mock with a stored cosine similarity value per checkpoint. [Compute hash + capture probe fingerprint] captures both in one mock action.
164. Integrity status badge: shown on each checkpoint card in experiment tracker. Green shield "✓ Verified" for passing, red shield "✗ Integrity failure" for failing. Legal AI Corp ckpt_legal_v2.0 shows red.
165. Integrity failure UI: full-screen warning replacing the normal eval results view. Not dismissable — user must choose [Register New Checkpoint] or [View Audit Log]. No way to proceed with a failed integrity check.
166. Probe similarity warning (0.85–0.95): shown as an amber interstitial before eval results. User can proceed — logged if they do.
167. Base model change warning: shown once per experiment creation when base_model differs from previous experiment's checkpoint. Proceed closes the modal. "Run calibration" opens a new experiment form pre-filled with 200-row subset config.
168. Audit log entry for integrity failure: auto-created, visible to Admin role in audit log. Shown with red background in the log list. Filterable by event type "checkpoint_integrity_failure".

## 51. DELTA — V1 EVALUATION CORE (SEEDED COMPARISON + GROUNDED CHECKING)

### Overview
This delta re-scopes the product to a shippable v1: the **evaluation-and-audit layer only**. It does not invent a new eval engine — Section 37 (reference-grounded scoring, judge selection) and Delta 47 (bring-your-own-judge, agreement report) already carry most of it. It makes three changes:

1. Elevates **seeded ground-truth comparison** to the *primary* validation mode (Section 37's grounded scoring becomes the complement, not the headline).
2. Adds the pieces that mode needs — a comparison-judge rubric, seed-set authoring and coverage tracking, a regression suite — plus retrieval transparency for the grounded complement.
3. Adds a **v1 evaluation-only deployment profile** (extending Delta 48) that hides everything deferred, so the product and the spec both stop presenting the full loop.

Core logic of v1: point Cueval at a model the customer already built and a reference corpus; validate its answers two ways — against a fixed expert-approved test set (primary) and against the corpus for open-ended questions (complement); wrap both in human calibration; emit one audit report. Everything else in the eight-stage loop is deferred, not built.

---

### Validation mode 1 — Seeded ground-truth comparison (PRIMARY)

A fixed test set of `(question, expert-approved answer)` pairs. The customer's model answers each question; a judge decides whether the model's answer is equivalent to the baseline and where it differs.

Why primary: no retrieval dependency (the correct answer is pinned, not fetched), most legible to a regulator/reviewer, gives the judge the *easy* task (compare to a known answer, not grade open-ended), and doubles as a regression suite.

**Comparison-judge rubric** (distinct from Section 37's grounded rubric):

```
Given QUESTION, BASELINE (expert-approved), and CANDIDATE (model output):
  verdict ∈ { match | partial | mismatch }
  differences: [ what the candidate adds, omits, or contradicts vs baseline ]
  per-dimension where applicable (factual equivalence, completeness, safety)
Return structured JSON. Do NOT answer the question — only compare.
```

`partial` covers "same facts, extra caveats" or "correct but incomplete" — the common real case. The judge never generates an answer; it only compares, which is why a small local model suffices (see judge tiers).

**Test-set / run data model:**

```javascript
eval_sets = [
  { id, tenantId, projectId, name, purpose,   // e.g. "GFR core Q&A v1"
    provenance,                                 // authored | synthesis_drafted_then_approved
    coverage,                                   // { intents:[], nQuestions, samplingMethod, notes }
    approvedBy, approvedAt }
]
eval_set_items = [
  { id, setId, question, baselineAnswer, expertId, tags:[] }
]
eval_runs = [
  { id, setId, modelRef, judgeConfigId, createdAt,
    results:[ { itemId, candidateAnswer, verdict, differences, dims } ],
    summary:{ match, partial, mismatch, scorePct } }
]
```

---

### Validation mode 2 — Reference-grounded checking (COMPLEMENT)

Reuses Section 37's grounded scoring for open-ended and production questions the seed set can't cover. Two additions required by this architecture:

- **Retrieval citation on every verdict.** Each grounded judgment records *which corpus chunk it graded against*, shown in the report, so mis-retrieval is visible and correctable — not silent.
- **Fourth verdict: `no_relevant_source`.** When retrieval returns nothing above a similarity threshold, the judge abstains and routes to a human instead of grading against weak chunks. This bounds the confident-wrong failure mode.

Grounded mode's accuracy is explicitly framed as retrieval-bounded; the citation trail and abstention make that boundary auditable.

---

### Judge tiers (v1)

One pluggable judge interface, three deployments (extends Section 37 / Delta 47):

```
Default        Cloud API judge (best accuracy, zero setup)      non-sensitive pilots
Sovereign      Local general-instruct model (7–8B class)        data can't leave network
BYO            Customer's own model / endpoint (Delta 47)       skeptical / self-standardised buyers
```

The sovereign tier is a NEW addition: an off-the-shelf **general** instruct model (not a self-trained rater), prompted with the comparison rubric — private, cheap, no training data required. Selecting the sovereign tier disables the cloud-API option and skips the DLP gate. A self-trained comparator is explicitly out of v1 (revisit only once pilots have produced labelled comparison data; see deferrals).

---

### Calibration wraps both modes

Reuses the Delta 47 agreement report, extended:

- Covers **both** the comparison judge's equivalence decisions and the grounded judge's verdicts.
- Reported **per dimension** (e.g. factual equivalence α 0.85, tone α 0.55) — never a single "it works" number. Dimensions below a threshold are marked human-review-only.
- Disagreements are separated into **bad judging vs bad retrieval** (grounded mode), so the reported agreement reflects the end-to-end pipeline the customer actually runs, not a judge in isolation.

The pilot's real output is this per-dimension agreement profile, not a headline score.

---

### Seed-set construction & coverage (the attackable point, made explicit)

Because a passing score on a narrow or model-friendly set is false comfort, seed-set provenance is first-class:

- Questions may be **synthesis-drafted from the corpus** (reusing existing templates) then **expert-approved** — turning authoring into faster review. `provenance` records which.
- **Coverage** is tracked: which intents/topics the set spans, how questions were sampled, how much of the real question distribution is (and isn't) represented.
- The audit report includes a **coverage statement** alongside the score: what this set does and doesn't validate. The deliverable is never the number alone.

---

### Regression suite

An `eval_set` can be re-run against a new `modelRef` after each fine-tune; the UI shows score deltas per item and per dimension, and flags regressions (items that were `match` and became `mismatch`). This is the reusable-benchmark value of the seeded mode.

---

### v1 deployment profile — `evaluation_only`

Extends Delta 48's profiles. Visible: reference-corpus ingest (minimal), Eval (both modes), Calibration/Agreement, Audit export, Settings. Hidden/deferred (shown as "future stage" per Delta 48): Synthesis-as-product, the full annotation/correction cockpit, Training, Release, Monitor, Agents.

This profile is what makes the spec itself reflect the narrowed product rather than the eight-stage loop.

---

### What v1 explicitly defers

Training · data synthesis as a product surface · the full Delta 46 correction cockpit (only minimal corpus ingest remains) · production-monitoring SDK · serving/deployment · agents · Delta 48's other profiles beyond evaluation_only · a self-trained judge model. Each is earned later by customer pull, not built up front:
- **v2:** review/correction cockpit — pulled by errors the eval surfaces.
- **v3:** production monitoring (grounded checks on live traffic) + optional distilled small rater, funded by accumulated pilot labels.

---

### Mock data updates

- Add one `eval_set` ("GFR core Q&A v1", 40 items, `provenance: synthesis_drafted_then_approved`, coverage across 6 intents) with expert-approved baselines.
- One `eval_run` of that set against the CAG mock model: 40 items → 31 match / 6 partial / 3 mismatch, scorePct 77.5, with 2 mismatches carrying `differences` (wrong rule number; omitted exception clause).
- One grounded run on 10 open-ended questions using the same corpus, including 2 `no_relevant_source` abstentions with retrieval citations shown.
- Extend the agreement report: per-dimension α for both modes (factual equivalence 0.82, completeness 0.74, tone 0.58), with tone flagged human-review-only, and 3 disagreements tagged bad-retrieval vs bad-judgment.
- Set the CAG tenant to `deploymentProfile: 'evaluation_only'`.

---

### Implementation Notes for Claude Code (delta)

221. Add seeded comparison as the primary eval mode. New screens: Eval Sets (list/create), Set editor (question + baseline + expert), Run (pick model + judge), Results (match/partial/mismatch with per-item differences). Reference-grounded mode (Section 37) remains available as a second tab, relabelled "Open-ended / grounded".
222. Comparison judge: implement the comparison rubric as a distinct prompt from Section 37's grounded rubric. Judge receives question + baseline + candidate, returns `{verdict, differences, dims}`. It must never generate an answer to the question. Mock the judge call; do not require a real model.
223. Verdict set for comparison: `match | partial | mismatch`. `scorePct` = (match + 0.5·partial) / total, shown with the raw counts. Never show scorePct without the counts beside it.
224. Grounded mode additions: every grounded verdict stores and displays the `chunkId` it graded against (retrieval citation). Add the `no_relevant_source` verdict when top-retrieval similarity is below a threshold (mock: flag ~10% of grounded items), routing them to a human-review list rather than scoring them.
225. Judge tiers: selector with Cloud API (default) / Sovereign local (general instruct) / BYO (Delta 47). Sovereign disables the cloud option and hides the DLP gate. Do not implement a self-trained rater.
226. Calibration/agreement (extend Delta 47): compute and display per-dimension α for both comparison and grounded runs; mark dimensions below threshold (mock: <0.65) as "human-review only"; tag grounded disagreements as bad-retrieval vs bad-judgment.
227. Seed-set provenance & coverage: `provenance` field (authored | synthesis_drafted_then_approved) and a `coverage` block; render a coverage statement on the audit report next to the score. The audit report must never present a score without its coverage statement.
228. Regression: allow re-running an `eval_set` against a new `modelRef`; show per-item and per-dimension deltas; highlight items that regressed from match to mismatch.
229. Add `evaluation_only` to the Delta 48 profile enum. Under it, show only corpus ingest, Eval (both modes), Calibration, Audit export, Settings; render all other modules as deferred "future stage" cards. Set it as the CAG tenant's profile in mock data.
230. Audit report (extend Delta 49 export): a single exportable, lineage-backed report combining the seeded-set score + coverage statement, the grounded-mode findings with retrieval citations, and the per-dimension agreement profile. This is v1's headline deliverable.