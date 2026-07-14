Verify a change end-to-end against the CLAUDE.md §12 discipline. There are no
automated tests in this prototype — manual verification IS the test suite. Do not
stop at "implementation complete"; prove it works.

If the user named a screen/flow/role in the arguments, focus there. Otherwise
verify whatever the current diff touched (`git diff --stat`).

## Steps

1. **Serve it.** Start a static server in the repo root and load index.html:
   `python3 -m http.server 8000` then open http://localhost:8000
   (Kill the server when done: `pkill -f "http.server 8000"`.)

2. **Console must be clean.** Open DevTools console. It must be free of errors
   AND warnings for the flow you touched. Report anything that appears.

3. **Every affected role.** Log in as each role the change touches and confirm
   role-gated UI appears/hides correctly (Settings = Admin; approvals =
   Architect/PM/Admin; SuperAdmin gets a different shell).

4. **Every affected tenant + isolation.** Where multi-tenancy is involved, log in
   as Pravartak, Legal AI, and HealthBot, plus SuperAdmin. Confirm data shown
   belongs ONLY to the current tenant — no leakage. Confirm plan-limit states
   render (HealthBot yellow at 82%). Treat any leak as a security defect (§5).

5. **Exercise the real interaction**, not just the initial render — upload,
   review, run eval, approve release, whatever the change enables.

6. **Keyboard + motion.** Tab through new interactive elements (visible coral
   focus ring, icon-only buttons have aria-label). Toggle OS "reduce motion" and
   reconfirm nothing breaks.

7. **Empty state.** Confirm the empty case renders icon + one-line + CTA — no
   blank region, no "coming soon".

## Report

State exactly what you did, what role/tenant you were in, and what you OBSERVED
— never "should work". If anything failed, quote the console output. End with a
one-line pass/fail per checklist item above.
