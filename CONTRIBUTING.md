# Contribute to the CryptoMate documentation

## Flow

1. Branch off updated `main` (`docs/EN-1234-short-slug`).
2. Edit the MDX pages, and `docs.json` if the navigation changes.
3. Run `mint dev` and `mint broken-links` before pushing.
4. Open a PR against `main`. Mintlify deploys automatically once merged.

Never add commits to a branch whose PR is already merged — branch again off updated `main`.

## Writing style

- Active voice, second person ("you").
- Sentence case for headings.
- One idea per sentence. Lead with the reader's goal.
- Bold for UI elements (Click **Settings**); code formatting for file names, commands, paths, and identifiers.
- All content in **English**, including titles, descriptions, table headers, and callouts.

## Content boundaries

- `products/*.mdx` are commercial guides: no endpoint paths, `curl` examples, JSON bodies, or auth details.
- `api-reference/**` carries every technical detail, one page per endpoint.
- Only the public `/api` surface is documented. `/portal` endpoints are internal — never document them.
- Generic errors (`401` invalid key, `400` validation, `500`) live only in `/integration/errors`; endpoint pages document endpoint-specific errors.

Full conventions: [`CLAUDE.md`](CLAUDE.md).
