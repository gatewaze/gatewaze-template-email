# Static assets

The Barebone starter templates in `../emails/` reference asset URLs under
`/static/shared/` and `/static/barebones/` (logo, social icons, hero image).
React Email's local dev server serves the contents of this directory as
`/static/...` so previews resolve.

Once a newsletter is rendered through the platform, those URLs are rewritten
to absolute CDN URLs by the publish step (the `baseUrl` constant in each
template reads `process.env.VERCEL_URL` for local dev / Vercel preview
deployments — set `EMAIL_BASE_URL` to override in production).

Drop your brand logo, social icons, and hero imagery here. The originals
live in the [react-email demo repo](https://github.com/resend/react-email/tree/canary/apps/demo/public/static)
under `apps/demo/public/static/` if you want a starting set.

Subdirectories:
- `shared/` — brand-wide assets (logo, social icons) used by every template
- `barebones/` — template-specific imagery (hero photos, illustrations)
