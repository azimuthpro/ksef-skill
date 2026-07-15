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
- **No high-entropy literals** — example hashes/signatures from the official
  docs must be replaced with structural placeholders like
  `{invoiceHashBase64Url}`; entropy-based secret scanners (Snyk Agent Scan
  W008) flag realistic Base64 values as leaked secrets. Scan a clean export
  (`git archive HEAD | tar -x -C <tmpdir>`), not the working tree — `.git`
  objects are high-entropy noise.
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

- Type-check examples (`node_modules/` here is gitignored; a bare `npx tsc`
  resolves to an unrelated decoy package, and without `@types/node` you get
  ~39 spurious `TS2591` errors):

  ```bash
  cd assets/examples
  npm i -D typescript @types/node
  npx tsc --noEmit --strict --target es2022 --module nodenext \
    --moduleResolution nodenext --skipLibCheck --types node *.ts
  ```
- Sanity-run pure-crypto helpers with `npx tsx` (AES round-trip, hash
  helpers) — they have no network dependencies.
- Verify claims against the live OpenAPI rather than the narrative docs, which
  lag it (`curl -s https://api-test.ksef.mf.gov.pl/docs/v2/openapi.json`).
  Response-shape bugs (wrong nesting, wrong field level) type-check clean and
  are invisible to secret/SAST scanners — exercise the parse with a realistic
  body instead.
- Sanity-run pure-crypto helpers with `npx tsx` (AES round-trip, hash
  helpers) — they have no network dependencies.
- Live testing requires TEST-environment credentials; the bootstrap
  walkthrough is in `references/errors-limits-and-testing.md`.
