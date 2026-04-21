# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

CryptoMate's public documentation site, built on Mintlify. The source of truth for every user-facing page is an MDX file plus `docs.json` for navigation, theme, and API-playground configuration. Content is deployed by the Mintlify GitHub app on push to the default branch.

## Commands

```bash
mint dev              # preview at http://localhost:3000 (run from this directory)
mint broken-links     # verify internal links resolve
mint update           # upgrade the Mintlify CLI if the dev server misbehaves
```

Install once: `npm i -g mint`.

## Where content lives

```
docs.json                 single source of truth for navigation + theme + api playground
index.mdx                 Guides → home (/)
quickstart.mdx            Guides → Getting started
authentication.mdx        Guides → Getting started (API keys)
integration/webhooks.mdx  Guides → Integration (envelope + event catalog)
integration/errors.mdx    Guides → Integration (HTTP status + error codes)
products/*.mdx            Guides → Products (treasury, virtual-wallets, cards)
api-reference/index.mdx   API reference overview anchor
api-reference/<domain>/   one anchor per product (treasury, virtual-wallets, cards, management)
  index.mdx               anchor landing — `title: "Overview"` + `sidebarTitle: "Overview"`
  <group>/<op>.mdx        one file per endpoint
logo/, favicon.ico        brand assets (logo-black.png for light, logo-white.png for dark)
snippets/                 reusable MDX snippets
```

The API reference tab uses **anchors, not groups**, so clicking a product name (Treasury / Active Management / Cards / Management) loads its `index.mdx` and scopes the sidebar to that product's endpoints.

## Naming conventions that aren't obvious

- **The product is called "Active Management"** in every user-facing label, but the URL slug (`/products/virtual-wallets`, `/api-reference/virtual-wallets/...`) was intentionally left as-is. The individual wallet entity is still called a "virtual wallet" — only change the product-level label, not entity references.
- **Overview landing pages** use `title: "Overview"` + `sidebarTitle: "Overview"` so the anchor label doesn't get duplicated in the sidebar.
- **Do not document** Payments, NFTs (Media NFT), or Individual Cards (Rain) — those products are deprecated. `portal/` Spring controllers are also internal-only; only controllers under `.../{domain}/api/controllers/` are public.

## Endpoint page structure

Every endpoint MDX follows this shape:

```mdx
---
title: "<human summary>"
description: "<one line>"
api: "<METHOD> <path>"
---

<short description + access level>

## Path parameters
<ParamField path="..." type="..." required>...</ParamField>

## Body
<ParamField body="..." type="..." required>...</ParamField>

## Response
<ResponseField name="..." type="...">...</ResponseField>

<RequestExample>
```bash cURL
curl -X <METHOD> "https://api.cryptomate.me<path>" \
  -H "X-API-KEY: $CRYPTOMATE_API_KEY" ...
```
</RequestExample>

<ResponseExample>
```json 200 OK
{ ... }
```
```json 404 Not Found
{ "code": "NOT_FOUND", "message": "..." }
```
</ResponseExample>
```

Rules baked into the existing pages — preserve them when editing:

- **Errors go in the right-side `<ResponseExample>` as additional status-code blocks**, never in a `## Errors` section in the body. Labels follow the `<status> <short label>` format (e.g. `404 Not Found`, `412 Has Balance`, `412 Gas Recovery`).
- **Error body schema is `{ "code": "...", "message": "..." }`** — not `error_code`. That field name was used by the old theneo.io docs but does not match the real Java `ErrorResponseApi` record.
- **Do not repeat generic errors** (`401` invalid key, `400` BAD_REQUEST for Spring validation, `500` SERVER_ERROR) on every endpoint — those live only in `/integration/errors`. Endpoint pages document endpoint-specific errors only.

### snake_case everywhere Jackson touches the wire

The API Layer is configured with `PropertyNamingStrategy.SNAKE_CASE`, so every JSON field CryptoMate reads or writes is snake_case. The Java models themselves are camelCase — do not copy the Java field names into the docs as-is.

**Convert to snake_case:**

- JSON keys in request bodies and response bodies (inside every ```` ```json ```` block, including the 200 body and every error block in `<ResponseExample>`).
- `<ParamField body="...">` and `<ParamField query="...">` name attributes.
- `<ResponseField name="...">` name attributes.
- Query param names in cURL URLs (`?from_date=...&to_date=...`).
- Webhook payload fields (e.g. `operation_id`, `event_type`, `company_id`).
- Backticked field references inside ParamField/ResponseField descriptions.

**Leave as-is:**

- Path variables in URL templates — `/mpc/accounts/{accountId}/wallets/{walletId}`. These are literal `@RequestMapping` placeholders in Spring, not JSON.
- Header names — `X-API-KEY`, `X-Webhook-Key`, `X-Request-Timestamp`, `X-Trace-Id`, `Idempotency-Key`, `Content-Type`.
- Enum values — `POLYGON`, `PENDING`, `SUCCESS`, `FAILED`, `TOP_UP`, `NOT_FOUND`, `VAL`, `APP_ERROR`, etc. These are already uppercase / constant-case.
- Error codes inside the `"code"` JSON value — same as above.
- Example UUID / hex / email / amount values.
- Frontmatter `api:` path templates.

Example — the Java model `Cards_CreateVirtualCardApiRequest.cardHolderName` shows up in docs as:

```mdx
<ParamField body="card_holder_name" type="string" required>
  Name of the cardholder.
</ParamField>
```

```json
{
  "card_holder_name": "John Doe",
  "approval_method": "TOP_UP"
}
```

The "Try it" playground uses `docs.json` → `api.mdx.server` (Sandbox base URL) and `api.mdx.auth` (`X-API-KEY` header). Don't duplicate that config per file.

## Architecture the content maps to

Request flow, per product:

| Product (anchor) | API Layer controller dir | Primary downstream | Notes |
|---|---|---|---|
| Treasury | `apps/api-layer-services/.../treasury/api/controllers/` | `apps/operations` (treasury) → `apps/mpc`, `apps/web3-gateway-*` | MPC wallets, on-chain transfers |
| Active Management | `.../virtualWallets/api/controllers/` | `apps/operations-active-management` → `apps/ramps` for bank ramps | Virtual + holding wallets, ramps |
| Cards | `.../cards/api/controllers/` | `apps/operations-cards` → `apps/cards` (card issuer integration) | External authorization webhook is the 1,200 ms path |
| Management | `.../management/api/controllers/` | `apps/companies-api` (most), `apps/ramps` (bank accounts), `apps/operations` (ops) | Company, clients, bank accounts, config |

Webhooks are delivered by `apps/messenger` (`WebhookService`). The `X-Webhook-Key` header carries the company's shared secret configured via `/api-reference/management/company/subscribe-webhook-key`. Card `authorization` events have a hard **1,200 ms** deadline; all other events use 10 retries with exponential backoff.

Shared exception model in `apps/java-commons` (`com.cryptomate.commons.models.exceptions`) maps directly to the status codes used in docs: `AppErrorException`→400 (`APP_ERROR`), `NotFoundException`→404 (`NOT_FOUND`), `ValidationException`→412 (`VAL`), `WarningException`→302 (`WARNING`), unhandled `Exception`→500 (`SERVER_ERROR`).

When tracing a new endpoint's possible errors: start in the API Layer controller method, follow into the service and provider, then into the downstream microservice's controller/service — every `throw new XxxException(code, msg)` along the path becomes a row in the endpoint's `<ResponseExample>`.

## Writing style (from AGENTS.md / CONTRIBUTING.md)

- Active voice, second person ("you").
- Sentence case for headings.
- Bold UI elements (e.g. Click **Settings**). Code formatting for file names, commands, paths, and identifiers.
- One idea per sentence. Lead with the user's goal.
