# CLAUDE.md

Guidance for working on this repository.

## Project Overview

This is the **KSeF Next.js Agent Skill** — a publishable agent skill
(installed via `npx skills add azimuthpro/ksef-skill`) that teaches AI agents
to build KSeF API 2.0 integrations (Poland's national e-invoicing system) in
Next.js apps on Vercel.

It is a **documentation artifact**, not an application: there is no build, no
tests, no dependencies. The deliverables are `SKILL.md`, `references/*.md`,
and runnable examples in `assets/examples/*.ts`.

## Structure & conventions

- `SKILL.md` — thin router: frontmatter (name/description drive skill
  activation), critical facts, and a Reference Directory table mapping trigger
  keywords to `references/*.md`. Keep it under ~150 lines; details belong in
  references.
- `references/*.md` — one focused document per scenario, each ending with a
  `## Sources` section linking the official docs at
  https://github.com/CIRFMF/ksef-api. Keep each under ~500 lines.
- `assets/examples/*.ts` — standalone scripts mirroring the code shown in
  references (`npx tsx <script>`). Only `node:crypto` + `fetch`; third-party
  deps (`qrcode`, `fflate`, `@peculiar/x509`) appear only as clearly marked
  optional snippets.
- **No secrets anywhere** — examples read env vars (`KSEF_KSEF_TOKEN` etc.)
  and never print token values. Preserve this property in every edit.
- English prose; Polish domain terms (UPO, NIP, sesja wsadowa) kept with a
  translation on first use.

## Maintenance rules

- **Source of truth**: the official docs and OpenAPI spec at
  https://github.com/CIRFMF/ksef-api (Swagger:
  https://api-test.ksef.mf.gov.pl/docs/v2). Any change to endpoint shapes,
  status codes, limits, or crypto parameters must be verified there first —
  never from memory. Check `api-changelog.md` in that repo for what changed.
- Volatile facts (statutory dates, rate-limit numbers, Vercel platform
  limits) are deliberately phrased as snapshots with pointers to the live
  source (`GET /rate-limits`, podatki.gov.pl, Vercel docs). Keep that framing.
- When editing reference files, keep SKILL.md's Reference Directory table and
  trigger keywords in sync.
- Bump `metadata.version` in SKILL.md and add a README Version History entry
  for every released change.
- Internal links are relative (`references/...`, `assets/...`) — verify they
  resolve after renames. The skill must stay self-contained (no links to
  local paths outside the repo).

## Verifying changes

- Type-check examples: `cd assets/examples && npx tsc --noEmit --strict --target es2022 --module nodenext --skipLibCheck *.ts` (ambient shims for optional deps may be needed).
- Sanity-run pure-crypto helpers with `npx tsx` (AES round-trip, hash
  helpers) — they have no network dependencies.
- Live testing requires TEST-environment credentials; the bootstrap
  walkthrough is in `references/errors-limits-and-testing.md`.
