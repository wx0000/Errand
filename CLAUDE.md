# CLAUDE.md — Buggy Maestro E2E suite

> **Read this file at the start of every session.**
> Conversation with the CLI is in Polish; **every file in this repo is written in English.**

---

## What we are building

A **Maestro E2E test suite** for the Android app **Buggy**, asserting the behavior the
**task specification** requires. The deliverable is ≥5 E2E tests plus a README that brings
the suite up from a clean clone on the reviewer's machine.

The reviewer's environment equals the dev environment: **macOS, MacBook Air M2, an Android
ARM64 emulator driven over `adb`.**

## Project facts

| | |
|---|---|
| App | **Buggy** — Flutter **release** build, `arm64-v8a` |
| appId | `com.example.buggy` |
| Network | **Offline** — the app has **no `INTERNET` permission**; do not write tests that assume network |
| Tooling | **Maestro** (flows in YAML) |
| Device | Android **ARM64 emulator** via `adb` |
| Repo | **Public** — see Hygiene below |

---

## Cardinal rule (binding)

**Tests assert the SPECIFICATION, never the app's behavior.**

The app almost certainly contains **intentional defects** versus the spec. A test that fails
because the app violates the spec is doing its job — that is a *finding*, not a test bug.

> **"Adjust the assertion so it passes" against behavior that contradicts the spec is a
> disqualification.** If that move ever surfaces in reasoning — even as a tempting shortcut —
> **name it explicitly** and reject it. We never bend an assertion to match a buggy app.

This session (and recon generally) produces **material**; the human issues the **BUG/PASS
verdicts** from the screenshots. Claude does not rule defects.

---

## SPEC MATRIX

**SPEC MATRIX = binding ground-truth, lives in `docs/SPEC-MATRIX.md`** (single source: R1–R5
expected-per-spec, recon `actual`, bug candidates, anti-case placeholders; `verdict` column is
the human's, never Claude's). Do not keep a second copy of the skeleton here.

---

## Plan-bridge (how we work)

- **One terminal, plan mode on a toggle.** When thinking is needed before writing (recon →
  SPEC MATRIX), switch plan mode ON in the same window, design the plan, save it to
  `.claude/plans/<phase>-<topic>.md`, switch plan mode OFF, then build from the file.
- **The plan bridge is the file** in `.claude/plans/`. Plan mode is a *mode*, not a separate
  terminal.
- **Judgment stays in the main window** (stress-testing the plan, the defect-or-not call) —
  the transcript is part of the deliverable. Mechanics may be delegated to a subagent.
- **Build verifies the plan against the real emulator before implementing** — it does not
  execute the plan blindly. If reality contradicts the plan, stop and reconcile.

## Self-Validation Protocol (apply before showing anything)

1. **Every bug candidate** names: which requirement it asserts, the expected-per-spec, and the
   **real evidence** — a string quoted from the hierarchy dump, or a screenshot filename.
   Never a *predicted* result.
2. **Paste real run outputs** (maestro, adb). Never fabricate command output.
3. **Flag your own flailing.** If bash/maestro has not returned a result yet, do **not** narrate
   a result you don't have — stop, wait for the real output, then conclude.
4. **Anti-sycophancy.** When a move is questioned, defend it with evidence or concede with a
   stated reason. No reflexive agreement.
5. **Explain before every approval gate.** Before every action that triggers a user approval
   prompt — file edit, bash command, file read, anything — state in 1–3 plain-language sentences
   what it does and why, in the prose immediately before it. No bare action at an approval gate.

## Maestro / selector conventions (P1 recon — load-bearing for P2)

Learned from real recon (P1) and the P2 build against Buggy; load-bearing for P3/P4.

- **Selectors are regex anchored to the FULL node text (DOTALL).** Flutter exposes text as
  `accessibilityText` / `hintText` in a `"label\nvalue"` shape, so a bare substring does **not**
  match — wrap tokens (`.*token.*`) or use the exact full string.
- **Tap the clickable `Button`, not the wrapper `View`** — the wrapper is often
  `clickable=false` and the tap fails.
- **Pass `STAGE` only via `-e STAGE=<act>`.** Do **not** add `env: STAGE: …` in the flow header:
  in Maestro 2.6.1 the flow `env` default overrides `-e` and silently pins every run to the default.
- **Subflows take params via `runFlow`'s `env:` — and declare NO `env:` defaults.** Same Maestro
  2.6.1 precedence bug as `-e`: a subflow's `env` default overrides the value the caller passes,
  silently resolving every `${VAR}` to that default. Reference `${VAR}` directly with no defaults
  block; a missing param then errors loudly instead of typing blank. (P2: confirmed the hard way —
  empty defaults made every registration field submit blank → 3/3 false red until removed.)
- **Form flows (`register-user` etc.): `hideKeyboard` after each `inputText`, before the next field
  tap.** Gboard has a transitional floating-toolbar state that absorbs a tap fired mid-transition, so
  focus does not move and the next `inputText` lands in the previously-focused field (intermittent
  flake). Dismissing the keyboard between fields makes every tap land on a clean screen. Confirmed on
  artifact (P3).
- **Snackbar = transient, absent from the a11y hierarchy.** Catch it in-flow with a screenshot +
  `extendedWaitUntil` on its token (`.*Applied to.*`); the **screenshot is the evidence**.
- **Picker = modal, parkable** → dump its hierarchy out-of-band (`maestro hierarchy`).
- **`maestro hierarchy`** (no flag) = JSON; **`--compact`** = CSV.

The full per-screen selector map lives in the comments of `recon.yaml` (repo root).

## End-of-Session Cycle (run BEFORE closing a session, in order)

If a step does not apply, **say which and why — never skip silently.**

1. **Suite green/red** — run the suite, report the result.
2. **Update CLAUDE.md** — Current Status (phase, next task, R1–R5 coverage) + a Session Log entry
   (what was done, decisions, judgment moments).
3. **Update PROGRESS.md** — check off what's done, note any drift if the plan changed.
4. **`/export`** this session to `docs/ai-history/` as `YYYY-MM-DD-<phase>-<role>.txt`.
5. **Archive the plan — UNCONDITIONAL.** Every plan created this session is copied from
   `.claude/plans/` to `docs/ai-history/` as `<phase>-<topic>.md` before the session closes —
   **always, even a minor plan.** Reason: `.claude/` is gitignored, and the plans are the
   **decision layer** of the delivery docs that the build transcript alone cannot reconstruct.
6. **Update the index** in `docs/ai-history/README.md` with the new files.
7. **Local commit** — conventional commits, one commit = one logical unit (scaffold separate
   from recon), no squash; history tells the process. **Push is the human's call — do not
   commit without explicit consent.**
   **Never add `Co-Authored-By` or "Generated with Claude Code" trailers to commits or PRs.**
   (Attribution is disabled in `.claude/settings.local.json`; this line is the backup, since the
   setting is intermittently ignored — known bug.)
8. **Handoff prompt** for the next session (what's done, what's next, what to watch, which file
   to read first) + a recommended effort with a one-sentence justification.

## Hygiene

- **`buggy.apk` and the task PDF are in `.gitignore` from the first commit** and must never
  enter the repo or its history (public repo, company-owned artifacts).
- These are the **only** forbidden files. Recon artifacts (hierarchy dumps, screenshots) are
  committed as evidence for the human's verdicts.
- `.claude/` is gitignored; plans reach the deliverable only via the archival step above.
- **Scratch artifacts are NEVER committed.** Throwaway/scratch artifacts (smoke tests,
  `_`-prefixed scratch, temp files, `/tmp` dumps) never enter a commit. Before **any** commit,
  `git status` shows only real deliverables. At End-of-Session, confirm no scratch file is staged
  or tracked — only real recon evidence under `docs/recon/` is committed.

---

## Current Status

- **Phase:** **P5 — repo-build DONE; Desktop closeout (lead) pending** (local commits, **not pushed**).
  Delivery layer shipped: `README` (clean-clone bring-up + cardinal rule + architecture + design
  decisions + known limitations incl. **BUG-B-on-list gap** + AI-assisted section + future work),
  `docs/TEST_PLAN.md` (R1–R5 matrix + anti-cases + ~2-min manual procedure), `docs/BUGS.md` finalized
  (P2–P4 red-run artifacts linked under BUG-A→01 / BUG-B→04 / BUG-C→02+05, alongside recon). Suite
  **3× stable** — **4 pass / 4 red, 0 flake, 0 state-flip** (lead ruled Gate 1); **fresh-clone bring-up
  proof** = 4 pass / 4 red (lead accepted as the pre-commit Gate-2 proof; clean-machine = human
  post-push). IP hygiene: **zero APK/PDF in tree + history**; `.idea/` now git-ignored.
- **Scope boundary (lead):** repo-build = Claude Code; **3 meta-tasks are lead/Desktop-owned, executed
  in P5 outside the repo-build** — `WORKING-WITH-AI.md` consolidation, orphaned-decision audit,
  `STRATEGIA-PROJEKT.md` §10 sync. **Full P5 DONE = repo-build (Code) + Desktop closeout (lead).**
- **Gated build (both gates lead-ruled):** Gate 1 (stability verdict + any unexpected red) and Gate 2
  (clean-machine) were the human's calls; Code ran, reported real output verbatim, restored `docs/runs/`
  after each run (zero screenshot churn), and stopped at each gate. No assertion ever bent.
- **Next task:** human `/export` → `docs/ai-history/2026-06-17-p5-build.txt`, then the Desktop closeout
  (3 meta-tasks) + **push (human's call)**. Plan archived to `docs/ai-history/p5-closeout.md`.
- **R1–R5 coverage — ALL ASSIGNED:** **R1 → flows 01+02** (BUG, red = findings ✓), **R2 → flow 03**
  (PASS ✓), **R3 → flows 04+05** (BUG, red = findings ✓), **R4 → flows 06+07** (PASS ✓), **R5 → flow
  08** (PASS ✓).

## Session Log

> Newest entries at the top. Each entry: date, phase, what was done, decisions, judgment moments.

### 2026-06-17 — P5 — build (repo-build slice)

- **Done:** built P5 repo-build from the revised plan (archived `p5-closeout.md`). `README` rewritten
  from the stub (8 sections + "How this was built (AI-assisted)"); `docs/TEST_PLAN.md` created (R1–R5
  matrix + anti-cases + ~2-min manual procedure, paraphrased from SPEC-MATRIX); `docs/BUGS.md`
  finalized — suite red-run artifacts linked under each defect (BUG-A→`01`, BUG-B→`04` + documented
  list-gap, BUG-C→`02`+`05`) alongside the recon links; `.idea/` git-ignored; IP hygiene proven (zero
  APK/PDF in tree + history). Full suite **3× stable** (4 pass / 4 red, 0 flake, 0 state-flip);
  **fresh-clone bring-up** = 4 pass / 4 red. Local commits, **not pushed**. Plan-archive + index +
  `/export` transcript = the human's ai-history step.
- **Decisions (lead-directed):** two **hard stop-gates** — Gate 1 (stability verdict + any unexpected
  red) and Gate 2 (clean-machine) — both **the lead's adjudication, not Code's**. Clean-machine ruling
  deferred to the human post-push; Task F accepted as the pre-commit bring-up proof. **3 meta-tasks
  (WORKING-WITH-AI consolidation, orphaned-decision audit, STRATEGIA §10 sync) = lead/Desktop-owned**,
  outside the repo-build → **P5 NOT marked fully DONE** (repo-build slice only). README §3 prereqs from
  the **real** env (OpenJDK 21.0.11 LTS, Maestro 2.6.1, macOS 26.5.1, `arm64-v8a`), never assumed.
- **Judgment moments:**
  1. **Surfaced the scope conflict before planning.** PROGRESS listed 3 closeout meta-tasks the prompt's
     "nic poza tym" excluded — asked rather than silently dropping or doing them; took the lead's
     "lead/Desktop-owned, P5-outside-repo-build" ruling and labelled them so PROGRESS stays consistent.
  2. **Did NOT self-rule the verdicts.** Reported all 3 stability matrices + the fresh-clone result
     verbatim and **stopped at each gate**; the lead ruled stable / clean-machine. Cardinal rule held —
     no assertion bent; the 4 reds are the app violating spec, confirmed eye-vs-assertion.
  3. **Zero evidence churn.** Every suite run (3× stability + fresh-clone) regenerated `docs/runs/`
     screenshots; restored the whole dir with `git checkout` after each so the runs produced
     verdict + console evidence without re-committing the P4 artifacts of record.
  4. **Faithful pre-commit clean-clone test.** Couldn't commit before Gate 2, so overlaid the 4
     uncommitted P5 docs into the throwaway clone to simulate the post-commit state — disclosed the
     overlay; the lead accepted it as the Gate-2 bring-up proof.

### 2026-06-16 — P4 — build

- **Done:** built P4 from the plan-mode design (archived `p4-r1-offers.md`) → flows
  `01-offers-locations` (R1/C1→BUG-A) + `02-offers-salary` (R1/C3→BUG-C). Both read-only on the list,
  reuse `register-user`, no new subflow. R1 **red = findings** (list concatenates multi-location as
  `WarsawGdansk`, and leaks raw JSON `{"min":10000,"max":15000}` for the object salary). Full suite =
  **8 flows: 4 green + 4 red-findings**; flake **3× / 0** (incl. 01's scroll-back UP). SPEC-MATRIX R1
  `Flow` cell → `01, 02` (only edit). Feat commit `5474ef5`; docs + plan-archive commits local
  (**not pushed**). Evidence: `docs/runs/screens` + `docs/runs/hierarchy` 01/02.
- **Decisions (lead-directed):** 2 flows in the 01–08 scheme (no renumber); BUG-A target = **BigTech
  (`WarsawGdansk`)** — only tile with *just* BUG-A (clean salary) → cleanest screenshot evidence; the
  5-offer green proof scrolls past it, so a `scrollUntilVisible direction: UP` re-positions before the
  red. **BUG-B on the LIST = accepted documented gap** (Maestro aborts on first failure → can't share
  01/02 without masking; root defect already red in `04` + anti-case; gap note → README at P5).
  `01` asserts `.*Warsaw, Gdansk.*` (spec comma form), `02` asserts `.*Salary: 10000-15000.*` (spec
  range) — never the run-together string or the JSON (anti-case).
- **Judgment moments:**
  1. **Re-confirmed ground-truth before writing assertions (hard STOP discipline).** Fresh
     `maestro hierarchy` vs the 2026-06-14 dumps: all per-tile strings matched, and the spec forms
     (`Warsaw, Gdansk`, `10000-15000`) were absent everywhere — so the reds are genuine spec
     violations, not selector misses. Two first-pass grep "misses" were JSON/CSV escaping artifacts,
     proven on the raw lines — not a divergence, so no false STOP.
  2. **Verified the screenshot framing the lead flagged, on the real run.** The scroll-back UP could
     have left `WarsawGdansk` at the viewport edge; inspected `01-offers-locations.png` and confirmed
     the location node is fully framed (BigTech centred, clean salary beside it) — no reconcile needed.
  3. **Flagged BUG-B-on-list as a real (narrow) coverage gap rather than silently accepting.** Surfaced
     it to the lead with the masking constraint; took the "accept + document" call only after the lead
     ruled. Mechanic-proof precedes every red, so each red is the app violating spec, not a drive miss.
  4. **Kept the commit clean.** The full-suite run overwrote P2/P3 screenshots (03/04/05); restored
     them (`git checkout`) so the P4 commit carries only P4 deliverables. Left untracked `.idea/` out.

### 2026-06-15 — P3 — build

- **Done:** built P3 from the plan-mode design (archived `p3-apply-history.md`) → subflow
  `apply-to-offer` (tap-only, param `OFFER`) + flows `03-apply` (R2), `04-history-no-salary`
  (R3/C4→BUG-B), `05-history-salary-format` (R3/C5→BUG-C). R2 **green** (PASS); R3 **red = findings**
  (history renders a salary line `—` for a no-salary offer, and raw JSON `{"min":…}` for an object
  salary — adjudicated on screenshot + hierarchy in `docs/runs/`). Hardened `register-user`; full
  suite **3× clean, 0 flake**. `## Anti-cases` replaced (R1+R3 BUG rows, snackbar title-only,
  persistence). Commits `c8c90a6`/`4664018`/`8a61dcb`, pushed to `origin/main`.
- **Decisions:** `apply-to-offer` = tap-only (param `OFFER` = company anchor; picker + snackbar stay
  in the flow) — single-responsibility like `register-user`. R2 = one flow (single + multi, same
  requirement, both PASS → nothing masks). R3 = **two isolated flows** (Maestro aborts on the first
  failed assertion → one combined flow would mask the second finding). C4 = `assertNotVisible
  ".*Applied job salary.*"` (no salary line at all; single-entry history); C5 = `assertVisible
  ".*…salary: 10000-15000.*"` (positive spec range) — never assert the JSON/placeholder (anti-case).
- **Judgment moments:**
  1. **Mechanics bug, NOT an app finding (again).** First `03` run red at the `Job Offers` gate; the
     screenshot showed PASSWORD concatenated into the EMAIL field. Resisted misreading it as an R2
     finding — re-ran `07` (green) + `03`×3 (green) to prove `register-user` flakes intermittently,
     not the app.
  2. **Confirmed the flake mechanism on an artifact before fixing (no guessing).** A scratch `/tmp`
     probe screenshot caught the real cause — a Gboard transitional floating-toolbar overlay absorbing
     a field tap fired mid-transition, so focus doesn't move. Fix = `hideKeyboard` between fields
     (pure mechanics, no assertion change). Verified by 3× suite / 0 flake.
  3. **Red R3 = the correct result; proved the flow drove the app first.** Each R3 flow asserts the
     green mechanic-proof (reached the right entry) before the spec assertion, so the red is the app
     violating spec, not the flow failing. Never bent an assertion to green.

### 2026-06-15 — P2 — build

- **Done:** designed P2 in plan mode (archived `p2-subflows-reg-login.md`) → built **8 suite files**:
  `config.yaml` (workspace scope = `flows/` only), subflows `register-user`/`login`/`logout`/
  `open-profile`, flows `06-registration-validation` (R4), `07-registration-autologin` (R4),
  `08-login-logout` (R5). Suite **3/3 green**. Commit `5f61654`.
- **Decisions:** `register-user` is a generic registration-**attempt** helper (`EMAIL/PASSWORD/
  CONFIRM`) with **no outcome assertion inside** — the flow asserts success (07) or the validation
  error (06); each subflow asserts only its own readiness gate. R4 "error **ABOVE** field" asserted
  with Maestro's relative `above:` selector (programmatic equivalent of the recon y-coordinates).
  Unique email per run via `evalScript` + `Date.now()`. **No `tags:`/selective-run** (rejected
  extras → README future work, not suite code).
- **Judgment moments:**
  1. **Verify-against-reality caught a mechanics bug, NOT an app finding.** First run **3/3 red**;
     the failure screenshot showed the register form submitted with **empty fields** ("Email is
     required"). Root cause: the subflows' empty `env:` defaults **overrode** the values passed via
     `runFlow` — the exact Maestro 2.6.1 precedence bug noted for `-e`, now confirmed for subflow
     defaults too. Reconciled (removed the defaults) → 3/3 green. Resisted misreading the red as an
     R4/R5 finding (recon already proved them PASS) and never bent an assertion.
  2. Surfaced the `register-user` signature as a design choice (asked); took the recommended 3-param
     option only after the human deferred — flagged correctable at the gate.
  3. Caught my **own plan flaw mid-build** (the plan had prescribed the empty `env:` defaults block);
     reconciled per the plan's "verify before implement" clause and folded the corrected convention
     back into the Maestro conventions above.

### 2026-06-14 — P1 — adjudication

- **Done:** human issued the R1–R5 verdicts → recorded in `docs/SPEC-MATRIX.md` (`verdict` column;
  `actual` left byte-untouched) and created `docs/BUGS.md` — **3 root defects**: BUG-A (multi-loc
  no separator, R1), BUG-B (missing-salary placeholder, R1 list + R3 history), BUG-C (object salary
  as raw JSON, R1 list + R3 history); 5 manifestations on 2 surfaces. C1–C5 mapped to bug-ids.
  Committed `cd4bf26`.
- **Decisions:** `docs/BUGS.md` written in **English** (repo convention) though the human drafted
  it in Polish — confirmed with the human before writing. Verdicts are the **human's**; Claude only
  recorded them (verdict-column rule re-stated to "records, never authors").
- **Judgment moments:**
  1. Caught a prompt-internal conflict — verbatim Polish "exact content" vs. the binding "every
     file in English" rule + "keep repo style" — surfaced it and asked instead of silently choosing.
  2. Self-audited the diff before commit: confirmed the `actual` column is byte-identical (only the
     `verdict` cell appended), so no spec-evidence was altered.
  3. Found `Current Status` stale (still "P0 done / P1 recon blocked") and the recon session left
     no CLAUDE.md Session Log entry — refreshed Current Status; flagged the recon-entry gap to the
     human rather than fabricating it (backfilled in `dd181e3`).

### 2026-06-14 — P1 — recon

- **Done:** full recon R1–R5 against the live emulator. `recon.yaml` (**12 STAGE acts**;
  proof-of-method, **not** the final suite), **17 screenshots + 12 hierarchy dumps** under
  `docs/recon/`, SPEC-MATRIX `actual` filled, bug **candidates C1–C5** (R1 list + R3 history — a
  salary/location formatting cluster; R2/R4/R5 clean) — **no verdicts**. Locale = EN; session +
  history survive an app restart (spec silent → recorded, **not** a candidate). Commit `7a15bd4`.
- **Decisions:** Maestro/selector conventions captured in `CLAUDE.md` (load-bearing for P2);
  `STAGE` passed only via `-e` (no flow `env:` default); `recon.yaml` + all recon artifacts
  committed as evidence (only `buggy.apk` + the task PDF stay out); attribution trailer disabled +
  scaffold history rewritten clean; `verdict` column left to the human.
- **Judgment moments:**
  1. KROK 0 caught the P0 "env empty" note as **stale** — `maestro`/`adb` were a PATH-gap, not
     not-installed (binaries dated after the P0 session), and the human had upgraded Java 9.0.4 →
     21 LTS; proceeded instead of a false STOP.
  2. Discover-then-target: dumped the `offers` list **first** and derived the apply/register/
     history selectors from the real JSON — recon must not assume a no-salary or multi-location
     offer exists.
  3. Per-act `clearState` for isolation, with deliberate exceptions — `login-logout` shares one
     session, `persistence` issues no `clearState` across the restart (else the wiped datastore
     would read as a false BUG).
  4. Multi-location picker confirmed **modal/parkable** → dumped its hierarchy (exact location
     strings) rather than screenshot-only; snackbar handled as transient (screenshot +
     `extendedWaitUntil`), a snackbar timeout treated as **evidence**, not a recon failure.

### 2026-06-12 — P0 — build

- **Done:** scaffold built (`.gitignore`, `CLAUDE.md`, `PROGRESS.md`,
  `docs/ai-history/README.md`). Environment verified — **empty**: no Maestro, no adb/Android
  SDK (`ANDROID_HOME` unset), JDK present but **Java 9.0.4 (EOL)**. Install commands handed to
  the human; **I did not install.** Recon **not** started — double-blocked (toolchain + R1–R5).
- **Decisions:** Polish conversation / English files. Recon artifacts are committed as evidence;
  only `buggy.apk` + the task PDF are excluded. `.claude/` gitignored. Plan archival is an
  unconditional end-of-session rule.
- **Judgment moments:**
  1. Caught a prompt-vs-reality mismatch — the dir was **not** "empty + .gitignore"; it had a
     tracked `README.md`, **no** `.gitignore`, and a **live public remote** (`origin/main`).
  2. Made `.gitignore` the **first** scaffold action precisely because that public remote was
     already live (APK/PDF leak risk is immediate, not hypothetical).
  3. Distinguished **PATH-gap vs not-installed** — probed the default install locations
     (`~/.maestro/bin`, `~/Library/Android/sdk`, Homebrew) before telling the human to install;
     confirmed genuinely absent, not merely off-PATH.

<!-- entries appended at end of each session -->
