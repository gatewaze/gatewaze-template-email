# gatewaze-template-email

Boilerplate email theme for the gatewaze newsletters / events / calendars modules. Cloned per-newsletter on creation; rendered editions are written back to the cloned repo's `publish` branch.

This repo follows the [react-email](https://react.email) project layout, seeded with the **Barebone** demo templates. Three coexisting authoring paths:

1. **React-Email starter templates (`emails/*.tsx`)** — full-edition TSX templates ported from [react-email's Barebone demo](https://github.com/resend/react-email/tree/canary/apps/demo/emails/01-Barebone). Operators preview them locally with `pnpm email:dev` and copy any of them as the starting point for a new edition.
2. **React-email block registry (`manifest.json` → `enabled_blocks`)** — declarative manifest enabling the platform's react-email block components (Header / ContentSection / AiContentSection / Footer / Heading / Text / Button) for newsletters cloning this boilerplate. This is the path the editor's slash-command palette uses today.
3. **Mustache (`source.html`, legacy)** — single file marked up with the templates parser's WRAPPER / BLOCK / BRICK comment grammar. Any block authored here ends up in the email block library as `render_kind='mustache'`. Still fully supported for the original gatewaze authoring workflow.

## Local preview

Install once:

```bash
pnpm install
```

Run the react-email dev server:

```bash
pnpm email:dev
```

Open <http://localhost:3000> to browse the eight starter templates with hot reload. Each template's `PreviewProps` block (at the bottom of its TSX file) supplies the data passed to the component during preview — adjust them as needed.

Static assets (logo, social icons, hero imagery) live under `static/` and are served at `/static/...` by the dev server. See `static/README.md` for layout.

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
│  Reads manifest.json → `enabled_blocks` populates the editor    │
│  slash-command palette (Header / Content / Footer / …).         │
│  `starter_templates` exposes the Barebone TSX files as          │
│  "start from template" presets in the new-edition flow.         │
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

## Repo layout

```
gatewaze-template-email/
├── manifest.json            ← which platform-registry blocks the editor exposes,
│                              + `starter_templates` listing the TSX files in emails/
├── emails/                  ← React-Email starter templates (Barebone-style)
│   ├── theme.ts             ←   shared Tailwind config (colors, fontScale, mobile variant)
│   ├── theme-fonts.tsx      ←   Inter @font-face + <Font> fallback registration
│   ├── welcome.tsx          ←   each TSX file is a complete edition: <Html><Body>…</Body></Html>
│   ├── activation.tsx
│   ├── password-reset.tsx
│   ├── feature-announcement.tsx
│   ├── product-update.tsx
│   ├── subscription-confirmation.tsx
│   ├── subscription-update.tsx
│   └── text-only.tsx
├── static/                  ← logo, social icons, hero imagery served by `email dev`
├── content/                 ← Mustache content fixtures (legacy path)
├── source.html              ← Mustache wrapper + blocks (legacy path)
├── theme.json               ← Mustache theme tokens (legacy path)
├── package.json
└── tsconfig.json
```

## manifest.json

`manifest.json` is the source of truth for what the editor exposes:

- `enabled_blocks` — IDs of platform-registry blocks that should appear in the slash-command palette. The platform supplies the implementation; this file just opts in.
- `starter_templates` — full-edition TSX files in `emails/`. The platform reads this list to populate the "start from template" picker when an operator creates a new edition.
- `publish_branch` — branch name where rendered editions are committed (default `publish`).
- `publish_layout` — pathname template for `editions/<id>.html` and `editions/<id>.json`.

Per-tenant TSX components committed directly to `emails/` are surfaced today as starter templates (full editions). Phase 2 will add runtime TSX compilation so individual block-level components committed by tenants can also flow into the slash-command palette.

## Customising

### Brand the starter templates

1. Edit `emails/theme.ts` — swap the `colors` and `fontScale` values, or add new design tokens. The Tailwind plugin auto-generates `font-{step}` utility classes for every key in `fontScale`.
2. Replace the Inter fallback URLs in `emails/theme-fonts.tsx` with your brand font.
3. Drop your logo + social icons into `static/shared/` (see `static/README.md`).
4. Run `pnpm email:dev` to verify in the local preview before pushing.

### Add a new starter template

1. Create `emails/<name>.tsx` following the Barebone pattern: import `Body / Container / Tailwind / …` from `@react-email/components`, wrap the email body in `<Tailwind config={barebonesBoxedTailwindConfig}>`, and export both a named export and `export default`.
2. Add a `<Component>.PreviewProps = {…} satisfies <Props>` block at the bottom for local preview.
3. Append an entry to `manifest.json` → `starter_templates.templates`.
4. Push. Newsletters cloned from this boilerplate will see it on their next manifest fetch.

### Add a platform-registry block to the palette

The platform owns the implementation of `header / content_section / ai_section / footer / heading / text / button`. To enable a new one for newsletters cloning this boilerplate, add it to `manifest.json` → `enabled_blocks`. To author a new block that the platform doesn't ship yet, see the platform's `gatewaze-modules/modules/newsletters/admin/components/puck/email-blocks/` registry — that's where the component lives until Phase 2 lifts arbitrary tenant blocks out of the platform monorepo.

### Customise the legacy Mustache path

`source.html` is parsed by the templates module (`<!-- WRAPPER:key -->` / `<!-- BLOCK:key -->` / `<!-- BRICK:key -->` markers + `<!-- SCHEMA:{...} -->` JSON Schema). See the **Marker grammar reference** below.

## Marker grammar reference (legacy Mustache path)

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

## Compatibility

- The Barebone templates target Gmail, Outlook (modern), Apple Mail, Yahoo, ProtonMail. They use react-email's table-based primitives (`Section / Row / Column`) so Outlook on Windows renders cleanly.
- The Tailwind utilities are inlined to per-element styles by `@react-email/render` at publish time — no external stylesheet ships in the email body.
- The `mobile:` variant in `theme.ts` adds a `@media (max-width: 600px)` rule that all major email clients respect.

## Credits

Starter templates adapted from [react-email's Barebone demo](https://github.com/resend/react-email/tree/canary/apps/demo/emails/01-Barebone) (MIT). The Inter font fallback URLs and font-scale tokens are imported as-is.

## License

Apache-2.0.
