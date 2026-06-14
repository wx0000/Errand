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
> - **The `verdict` column carries the human's review ONLY — Claude records the human's PASS/BUG, never authors one.** (Empty through recon; filled at P1 adjudication from the human's ruling.)

## Main table — requirements R1–R5

> `expected per spec` below is a **paraphrase** of the R1–R5 the human provided (no verbatim
> spec text, no IP). `flow` is assigned in P2; `actual` + bug candidates come from recon.

| Req | Flow | Expected per spec | Actual (from recon) | Verdict (PASS/BUG) |
|-----|------|-------------------|---------------------|--------------------|
| R1  | _TBD — P2_ | **Offers list — tile data.** Each tile shows title, company, location, salary range. Location: single = one value; multiple = comma-separated. Salary: present → range shown; **absent → the salary element does not exist at all** (no placeholder, no "no data"). | "Job Offers" list, **5 offers**. Multi-location values are concatenated **without separators**: `WarsawBerlinLondon`, `WarsawGdansk`, `WarsawBerlinLondonParis`. No-salary offer renders a salary element: `Salary: not specified` + value `null` (Senior QA Engineer). One salary rendered as raw JSON: `{"min": 10000, "max": 15000}` (Senior QA / QualityAssurance). Normal ranges fine (`5000-8000 PLN`). Single-location fine (`Krakow`). Evidence: `docs/recon/hierarchy/02-offers-list-top.txt` + `02-offers-list-bottom.txt`; `docs/recon/screens/02-offers-top.png` + `02-offers-bottom.png`. → candidates **C1–C3** | **BUG** — C1→BUG-A, C2→BUG-B (list), C3→BUG-C (list) |
| R2  | _TBD — P2_ | **Apply from the list.** Apply directly via an "Apply" button on the tile, **without opening the offer detail**. Confirmation **snackbar contains the offer title AND the applied location** (both, not title alone). Multi-location offer: "Apply" **first opens a location-picker popup**; after choosing, the application is saved for the chosen location and the snackbar shows title + that chosen location. | "Apply" sits **on each tile** (no detail screen). Single-loc (Junior Flutter Developer): snackbar `Applied to Junior Flutter Developer - Krakow` (title + location). Multi-loc (QA Automation Engineer): Apply opens a **modal popup** `Select Location for QA Automation Engineer` with separate options `Warsaw` / `Gdansk`; choosing Gdansk → snackbar `Applied to QA Automation Engineer - Gdansk` (title + chosen location). Picker splits locations correctly (contrast C1). Evidence: `docs/recon/screens/03-apply-single-snackbar.png`, `04a-apply-multi-picker.png`, `04b-apply-multi-snackbar.png`; `docs/recon/hierarchy/04-apply-multi-picker.txt`. No candidate observed in recon (human adjudicates). | **PASS** |
| R3  | _TBD — P2_ | **Application history.** Reached from the profile (header profile icon); lists the offers applied to. Entry = **4 lines**: (1) title, (2) company, (3) chosen location, (4) salary range. No-salary offer → **no salary line** (consistent with the list). | Profile button (header) → "Profile" with "User Information" (email) + "Application History". Entry = title / company / **chosen location** / salary; chosen location persists correctly (`Warsaw`, `Krakow`, `London`). Salaried entry fine (`5000-8000 PLN`). **No-salary entry STILL shows a salary line** rendered `—` (a11y `Applied job salary: not specified`) → **C4**. **Raw-JSON salary leaks into history**: `{"min": 10000, "max": 15000}` (Senior QA / QualityAssurance) → **C5**. Evidence: `docs/recon/hierarchy/09-history.txt`; `docs/recon/screens/09-history.png`. | **BUG** — C4→BUG-B (history), C5→BUG-C (history) |
| R4  | _TBD — P2_ | **Registration.** Successful registration **saves the user and auto-logs them in**. Validation: email basic format; password **≥ 8 characters**; confirm-password **must equal** the password. Validation error messages rendered **ABOVE the relevant field (not below)**. | **Auto-login confirmed** (valid register → lands on "Job Offers", no separate login). All 3 validations present; each error rendered **ABOVE** its field (objective: error top-y < input top-y, + visual): email `Please enter a valid email address` (936 < 1002); password `Password must be at least 8 characters` (1135 < 1201); confirm `Passwords do not match` (1335 < 1401). Evidence: `docs/recon/hierarchy/06-reg-email-error.txt`, `07-reg-pwd-error.txt`, `08-reg-confirm-error.txt`; `docs/recon/screens/06/07/08-reg-*-error.png` (06 shows the red error above the Email box). **No candidate observed** (human adjudicates). | **PASS** |
| R5  | _TBD — P2_ | **Login / logout.** User can **log out from the profile**, and can **log back in** with the email + password created at registration. | **Logout** button in profile → returns to the login screen (`Welcome Back`) ✓. **Re-login** with the registration creds (`recon.user@example.com` / `ReconPass123`) → back to `Job Offers` ✓ (same session). Evidence: `docs/recon/screens/10-after-logout.png` (login), `11-after-relogin.png` (offers); `docs/recon/hierarchy/11-after-relogin.txt`. **No candidate observed** (human adjudicates). | **PASS** |

## Bug candidates (recon output — NO verdicts)

> Filled during recon. Each candidate = `R#` + expected-per-spec + actual + **artifact**
> (hierarchy string or screenshot filename), tagged **"candidate — human adjudicates on
> screenshot."** Claude does not rule BUG/PASS.

**C1 — R1 — multi-location values rendered without separators.** → **BUG-A**
- Expected per spec: multiple locations shown as **comma-separated** values.
- Actual: concatenated with no separator — `Locations: WarsawBerlinLondon`, `WarsawGdansk`, `WarsawBerlinLondonParis`.
- Artifact: `docs/recon/hierarchy/02-offers-list-top.txt` + `02-offers-list-bottom.txt`; `docs/recon/screens/02-offers-top.png` + `02-offers-bottom.png`.
- _candidate — human adjudicates on screenshot._

**C2 — R1 — no-salary offer still renders a salary element (placeholder + literal `null`).** → **BUG-B** (list)
- Expected per spec: when salary is absent, the **salary element does not exist at all** (no placeholder, no "no data").
- Actual: Senior QA Engineer (TechCorp) shows `Salary: not specified` with value `null`.
- Artifact: `docs/recon/hierarchy/02-offers-list-top.txt` (`Salary: not specified\nnull`); `docs/recon/screens/02-offers-top.png`.
- _candidate — human adjudicates on screenshot._

**C3 — R1 — salary rendered as raw JSON instead of a formatted range.** → **BUG-C** (list)
- Expected per spec: salary shown as a range (e.g. `10000-15000 PLN`).
- Actual: Senior QA (QualityAssurance Ltd) shows `Salary: {"min": 10000, "max": 15000}`.
- Artifact: `docs/recon/hierarchy/02-offers-list-bottom.txt`; `docs/recon/screens/02-offers-bottom.png`.
- _candidate — human adjudicates on screenshot._

**C4 — R3 — no-salary offer still renders a salary line in history.** → **BUG-B** (history)
- Expected per spec: no-salary offer → **no salary line** in the history entry.
- Actual: Senior QA Engineer entry shows a 4th line `—` (a11y `Applied job salary: not specified`).
- Artifact: `docs/recon/hierarchy/09-history.txt`; `docs/recon/screens/09-history.png`.
- _candidate — human adjudicates on screenshot._

**C5 — R3 — salary rendered as raw JSON in history (same root as C3, history surface).** → **BUG-C** (history)
- Expected per spec: salary shown as a formatted range.
- Actual: Senior QA (QualityAssurance Ltd) history entry shows `Applied job salary: {"min": 10000, "max": 15000}`.
- Artifact: `docs/recon/hierarchy/09-history.txt`; `docs/recon/screens/09-history.png`.
- _candidate — human adjudicates on screenshot._

> Note: no-salary rendering differs by surface — list shows `null` (C2), history shows `—` (C4);
> raw-JSON salary appears on both list (C3) and history (C5). Human adjudicated: **3 root
> defects** (BUG-A / BUG-B / BUG-C), **5 manifestations** across 2 surfaces (list R1, history R3)
> — see `docs/BUGS.md`.

_Recon complete — all R1–R5 acts run. Total candidates: **C1–C5** (all under R1 / R3). R2, R4, R5 clean (no candidate observed)._

## Recon notes (non-requirement observations — not candidates)

- **Locale.** UI renders in **English**; emulator system locale `ro.product.locale = en-US`
  (`persist.sys.locale` unset, `system_locales` null). This answers "which language to assert"
  (= EN). Whether the UI dynamically **follows** a non-English system locale is **not required by
  R1–R5** and was not tested (would need a system-locale switch + reboot). Evidence:
  `docs/recon/hierarchy/00-login.txt`; `docs/recon/screens/00-login.png`.
- **Persistence across app restart (exploration; spec is SILENT).** After `stopApp` + relaunch
  **without** clearState, the **session persists** (app reopens on "Job Offers", still logged in)
  and the **application history persists** (the seeded Junior Flutter entry is in the profile).
  **R3 does not require surviving a restart → recorded actual, NOT a candidate** (and it would not
  be a candidate if it had been lost either). No recon flow **asserts** post-restart persistence —
  asserting it would be an anti-case (over-interpreting the app). Evidence:
  `docs/recon/screens/12-after-restart.png`, `13-after-restart-profile.png`;
  `docs/recon/hierarchy/13-after-restart-profile.txt`.

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
