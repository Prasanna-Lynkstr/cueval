# Cueval — Delta Specs: High-Priority Workflow Gaps

> **Status:** Proposed. These sections extend the main spec (`PRODUCT_SPEC.md`)
> and are **not yet implemented**. They cover the nine high-priority gaps that
> break common real-world annotation, eval, and experiment workflows.
> Implementation-note numbering continues from the main spec (last note: 100).
>
> **Note on §35:** the original gap was phrased as "page-aware navigation in
> entity labelling." Entity Labelling was withdrawn in main-spec §34, so §35 is
> re-scoped to the surviving multi-page context — OCR Correction and document
> review — where the same need is real.

---

## 35. MULTI-PAGE DOCUMENT NAVIGATION IN CORRECTION TASKS

### Problem
Scanned documents span many pages (24, 32, 500+). Today the OCR Correction queue
presents regions as a flat, worst-first list with no sense of *where in the
document* you are. Annotators lose context, can't jump to a page, and can't tell
how much of a given document remains.

### Where It Lives
Review Queue → OCR Corrections. Adds a **page rail** and a **document scope
switch** to the existing correction pane.

### UI

```
┌──────────────────────────────────────────────────────────────────────┐
│  OCR Corrections    Scope: [Audit_Report_2024.pdf ▾]  Region 3 of 67 │
│  Page 4 of 24        [◄ Prev page] [Next page ►]     [All documents ▾]│
├───────────┬──────────────────────────────────────────────────────────┤
│  PAGES    │  [ current region correction UI — scan | text | actions ]│
│  ┌──────┐ │                                                           │
│  │ p.1 ✅│ │  ◀ region-level [◄][►] still work within the page        │
│  │ p.2 ✅│ │                                                           │
│  │ p.3 ⚠️│ │                                                           │
│  │►p.4 🔴│ │                                                           │
│  │ p.5 ⏳│ │                                                           │
│  │ ...  │ │                                                           │
│  └──────┘ │                                                           │
│  6 pages  │                                                           │
│  flagged  │                                                           │
└───────────┴──────────────────────────────────────────────────────────┘
```

**Page rail** (left, ~90px): one chip per page. Chip status:
- ✅ all regions corrected/accepted on that page
- ⚠️ some regions still pending
- 🔴 the page containing the current region (active)
- ⏳ not yet started

Clicking a page chip jumps to that page's first pending region. The rail scrolls
independently and highlights the active page.

**Scope switch (top):** `All documents` (default — global worst-first queue,
today's behaviour) vs a single document. When scoped to a document, the queue is
filtered to that document and ordered by page → region index (reading order, not
worst-first), so corrections proceed naturally through the document.

**Progress line:** `Page 4 of 24 · Region 3 of 67 · document 41% corrected`.

### Mock Logic
`ocr_regions` already carries `pageNumber`. Derive per-page status from the
regions on that page:

```javascript
function pageStatus(docId, page) {
  const regs = regionsForDoc(docId).filter(r => r.pageNumber === page);
  if (!regs.some(r => r.confidence < 60)) return 'clean';      // no flagged regions
  const pending = regs.filter(r => r.correctionStatus === 'pending').length;
  if (pending === 0) return 'done';
  if (regs.some(r => r.correctionStatus !== 'pending')) return 'partial';
  return 'todo';
}
// pages list = unique pageNumbers for the doc, ascending
```

### Implementation Notes
101. Page rail renders one chip per page that has ≥1 region; pages with no OCR
     regions are omitted (they had no low-confidence text). Cap the visible rail
     height; scroll internally.
102. Scope switch defaults to "All documents" to preserve existing worst-first
     triage. Selecting a document persists in `appState` until changed or the
     queue empties.
103. When scoped to a document, sort regions by `(pageNumber, regionIndex)`;
     when "All documents", keep `confidence` ascending (worst first).
104. Keyboard: `[` / `]` move to previous / next page (in addition to the existing
     `←` / `→` region navigation). Guarded by the shortcut-scope rules in §43.
105. Jumping to a page selects its first `pending` region; if none pending, its
     first region. `appState.reviewIndex` is recomputed from the (filtered) queue.

---

## 36. INDIC IME / SCRIPT KEYBOARD SUPPORT IN CORRECTION FIELDS

### Problem
Correcting Tamil/Hindi/Telugu OCR and ASR output requires typing Indic script.
Annotators on machines without an OS-level Indic IME cannot enter the correct
text, silently degrading correction quality on exactly the languages that need
it most.

### Where It Lives
Every correction textarea that can hold Indic text: OCR Corrections, Audio
Corrections. A small **script/IME control** sits above the textarea.

### UI

```
┌──────────────────────────────────────────────────────────────┐
│  Your Correction        Script: [Tamil ▾]  ⌨ Transliterate ⓘ │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ நீதிமன்ற நடவடிக்கை...                                    │  │
│  └────────────────────────────────────────────────────────┘  │
│  Type in Latin (e.g. "neethimandram") → converts on space.   │
│  [Aa Latin] [அ Tamil]  ·  Detected script: Tamil             │
└──────────────────────────────────────────────────────────────┘
```

- **Script selector** defaults to the region/segment's detected script.
- **Transliterate toggle** (⌨): when on, the annotator types Latin phonetic
  input and it converts to the selected Indic script on space/punctuation
  (ITRANS-style). When off, the textarea accepts native input directly (for
  users who *do* have an OS IME).
- A one-line hint shows an example. A quick **Latin ↔ script** switch lets the
  annotator drop back to Latin mid-word (for proper nouns, numbers, English
  terms in mixed text).

### Mock Logic
Prototype ships a small deterministic transliteration map per script (not a full
engine — enough to demonstrate). Convert the last token on a word boundary:

```javascript
// Minimal ITRANS-ish maps; real impl would use a library (e.g. Aksharamukha).
const TRANSLIT = {
  ta: { 'a':'அ','aa':'ஆ','i':'இ','ka':'க','neethi':'நீதி', /* ... */ },
  hi: { 'a':'अ','ka':'क','dharma':'धर्म', /* ... */ },
  te: { /* ... */ }, kn: { /* ... */ }
};
function transliterateToken(latin, script) {
  const m = TRANSLIT[script] || {};
  return m[latin.toLowerCase()] || latin;   // fall through unknown tokens as-is
}
```

The point in the prototype is the **interaction and control**, not linguistic
completeness — unknown tokens pass through unchanged and can be edited by hand.

### Implementation Notes
106. Script selector seeds from the region/segment `script`. Changing it does not
     alter already-typed text; it only affects subsequent transliteration.
107. Transliteration fires on `keyup` when the key is space, Enter, or
     punctuation — convert the just-completed token, preserve caret position.
108. Provide a visible **undo affordance**: `Ctrl/Cmd+Z` must undo a
     transliteration as a single step (store the pre-conversion token).
109. `Aa Latin` toggle temporarily disables conversion so the next token stays
     Latin (for English words / numbers in mixed-script text). Auto-resets after
     one token, or stays until toggled — show which mode is active.
110. Never block native input: if the OS IME is producing script directly, the
     transliterate toggle should be off by default when native Indic characters
     are detected in the field.

---

## 37. SESSION RESUMPTION FOR ANNOTATION TASKS

### Problem
The prototype is memory-only and resets on reload; even within a session, moving
away from a queue loses the cursor. Real annotators work in long sessions across
interruptions and expect to resume exactly where they left off, per task type.

### Where It Lives
Cross-cutting: Review Queue (all tabs), Documents chunking/synthesis, Bulk jobs.
Surfaces as a **"Resume where you left off"** affordance and silent cursor
persistence.

### UI

```
Review Queue banner (shown when a saved position exists for the active tab):
┌──────────────────────────────────────────────────────────────┐
│  ⏱ Resume: Audio Corrections — you were on segment 34 of 98   │
│     (last active 12 min ago)          [Resume]  [Start fresh] │
└──────────────────────────────────────────────────────────────┘
```

Dashboard "My Work Today" also gains a resume hint when a session is in progress:
`"You were reviewing Legal AI v1.1 — 12 of 20 done" → [Resume]`.

### Mock Logic
Maintain a per-tenant, per-task-type **checkpoint** in an in-memory session
store (and, for a browser prototype, optionally mirror to `sessionStorage` so a
reload restores it — *this is the one sanctioned exception to the no-storage
rule, gated behind an ADR because resumption is the feature*):

```javascript
session.checkpoints = {
  // key: `${tenantId}:${taskType}`
  "tenant_pravartak:audio":    { index: 34, itemId: "seg412", ts: "…", label: "Audio Corrections" },
  "tenant_pravartak:datasets": { index: 12, itemId: "r88",    ts: "…", label: "Text Review" },
  // taskType ∈ datasets | ocr | audio | video | pref
};
function saveCheckpoint(taskType, index, itemId) { … }   // called on every advance
function resumeCheckpoint(taskType) { … }                 // sets appState cursor + reviewTab
```

Checkpoint is written on every approve/reject/skip/next. Cleared when its queue
empties. ETA/"last active" derived from the stored timestamp.

### Implementation Notes
111. Save a checkpoint on every cursor advance across ALL five task types, keyed
     by `tenant + taskType`. Never leak across tenants (write through the same
     isolation boundary).
112. On entering a Review tab with a saved checkpoint whose item still exists and
     is still pending, show the resume banner. If the item was completed by
     someone else, resume at the nearest still-pending item and say so.
113. `[Start fresh]` clears the checkpoint and starts at the worst-first item.
114. **ADR required:** persisting checkpoints to `sessionStorage` overrides the
     main-spec "no localStorage/sessionStorage" rule *only for the resumption
     checkpoint object* — no domain content is stored, only cursor indices/ids.
     If the ADR is declined, keep resumption in-memory (survives navigation, not
     reload).
115. Bulk jobs and chunking/synthesis get the same treatment — store the current
     step/page so returning to the screen restores position (see §39 checkpoint
     for bulk).

---

## 38. BULK FIND-AND-REPLACE FOR SYSTEMATIC ASR ERRORS

### Problem
ASR models make the *same* mistake repeatedly — a domain term consistently
mis-transcribed ("Section 302" → "section three oh two", a name always wrong).
Fixing each segment by hand is wasteful when one rule fixes hundreds.

### Where It Lives
Audio Corrections tab → **[Find & Replace]** button (Reviewer/Admin, or any
annotation role — configurable). Operates over the current audio file's segments
(or, scoped up, all pending segments in the tenant).

### UI

```
┌──────────────────────────────────────────────────────────────┐
│  Find & Replace — systematic ASR corrections                 │
├──────────────────────────────────────────────────────────────┤
│  Scope:  ● This file (court_hearing_01.mp3)  ○ All pending    │
│  Find:     [ section three oh two           ]  ☐ Whole word   │
│  Replace:  [ Section 302                     ]  ☐ Case-sensitive│
│  Script:   [ Any ▾ ]   (limit to segments of one script)      │
│                                                              │
│  MATCHES (23 segments)                              [Preview] │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ seg 004  …ke antargat section three oh two…             │  │
│  │          →…ke antargat Section 302…            [skip ▢] │  │
│  │ seg 019  …under section three oh two of…                │  │
│  │          →…under Section 302 of…               [skip ▢] │  │
│  │ … 21 more …                                             │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Applies to correctedText; leaves the original transcript.   │
│  [Cancel]   [Apply to 23 segments]                           │
└──────────────────────────────────────────────────────────────┘
```

- Live match count as you type "Find".
- Per-match **skip** checkboxes (opt out of specific segments).
- Preview shows before → after with the replaced span highlighted.
- Applying writes each match's `correctedText`, marks those segments
  `corrected`, logs one bulk event, and is **undoable as a single action** for
  10 minutes (consistent with the agent-undo pattern).

### Mock Logic
```javascript
function findMatches(scope, find, opts) {
  const segs = scope === 'file' ? segmentsForAudio(activeAudio().id)
                                : getAudioSegments().filter(s => s.status === 'pending');
  const re = buildRegex(find, opts);   // whole-word / case flags
  return segs.filter(s => re.test(s.correctedText ?? s.rawText ?? s.cleanText));
}
function applyReplace(matches, find, replace, opts, skipSet) {
  const re = buildRegex(find, opts);
  matches.filter(m => !skipSet.has(m.id)).forEach(m => {
    const base = m.correctedText ?? m.cleanText;
    m.correctedText = base.replace(re, replace);
    m.status = 'corrected';
  });
}
```

### Implementation Notes
116. Match count recomputes live on "Find" input (debounced). Empty "Find" → no
     matches, Apply disabled.
117. Replace operates on `correctedText` (falling back to the clean/auto text);
     it never mutates the original ASR `rawText`, preserving provenance.
118. Applying records ONE bulk log entry ("Replaced 'x' → 'y' in 23 segments")
     and reduces the Audio Corrections pending badge accordingly.
119. Single-action **Undo** (10-minute window) restores every affected segment's
     prior `correctedText`/`status`. Reuse the agent-log undo pattern/timer.
120. "Whole word" wraps the pattern in word boundaries; escape regex
     metacharacters in the Find term so users type literal text, not regex.

---

## 39. DIFF VIEW FOR EDITED OUTPUTS (TEXT REVIEW)

### Problem
When an annotator edits a row's output, reviewers need to see *what changed*, not
re-read two blocks of text. The current row panel shows original (struck through)
and edited as separate blocks — fine for short strings, useless for paragraphs.

### Where It Lives
Datasets → row detail panel, and Review Queue → Text Review, wherever a row has
`originalOutput` (i.e., `reviewStatus === 'edited'`). Replaces the current
before/after blocks with an inline word-level diff, with a toggle.

### UI

```
┌──────────────────────────────────────────────────────────┐
│  Output   [ Diff ▾ | Original | Edited ]                 │
│  ┌────────────────────────────────────────────────────┐  │
│  │ The clause means neither party is liable for delays │  │
│  │ caused by ~~unforeseeable events~~ ++events beyond   │  │
│  │ their control++ such as natural disasters.          │  │
│  └────────────────────────────────────────────────────┘  │
│  − 2 words removed   + 4 words added   · edited by Ravi   │
└──────────────────────────────────────────────────────────┘
```

- **Deletions** struck through in red (`--red`), **additions** underlined in
  green (`--green`), unchanged text in normal colour. (Never colour-only — the
  strike/underline marks carry the meaning too, per accessibility rules.)
- View switch: **Diff** (default for edited rows) · **Original** · **Edited**.
- Summary line: `− N removed · + M added · edited by <name>`.

### Mock Logic
Word-level LCS diff (no library):

```javascript
function wordDiff(a, b) {
  const A = a.split(/(\s+)/), B = b.split(/(\s+)/);   // keep whitespace tokens
  // classic LCS table → emit ops: 'eq' | 'del' | 'ins'
  // render: eq→plain, del→<del>, ins→<ins>
  return ops; // [{type,text}]
}
```

Escape each token before wrapping in `<del>`/`<ins>` (untrusted content).

### Implementation Notes
121. Diff is the default view when `row.originalOutput != null`; rows never
     edited show the output plainly with no view switch.
122. Word-level (whitespace-preserving) LCS, not character-level — reads far
     better for prose. Collapse runs of identical ops.
123. `<del>`/`<ins>` semantic elements styled with `text-decoration` +
     colour so the diff survives greyscale / colour-blind viewing.
124. Summary counts words, not tokens (ignore pure-whitespace ops).
125. Same diff component is reusable anywhere an original/edited pair exists
     (e.g., OCR corrected vs raw, if desired later).

---

## 40. EVAL COST PRE-ESTIMATE (BEFORE RUNNING)

### Problem
Today cost only appears *after* an eval completes. Users can't decide whether a
run is worth it, and there's no guardrail before spending tokens on a large
rubric × sample set.

### Where It Lives
Eval Harness → run configuration, directly above the **[Run Eval]** button. A
live estimate that updates as the rubric/sample settings change.

### UI

```
┌──────────────────────────────────────────────────────────┐
│  Sample size:   [ 200 rows ▾ ]   of 847 in v1.1          │
│  Judge model:   [ Mock (prototype) ▾ ]                   │
│  Rubric:        5 dimensions · weights total 100% ✅      │
│                                                          │
│  ESTIMATED COST                                          │
│  ~168,000 tokens   ≈ ₹33.60   ·   ~40 s wall-clock       │
│  200 rows × 5 dims × ~168 tok/judgement                  │
│                                                          │
│  [ Run Eval ]   [ Run on 20-row sample first (₹3.40) ]   │
└──────────────────────────────────────────────────────────┘
```

- Estimate = `rows × dimensions × tokensPerJudgement × pricePerToken`, recomputed
  on every change to sample size / rubric / judge model.
- A **"Run on a small sample first"** secondary action (cheap dry-run) is offered
  for large/expensive configs.
- If estimated cost exceeds a threshold (or the tenant's remaining eval budget),
  show a warning band and require confirmation.

### Mock Logic
```javascript
const TOK_PER_JUDGEMENT = { 'Mock': 168, 'Claude API': 210, 'Local Model': 168 };
const PRICE_PER_TOKEN    = 0.0002;   // ₹ (matches existing cost tracker)
function estimateEval(sampleRows, rubric, judge) {
  const tok = sampleRows * rubric.length * (TOK_PER_JUDGEMENT[judge] || 168);
  return { tokens: tok, cost: +(tok * PRICE_PER_TOKEN).toFixed(2),
           seconds: Math.max(3, Math.round(sampleRows / 5)) };
}
```

### Implementation Notes
126. Estimate updates live as sample size, rubric dimension count, or judge model
     change (reuse the live-input listener; no full re-render — patch the number).
127. Show the arithmetic (`rows × dims × tok/judgement`) so the number isn't a
     black box.
128. If `estimate.cost` would push the tenant over its monthly eval budget (plan
     limit), render an orange warning and gate `[Run Eval]` behind a confirm.
129. The small-sample dry-run reuses the normal eval flow with a fixed 20-row
     sample and shows its own (smaller) estimate.
130. On actual run, the real post-run cost tracker (§8.5) reconciles against the
     estimate; show "estimated ₹33.60 · actual ₹32.90" for trust.

---

## 41. LONG-RUNNING EVAL PROGRESS DISPLAY

### Problem
The current eval "run" is a 3–4 s canned animation. Real evals over hundreds of
rows take minutes; users need per-dimension / per-batch progress, a running
partial score, and the ability to walk away or cancel.

### Where It Lives
Eval Harness → after **[Run Eval]**, an in-place **progress panel** replaces the
canned modal. Uses the live-timer registry (auto-cancels on navigation).

### UI

```
┌──────────────────────────────────────────────────────────┐
│  Running eval — v1.0 vs v1.1        [ Cancel ]           │
│  ████████████░░░░░░░░  118 / 200 rows scored (59%)       │
│  Elapsed 0:24 · ETA 0:17 · ~4.9 rows/s                   │
│                                                          │
│  Per-dimension (partial, updates live):                  │
│  Factuality          ▓▓▓▓▓▓▓░░  72.9   (118 rows)        │
│  Instruction Follow  ▓▓▓▓▓▓▓▓░  80.4                     │
│  Tone                ▓▓▓▓▓▓▓▓▓  88.1                     │
│  Task Completion     ▓▓▓▓▓▓░░░  70.2                     │
│  Refusal Behaviour   ▓▓▓▓▓▓▓▓▓  93.8                     │
│                                                          │
│  12 rows routed to human review (judge confidence <70%)  │
└──────────────────────────────────────────────────────────┘
```

- Overall progress bar + rows scored / total, elapsed, ETA, throughput.
- **Partial per-dimension scores** update as batches complete (a running mean).
- Low-confidence count accumulates live.
- **[Cancel]** stops the run; already-scored rows can be kept as a partial result
  ("Scored 118 of 200 — keep partial / discard").
- Completing transitions straight into the normal results view (radar + table).

### Mock Logic
Batch the mock scoring across ticks instead of one setTimeout:

```javascript
function startEvalRun(run, totalRows) {
  let scored = 0, acc = {};                 // running sums per dimension
  const batch = Math.max(1, Math.round(totalRows / 40));  // ~40 ticks
  const timer = registerTimer(setInterval(() => {
    for (let i = 0; i < batch && scored < totalRows; i++, scored++) {
      run.rubric.forEach(d => acc[d.dimension] = (acc[d.dimension]||0) + sampleScore());
    }
    renderEvalProgress(run, scored, totalRows, mean(acc, scored));
    if (scored >= totalRows) { clearScreenTimers(); finaliseEval(run, acc, totalRows); }
  }, 200));
}
```

### Implementation Notes
131. Progress driven by a registry timer (~200 ms tick) so it auto-cancels on
     navigation; never a runaway interval.
132. Partial per-dimension score = running mean over rows scored so far; must be
     mathematically consistent (never exceed the final value's plausible range).
133. ETA = `(total − scored) / throughput`, throughput smoothed over recent ticks.
134. `[Cancel]` offers keep-partial vs discard. Keeping stores a result flagged
     `partial: true` and labels it as such in eval history.
135. On completion, reuse the existing radar + score-table result view unchanged;
     the progress panel is purely the running state.

---

## 42. EXPERIMENT SEARCH & FILTER BY PARAMETER

### Problem
The experiment timeline is chronological only. With dozens of runs, finding "all
LoRA rank 64 runs on Llama-3.1-8B with eval > 80" is impossible without scrolling
and eyeballing.

### Where It Lives
Experiment Tracker → a **filter bar** above the timeline.

### UI

```
┌──────────────────────────────────────────────────────────────────────┐
│  🔍 [ search name / notes… ]   Base model [Any ▾]  LoRA rank [Any ▾]  │
│  Status [Any ▾]   Eval score [ ≥ 80 ]   Sort [ Newest ▾ ]  [Clear]    │
│  Showing 4 of 17 experiments                                          │
└──────────────────────────────────────────────────────────────────────┘
  … filtered timeline nodes …
```

Filterable fields (from `trainingConfig` + eval link):
- **Text search** across name + notes.
- **Base model** (dropdown of distinct values).
- **LoRA rank**, **epochs**, **batch size**, **optimizer** (distinct-value
  dropdowns).
- **Status** (completed / running).
- **Eval score** threshold (≥ N).
- **Sort:** newest, oldest, highest eval, lowest cost.

Active filters render as removable chips. `[Clear]` resets. The count line
("Showing 4 of 17") is always visible. Empty result → an empty state with a
"clear filters" CTA.

### Mock Logic
```javascript
function filterExperiments(list, f) {
  return list.filter(x => {
    if (f.q && !(x.name + ' ' + x.notes).toLowerCase().includes(f.q.toLowerCase())) return false;
    if (f.baseModel && x.trainingConfig.baseModel !== f.baseModel) return false;
    if (f.loraRank && x.trainingConfig.loraRank !== +f.loraRank) return false;
    if (f.status && x.status !== f.status) return false;
    if (f.minEval != null) { const ev = byId(evalRuns, x.evalRunId);
      if (!(ev && ev.results && ev.results.B.Overall >= f.minEval)) return false; }
    return true;
  }).sort(sorters[f.sort]);
}
// dropdown options = [...new Set(experiments.map(x => x.trainingConfig.baseModel))]
```

### Implementation Notes
136. Dropdown options derive from distinct values present in the tenant's
     experiments (never a hardcoded list) so they stay meaningful.
137. Text search matches name + notes, case-insensitive, debounced live.
138. Filters compose (AND). Each active filter shows as a removable chip; the
     result count updates live.
139. Also expose the filter through the command palette ("Filter experiments by
     …") so power users skip the bar.
140. Empty state after filtering: icon + "No experiments match" + [Clear filters].

---

## 43. KEYBOARD SHORTCUT SCOPE GUARD

### Problem
Single-key shortcuts (`A`/`R`/`E`/`S`, `A`/`B`/`T`/`X`, `[`/`]`) currently risk
firing while the user is typing in a textarea/input, or on the wrong screen, or
inside a modal — e.g., typing "add" in a reasoning field triggers Approve. This
is a correctness bug waiting to happen as more shortcuts are added.

### Where It Lives
Cross-cutting: the global `keydown` handler. Introduces a single **scope guard**
that all shortcut handlers pass through.

### Design

```javascript
function shortcutContext(e) {
  const t = e.target;
  const typing = t && (/^(INPUT|TEXTAREA|SELECT)$/.test(t.tagName) || t.isContentEditable);
  const modalOpen = !!el('overlays').innerHTML;   // palette, modals, integration guide
  const panelOpen = el('panel')?.classList.contains('is-open');
  return {
    typing, modalOpen,
    // which shortcut set, if any, is currently allowed
    scope: modalOpen ? 'modal'
         : currentView !== 'review' ? 'none'
         : appState.reviewTab,          // datasets | ocr | audio | video | pref
  };
}
```

Rules:
- **Global** shortcuts (`Cmd/Ctrl+K`, `Esc`) always work.
- **Task shortcuts** fire only when `scope` matches the active task AND `typing`
  is false — *except* navigation keys (`←`/`→`, `[`/`]`) which are allowed even
  while typing (they don't conflict with text entry) but NOT while a modal is
  open.
- When a **modal/palette/overlay** is open, task shortcuts are fully suppressed;
  only that surface's own keys (palette arrows/enter, `Esc`) apply.
- A shortcut that has no meaning on the current tab is a no-op (never falls
  through to another tab's handler).

### UX
- Shortcut hints (`[A]`, `[R]`) are only shown on the surface where they're
  currently live, so users aren't told about keys that won't fire.
- Optional: a subtle "typing — shortcuts paused" affordance when focus is in a
  correction field, so the annotator understands why `A` didn't approve.

### Implementation Notes
141. All existing task-shortcut branches route through `shortcutContext()` — no
     handler reads `e.target.tagName` ad hoc anymore. One guard, one behaviour.
142. Navigation keys (`←`/`→`/`[`/`]`) are the only shortcuts permitted while
     `typing` is true, and only when no modal is open.
143. When `scope === 'modal'`, suppress every task shortcut; delegate solely to
     the open surface (palette/modal) handler.
144. Shortcut hint chips render conditionally on the active scope so on-screen
     hints never advertise an inactive key.
145. Adding a new shortcut requires declaring its scope in one place; unknown or
     mismatched scope = the key does nothing (fail safe, not fail open).
