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

- [ ] **P2 — Subflows + registration / login**
  - [ ] Reusable subflows (launch, auth helpers, …)
  - [ ] Registration flow
  - [ ] Login flow

- [ ] **P3 — Applying + history**
  - [ ] Apply-to-offer flow
  - [ ] History flow

- [ ] **P4 — Offers list**
  - [ ] Offers list flow(s)

- [ ] **P5 — Stabilization + delivery**
  - [ ] Run the full suite 3× (stability)
  - [ ] `BUGS.md` (human verdicts → documented defects)
  - [ ] `TEST_PLAN`
  - [ ] `README` (clean-clone bring-up)
  - [ ] Fresh-clone verification

## Session Log

> Newest entries at the top. Format:
> `### YYYY-MM-DD — <phase> — <role>` then bullets: **Done**, **Decisions**, **Drift**.

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
