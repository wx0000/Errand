# SPEC MATRIX — Buggy Maestro E2E suite

> **Binding ground-truth of the delivery.** Single source linking each requirement to its flow,
> the spec's expectation, what recon actually observed, and (later) the human's verdict.
> This file replaces any notion of a "golden dataset."
>
> **Rules (from `CLAUDE.md` → Cardinal rule + Self-Validation):**
> - `expected per spec` comes from the **R1–R5** text pasted by the human — **never invented**.
> - `actual` comes from **real recon evidence**: an exact string quoted from a hierarchy dump
>   (`docs/recon/hierarchy/…`) and/or a screenshot filename (`docs/recon/screens/…`).
>   No artifact = no claim.
> - **The `verdict` column stays EMPTY until the human's review — Claude never fills it.**

## Main table — requirements R1–R5

| Req | Flow | Expected per spec | Actual (from recon) | Verdict (PASS/BUG) |
|-----|------|-------------------|---------------------|--------------------|
| R1  | _TBD — P2_ | _TBD — awaiting R1–R5_ | _TBD — awaiting recon_ |  |
| R2  | _TBD — P2_ | _TBD — awaiting R1–R5_ | _TBD — awaiting recon_ |  |
| R3  | _TBD — P2_ | _TBD — awaiting R1–R5_ | _TBD — awaiting recon_ |  |
| R4  | _TBD — P2_ | _TBD — awaiting R1–R5_ | _TBD — awaiting recon_ |  |
| R5  | _TBD — P2_ | _TBD — awaiting R1–R5_ | _TBD — awaiting recon_ |  |

## Bug candidates (recon output — NO verdicts)

> Filled during recon. Each candidate = `R#` + expected-per-spec + actual + **artifact**
> (hierarchy string or screenshot filename), tagged **"candidate — human adjudicates on
> screenshot."** Claude does not rule BUG/PASS.

_TBD — awaiting recon._

## Anti-cases — what a test must NOT assert

These guard against **over-interpreting the app**: asserting something the app happens to show
but the spec does not require (or forbids). Asserting an anti-case bakes a bug into a "green"
test.

> **The rows below are FORMAT PLACEHOLDERS taken from the prompt — NOT real Buggy findings.**
> They show the shape/columns only. **Real anti-cases are filled only after R1–R5 + recon, and
> are the human's judgment.** Do not treat these as ready app cases.

| Anti-case (do NOT assert) | Why it would be over-interpretation | Related Req |
|---------------------------|-------------------------------------|-------------|
| _format example:_ snackbar shows only the title instead of title + location | Spec requires title **and** location; asserting title-only locks in the defect | _TBD_ |
| _format example:_ a placeholder text the spec forbids | Spec forbids the placeholder; asserting it present contradicts spec | _TBD_ |
| _format example:_ validation error rendered **below** the input instead of above | Spec dictates the error position; asserting the wrong position rubber-stamps it | _TBD_ |
