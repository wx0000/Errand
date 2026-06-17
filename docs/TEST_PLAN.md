# TEST PLAN — Buggy Maestro E2E suite

Condensed, reviewer-facing view of the test strategy and coverage. The binding ground-truth is
[`SPEC-MATRIX.md`](SPEC-MATRIX.md) (requirement → flow → expected → recon `actual` → human verdict);
the adjudicated defects are in [`BUGS.md`](BUGS.md). This file paraphrases those — it is not a second
source of truth, and it contains no verbatim spec text.

## Strategy

Every test asserts the **specification**, never the app's observed behaviour. Buggy contains
intentional defects versus the spec, so a flow that goes **red because the app violates the spec is
producing the correct result — that red is a finding, not a test bug.** We never weaken an assertion
to match buggy output (doing so would rubber-stamp the defect; see Anti-cases). The human issues the
BUG/PASS verdict from the screenshot + hierarchy evidence; the suite's job is to drive the app to the
spec-defined moment and assert what the spec requires.

Consequence for reading results: **the green suite state is 4 pass / 4 red.** Four flows are expected
to fail — they are the four findings — and `maestro test .` therefore exits non-zero by design.

## Coverage matrix (R1–R5)

| Req | What the spec requires (paraphrase) | Flow(s) | Expected result | Defect |
|-----|-------------------------------------|---------|-----------------|--------|
| **R1** | Offers list tiles: title, company, location, salary. Multiple locations → **comma-separated**; salary present → range, **absent → no salary element at all** (no placeholder). | `01-offers-locations`, `02-offers-salary` | **RED = finding** | BUG-A (locations run together), BUG-C (object salary as raw JSON) |
| **R2** | Apply from the tile (no detail screen); confirmation snackbar shows **title AND applied location**; multi-location offer opens a **location picker** first, then the snackbar shows the chosen location. | `03-apply` | **PASS** | — |
| **R3** | Application history from the profile: entry = title / company / chosen location / salary range; no-salary offer → **no salary line**. | `04-history-no-salary`, `05-history-salary-format` | **RED = finding** | BUG-B (placeholder salary line shown), BUG-C (object salary as raw JSON) |
| **R4** | Registration saves the user and **auto-logs in**; validation (email format, password ≥ 8, confirm = password); error messages render **above** the field. | `06-registration-validation`, `07-registration-autologin` | **PASS** | — |
| **R5** | Log out from the profile, then log back in with the registration credentials. | `08-login-logout` | **PASS** | — |

Defect detail and per-flow red-run evidence: [`BUGS.md`](BUGS.md) (BUG-A → flow 01; BUG-B → flow 04
+ a documented list-surface gap; BUG-C → flows 02 + 05).

## Anti-cases — what the suite must NOT assert

Each is a way a test could over-read the app: either rubber-stamping a defect by asserting the broken
output as if correct, or asserting behaviour the spec never requires.

| Do NOT assert | Why it would be wrong | Req |
|---|---|---|
| Run-together locations (`WarsawGdansk`) as a correct display | Spec wants a comma-separated list; asserting the concatenation locks in BUG-A | R1 |
| A salary placeholder for a no-salary offer (`null` on the list, `—` in history) | Spec forbids any placeholder when salary is absent; asserting it rubber-stamps BUG-B | R1, R3 |
| Raw JSON salary (`{"min":10000,"max":15000}`) as the displayed value | Spec wants a formatted range like every other offer; asserting the JSON rubber-stamps BUG-C | R1, R3 |
| Snackbar with the title alone (location omitted) | Spec requires title **and** applied location; a title-only check passes on an under-spec snackbar | R2 |
| Application history surviving an app restart | Spec is silent on restart; the app does persist, but asserting it reads app behaviour, not a requirement | R3 |

## Manual verification procedure (~2 minutes of active attention)

For a reviewer who wants to confirm the suite and eyeball the findings by hand. The ~2 minutes is your
*active* attention (kick off the run, read the split, open four screenshots); the `maestro test .` run
itself takes ~6–8 minutes unattended in between.

1. **Prerequisites up** — Android ARM64 emulator running and visible to `adb devices`; `buggy.apk`
   installed (`adb install buggy.apk`). See the README "How to run" for versions.
2. **Run the suite** — from the repo root: `maestro test .` (≈6–8 min; `config.yaml` runs all 8 flows).
3. **Check the split** — expect **4 passed / 4 failed**. Passing: `03`, `06`, `07`, `08`. Failing (the
   findings): `01`, `02`, `04`, `05`. Any other split → see the README troubleshooting / SPEC-MATRIX,
   and treat an unexpected result as something to adjudicate, not to "fix" by editing an assertion.
4. **Eyeball the findings** — open the captured screenshots and confirm the defect with your own eyes:
   - `docs/runs/screens/01-offers-locations.png` — locations rendered `WarsawGdansk` (no comma) → BUG-A.
   - `docs/runs/screens/02-offers-salary.png` — salary as raw JSON `{"min":…}` on the list → BUG-C.
   - `docs/runs/screens/04-history-no-salary.png` — a salary line present for a no-salary offer → BUG-B.
   - `docs/runs/screens/05-history-salary-format.png` — raw JSON salary in history → BUG-C.
5. **Cross-reference** — each finding maps to a defect in [`BUGS.md`](BUGS.md) and a row in
   [`SPEC-MATRIX.md`](SPEC-MATRIX.md). The red assertion in each flow targets the **spec** form
   (e.g. `Warsaw, Gdansk`, `Salary: 10000-15000`), so the failure is the app diverging from the spec.
