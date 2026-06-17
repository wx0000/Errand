# Buggy — Maestro E2E test suite

End-to-end [Maestro](https://maestro.mobile.dev/) test suite for the Android app **Buggy**
(`com.example.buggy`), a Flutter release build. The suite asserts the behaviour the **task
specification** requires across five requirement areas (R1–R5): the offers list, applying from the
list, application history, registration/validation, and login/logout.

## 1. Overview — the cardinal rule

**Every test asserts the SPECIFICATION, never the app's observed behaviour.** Buggy ships with
intentional defects versus the spec, so a flow that fails **because the app violates the spec is
producing the correct result — that red is a finding, not a test bug.** An assertion is never weakened
to match buggy output (that would rubber-stamp the defect). The human issues the BUG/PASS verdict from
the screenshot + hierarchy evidence the suite captures.

Practical consequence: **the healthy state of this suite is 4 passed / 4 failed.** Four flows are
*expected* to fail — they are the four findings — so `maestro test .` exits non-zero **by design**.

| | |
|---|---|
| **Passing (spec satisfied)** | `03-apply` (R2), `06-registration-validation` (R4), `07-registration-autologin` (R4), `08-login-logout` (R5) |
| **Red = findings (app violates spec)** | `01-offers-locations` ([BUG-A](docs/BUGS.md#bug-a--multiple-locations-without-a-separator-r1)), `02-offers-salary` ([BUG-C](docs/BUGS.md#bug-c--object-shaped-salary-leaks-as-raw-json-r1--r3)), `04-history-no-salary` ([BUG-B](docs/BUGS.md#bug-b--missing-salary-renders-a-placeholder-instead-of-omitting-the-element-r1--r3)), `05-history-salary-format` ([BUG-C](docs/BUGS.md#bug-c--object-shaped-salary-leaks-as-raw-json-r1--r3)) |

Each red maps 1:1 to a human-adjudicated defect with two-layer evidence (recon + suite red-run) in
[BUGS.md](docs/BUGS.md). The requirement→flow→verdict ground-truth is in
[`docs/SPEC-MATRIX.md`](docs/SPEC-MATRIX.md); the coverage matrix + a ~2-minute manual check in
[`docs/TEST_PLAN.md`](docs/TEST_PLAN.md).

## 2. What it tests — and what it deliberately does not

**Covered (R1–R5):** offers-list tile rendering (locations, salary); apply-from-list with the
location picker and confirmation snackbar; application history; registration + field validation +
auto-login; logout and re-login.

**Consciously out of scope:**
- **Restart persistence** — the spec is silent on surviving an app restart. The app *does* persist the
  session and history, but asserting that would test app behaviour, not a requirement (it is an
  anti-case). Recorded in SPEC-MATRIX, never asserted.
- **Network behaviour** — Buggy has **no `INTERNET` permission** and runs fully offline; there are no
  tests that assume connectivity.
- **The BUG-B "list" manifestation** has no dedicated assertion flow (it is covered transitively) — see
  [§6 Known limitations](#6-known-limitations).

## 3. How to run (primary path — macOS)

The reviewer environment equals the dev environment: **macOS on Apple Silicon (M2), an Android ARM64
emulator driven over `adb`.** The versions below are the ones this suite was developed and verified
against on this machine:

| Tool | Version used |
|---|---|
| macOS | 26.5.1 (build 25F80), Apple Silicon |
| JDK | OpenJDK **21.0.11** LTS (Temurin) — `java -version` |
| Maestro | **2.6.1** — `maestro --version` |
| Android emulator ABI | **`arm64-v8a`** — `adb shell getprop ro.product.cpu.abi` |

### Steps

1. **Clone** this repo and `cd` into it.
2. **Start an Android ARM64 emulator** and confirm it is attached:
   ```sh
   adb devices            # the emulator must appear as "device" (not "offline"/empty)
   adb shell getprop ro.product.cpu.abi   # must print: arm64-v8a
   ```
3. **Install the app.** Use the **same `buggy.apk` supplied with the task** — it is **not in this repo**
   (company-owned IP, git-ignored):
   ```sh
   adb install buggy.apk
   adb shell pm list packages | grep buggy   # expect: package:com.example.buggy
   ```
4. **Run the suite** from the repo root (≈6–8 min):
   ```sh
   maestro test .         # config.yaml scopes discovery to flows/*.yaml → all 8 flows
   ```
   Expect **4 passed / 4 failed** and a **non-zero exit code — this is the correct result** (the four
   failures are the findings). See [§7 Example output](#7-example-output).

### Troubleshooting — ABI / device

- **`maestro test` finds no device / hangs on "Waiting for flows".** `adb devices` is empty or shows
  `offline` → start/cold-boot the emulator, then retry.
- **Wrong ABI.** Buggy is `arm64-v8a` only. On an M2 the emulator system image **must be ARM64**; an
  `x86_64` image will not install/run the APK. Recreate the AVD from an `arm64-v8a` system image.
- **App not installed.** If flows fail immediately at the login screen, confirm
  `adb shell pm list packages | grep buggy` returns the package; re-`adb install` if not.
- **The 4 expected reds are not failures of the harness** — they are findings. A *different* split
  (e.g. a green flow going red) is what warrants investigation; start with the troubleshooting above
  and the [TEST_PLAN](docs/TEST_PLAN.md) before suspecting the app.

## 4. Architecture

```
config.yaml            # Maestro workspace config — scopes `maestro test .` to flows/*.yaml only
flows/                 # the suite — 8 numbered flows, each = one requirement scenario
  01-offers-locations.yaml      05-history-salary-format.yaml
  02-offers-salary.yaml         06-registration-validation.yaml
  03-apply.yaml                 07-registration-autologin.yaml
  04-history-no-salary.yaml     08-login-logout.yaml
subflows/              # reusable fragments, invoked via `runFlow:` — never run standalone
  register-user.yaml  login.yaml  logout.yaml  open-profile.yaml  apply-to-offer.yaml
recon.yaml             # P1 proof-of-method recon flow (STAGE-parameterized); NOT part of the suite
docs/                  # SPEC-MATRIX.md, BUGS.md, TEST_PLAN.md, recon/ + runs/ evidence, ai-history/
```

- **Flows** are the suite. Each is self-contained: it sets up state (`clearState`, register a unique
  user), drives to the spec-defined moment, *proves the mechanic in green* (e.g. the list loaded, a
  clean tile renders), then makes the **spec assertion**. For the four findings, that last assertion is
  the red.
- **Subflows** are single-responsibility helpers reused across flows. Each asserts only its own
  readiness gate and **never asserts an outcome** — the calling flow owns the verdict. `register-user`
  (params `EMAIL`/`PASSWORD`/`CONFIRM`), `apply-to-offer` (param `OFFER` = company anchor), `login`,
  `logout`, `open-profile`.

### Selector conventions (the load-bearing ones)

The full canonical list is in [`CLAUDE.md`](CLAUDE.md) ("Maestro / selector conventions") and the
per-screen map is in the comments of [`recon.yaml`](recon.yaml). The essentials:

- **Selectors are regex anchored to the FULL node text (DOTALL).** Flutter exposes text as a
  `"label\nvalue"` shape, so a bare substring does not match — wrap tokens as `.*token.*` or use the
  exact full string.
- **Tap the clickable `Button`, not the wrapper `View`** (the wrapper is often `clickable=false`).
- **Pass params only via the CLI `-e` / `runFlow` `env:` — declare NO `env:` defaults.** In Maestro
  2.6.1 a flow/subflow `env` default *overrides* the value passed in, silently blanking every field.
- **`hideKeyboard` between form fields.** Gboard has a transitional state that absorbs a tap fired
  mid-transition; dismissing the keyboard between fields makes each tap land on a clean screen.
- **A snackbar is transient** (absent from the a11y hierarchy) → caught in-flow with
  `extendedWaitUntil` on its token + a screenshot; the screenshot is the evidence.

## 5. Design decisions & trade-offs

- **Why Maestro.** Declarative YAML flows, fast bring-up, and it drives the app fully **black-box**
  against a **release** APK over `adb` — no app source, no instrumentation, no rebuild. That fits the
  brief exactly (we have an APK and a spec, not the source).
- **Rejected Appium.** Heavier — a WebDriver server + client, language bindings, and capability/driver
  boilerplate — for no benefit here; the assertions we need are straightforward UI presence/position
  checks Maestro expresses directly.
- **Rejected Patrol.** Patrol's strength is integration testing *from inside* the Flutter app, which
  needs the app source and a test harness build. We only have a release APK → black-box is the only
  option, and Maestro covers it.
- **One flow per finding (isolation).** Maestro aborts a flow on its first failed assertion. If two
  findings shared a flow, the first red would mask the second. So R1's two findings live in `01`/`02`
  and R3's two in `04`/`05`, each isolated, so every defect surfaces independently.
- **Subflows assert no outcome.** A registration helper that asserted "success" could not be reused by
  the validation flow that expects an error. Keeping subflows outcome-neutral makes them composable;
  the caller asserts success or the specific error.

## 6. Known limitations

- **BUG-B on the offers list has no dedicated assertion flow (documented gap, by design).** BUG-B (a
  placeholder shown for an absent salary) appears on two surfaces: the **list** (`null`) and **history**
  (`—`). History is asserted directly by `04-history-no-salary` (red). The **list** manifestation is
  *not* given its own red assertion: flows `01`/`02` already fail earlier on BUG-A / BUG-C, and Maestro
  aborts on the first failed assertion, so a list-BUG-B assertion added to them would never be reached
  (masked). The root defect is still proven — red in `04`, and locked as an anti-case in
  [`SPEC-MATRIX.md`](docs/SPEC-MATRIX.md) and a defect in [`BUGS.md`](docs/BUGS.md) — so it is covered
  transitively rather than by a standalone list flow.
- **Requires a live ARM64 emulator + a supplied APK.** The suite cannot run headless/offline without an
  emulator, and the APK is intentionally not committed (IP).
- **Emulator-only.** Verified on the Android ARM64 emulator; not exercised on a physical device.

## 7. Example output

Real output from `maestro test .` (one of three stability runs; identical split on all three):

```
Waiting for flows to complete...
[Failed] 05-history-salary-format (59s) (Assertion is false: ".*Applied job salary: 10000-15000.*" is visible)
[Passed] 03-apply (39s)
[Failed] 02-offers-salary (48s) (Assertion is false: ".*Salary: 10000-15000.*" is visible)
[Failed] 01-offers-locations (48s) (Assertion is false: ".*Warsaw, Gdansk.*" is visible)
[Passed] 06-registration-validation (1m 19s)
[Passed] 08-login-logout (49s)
[Passed] 07-registration-autologin (28s)
[Failed] 04-history-no-salary (56s) (Assertion is false: ".*Applied job salary.*" is not visible)

4/8 Flows Failed
```

The four `[Failed]` flows are the findings (BUG-A, BUG-B, BUG-C ×2); each asserts the **spec** form
(`Warsaw, Gdansk`, `Salary: 10000-15000`, salary line absent), so the failure is the app diverging
from the spec.

## How this was built (AI-assisted)

Built across two channels: Claude Desktop for strategy, adversarial review, and decisions; Claude Code
for execution. Every phase was designed in plan mode — the plan reviewed and revised before any build —
with hard stop-gates where a human adjudicates each finding, the stability verdict, and the clean-clone
check (nothing is auto-ruled). The full process record lives in `docs/ai-history/`: per-session
transcripts, the archived plans, and `WORKING-WITH-AI.md` — the judgment moments where the human caught
or corrected the model. Kept so the workflow is verifiable, not just its output.

## 8. Future work (considered and deliberately deferred)

- **CI integration** — no hosted ARM64 Android emulator is readily available, the app is offline, and
  the APK cannot be committed, so a hosted pipeline would need a self-hosted ARM runner + an
  out-of-band APK. Out of scope for this deliverable.
- **JUnit / machine-readable report export** — useful for dashboards; not needed for a human-reviewed
  suite of this size.
- **Selective-run tags** (`tags:`) — handy at larger scale; the 8-flow suite runs end-to-end quickly.
- **iOS coverage** — the brief is Android-only.
- **Revisit Appium/Patrol** — only if source-level or cross-framework access becomes available.
- **Performance / load assertions** and **visual (screenshot-diff) regression** — beyond the
  functional-spec scope asserted here.
