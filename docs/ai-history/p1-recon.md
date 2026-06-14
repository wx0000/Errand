# P1 Recon Plan — Buggy Maestro E2E suite

## Context

We are building a Maestro E2E suite for the Android app **Buggy** (Flutter release, arm64-v8a,
**offline**, appId `com.example.buggy`). Tests must assert the **specification (R1–R5)**, never
the app's behavior — the app likely contains intentional defects, and a spec-violating failure
is a *finding*, not a test bug. **Claude does not rule BUG/PASS; the human adjudicates on
screenshots.**

This session is **P1 recon**: gather *material* (per-screen view-hierarchy dumps + screenshots,
including transient states) covering **every screen and variant**, fill the `actual` column of
the SPEC MATRIX from real evidence, and list **bug candidates** (expected-per-spec vs actual +
artifact) **without verdicts**. Recon does not write the final suite (that is P2+).

## KROK 0 — corroboration (DONE; real outputs captured this session)

| Check | Result | Status |
|---|---|---|
| `eff2394` on remote | `origin/main` contains `eff2394`; `main...origin/main` no ahead/behind; `fetch --dry-run` empty | ✅ on remote — **do not push** |
| maestro | `2.6.1` at `~/.maestro/bin/maestro` (off-PATH) | ✅ installed |
| adb / emulator | `emulator-5554  device`; `get-state=device` | ✅ up |
| Buggy package | `package:com.example.buggy` | ✅ installed |
| Java | `openjdk 21.0.11 LTS` (P0 log said 9.0.4 — human upgraded) | ✅ modern |

**One mismatch, resolved:** `maestro`/`adb` are **not on PATH** (first probe `command not found`);
default-location probe proved **PATH-gap, not not-installed** (binaries dated 12 Jun 23:37/23:43,
after the P0 session). CLAUDE.md/PROGRESS "env empty" is **stale**. **No STOP** — toolchain
prerequisite is satisfied; only open blocker is R1–R5 (pasted at acceptance, KROK 3).

**Consequence:** every recon invocation must set env (off-PATH tools) and run from repo root
(so `takeScreenshot` workspace-relative paths land under the repo):
```sh
export ANDROID_HOME="$HOME/Library/Android/sdk"
export PATH="$HOME/.maestro/bin:$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator:$PATH"
cd /Users/wx0000/AIAIAI/Claude/Errand
```

## Maestro 2.6.1 mechanisms (verified from docs; re-verify in build before relying)

- **Transient capture without sleep:** `extendedWaitUntil: { visible: <sel>, timeout: <ms> }`
  returns the instant the element appears → the next `takeScreenshot` fires while it is on
  screen. Snackbar **lifetime** = `visible` screenshot, then `extendedWaitUntil: { notVisible }`
  → optional "dismissed" screenshot. No hardcoded wait.
- **No in-flow hierarchy dump.** `maestro hierarchy --compact` is a **CLI** command on the
  *current* screen (JSON: `attributes.text`, `bounds`, `children`); the app must be **parked**.
  `--debug-output` saves only `maestro.log` + `commands-*.json` (NOT hierarchy/screenshots).
- **`takeScreenshot: <path>`** → `.png`, path **relative to workspace (= cwd)**; subdirs honored.
- **State:** `launchApp: { clearState: true }` wipes app data; `stopApp` then `launchApp`
  (no clearState) **preserves** the datastore → this is the restart-for-persistence move.
- **One-file parameterization (no subflows):** `runFlow: { when: { true: ${STAGE == "..."} },
  commands: [...] }` driven by `maestro test -e STAGE=<act> recon.yaml`. `when:` accepts
  `visible/notVisible/platform/true(JS)`. **← verify this combo in build first (smoke test).**
- Text selectors are **regex**; `copyTextFrom` → `${maestro.copiedText}`. Exact-text evidence
  comes from the **hierarchy `text` field**, not log-scraping.

## Recon architecture

A **single committed `recon.yaml`** in repo root (the only YAML this session; **not** part of
the final suite — it is proof-of-method + the screenshot evidence trail). It is **parameterized
by `STAGE`**: each `maestro test -e STAGE=<act>` run navigates one act, takes in-flow
screenshots, and **parks** on the act's target screen; a shell step then dumps that parked
screen's hierarchy. Transients (snackbar, picker) are evidenced by **in-flow screenshots only**
(they auto-dismiss → cannot be parked); stable screens get **both** screenshot + hierarchy.

- Screenshots → `docs/recon/screens/NN-<screen>[-variant].png`
- Hierarchy → `docs/recon/hierarchy/NN-<screen>.json`
- Both **committed** as evidence (only `buggy.apk` + task PDF are forbidden).

### Acts (one STAGE ≈ one parked-hierarchy target)

| STAGE | Navigation (selectors DERIVED from earlier dumps, not pre-written blind) | In-flow screenshots | Parked hierarchy | clearState |
|---|---|---|---|---|
| `locale` | launch, land on first screen | first screen | first screen | start |
| `offers` | go to offers list; scroll | list-top, list-scrolled | offers list (discover salary / no-salary / multi-location rows) | start |
| `apply-single` | open salaried offer → tap Apply | offer-detail, **snackbar (visible)**, snackbar-dismissed | (post-dismiss) | start |
| `apply-multi` | open multi-location offer → Apply | **picker popup**, **snackbar w/ chosen location** | (post) | start |
| `reg-email` | register form, bad email, submit | error state | error node `text`+`bounds` vs input `bounds` | start |
| `reg-pwd` | register form, password <8, submit | error state | error `text`+`bounds` | start |
| `reg-confirm` | register form, confirm mismatch, submit | error state | error `text`+`bounds` | start |
| `reg-valid` | register valid unique creds, submit | auto-login landing | auto-login screen | start |
| `login-logout` | register acct → logout → login (same creds) | logged-in, logged-out, logged-in | final logged-in | start; **none** between logout↔login |
| `profile-history` | register → apply to salaried **and** salary-less offer → open history | both entry formats | history (entry format incl. no-salary variant) | start |
| `persistence` | register → apply (seed) → history → `stopApp` → `launchApp` (no clearState) → reopen history | history-before, history-after-restart | history-after-restart | start; **none mid-act** |

**Locale:** primary evidence = current UI language read from the `locale` hierarchy `text`
fields + `adb shell getprop persist.sys.locale` / `settings get system system_locales`.
Secondary = attempt a system-locale switch + relaunch to test "follows system"; **flag as
uncertain** if the switch mechanism (may need reboot/root on the emulator) is unreliable —
do not fabricate a result.

### Adversarial stress-test of this plan (judgment kept in main transcript)

1. **Discover-then-target (biggest risk):** recon CANNOT assume an offer-without-salary or a
   multi-location offer exists. Order: `offers` dump **first** → read JSON to identify which
   rows are salary-less / multi-location → derive the `apply-*`/`history` selectors from that.
   Capture **pre- and post-scroll** so a below-the-fold variant is not missed.
2. **State collisions:** registration auto-logs-in → a naive linear flow can't reach a fresh
   login. Hence per-act `clearState`. **Exceptions:** `login-logout` shares one session across
   logout→login; `persistence` issues **no** clearState between seed and restart (else the
   datastore is gone → false BUG).
3. **Offline + clearState:** the account is local; `login-logout`/`history` must **register
   first** within the act (clearState wipes any prior account) — there is no network/remote acct.
4. **Transient race:** `extendedWaitUntil: visible` minimizes but cannot zero the appear→capture
   gap; the **screenshot is canonical**, backed by an `assertVisible` regex confirming the text
   appeared. Lifetime proven by the `visible → notVisible` transition, not a guessed duration.
5. **Error position:** objective evidence = `bounds.y` (error node vs input node) from the
   parked hierarchy, corroborated by the screenshot — never eyeballed alone.
6. **Mechanism risk:** the `STAGE` `when:`/inline-`commands` combo is doc-verified but unproven
   here → **build step 1 is a 2-branch smoke test** before authoring the full flow.
7. **Screens a linear flow skips:** no-salary offer, multi-location offer, no-salary *history*
   entry, logout state, picker popup, distinct auto-login state, persistence-after-restart —
   all explicitly enumerated as STAGES above.

### Verify-in-build refinements (human, post-approval)

1. **Multi-location picker is probably MODAL, not transient.** Confirm in build whether it
   **blocks until a location is chosen**. If it blocks → it is parkable → `maestro hierarchy`
   it (exact location strings = stronger evidence than a screenshot alone), not screenshot-only.
   If it auto-dismisses → treat as transient (screenshot via `extendedWaitUntil`).
2. **Snackbar selector comes from the `offers` dump** — use the chosen offer's **title token**
   as the `extendedWaitUntil: visible` selector. A **timeout is EVIDENCE** (candidate: snackbar
   missing the title), **not** a recon failure — capture it as such, do not let it abort framing.
3. **Apply path per spec (partial R2 leaked: "from the tile, without entering details").** From
   the `offers` dump confirm whether **"Apply" sits on the list tile** (R2 path) or **only in
   the detail screen**. Capture the path the **spec** dictates; if Apply is only in detail, that
   gap is itself a candidate. (Full R1–R5 still required before framing candidates.)

## Build sequence (KROK 3 — only after acceptance + R1–R5 pasted; plan mode OFF)

> If I reach SPEC MATRIX `expected per spec` and R1–R5 are NOT pasted → **STOP and ask**.
> Never invent requirements.

1. **KROK 1 — SPEC MATRIX migration (was blocked by plan mode):** create
   `docs/SPEC-MATRIX.md` as the filled deliverable — main table R1–R5 (cols: area/req, flow
   `TBD-P2`, expected per spec [from R1–R5], actual [from recon], **verdict EMPTY**) + anti-cases
   kept as **format placeholder only** (no content — human's call). In `CLAUDE.md` replace the
   SPEC MATRIX skeleton with one line — *"SPEC MATRIX = binding ground-truth, lives in
   docs/SPEC-MATRIX.md"* — and **keep** the Self-Validation protocol. Commit:
   `refactor(spec): move SPEC MATRIX skeleton to docs/SPEC-MATRIX.md as single source`.
2. **Mechanism smoke test:** tiny 2-branch `recon.yaml`, confirm `-e STAGE=` parking +
   `maestro hierarchy --compact` dump land correctly (paths under repo). Paste real output.
3. **Build recon.yaml incrementally:** author `locale` + `offers`, run, dump → **read the real
   labels/ids** → derive and author the deeper acts (apply/register/history) from the dumps.
   recon.yaml lives in repo root, committed.
4. **Run all STAGES**, collect screenshots → `docs/recon/screens/`, hierarchies →
   `docs/recon/hierarchy/`. Paste real `maestro`/`adb` output for each.
5. **Fill `docs/SPEC-MATRIX.md` `actual`** with exact hierarchy strings / screenshot filenames.
6. **List bug candidates:** each = `R#` + expected-per-spec + actual + **artifact** (hierarchy
   string or screenshot filename), tagged **"candidate — human adjudicates on screenshot"**.
   **No verdicts.**

**Self-validation on every material drop (same turn):** (a) which R, on which exact
string/screenshot the `actual` stands; (b) no R without `actual` where recon reached it, no
candidate without an artifact; (c) paste REAL output, not predicted; (d) mark uncertainty,
name any skipped step. Prove-don't-declare: no artifact = no claim.

## What this session does NOT do

- No flows/subflows (P2). `recon.yaml` is the only YAML and is not in the final suite.
- No BUG/PASS verdicts. No anti-case content. No invented requirements.
- No push. No app install (already present). 

## Verification

- KROK 0 re-checks already pass (emulator up, package present, maestro/adb resolve via the env
  block). Build step 2 smoke-tests the STAGE-parking mechanism with real output before scaling.
- Each material drop pastes real `maestro hierarchy` / `takeScreenshot` artifacts; SPEC MATRIX
  `actual` cells and every candidate quote a committed file. Hygiene check: `git status` shows
  no `buggy.apk`/PDF.

## End-of-session cycle (KROK-mapped)

SPEC-MATRIX (actual filled where recon reached, candidates w/ artifacts, verdict empty) →
CLAUDE.md (Current Status P1→P2, coverage, Session Log entry) → PROGRESS.md (check P1, note
drift) → hygiene (no APK/PDF) → `/export` → `docs/ai-history/2026-06-13-p1-recon.txt` →
**archive plan**: copy this file → repo `.claude/plans/p1-recon.md` → `docs/ai-history/p1-recon.md`
→ update `docs/ai-history/README.md` index → local commits (conventional, no squash:
`refactor(spec)`, `feat(recon)`, `docs(recon)`, `docs(p1)`) → **push is the human's call** →
handoff + P2 effort recommendation.
