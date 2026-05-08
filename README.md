# gatewaze-template-email

Boilerplate email theme for the gatewaze newsletters / events / calendars modules. **Two coexisting authoring paths:**

1. **Mustache (`source.html`)** — single file marked up with the templates parser's WRAPPER / BLOCK / BRICK comment grammar. Any block authored here ends up in the email block library as `render_kind='mustache'`. The original gatewaze authoring path; still fully supported.
2. **React-email registry (`manifest.json`)** — declarative manifest enabling the platform's react-email block components (Heading / Text / Button / …) for newsletters cloning this boilerplate. See **Newsletter clone + publish workflow** below.

When a newsletter is created with `NEWSLETTERS_BOILERPLATE_URL` pointing at this repo, the platform clones it as a per-newsletter internal Git repo. Edition publishes write rendered HTML back to the cloned repo's `publish` branch (`editions/<edition_id>.html`); operators can graduate the internal repo to an external GitHub remote at any time.

## Newsletter clone + publish workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  Operator creates a newsletter "Daily Digest"                   │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  Platform clones gatewaze-template-email                        │
│  → bare repo at /var/gatewaze/git/newsletter/<id>.git           │
│  → row in gatewaze_internal_repos with host_kind='newsletter'   │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  Reads manifest.json → enabled_blocks → editor's palette shows  │
│  Heading + Text + Button (the platform-provided react-email     │
│  registry components, filtered by manifest)                     │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  Editor edits "May 8" edition; clicks Publish                   │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  Server renders the edition via `await render(<EditionEmail/>)` │
│  Commits to `publish` branch:                                   │
│    editions/<edition_id>.html  ← inlined-CSS email-safe HTML    │
│    editions/<edition_id>.json  ← raw block tree                 │
└─────────────────────────────────────────────────────────────────┘
```

## manifest.json

`manifest.json` is the source of truth for which platform-registry blocks the newsletter palette exposes. Operators may extend it with per-block overrides (label, defaults). Per-tenant TSX components committed directly to this repo are Phase 2.

## How gatewaze ingests source.html (legacy Mustache path)

When an operator connects this repo as a templates source for a newsletter / event / calendar collection (`templates_libraries.theme_kind = 'email'`):

1. The templates parser walks `source.html`.
2. Extracts every `<!-- WRAPPER:key -->` / `<!-- BLOCK:key -->` / `<!-- BRICK:key -->` block + the adjacent `<!-- SCHEMA:{...} -->` JSON Schema.
3. Inserts/updates rows in `templates_wrappers`, `templates_block_defs`, `templates_brick_defs` (per the `templates_apply_source` RPC).
4. Editors can then compose newsletter editions / event pages / calendar pages from the available block palette.

## What's in source.html

| Marker | Purpose |
|--------|---------|
| `WRAPPER:default` | The email shell — `<!doctype html>` through `</body>`. The `{{>page_body}}` slot is where the composed block list lands. |
| `BLOCK:heading` | Single heading; `level` ∈ `{h1, h2, h3}`. |
| `BLOCK:paragraph` | Rich-text body. The `format: 'html'` field hint tells the editor to use the rich-text editor. |
| `BLOCK:cta_button` | Primary CTA link with brand styling. |
| `BLOCK:event_list` | A repeating list of events (`has_bricks=true`); each item is a `BRICK:event_list_item`. |
| `BRICK:event_list_item` | Single event row inside `event_list`. |
| `BLOCK:divider` | Horizontal rule for visual separation. |
| `BLOCK:image` | Centered image with optional caption + link. |

## Marker grammar reference

```
<!-- WRAPPER:key | name=... -->
<!-- META:slot_key -->...<!-- /META:slot_key -->
<!-- SCHEMA:{ JSON Schema } -->
...HTML...
<!-- /WRAPPER:key -->

<!-- BLOCK:key | name=... | description=... | has_bricks=... | sort_order=N -->
<!-- SCHEMA:{ JSON Schema } -->
<!-- DATA_SOURCE:{ ...optional adapter config... } -->
...HTML...
<!-- /BLOCK:key -->

<!-- BRICK:key | name=... | sort_order=N -->     (nested inside a BLOCK with has_bricks=true)
<!-- SCHEMA:{ JSON Schema } -->
...HTML...
<!-- /BRICK:key -->
```

Field interpolation uses Mustache:
- `{{field}}` — escaped string from the editor's content.
- `{{{field}}}` — unescaped HTML (use with care; only for `format: 'html'` fields).
- `{{>page_body}}` — wrapper slot for the composed block list.
- `{{#bricks}}{{> brick_key}}{{/bricks}}` — repeating bricks inside a block.
- `{{#field}}…{{/field}}` — conditional rendering when field is truthy.

See `gatewaze-modules/modules/templates/lib/parser/markers.ts` for the parser implementation.

## Customising

- **Brand styling** — edit the `<style>` block inside `WRAPPER:default`. Email clients vary in CSS support; keep it inline-friendly.
- **Add a block** — append a new `<!-- BLOCK:key -->...<!-- /BLOCK:key -->` section with its own `<!-- SCHEMA:... -->`. Push to your repo. Run **Apply theme** in the gatewaze admin's Source tab.
- **Add a brick** — only allowed inside a `BLOCK` declared with `has_bricks=true`. Mirror the `event_list` / `event_list_item` pattern.
- **Test rendering** — use any Mustache renderer locally (e.g. the `mustache` npm package) with sample content to preview before committing.

## Compatibility

- Renders cleanly in Gmail, Outlook (modern), Apple Mail, Yahoo, ProtonMail.
- Outlook on Windows is the most picky — keep table layouts simple, avoid CSS Grid / Flexbox at the top level.

## License

Apache-2.0.
