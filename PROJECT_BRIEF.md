# EOB Guided Review Assistant — Project Brief

## One-line description
A local-first web app that walks a patient step-by-step through comparing their Explanation of Benefits (EOB) against their itemized medical bill, runs deterministic reconciliation checks on the data they enter, screens for possible No Surprises Act (NSA) issues via guided questions, and generates dispute/appeal letters.

## Product thesis
The automated version of this tool (OCR/LLM extraction of EOBs) fails if extraction accuracy is imperfect, because it produces confident-but-wrong claims about money and legal rights. This variant inverts the risk: **the patient transcribes the data with visual guidance; the tool does only deterministic math and rule checks on confirmed data.** Accuracy burden shifts from the machine to the user, where it is verifiable.

## Non-negotiable constraints
1. **Local-first, zero backend for MVP.** All documents and entered data stay in the browser (IndexedDB). No network calls in the MVP build. No analytics, no telemetry.
2. **No CPT descriptions.** CPT codes and their long/short descriptions are AMA-copyrighted. The app may store and display the *codes the user types in*, but must never ship a bundled CPT description dataset or auto-describe codes. HCPCS Level II descriptors (public) are acceptable in a later phase, clearly separated.
3. **Information, not advice.** Every findings screen and generated letter carries a persistent disclaimer: this is an organizational aid, not legal, medical, or billing advice. NSA screening output language is always "possible issue — worth asking about," never "violation."
4. **User can delete everything.** One-tap "Delete all my data" that clears IndexedDB and in-memory state, with confirmation.

## Target user & core scenario
A patient with (a) one EOB from their insurer and (b) one itemized bill from a provider for the same episode of care, who suspects the numbers don't line up. Assume low familiarity with billing terminology; define every term inline on first use (plain-language tooltips).

## User flow (wizard)

### Step 0 — Welcome & scope
- What the tool does / doesn't do. Disclaimer acknowledgment (checkbox).
- Choose scenario: "I have an EOB and a bill" (primary path) | "I only have a bill" (limited path: bill sanity checks + NSA screen only) | "I'm self-pay with a Good Faith Estimate" (GFE path).

### Step 1 — Document upload (reference only)
- User uploads/photographs EOB and bill. Files are rendered locally (PDF.js for PDFs, native img for photos) in a side-by-side viewer next to the entry forms.
- Files are never parsed, transmitted, or interpreted in MVP. They exist so the user can read from them while typing.

### Step 2 — Guided EOB entry
- Wizard walks through one section at a time with visual guidance text ("Look for a box usually labeled 'Amount billed' or 'Provider charges' — it's often the leftmost money column").
- Header fields: insurer name, claim number, service date(s), provider name, network status if shown (in-network / out-of-network / not stated).
- Per line item (repeatable form row): service date, procedure code (free text, optional), billed amount, allowed/negotiated amount, plan paid, patient responsibility, remark/reason codes (free text, optional).
- Summary fields: total patient responsibility, deductible applied, coinsurance, copay, and "amount not covered" if shown.
- Every field optional except amounts needed for the checks the user wants; the engine degrades gracefully and reports which checks were skipped for missing data.

### Step 3 — Guided bill entry
- Same pattern: provider name, statement date, account number (optional), then repeatable line items (date, code if shown, description as printed — user-typed, so no licensing issue — amount), then total charges, payments/adjustments shown, and **balance due**.

### Step 4 — Reconciliation (deterministic engine)
Run all applicable checks; each check returns `pass | flag | skipped(reason)` with a plain-language explanation and the exact numbers involved.

Checks (MVP set):
1. **Balance vs. EOB responsibility.** Bill balance due > EOB total patient responsibility → flag with the delta. This is the headline check.
2. **Line-item match.** Match bill lines to EOB lines by (code, date) then (amount, date). Report unmatched lines on either side.
3. **Duplicate lines.** Identical (code/description, date, amount) appearing more than once on the bill.
4. **Internal EOB math.** Per line: allowed − plan paid should ≈ patient responsibility components; totals should equal sum of lines. Flag arithmetic that doesn't reconcile (tolerance: $0.01 rounding).
5. **Internal bill math.** Line items − payments/adjustments should equal balance due.
6. **Already-paid check.** User enters any payments they've made; verify they're reflected in the bill.
7. **GFE delta (self-pay path).** Bill total vs. Good Faith Estimate; flag if delta ≥ $400 (patient-provider dispute resolution threshold) with a note about the PPDR process.

### Step 5 — NSA screening (guided questionnaire, not a determination)
Plain-language questions, branching:
- Was this emergency care (ER, urgent stabilization, post-stabilization)?
- Was the *facility* in-network but this specific provider possibly out-of-network (anesthesia, radiology, pathology, assistant surgeon, hospitalist)?
- Air ambulance involved?
- Did you sign a notice-and-consent waiver agreeing to out-of-network billing? (explain what that looks like)
- Insurance type gate: employer/marketplace plan vs. Medicare/Medicaid/TRICARE (NSA doesn't apply to the latter — different protections; say so and stop the NSA branch).
Output: a list of "possible NSA-related issues to raise," each with a one-paragraph explanation and what to ask the provider/insurer. Link targets (rendered as plain URLs, not fetched): CMS No Surprises Help Desk info, 1-800-985-3059.

### Step 6 — Findings report & letters
- Findings report: ordered by severity (money-delta flags first), each finding shows the numbers, what it might mean, and a suggested next step.
- Letter generator (template merge, no LLM): (a) itemized-bill request letter, (b) billing-discrepancy letter to provider citing the specific line deltas, (c) claim-review/appeal request letter to insurer, (d) NSA inquiry letter. User fills name/address merge fields; output is copyable text and a printable view.
- Export: findings + entered data as a JSON download and a printable HTML report. (No PDF generation in MVP.)

## Technical decisions (correctable, but pick these unless overridden)
- **Stack:** React + TypeScript + Vite. Tailwind for styling. PDF.js for local PDF rendering. Dexie.js over IndexedDB for persistence. No router needed beyond wizard state, but use one if it simplifies deep-linking steps.
- **Architecture:** Reconciliation engine and NSA screening logic as **pure TypeScript modules with zero DOM/React imports** (`/src/engine/*`). All money math in integer cents. This is the unit-test surface.
- **State:** Single wizard store (Zustand or reducer). Autosave to IndexedDB on every step change.
- **Accessibility:** WCAG 2.1 AA target. Full keyboard operability, labels on every input, visible focus, no color-only meaning on flags.
- **Testing:** Vitest. The engine must have exhaustive unit tests including golden-case fixtures (see TASKS.md M3). UI testing is secondary for MVP.

## Explicitly out of scope for MVP
- OCR / LLM extraction of any kind
- CPT/HCPCS descriptor lookup
- State-specific balance-billing law content
- Accounts, sync, backend, PDF export
- Price benchmarking

## Success criteria
1. A test user with a real EOB + bill can complete the flow in ≤ 20 minutes and gets at least the balance-vs-responsibility check with correct math.
2. Engine unit tests cover every check with pass, flag, and skipped fixtures; all money math is exact.
3. No network request is issued at runtime (verify with devtools; CI check if feasible).
4. Every findings/letter surface carries the disclaimer.
