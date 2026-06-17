# Working with AI on this project

This test suite was built with AI across two surfaces that did different jobs. The point of this
note is the part a reader cannot reconstruct from the raw build transcripts: the decision layer.

**Claude Code** — one terminal, plan mode toggled on for design and off for execution — wrote the
flows, the subflows, and the docs, and ran the suite on the emulator. Its raw record (the per-session
`/export` transcripts and the archived plans) lives alongside this file in `docs/ai-history/`.

**Claude Desktop** was the strategy and direction layer: scoping, sequencing the phases, adversarial
review of every plan before it was built, and the decisions Code executed against. This document is
about that layer.

The working assumption throughout was that the agent is a fast but fallible senior engineer, not an
oracle. The leverage came from the structure built around it and the discipline of verification — not
from generation. Every point below is anchored to a real moment in this project; nothing is a general
statement about "using AI."

## How the work was run

The work was closed phase by phase: recon to establish ground truth, then a `SPEC-MATRIX` of the
requirements; the flows built in scoped passes — registration and login (R4/R5), apply and
application history (R2/R3), the offers list (R1); then stabilization and the delivery docs. Recon
was run as proof-of-method, not a formality — most of the early judgment moments below come from it,
because that is where the assumptions about the app were first tested against the app.

`CLAUDE.md` and `PROGRESS.md` carried the binding rules into every session: the `SPEC-MATRIX` (the
golden set of requirements plus the anti-cases), a Self-Validation Protocol, and an explicit
anti-sycophancy clause. Keeping the rules in files rather than in a person's head is what let them
survive a context reset between sessions.

Single source of truth was enforced: only the build terminal wrote to the repo. Desktop and plan mode
advised and drafted; they did not commit. Scopes were separated with `/clear` so one phase's context
did not bleed into the next.

One pattern is worth naming, because it carried a lot of the reliability. The checkpoints where a
human had to render a verdict were built into the plan as hard STOP gates, not left to the operator to
remember. The division was deliberate: if a step produced a judgment only a human could make — is this
red a finding, is the suite stable, did the clean clone actually come up — the agent reported the real
output verbatim and stopped. If a step could be checked without judgment — `git status`, no commit
trailer, the diff touches only the expected files, every linked artifact path resolves — the agent ran
that as a self-audit and reported the result for the human to read. Vigilance lived in the structure,
not in the operator's attention.

## 1. The agent is fallible — anti-sycophancy was engineered, not hoped for

Every generation was probed rather than accepted: which requirement does this flow assert; prove it by
the run, not by description; what is the weakest part of this; are you agreeing because I am right or
because I pushed? The aim was to make a change of mind cost a reason, so a justified correction could
be told apart from a reflexive flip. It ran in both directions, and it included the human being wrong.

The agent argued the lead down, with a reason. The boundary of the `apply-to-offer` subflow was
sketched loosely at first — "drive the apply, handle the picker if the offer is multi-location." In
plan mode, Code broke that into options and pushed back: a conditional "handle the picker if multi"
forces a branch inside the subflow, and Maestro handles conditional step-skipping badly — the same
fragility class that had already produced a false red earlier. The boundary was tightened to a
tap-only subflow (one required parameter, no default, no outcome assertion), with the picker choice and
the confirmation-snackbar assertion left in the calling flow, which is the only place that knows
whether the offer is single- or multi-location. The change was taken because the argument held, not
because it was asserted.

The agent declined a human instruction because checking showed it was wrong. At the close of an
earlier phase the lead's instruction was to overwrite an existing plan file — "it's the same plan."
Corroborating before overwriting found two distinct plans (an earlier combined plan and the recon
plan); obeying would have deleted the first. The agent surfaced the discrepancy instead of proceeding
and archived the recon plan under its own name; the lead confirmed the correction. Not complying was
the correct move.

The agent refused to rubber-stamp a senior's guess. When three registration/login flows went red, the
lead's adversarial hypothesis was a selector collision between the password and confirm fields. Code
rejected it with evidence from the run rather than nodding: the evaluated command log showed the input
never resolved at all, so the taps never reached any field and the selectors were never in play. The
evidence beat the guess (the real cause is in §2).

And the lead was argued down on the merits. Late in the project the lead wanted to add Windows run
instructions to the README and run the clean-machine check on a Windows laptop. Both were pushed back:
the project's rule is that the README's primary path is the one actually walked, and the reviewer
environment is macOS, so unverified Windows steps would be theory in a document meant to be a walked
path; and there is a hard technical blocker — the app is `arm64-v8a` only, so an ARM64 Android emulator
on an x86 Windows host runs under full software emulation and is effectively unusable. The Windows path
was dropped and the verified macOS path stayed.

## 2. Corroborate over declaration

Rules and checklists are snapshots, not live state, so "done" was always checked against the real
thing — `git status`, an actual suite run on the emulator — never a ticked box. Even the selector
semantics were established by a run, not assumed: a throwaway diagnostic flow in recon confirmed that
Flutter exposes text as a `label\nvalue` node and that selectors must be full-text-anchored regex,
rather than guessing that a bare substring would match.

The discipline shows best in one Maestro hazard met three times. First, in recon, the agent declared a
STAGE-parameterization mechanism "confirmed" after a smoke test whose `-e` value happened to equal the
flow's `env:` default — so the test never actually exercised the override. A real run exposed that the
`env:` default beats `-e`; the agent named its own over-claim and fixed it by dropping the default, and
the lesson — what the run did not actually exercise is not "confirmed" — went into `CLAUDE.md`. The
same precedence bug resurfaced in the build phase, now inside the subflows: the first full suite run
was 3/3 red. The cardinal rule primes "red = a finding," but those flows had been adjudicated PASS in
recon, so a finding was implausible. The failure screenshot showed the form submitted with empty
fields; the empty `env:` defaults were overriding the values passed from `runFlow`, so the harness was
feeding the app emptiness and the app was correctly rejecting it. Removing the defaults turned it
green — no assertion bent, and a harness bug not mislabeled an app defect. And the block that caused it
was in the plan the agent had written and the lead had approved; per the plan's own "verify before
implement" clause the agent reconciled against reality rather than forcing the approved plan through.
An approved plan is still a snapshot.

The same discipline governed the rest. The three stability runs and the fresh-clone run were reported
verbatim and the run stopped for the human to rule stable / clean-machine; nothing was self-ruled.
Conversely, the mechanical checks — no commit trailer, the diff touches only the expected files, every
linked artifact resolves — were run as a self-audit and reported for the human to read. A small but
representative one: a pasted copy of `BUGS.md` once looked like it was missing the suite-run evidence
links under one defect; rather than act on the paste, the file on disk was read — the links were there,
the paste was stale. The artifact of record is the file, not a snapshot of it in chat. And a recon
sequencing error was caught and named: running a second action between parking on a screen and dumping
its hierarchy relaunched and clobbered the parked state, so the rule became dump immediately after the
parking run, nothing in between.

The biases this guards against were named explicitly so they could be watched for: anchoring on the
first hypothesis, treating a checked box as done, the planning fallacy, optimizing a metric instead of
the goal, and trusting a single signal.

## 3. Adversarial review caught real errors before they shipped

The 3/3-red reconcile in §2 was itself the largest save: left unexamined it becomes a fabricated bug
report against a feature that actually works. Three more.

A comment had been written into a committed flow file asserting that "in Maestro 2.6.1 a subflow env
default OVERRIDES the runFlow-passed value." Because the repo is public and that is exactly the kind of
claim a reviewer will probe, it was flagged for verification — does the default actually override or
only shadow, and is it version-specific — before it was allowed to stand as interview ammunition.

The agent's own remediation call was overridden once. It correctly diagnosed an intermittent
focus-race in the foundational `register-user` subflow — a password landing in the email field because
a tap missed focus while the soft keyboard was up — but proposed to "note it for later and stay
vigilant." That was pushed back: `register-user` is used by every flow, stability is the top grading
criterion, and the assessor runs the suite three times, so a known intermittent fault in the foundation
is exactly what sinks a stability-graded submission. It was hardened then (the keyboard dismissed /
focus checked between fields, which touches mechanics not assertions) and the full suite was run three
times to confirm.

At delivery, when the commits were made, the agent bundled all of them into a single script that one
approval would have run end to end — quietly collapsing the per-commit review that had been asked for.
That was caught at review rather than after the fact, and the commits were made one at a time instead.

In every case the point is the same: the AI's output is the input to review, never the end of it.

## 4. Scope discipline as default-deny

Several attractive additions were rejected with the reason recorded rather than half-built: CI (no
hosted ARM64 emulator, an offline app, and an APK that cannot be committed), a JUnit / machine-readable
report, selective-run tags, iOS coverage, and a revisit of Appium or Patrol — the last two rejected at
the stack level (Appium is heavier for no benefit here; Patrol needs the app source, and only a release
APK was available). All of them live in the README's future-work section. A rejection with a reason
reads as a stronger judgment than an unfinished feature.

One structural decision belongs here:

> A single Claude Code terminal was used rather than the two-terminal planner/builder split from an
> earlier, larger project. That project — eight phases, a real production architecture — justified a
> separate planning terminal. This task is roughly a dozen declarative Maestro flows; the judgment is
> discovery-shaped (screenshot → defect vs spec → flow), it does not cleanly split into plan-then-
> execute, and Desktop already supplies the planning layer. A second terminal would have been a third
> layer where two suffice. Plan mode was used as a mode inside the one terminal, not a separate window.
> The lighter setup is matching the tool to the scale, not cutting a corner.

The same default-deny showed up at closeout: a set of housekeeping meta-tasks that the strict "nothing
beyond this" scope appeared to exclude were surfaced for an explicit ruling rather than silently
dropped or silently done, and the Windows instructions (§1) were declined to keep the README on the
path actually walked.

## 5. Creativity, bounded and grounded in the data

Two sharp moves carried most of the weight, each kept honest under the same check — the agent was asked
to say plainly whether a move added something or was decoration, and to drop it if it did not.

`BUGS.md` documents each defect with steps, the expected behaviour per spec, the actual behaviour, a
severity, and screenshot plus hierarchy evidence, on the thesis that a red test on a documented defect
is a working test, not a failure to fix. The three root defects — locations concatenated without a
separator, a placeholder shown for an absent salary where the spec wants the element omitted, and an
object-shaped salary leaking as raw JSON — each map to the flow whose spec assertion goes red because of
them, with two layers of evidence: the recon discovery and the suite's own red-run artifacts.

The `SPEC-MATRIX` pairs the positive half — the five requirement areas mapped to flows and expected
results — with an anti-case table: the things the suite must NOT assert, because asserting the broken
output (the concatenated string, the placeholder, the raw JSON) would rubber-stamp the very defect it
is meant to catch. Writing down what not to assert turned out to be as load-bearing as writing the
assertions, and it is what keeps a "make the test pass" instinct from quietly certifying a bug.

The cardinal rule sits behind both artifacts, and it cuts both ways: do not bend an assertion to match
a buggy app, but equally, do not invent a defect where the app conforms. In recon the anti-case
template primed a possible error-position defect on the registration screen; the objective check — the
error label renders above the input, confirmed by coordinates and screenshot, which is what the spec
requires — showed the app conformed, so it was recorded as "no candidate" rather than written up.
Restart-persistence was handled the same way: the spec is silent on it, so the human ruled it out as a
candidate up front; persistence was run as exploration and recorded as observed behaviour, but no flow
asserts it, because asserting behaviour the spec never requires is itself an anti-case.

## Deliberately out of scope

- **CI / hosted pipeline** — needs a self-hosted ARM runner and an out-of-band APK; not justified for a
  human-reviewed suite of this size.
- **JUnit / machine-readable export** — useful for dashboards, not for eight flows.
- **Selective-run tags** — handy at larger scale; the suite runs end to end quickly.
- **iOS** — the brief is Android only.
- **Appium / Patrol** — only if source-level or cross-framework access becomes available.
- **Performance and visual-diff regression** — beyond the functional-spec scope this suite asserts.
- **A dedicated red flow for the list-surface placeholder defect** — Maestro aborts a flow on its first
  failed assertion, so it would be masked behind the earlier reds on that screen; it is proven on the
  history surface and recorded as a known limitation instead of given a flow that would never be
  reached.

## The throughline

One line ties the way of working to the product: do not trust a single signal, assert the spec rather
than the app, document the defect instead of hiding it, and know the blind spots well enough to write
them down. The README's "Known limitations" and this note are part of that posture — engineering
maturity, not apology. The raw record is in `docs/ai-history/`.