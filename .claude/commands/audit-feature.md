Audit a feature area of index.html against the CLAUDE.md engineering charter
BEFORE writing any code for it. Surface surprises and confirm a plan first — do
not start editing.

The feature/area to audit is in the arguments (e.g. "datasets review panel",
"monitor config"). If empty, audit whatever the current diff touches.

## AUDIT FIRST — report findings, then confirm the plan

1. **Locate it.** Find the relevant section and screen/component functions:
   `grep -nE "SECTION [A-K]|function screen|function render" index.html | head -60`
   then grep the feature's own keywords to find its render fn, state, and events.
   Report the file-section (§2 A–K) each touched piece lives in.

2. **Tenant isolation (§5 — highest priority).** Confirm every data read in the
   area goes through a scoped accessor (getProjects/getDatasets/getReleases/
   getEvalRuns/getExperiments/getUsers), NOT a raw `.filter`/`.find` over a
   global collection. Confirm writes call `assertTenant(record)` before mutating.
   Flag any raw global access — that is a review-blocking violation.

3. **Escaping (§6).** Confirm all dynamic/mock text hits the DOM via
   `escapeHtml()`, `textContent`, or `el()` — never unescaped `innerHTML`.

4. **Design tokens (§8).** Confirm colors/fonts/spacing come from `:root`
   variables (or the mirrored `TOKENS` const in JS), no inline hex/font/px that
   duplicates an existing token. Confirm signature UI (orbital dock, slide-in
   panel, command palette) is preserved, not downgraded.

5. **Roles & plans (§11.8).** Confirm the area behaves correctly per role and
   per plan limit, and that the empty state (§13) is handled.

6. **Logging & errors (§7).** Confirm meaningful actions emit `log.event(...)`
   and risky/async code is wrapped in try/catch with a toast fallback.

## Then

Report what you found (grouped by the checks above), name any violations or
surprises, and propose a concrete plan. Wait for confirmation before writing code.
