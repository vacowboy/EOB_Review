# TASKS.md — Build Order

Work milestones in order. Each is a reviewable checkpoint; stop and summarize after each.

## M1 — Scaffold & shell
- Vite + React + TS strict + Tailwind + Vitest + Dexie
- Wizard shell: step navigation, progress indicator, persistent disclaimer footer component
- IndexedDB store with autosave on step change; "Delete all my data" with confirmation, verified to clear storage
- **Accept:** app runs, steps navigate, data survives refresh, delete works, zero network requests

## M2 — Entry schema & forms
- TypeScript schema for EOB entry, bill entry, payments, GFE (self-pay path) — cents-based money types
- Step 0 (welcome/scope/path selection), Step 2 (guided EOB entry), Step 3 (guided bill entry) with repeatable line-item rows, inline plain-language guidance copy, per-field "where to find this" hints
- Graceful optionality: every field skippable; schema tracks entered-vs-skipped
- **Accept:** full manual entry of a synthetic EOB + bill persists correctly; keyboard-only completion possible

## M3 — Reconciliation engine (the core)
- `/src/engine/reconcile.ts`: checks 1–7 from PROJECT_BRIEF Step 4, pure functions, shared result type
- Line-matching algorithm: exact (code+date) → (amount+date) → report unmatched; document ambiguity handling
- Fixture suite: pass/flag/skipped per check, rounding cases, ambiguous-match cases
- **Accept:** all tests green; engine has zero React/DOM imports; a deliberately mismatched fixture produces the correct delta in cents

## M4 — Document viewer
- Step 1 upload: PDF.js render + image render, side-by-side with entry forms (responsive: stacked on mobile)
- Files held as blobs in IndexedDB, never parsed or transmitted
- **Accept:** a multi-page PDF and a phone photo both render; entry forms usable alongside; delete-all removes blobs

## M5 — NSA screening questionnaire
- Branching question flow per PROJECT_BRIEF Step 5, including the Medicare/Medicaid/TRICARE gate
- All rule-dependent copy tagged `// DOMAIN-REVIEW:` for human verification
- Output: list of possible-issue cards with plain-language explanations and CMS help-desk contact info as static text
- **Accept:** every branch reachable and tested; no determination language anywhere in copy

## M6 — Findings report & letters
- Findings screen ordered by severity, numbers shown in context, suggested next steps
- Letter templates (a)–(d) with merge fields, copyable + printable view
- Export: JSON download of entered data + findings; printable HTML report
- **Accept:** end-to-end run from Step 0 to letters with a synthetic case; letters contain correct merged deltas

## M7 — Polish & a11y pass
- WCAG 2.1 AA sweep: labels, focus order, contrast, no color-only flag states
- Copy review pass for reading level and jargon definitions
- README with local-run instructions and the privacy model explained
- **Accept:** axe-core (dev-only dependency) clean on all steps; build size sane; final no-network verification

## Later phases (do not build now)
- P2: optional LLM-assisted pre-fill (every field user-confirmed) — this is the bridge to the automated variant and the source of a labeled eval set
- P2: HCPCS Level II public descriptors (separated from CPT)
- P3: state balance-billing content, price benchmarking, PDF export
