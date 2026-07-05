# CLAUDE.md — EOB Guided Review Assistant

Read PROJECT_BRIEF.md first. It is the source of truth for scope and constraints. TASKS.md defines build order. Do not skip milestones or pull future-phase features into the MVP.

## Hard guardrails (never violate)
1. **No network calls at runtime.** No fetch, no CDN-loaded data, no analytics, no fonts from remote origins. Everything bundles locally. If you believe a network call is necessary, stop and ask.
2. **No CPT descriptions anywhere.** Do not embed, generate, hardcode, or fetch CPT code descriptions. Codes are user-entered free text only. Do not "helpfully" add a lookup table.
3. **No legal determinations in copy.** NSA screening output says "possible issue," "worth asking about," "you may want to raise." Never "violation," "illegal," "you are owed." All findings/letter screens render the standing disclaimer component.
4. **Money is integer cents.** Never float arithmetic on currency. Parse user input to cents at the boundary; format for display only at the edge.
5. **Engine purity.** Nothing in `/src/engine/` imports React, DOM APIs, or the store. Pure functions in, typed results out.

## Conventions
- TypeScript strict mode. No `any` in `/src/engine/`.
- Every engine check returns `{ id, status: 'pass' | 'flag' | 'skipped', reason, details, amountsInvolved }` — one shared result type.
- Plain-language copy lives in a single `copy.ts` module per feature area, not scattered in JSX. Reading level target: 8th grade. Define jargon inline on first use.
- Component files small; wizard steps are independent routes/components sharing the store.
- Commit per task, conventional commits.

## Testing requirements
- Vitest. Run `npm test` before claiming any engine task complete.
- Every check: minimum three fixtures — pass, flag, skipped-for-missing-data. Plus rounding edge cases ($0.01 tolerance) and multi-line matching ambiguity cases.
- Golden fixtures live in `/src/engine/__fixtures__/` as JSON matching the entry schema. Fixtures use realistic but fully synthetic data — never real patient information, even as placeholder text.

## When uncertain
- Ambiguity about billing/insurance domain rules: do NOT invent rule content. Implement the mechanism, mark the rule content with `// DOMAIN-REVIEW:` comment, and surface it in your task summary for human verification.
- UI/UX ambiguity: pick the simpler option, note the decision.

## Definition of done (per milestone)
- Acceptance criteria in TASKS.md met
- `npm run build` clean, `npm test` green, no console errors in dev
- No runtime network requests (check the network tab)
- Brief summary of decisions made and any DOMAIN-REVIEW items
