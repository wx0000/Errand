# Working with AI — judgment log

Running log of the human-judgment moments in the AI sessions: where assumptions broke, where the
model caught itself, where a guardrail prevented over-interpretation. **Stubs — consolidate at
project closeout.**

## P1 recon — judgment moments

- **Selectors vs Flutter's real a11y.** Initial text selectors assumed substring matching; they
  broke on Flutter's `"label\nvalue"` accessibilityText. Resolved with a throwaway diagnostic flow
  (test the matching semantics) rather than guessing — anchored-DOTALL regex confirmed.
- **Caught an over-claim on myself.** Declared the STAGE mechanism "confirmed" after a smoke test
  whose `-e` equalled the flow's `env:` default — so it never tested override. The real run exposed
  that `env: STAGE` beats `-e`; named the over-claim and fixed it (drop the `env` default).
- **Park-then-dump sequencing error.** Ran a second act between parking on a screen and dumping its
  hierarchy, which relaunched and clobbered the parked state. Caught and named; rule: dump
  immediately after the parking run, nothing in between.
- **R4 — refused to fabricate a defect.** The anti-case format primed an error-position bug; recon's
  objective bounds (error top-y < input top-y) + screenshot showed conformance. Recorded "no
  candidate" — the cardinal rule cuts both ways (don't bend assertions to a buggy app; don't invent
  a defect where the app conforms).
- **Persistence — human guardrail stopped over-interpretation.** Spec is silent on surviving an app
  restart, so the human ruled it out as a candidate up front. Persistence was run as exploration and
  recorded as actual (session + history survive), with **no flow asserting persistence** (asserting
  it would be an anti-case).
- **Plan archival — refused a destructive overwrite.** The lead's closeout instruction ("overwrite
  `p0-scaffold.md`, it's the same plan") was wrong. Corroborating before overwriting found **two
  distinct plans** (the combined Session-0 plan vs the P1-recon plan) — overwriting would have
  deleted the P0 plan content. Surfaced it instead of proceeding; archived the P1 plan as its own
  `p1-recon.md` (unconditional, per-`<phase>-<topic>` rule). Lead confirmed the correction.
