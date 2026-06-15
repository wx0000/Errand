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

- **Phase:** **P3 — DONE** (apply + history; pushed). **+4 suite files** — subflow `apply-to-offer`
  (tap-only, param `OFFER`) + flows `03-apply` (R2), `04-history-no-salary` (R3/C4),
  `05-history-salary-format` (R3/C5). Suite = **6 flows: 4 green + 2 red-findings** (R3 reds are the
  correct result, adjudicated on `docs/runs/` artifacts). Also hardened `register-user` (IME
  focus-race → `hideKeyboard` between fields); suite **3× / 0 flake**. Commits `c8c90a6` (fix),
  `4664018` (feat), `8a61dcb` (docs) — pushed to `origin/main`.
- **Next task:** **P4 — offers list (R1).** R1 = **BUG** (list surface of the same cluster as R3):
  C1→BUG-A (multi-loc no separator), C2→BUG-B (no-salary `null`), C3→BUG-C (raw JSON). Expect
  **red = findings**; assert SPEC (comma-separated locations, no salary element when absent,
  formatted range), do not bend. The R1 anti-case rows (BUG-A, BUG-B-list, BUG-C-list) are
  **already in `docs/SPEC-MATRIX.md`** → don't re-add; the flows build against them. **Flow count =
  plan-mode decision:** 3 distinct findings + Maestro's first-failure abort imply 3 flows, but only
  `01`+`02` are free in the 01–08 scheme → reconcile with the lead (extend numbering vs accept masking).
- **R1–R5 coverage:** **R2 → flow 03** (PASS, green ✓), **R3 → flows 04+05** (BUG, red = findings ✓),
  **R4 → flows 06+07** (PASS, green ✓), **R5 → flow 08** (PASS, green ✓). Still unassigned: **R1**
  (offers list, BUG) → P4 (flows 01+02).

## Session Log

> Newest entries at the top. Each entry: date, phase, what was done, decisions, judgment moments.

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
