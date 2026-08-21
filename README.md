# CryptoMate documentation

Public documentation site for the CryptoMate API, built on [Mintlify](https://mintlify.com) and deployed by the Mintlify GitHub app on every push to `main`.

Live at **https://docs.cryptomate.me**.

## Local preview

```bash
npm i -g mint      # once
mint dev           # preview at http://localhost:3000
mint broken-links  # verify internal links resolve
```

## Layout

| Path | What it holds |
| --- | --- |
| `docs.json` | navigation, theme, API playground config, redirects |
| `index.mdx`, `authentication.mdx`, `portal-onboarding.mdx` | Guides → Getting started |
| `products/*.mdx` | commercial product guides (no API detail) |
| `integration/*.mdx` | webhooks, errors, rate limiting, API versioning |
| `api-reference/<domain>/**` | one MDX page per endpoint, grouped by product |
| `troubleshooting/*.mdx` | operational runbooks for integrators |

Page conventions (structure, naming, error documentation, English-only rule) live in [`CLAUDE.md`](CLAUDE.md). Contribution flow in [`CONTRIBUTING.md`](CONTRIBUTING.md).
