# CLAUDE.md — Cueval Engineering Charter

> **This file is law.** Every code change and new feature MUST comply with the rules
> here. When a request conflicts with these rules, surface the conflict before
> proceeding. Product requirements live in [`docs/PRODUCT_SPEC.md`](docs/PRODUCT_SPEC.md);
> *how* we build them lives here.

---

## 0. TL;DR — The Non-Negotiables

1. **One file.** The entire app ships as a single self-contained `index.html`. No build step, no bundler, no npm, no external runtime deps (Google Fonts CDN is the only exception).
2. **No frameworks.** Vanilla JS + CSS only. No React/Vue/jQuery/lodash/chart libs.
3. **Tenant isolation is a security boundary, not a filter.** Every data read goes through an accessor that scopes to `currentTenant.id`. Never touch a global data array directly from UI code.
4. **Design tokens only.** Never hardcode a hex color, font name, or spacing value that already exists as a CSS variable.
5. **Every interactive element is keyboard-reachable and labelled.** Icon-only buttons need `aria-label`. Respect `prefers-reduced-motion`.
6. **Log through `log.*`, never raw `console.*`.** Fail loud in dev, degrade gracefully in the UI.
7. **No dead ends.** Every screen works with mock data. No "coming soon", no unhandled empty state.
8. **Verify before you claim done.** Load the file in a browser, log in as the affected role/tenant, exercise the actual flow. See §12.

---

## 1. Architecture Decisions (ADRs)

These are settled. Do not relitigate without updating this section and stating why.

| # | Decision | Rationale | Consequence |
|---|----------|-----------|-------------|
| ADR-1 | **Single `index.html`, zero build** | Prototype must run by double-clicking. Portable, demoable offline, no toolchain rot. | All CSS in one `<style>`, all JS in one `<script>`. Discipline via strict internal sectioning (§2). |
| ADR-2 | **Vanilla JS, no framework** | Spec mandates it; keeps the file inspectable and dependency-free. | We hand-roll rendering, routing, and state. Use the patterns in §3–4 rather than inventing new ones per feature. |
| ADR-3 | **In-memory state, no persistence** | Prototype; no backend. `localStorage` explicitly excluded by spec (§3 Session). | State resets on reload. Never add `localStorage`/`sessionStorage`/IndexedDB without an ADR. |
| ADR-4 | **Database-per-tenant, simulated** | Multi-tenancy is a core selling point; isolation must be structural, not cosmetic. | ALL domain data carries `tenantId`; ALL access goes through scoped accessors (§5). This is the highest-risk area — treat bugs here as security bugs. |
| ADR-5 | **Mock logic is deterministic-ish and centralised** | Scoring/eval/PII must behave consistently across a demo. | All mock algorithms live in one `mock` module (§10 of product spec). UI never re-implements scoring inline. |
| ADR-6 | **Canvas for charts, SVG for icons** | No chart lib allowed; icons must be inline. | Radar/histograms drawn via Canvas 2D. Icons are inline `<svg>` (Heroicons style), no icon fonts. |
| ADR-7 | **Right panel overlays, never navigates** | Spec: detail views must not lose context. | Detail = absolutely-positioned panel with `translateX`. Never route away to show a row/experiment detail. |

If you need to break an ADR, add a new numbered ADR row explaining the trade-off, and call it out in your change summary.

---

## 2. File & Code Organization (inside the single file)

Because everything lives in one file, **internal structure is the only thing keeping it maintainable.** Enforce this layout top-to-bottom:

```
<!doctype html>
<head>
  <meta> tags, <title>
  <link> Google Fonts (Syne, Inter, JetBrains Mono)
  <style>
    /* 1. :root design tokens (colors, type, spacing, motion)      */
    /* 2. reset + base element styles                              */
    /* 3. layout primitives (shell, dock, topbar, canvas, panel)   */
    /* 4. components (cards, tables, pills, buttons, meters, modal) */
    /* 5. screen-specific styles, each under a .screen-<name> scope */
    /* 6. utilities + @media + prefers-reduced-motion overrides     */
  </style>
</head>
<body>
  <div id="app"></div>
  <script>
    /* ===== SECTION A: CONFIG & CONSTANTS ===== */   // tokens mirrored for JS, enums, role/permission matrix
    /* ===== SECTION B: MOCK DATA ===== */            // tenants, users, projects, releases, datasets, rows, evals, experiments
    /* ===== SECTION C: STATE ===== */                // currentUser, currentTenant, currentView, appState, impersonation
    /* ===== SECTION D: DATA ACCESSORS ===== */       // getProjects(), getDatasets()... ALL tenant-scoped (§5)
    /* ===== SECTION E: MOCK LOGIC ===== */           // scoreRow, detectLanguage, hasPII, runEval, etc.
    /* ===== SECTION F: LOGGING & ERRORS ===== */      // log.*, errorBoundary, toast
    /* ===== SECTION G: DOM/RENDER HELPERS ===== */    // el(), clear(), mount(), escapeHtml(), formatNum()
    /* ===== SECTION H: COMPONENTS ===== */            // reusable render fns: renderDock, renderTopbar, renderCard...
    /* ===== SECTION I: SCREENS ===== */               // one render function per screen: renderDashboard()...
    /* ===== SECTION J: ROUTING & SHELL ===== */       // route(view), renderShell(), tenant vs superadmin shell
    /* ===== SECTION K: EVENTS & INIT ===== */         // global listeners (Cmd+K), auth, boot()
  </script>
</body>
```

**Rules:**
- A new feature adds code to the **correct section**, never wherever the cursor happens to be.
- Every function has a one-line `//` comment stating its job if the name isn't self-evident.
- Keep functions small and single-purpose. If a render function exceeds ~80 lines, extract a component into Section H.
- Naming: `camelCase` for JS, `kebab-case` for CSS classes and DOM ids, `SCREAMING_SNAKE` for true constants/enums.
- CSS classes are namespaced by concern: `dock-*`, `topbar-*`, `panel-*`, `screen-<name>-*`. No bare generic class names like `.active` without a namespace — use `.dock-item.is-active`.

---

## 3. State Management

- Single source of truth: a top-level `appState` object plus `currentUser` / `currentTenant` / `currentView`. No hidden state stashed on DOM nodes beyond `data-*` used for event delegation.
- **Mutations go through small updater functions**, not scattered inline assignments, so we have one place to add logging/validation later. Example: `setView(name)`, `openPanel(kind, id)`, `enterImpersonation(tenantId)`.
- **Render is a function of state.** After any state change that affects the UI, call the relevant `render*()`. Prefer re-rendering the smallest affected region (a screen, the dock badge) over re-rendering the whole shell.
- No two-way data binding magic. Read state → build DOM → attach listeners that mutate state → re-render.
- Event handling uses **delegation** where lists are involved (one listener on the table, read `data-row-id`), not a listener per row.

---

## 4. Rendering & DOM

- Build DOM with a tiny helper (`el(tag, props, children)`) or template strings — pick one per section and stay consistent. If using template strings, **all interpolated user/mock text MUST pass through `escapeHtml()`** (see §6 Security).
- Never inject unescaped data into `innerHTML`. Instruction/output/name fields are "untrusted" mock content and may contain `<`, `&`, quotes.
- Idempotent renders: a `render*()` call clears its container first (`clear(node)`) then rebuilds. No appending duplicates.
- Animations via CSS classes/keyframes only (ADR-2). Score bars animate from 0 on mount; dock glow pulses via keyframes. All animation blocks are wrapped by the reduced-motion guard (§9).

---

## 5. Multi-Tenancy & Data Isolation (HIGHEST PRIORITY)

This is the feature most likely to leak data and the one reviewers scrutinise hardest. Treat every isolation bug as a **security defect**, not a cosmetic one.

**Rules:**
1. Every domain record (`project`, `release`, `dataset`, `row`-owning dataset, `evalRun`, `experiment`, `user`) carries a `tenantId`.
2. UI and screen code **must never** read `projects`, `datasets`, etc. directly. They call accessors:
   ```js
   getProjects()      // filters by currentTenant.id
   getDatasets()      // filters by currentTenant.id
   getReleases(), getEvalRuns(), getExperiments(), getUsers()
   ```
3. Accessors enforce the boundary in ONE place:
   ```js
   function scoped(collection) {
     if (currentUser.role === 'SuperAdmin' && !impersonating) return collection; // platform view only
     return collection.filter(r => r.tenantId === currentTenant.id);
   }
   ```
4. `currentTenant` is resolved **at login** and is immutable for the session except via explicit SuperAdmin **impersonation** (which sets a banner and is read-only).
5. SuperAdmin sees cross-tenant aggregates only through dedicated platform accessors — never by "turning off" a filter inside a tenant screen.
6. Writes (approve row, save experiment) must assert the target record's `tenantId === currentTenant.id` before mutating. Add a `assertTenant(record)` guard.
7. **When adding any new data type or query, wire it through an accessor in Section D. A raw `.filter`/`.find` over a global collection inside a screen function is a review-blocking violation.**

---

## 6. Security (prototype-appropriate, but real)

Even a browser-only prototype has real attack surface — it renders mock content into the DOM and reads uploaded files.

- **XSS:** All dynamic text rendered via `escapeHtml()` before hitting `innerHTML`, OR use `textContent`/`el()` which escape by construction. Mock rows contain fake PII and toxic strings by design — they must never execute.
- **File upload:** Parse uploaded files with `FileReader`. Validate extension against an allowlist (`.jsonl/.csv/.tsv/.parquet/.txt`). Guard against malformed content — wrap parsing in try/catch, cap row count, never `eval()` file content.
- **No secrets, ever.** No API keys, tokens, or real credentials in the file. Demo passwords are intentionally `demo123` and clearly labelled demo-only.
- **Tenant boundary** (§5) is part of the security model, not just UX.
- **Downloads:** Export via `Blob` + `URL.createObjectURL`; revoke the object URL after use.

---

## 7. Error Handling & Logging

There is no backend and no error tracker, so **the discipline is: fail loud in dev, degrade gracefully for the user, and leave a breadcrumb trail.**

**Logging facade** (Section F) — never call `console.*` directly:
```js
const log = {
  debug: (...a) => LOG_LEVEL <= 0 && console.debug('%c[cueval:debug]', 'color:#5A5A72', ...a),
  info:  (...a) => LOG_LEVEL <= 1 && console.info('%c[cueval:info]',  'color:#3B82F6', ...a),
  warn:  (...a) => console.warn('%c[cueval:warn]',  'color:#EAB308', ...a),
  error: (...a) => console.error('%c[cueval:error]', 'color:#EF4444', ...a),
  event: (name, payload) => log.info(`event:${name}`, payload), // user/domain events for demo traceability
};
```
- `LOG_LEVEL` is a top-level const (default `info`; set to `debug` while developing).
- Prefix every log with the subsystem: `log.info('[eval] run started', {releaseId})`.

**Error boundaries:**
- Wrap each top-level `render*()` and every async/timeout mock flow (upload curation, eval run) in try/catch.
- On a caught error: `log.error(...)` with context, then show a non-blocking **toast** ("Something went wrong rendering Datasets — check console") instead of a blank screen. Never let one broken screen white-screen the whole app.
- Install a global safety net:
  ```js
  window.addEventListener('error', e => { log.error('[uncaught]', e.error || e.message); showToast('Unexpected error', 'error'); });
  window.addEventListener('unhandledrejection', e => { log.error('[promise]', e.reason); showToast('Unexpected error', 'error'); });
  ```
- **Log domain events** (`log.event`) at every meaningful action — row approved, eval completed, release status change, tenant impersonation entered/exited, upload finished. This doubles as the audit trail the product spec's activity feed and notifications read from, and makes demos debuggable.
- User-facing errors are calm and specific; never dump a stack trace into the UI.

**Validation:** Validate inputs at the boundary (form submit, file parse, rubric weights must sum to 100). Show inline field errors with the `--red` token + an icon + text (never color alone, §9).

---

## 8. Design System Compliance

The look is the product. Adherence is mandatory, not aesthetic preference.

- **Tokens are the only source of color/type/spacing.** Define every palette value from product spec §2 as a `:root` CSS variable. JS that needs a color reads it from a mirrored `TOKENS` const — no inline hex in JS either.
- **Typography roles are fixed:** `Syne` = product name + big numbers only; `Inter` = all UI text; `JetBrains Mono` = code/JSON/rows/scores. Don't use display font for body copy.
- **Signature elements must be preserved and never downgraded to a plain sidebar:** the orbital dock (frosted glass, glow ring on active, breathing pulse for pending), the slide-in right panel, the command palette.
- **Status is always color + icon/text**, never color alone (accessibility + spec §12).
- **Motion budget:** page transition 200ms, panel slide 250ms ease-out, dock hover 150ms scale, score bars 600ms, dock pulse 2s loop. Match these; don't invent new durations.
- New components reuse existing primitives (pill, card, meter, button variants). Don't fork a new button style when a variant exists.

---

## 9. Accessibility (enforced per change)

- Desktop-first, min supported width 1024px; optimise for 1280px+.
- Every interactive element is `Tab`-focusable with a visible **coral 2px focus ring**.
- Icon-only buttons/links require `aria-label`. Dock items announce their destination.
- Color is never the sole signal — pair with icon or text (status pills, usage meters, deltas).
- Respect motion preference globally:
  ```css
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after { animation: none !important; transition: none !important; }
  }
  ```
- Modals/command palette trap focus and close on `Esc`. Restore focus to the trigger on close.

---

## 10. Performance

- Datasets have hundreds to thousands of rows. **Don't render 1,000 DOM rows at once** — paginate or windowize the row table; render the visible slice.
- Use event delegation for large lists (§3).
- Canvas charts: draw once per data change, not per frame; only animate the intro draw.
- Avoid layout thrash: batch DOM writes, read layout values before writing.
- No polling loops. Mock async flows use a single `setTimeout` chain, cleaned up if the user navigates away.

---

## 11. Conventions & Definition of Done

**Every change must, before you call it complete:**
1. Live in the correct file section (§2) and follow naming rules.
2. Route all data access through tenant-scoped accessors (§5).
3. Escape all dynamic text (§6).
4. Log meaningful events and wrap risky code in try/catch with a toast fallback (§7).
5. Use design tokens only; preserve signature UI (§8).
6. Be keyboard-reachable, labelled, and reduced-motion safe (§9).
7. Handle the empty state (spec §13) — icon + one-line + CTA. No blank regions.
8. Handle the affected roles/plans. If it touches dashboards, permissions, or limits, verify the behaviour differs correctly per role/plan (e.g. HealthBot at 82% shows warnings; SuperAdmin sees a different shell).
9. Leave no `TODO`/placeholder/"coming soon" in shipped code (spec §14.13).
10. Be verified in a real browser (§12).

---

## 12. Verification (do this, don't assume)

There are no automated tests in this prototype, so **manual verification is the test suite.** For any non-trivial change:

1. Open `index.html` in a browser (or run a static server: `python3 -m http.server` in the repo root, then load `http://localhost:8000`).
2. Open DevTools console — it must be **free of errors and warnings** for the flow you touched.
3. Log in as **each affected role**, and where multi-tenancy is involved, as **each affected tenant** (Pravartak/Legal AI/HealthBot) and **SuperAdmin**. Confirm:
   - Data shown belongs only to the current tenant (no leakage).
   - Role-gated UI appears/hides correctly (Settings = Admin; approvals = Architect/PM/Admin).
   - Plan-limit states render correctly (HealthBot yellow at 82%).
4. Exercise the actual interaction (upload, review, run eval, approve release), not just the initial render.
5. Tab through new interactive elements; toggle OS "reduce motion" and reconfirm.
6. State in your change summary exactly what you verified and what you observed — never "should work."

---

## 13. Git & Change Hygiene

- Repo maps to `https://github.com/Prasanna-Lynkstr/cueval` (`origin/main`).
- `.claude/settings.local.json` is machine-local and gitignored — never commit it.
- Commit in logical units with a clear message describing the *why*. Do not commit or push unless the user asks.
- If not already on a feature branch and the change is non-trivial, branch first.
- Keep `index.html` diffs reviewable — because it's one big file, scope each commit to one concern so diffs stay legible.
- Update `docs/PRODUCT_SPEC.md` when product behaviour changes; update **this file** when an architecture/standard decision changes (add/adjust an ADR).

---

## 14. When Requirements Are Ambiguous

- If the product spec and a request conflict, or a detail is unspecified, pick the option most consistent with the ADRs and design system, state the assumption in your summary, and proceed. Don't stall on trivia.
- If a request would violate a non-negotiable (§0) — e.g. "add React", "store in localStorage", "read data without tenant scoping" — flag it and propose the compliant alternative before implementing.
