# PROGRESS — Buggy Maestro E2E suite

Phase tracker. Tests assert the **spec**, not the app (see `CLAUDE.md` → Cardinal rule).
Update at the end of every session: check off what's done, note drift if the plan changed.

## Phases

- [x] **P0 — Scaffold + CLAUDE.md**
  - [x] `.gitignore` (buggy.apk + PDF excluded from the first commit; `.claude/` ignored)
  - [x] `CLAUDE.md` built from scratch (SPEC MATRIX, cardinal rule, protocols, cycle)
  - [x] `PROGRESS.md`
  - [x] `docs/ai-history/README.md` (transcript naming convention + index)
  - [x] Environment verification — **run; env empty** (no maestro/adb/SDK; JDK 9.0.4 EOL); exact install commands handed to human

- [x] **P1 — Recon + SPEC MATRIX** — **COMPLETE**
  - [x] `recon.yaml` — STAGE-parameterized recon flow (12 acts; proof-of-method, not the final suite)
  - [x] Per-screen Maestro hierarchy dumps → `docs/recon/hierarchy/` (12 files)
  - [x] Per-screen screenshots → `docs/recon/screens/` (17 files)
  - [x] Fill the `actual` column of the SPEC MATRIX from real evidence (R1–R5)
  - [x] List bug **candidates** C1–C5 (R1/R3) with artifacts — **no verdicts**
  - [x] **Adjudication** — human R1–R5 verdicts → SPEC-MATRIX `verdict` filled + `docs/BUGS.md` (3 root defects BUG-A/B/C); commit `cd4bf26`

- [x] **P2 — Subflows + registration / login** — **COMPLETE**
  - [x] Reusable subflows — `register-user`, `login`, `logout`, `open-profile` (+ `config.yaml` scope)
  - [x] Registration flows — `06-registration-validation` (R4), `07-registration-autologin` (R4)
  - [x] Login flow — `08-login-logout` (R5)
  - [x] Suite **3/3 green** (R4/R5 PASS, as adjudicated); commit `5f61654`

- [x] **P3 — Applying + history** — **BUILD DONE; R3 findings adjudicated on artifacts**
  - [x] `apply-to-offer` subflow (tap-only, param `OFFER`) + `03-apply` (R2) — suite **green** (PASS)
  - [x] History flows `04-history-no-salary` (R3/C4→BUG-B) + `05-history-salary-format` (R3/C5→BUG-C)
        — **red = findings, confirmed** (human adjudicated on screenshot + hierarchy in `docs/runs/`);
        linking the red artifacts into `BUGS.md` = P5
  - [x] Anti-case table filled (R2 + R3 + persistence) in `docs/SPEC-MATRIX.md`; R1/BUG-A row → P4
  - [x] Hardened `register-user` (IME focus-race → `hideKeyboard` between fields); full suite **3× green/red
        as expected, 0 flake** (18 register invocations clean)

- [x] **P4 — Offers list** — **BUILD DONE; R1 findings = red, human adjudicates on artifacts**
  - [x] Offers-list flows `01-offers-locations` (R1/C1→BUG-A: multi-loc concat) + `02-offers-salary`
        (R1/C3→BUG-C: raw-JSON salary) — **red = findings**; read-only on the list (reuse
        `register-user`, no new subflow). Suite **8 flows: 4 green + 4 red**; flake **3× / 0**.
  - [x] SPEC-MATRIX R1 `Flow` cell → `01, 02` (only edit). Evidence: `docs/runs/screens` +
        `docs/runs/hierarchy` 01/02. Feat commit `5474ef5`; local only (not pushed).
  - [x] Lead: 2 flows (no renumber); BUG-A target = BigTech (`WarsawGdansk`); **BUG-B-on-list =
        accepted documented gap** (root defect red in `04` + anti-case; gap note → README at P5)

- [ ] **P5 — Stabilization + delivery**
  - [ ] Run the full suite 3× (stability)
  - [ ] `BUGS.md` — **skeleton done in P1 adjudication** (3 root defects, commit `cd4bf26`); P5 finalizes it by linking the P2–P4 red-run Maestro artifacts
  - [ ] `TEST_PLAN`
  - [ ] `README` (clean-clone bring-up)
  - [ ] Fresh-clone verification
  - [ ] **Sync section 10 (instructions) of `STRATEGIA-PROJEKT.md`** — one-off merge of the live project
        instructions (the project "instructions" field) with section 10. Two-way divergence: do **NOT**
        overwrite blindly — each side holds content the other lacks. After the merge: master = the
        "instructions" field; section 10 becomes a mirror stamped with the resync date.
  - [ ] **Orphaned-decision audit** — sweep the project conversations for decisions that never landed
        in any document.

## Session Log

> Newest entries at the top. Format:
> `### YYYY-MM-DD — <phase> — <role>` then bullets: **Done**, **Decisions**, **Drift**.

### 2026-06-16 — P4 — build

- **Done:** flows `01-offers-locations` (R1/C1→BUG-A) + `02-offers-salary` (R1/C3→BUG-C), read-only
  on the list, reuse `register-user`. R1 **red = findings**; full suite **8 flows: 4 green + 4 red**;
  flake **3× / 0** (incl. 01's scroll-back UP). Ground-truth re-confirmed vs the 2026-06-14 dumps (no
  divergence). SPEC-MATRIX R1 `Flow` → `01, 02`. Feat commit `5474ef5`; docs + plan-archive local
  (not pushed). Plan `p4-r1-offers.md` archived to `docs/ai-history/`.
- **Decisions (lead):** 2 flows in the 01–08 scheme (no renumber); BUG-A target = BigTech
  (`WarsawGdansk`, only tile with just BUG-A → cleanest evidence); **BUG-B-on-list = accepted
  documented gap** (Maestro first-failure abort → can't share 01/02 without masking; root defect red
  in `04` + anti-case; gap note → README at P5).
- **Drift:** none vs the approved plan. All R1–R5 now have flows; remaining work is P5 (stabilization
  + delivery docs).

### 2026-06-15 — P3 — build

- **Done:** subflow `apply-to-offer` + flows `03` (R2 green/PASS), `04`+`05` (R3 **red = findings**,
  C4/BUG-B + C5/BUG-C, adjudicated on `docs/runs/` artifacts). Hardened `register-user` (IME
  focus-race → `hideKeyboard` between fields); full suite **3× / 0 flake**. `## Anti-cases` filled.
  Commits `c8c90a6`/`4664018`/`8a61dcb`, pushed to `origin/main`.
- **Decisions:** R2 = one flow (single+multi, same requirement); R3 = two isolated flows (first-failed
  assertion would mask the second finding); `apply-to-offer` tap-only (param `OFFER`); C4 =
  `assertNotVisible` salary line, C5 = `assertVisible` range — never the JSON/placeholder.
- **Drift:** none vs the approved plan. Lead-directed additions: `register-user` hardening (P2 subflow)
  as a separate fix commit; `docs/runs/` evidence dir; 2 new P5 closeout items (STRATEGIA-PROJEKT
  section-10 sync, orphaned-decision audit).

### 2026-06-15 — P2 — build

- **Done:** 8 suite files (`config.yaml` + 4 subflows + 3 flows: `06` R4-validation, `07`
  R4-autologin, `08` R5 login/logout). Suite **3/3 green** (R4/R5 PASS, as adjudicated). Commit
  `5f61654`. Plan `p2-subflows-reg-login.md` archived to `docs/ai-history/`.
- **Decisions:** `register-user` = generic registration-attempt helper (3-param), no outcome
  assertion inside; R4 "error above field" asserted via Maestro `above:`; unique email per run
  (`evalScript`+`Date.now()`); no `tags:`/selective-run (rejected extras → README future work).
- **Drift:** the approved plan prescribed an empty `env:` defaults block in the subflows — that
  contradicts Maestro 2.6.1 (a subflow `env` default overrides the `runFlow`-passed value → blank
  fields → **3/3 false red**). Reconciled in-build (removed the defaults) and updated the CLAUDE.md
  Maestro conventions so P3/P4 subflows inherit the fix.

### 2026-06-14 — P1 — adjudication

- **Done:** human issued R1–R5 verdicts → recorded in `docs/SPEC-MATRIX.md` (`verdict` column;
  `actual` untouched) + created `docs/BUGS.md`: **3 root defects** BUG-A/B/C, 5 manifestations
  across R1 list + R3 history. C1–C5 mapped to bug-ids. Commit `cd4bf26`. Next: **P2** (subflows
  + registration/login).
- **Decisions:** `BUGS.md` in English (repo convention; human drafted in Polish — confirmed
  first). Verdicts are the human's; Claude only recorded them.
- **Drift:** `BUGS.md` landed in P1 (originally a P5 item) — skeleton now; P5 only finalizes it by
  linking the P2–P4 red-run Maestro artifacts.

### 2026-06-14 — P1 — recon

- **Done:** full recon R1–R5. `recon.yaml` (12 STAGE acts), 17 screenshots + 12 hierarchy dumps in
  `docs/recon/`, SPEC-MATRIX `actual` filled. Candidates **C1–C5** (R1 list + R3 history — a
  salary/location formatting cluster). R2/R4/R5 clean. Locale = EN; persistence survives restart
  (spec silent → recorded, not a candidate).
- **Decisions:** Maestro selector conventions captured in `CLAUDE.md` (load-bearing for P2);
  attribution trailer disabled + scaffold history rewritten clean; verdict column left to the human.
- **Drift:** none vs the recon plan. Persistence kept as exploration only (no persistence assertion).

### 2026-06-12 — P0 — build

- **Done:** P0 scaffold complete; env verified empty (install handed to human).
- **Decisions:** scope split — P0 is its own transcript; recon deferred to a fresh session.
- **Drift:** none vs plan. P1 blocked on toolchain install + R1–R5.

<!-- entries appended at end of each session -->
