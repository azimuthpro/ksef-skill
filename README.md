# KSeF Next.js Agent Skill

An [agent skill](https://vercel.com/changelog/introducing-skills-the-open-agent-skills-ecosystem)
for building **KSeF API 2.0** integrations (Krajowy System e-Faktur — Poland's
national e-invoicing system) in **Next.js apps on Vercel**.

The official KSeF SDKs and documentation examples exist only in C# and Java.
This skill closes the gap for the TypeScript ecosystem: working `node:crypto`
implementations of every required primitive (AES-256-CBC invoice encryption,
RSA-OAEP key wrapping, KSeF-token auth, QR link signing), typed API-call
patterns, and serverless architecture guidance (no-webhook polling via Vercel
Cron, token persistence, function limits).

## What It Does

Guides AI agents (Claude Code, Cursor, v0, …) through:

- Authenticating with KSeF tokens — pure Node crypto, no XAdES at runtime
- Sending invoices in interactive and batch sessions (FA(3), encrypted)
- Polling statuses and retrieving UPO receipts
- Receiving and incrementally syncing purchase invoices (High Water Mark)
- Generating KOD I / KOD II QR verification codes and handling offline modes
- Managing KSeF tokens, certificates, and permissions
- Respecting rate limits and using the TEST environment / test-data helpers

## Install

```bash
npx skills add azimuthpro/ksef-skill
```

## Requirements

- Next.js app (App Router) deployed on Vercel — the patterns assume serverless
  functions, Vercel Cron, and a database (e.g. Neon/Supabase Postgres)
- A KSeF token for the target environment (the skill documents the one-time
  bootstrap on TEST and production)
- Env vars: `KSEF_BASE_URL`, `KSEF_KSEF_TOKEN`, `KSEF_CONTEXT_NIP` (see
  `references/architecture-and-vercel.md`)

## Security

- All examples read credentials from environment variables — no secrets in
  code blocks, ever. Agents are instructed to never log, echo, or embed
  credential values.
- Invoice XML received from KSeF is treated as untrusted third-party content:
  the skill instructs agents not to execute or interpolate it.
- The skill keeps all KSeF logic server-only (`import 'server-only'`) and
  documents encrypted-at-rest storage for tokens and session keys.

## Structure

```
├── SKILL.md                                  # Router: critical facts + reference directory
├── references/
│   ├── architecture-and-vercel.md            # Start-here: system model, DDL, crons, constraints
│   ├── crypto-and-client.md                  # node:crypto primitives + typed fetch client
│   ├── auth.md                               # Auth flows, token manager, XAdES bootstrap-once
│   ├── sending-interactive.md                # Online sessions, status codes, UPO
│   ├── sending-batch.md                      # ZIP/tar.gz pipeline, part uploads
│   ├── receiving-and-sync.md                 # Queries, exports, HWM incremental sync
│   ├── qr-codes-and-offline.md               # KOD I/II, offline24/awaryjny, tech correction
│   ├── certificates-tokens-permissions.md    # Tokens, CSR enrollment, permissions model
│   └── errors-limits-and-testing.md          # Rate limits, error codes, TEST bootstrap
└── assets/examples/                          # Runnable TypeScript (npx tsx ...)
    ├── crypto.ts            ├── ksef-client.ts
    ├── auth-ksef-token.ts   ├── send-invoice-online.ts
    ├── poll-session-status.ts └── qr-codes.ts
```

## Version History

### 1.0.1 (2026-07-15)

Correctness fixes found by re-verifying against the live OpenAPI spec:

- **KSeF number validator**: CRC-8 was computed over 33 characters (including
  the separating hyphen) instead of the specified 32, so `isValidKsefNumber`
  rejected every valid KSeF number — including the official docs' own example
- **Error code extraction**: `ksefCode` read `exceptionDetailList` at the body
  root; it is nested under `exception`. The getter always returned `undefined`,
  making the error-21470 stale-key refresh-and-retry path unreachable. Also
  reads the RFC 9457 `errors[].code` shape, and no longer mistakes a 429
  rate-limit body's HTTP status for a KSeF code
- Fixed an over-length challenge example, a checksum-invalid KSeF number
  example, and the claim that `/permissions/attachments/status` reports system
  availability (it reports attachment consent)
- Corrected CSR key length (RSA 2048 exactly, not a minimum), Owner rights
  (excludes `VatUeManage`), added `PefInvoicing` and part-upload `401`

### 1.0.0 (2026-07-06)

- Initial release covering KSeF API 2.0 (verified against the official
  CIRFMF/ksef-api documentation and OpenAPI spec, API v2, mid-2026)
- Nine reference documents + six runnable TypeScript examples
- Vercel-specific architecture guidance (cron polling, Fluid compute limits,
  Blob uploads, multi-tenancy)

## Links

- [Official KSeF API docs (CIRFMF/ksef-api)](https://github.com/CIRFMF/ksef-api)
- [KSeF API Swagger (TEST)](https://api-test.ksef.mf.gov.pl/docs/v2)
- [Official C# client](https://github.com/CIRFMF/ksef-client-csharp) · [Official Java client](https://github.com/CIRFMF/ksef-client-java)
- [KSeF at podatki.gov.pl](https://www.podatki.gov.pl/ksef/)
- [skills.sh](https://skills.sh)
