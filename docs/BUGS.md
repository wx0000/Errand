# BUGS.md — Buggy defects vs. the specification

Defects found during P1 recon, adjudicated by the human from screenshots and hierarchy dumps.
Every flow asserts the SPEC; a red result on any of the items below is the correct result. Two
evidence layers per defect: **recon** evidence (P1 discovery) lives in `docs/recon/`, and the
**suite red-run** artifacts — the flow whose spec assertion fails *because* the app violates the
spec — live in `docs/runs/` (linked under each defect below). Both confirm the same defect.

3 root defects, 5 manifestations across 2 surfaces (list R1, history R3).

---

## BUG-A — Multiple locations without a separator (R1)
- Requirement: R1, offers list, location.
- Expected (spec): multiple locations shown as a comma-separated list.
- Actual: locations concatenated with no separator. Affects every multi-location offer: QA Automation Engineer -> WarsawGdansk; Senior QA Engineer -> WarsawBerlinLondon; Senior QA -> WarsawBerlinLondonParis.
- Steps: clearState -> open the offers list -> offer "QA Automation Engineer" (BigTech Inc, 2 locations).
- Severity: Medium. "WarsawGdansk" reads as a single city; the data is misleading.
- Evidence (recon): docs/recon/screens/02-offers-top.png, docs/recon/screens/02-offers-bottom.png. The location is a single Flutter Semantics node (not a concatenation of child nodes), so the dump equals the render; the screenshot confirms the missing comma visually.
- Evidence (suite red-run): flow `01-offers-locations` asserts the spec form `.*Warsaw, Gdansk.*` → RED (the finding). docs/runs/screens/01-offers-locations.png + docs/runs/hierarchy/01-offers-locations.txt.

## BUG-B — Missing salary renders a placeholder instead of omitting the element (R1 + R3)
- Requirement: R1 list + R3 history.
- Expected (spec): when salary is absent, the salary element is NOT shown (no placeholder); in history the salary line is likewise omitted.
- Actual: the list shows a salary element reading "null"; history shows a salary line reading "—".
- Steps: clearState -> offer "Senior QA Engineer" (TechCorp, no salary): "null" visible on the list; apply -> profile -> the "Senior QA Engineer" entry shows "—".
- Severity: Medium. "null" on the list looks like a code artifact; the spec explicitly forbids a placeholder.
- Evidence (recon): list docs/recon/screens/02-offers-top.png + docs/recon/hierarchy/02-offers-list-top.txt; history docs/recon/screens/09-history.png + docs/recon/hierarchy/09-history.txt.
- Evidence (suite red-run, history surface): flow `04-history-no-salary` asserts the salary line is absent (`assertNotVisible ".*Applied job salary.*"`) → RED (the finding). docs/runs/screens/04-history-no-salary.png + docs/runs/hierarchy/04-history-no-salary.txt.
- List-surface coverage gap (documented, by design): the list manifestation (`null`) has **no dedicated red flow**. Flows `01`/`02` already fail on BUG-A / BUG-C, and Maestro aborts on the first failed assertion, so a list-BUG-B assertion bolted onto them would be masked (never reached). The root defect is proven red in `04` (history) and locked as a SPEC-MATRIX anti-case — covered transitively. See README §6 (Known limitations).

## BUG-C — Object-shaped salary leaks as raw JSON (R1 + R3)
- Requirement: R1 list + R3 history.
- Expected (spec): when an offer has a salary, the range is shown in the same format as the other offers (e.g. 10000-15000 PLN).
- Actual: raw JSON {"min": 10000, "max": 15000}, identical on the list and in history.
- Steps: clearState -> offer "Senior QA" (QualityAssurance Ltd) on the list; apply (location London) -> profile -> the "Senior QA" entry.
- Severity: High. Raw JSON in the UI is an obvious formatting defect. Offers whose salary is a string ("5000-8000 PLN") render correctly on both surfaces -> the defect is limited to object-shaped salaries.
- Evidence (recon): list docs/recon/hierarchy/02-offers-list-bottom.txt + docs/recon/screens/02-offers-bottom.png; history docs/recon/hierarchy/09-history.txt + docs/recon/screens/09-history.png.
- Evidence (suite red-run, list): flow `02-offers-salary` asserts the spec range `.*Salary: 10000-15000.*` → RED (the finding). docs/runs/screens/02-offers-salary.png + docs/runs/hierarchy/02-offers-salary.txt.
- Evidence (suite red-run, history): flow `05-history-salary-format` asserts the spec range `.*Applied job salary: 10000-15000.*` → RED (the finding). docs/runs/screens/05-history-salary-format.png + docs/runs/hierarchy/05-history-salary-format.txt.

---

Repro note: "Senior QA" (QualityAssurance Ltd, JSON salary, 4 locations) and "Senior QA Engineer" (TechCorp, no salary, 3 locations) are TWO different offers. Do not confuse them in the steps.
