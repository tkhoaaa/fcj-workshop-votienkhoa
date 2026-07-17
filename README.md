# fcj-workshop-votienkhoa

This repository is a Hugo-based bilingual documentation site for an internship report and workshop guide. It is primarily a content repo with a small set of local theme overrides on top of the bundled `hugo-theme-learn` theme.

## Architecture Map

- `config.toml`
  Site-wide Hugo configuration: base URL, theme selection, output formats, security settings for remote shortcode fetches, language configuration, and theme parameters.
- `content/`
  Source pages for the site. This is the main place to edit report sections, workshop steps, and translated content.
- `layouts/`
  Local Hugo overrides that take precedence over theme templates. This repo currently overrides a few partials and shortcodes.
- `static/`
  Site-owned assets copied directly into the generated site, including images, fonts, and custom CSS.
- `themes/hugo-theme-learn/`
  The bundled base theme. Most rendering behavior still comes from here unless a file is overridden in the local `layouts/` directory.
- `public/`
  Generated output from Hugo. Treat this as build output, not as the source of truth for long-term edits.

## Build And Content Flow

Hugo reads `config.toml`, loads Markdown from `content/`, applies templates from local `layouts/` first, falls back to `themes/hugo-theme-learn/` when no local override exists, copies files from `static/`, and writes the final rendered site into `public/`.

In practical terms:

1. Edit page content in `content/`.
2. Edit template behavior in `layouts/`.
3. Edit styling and images in `static/`.
4. Regenerate the site so changes appear in `public/`.

The existing `public/index.html` confirms that this flow is already active locally: it contains content from `content/_index.md`, layout output from the theme and overrides, and assets referenced from `/css`, `/js`, and `/images`.

## Current Customizations

The repo customizes the base theme in a few targeted places:

- `layouts/partials/logo.html`
  Replaces the default theme logo with an inline AWS SVG logo shown in the sidebar header.
- `layouts/partials/menu-footer.html`
  Adds custom footer content in the sidebar, including a dynamically rendered "Last Updated" date and a team link.
- `layouts/partials/custom-footer.html`
  Injects a Google Analytics tracking snippet into the page footer.
- `layouts/shortcodes/tabs.html`
- `layouts/shortcodes/tab.html`
  Local copies of the tabs shortcodes are present, so tab behavior can be customized without editing the bundled theme.
- `static/css/theme-workshop.css`
  Defines the active workshop color theme and a number of UI overrides on top of the base theme styles.

Other behavior still comes from `themes/hugo-theme-learn/`, including the main page skeleton, menu rendering, search UI, bundled JavaScript, and default shortcode implementations that are not overridden locally.

## Content Model

The site is organized as a report with numbered sections:

- `1-Worklog`
- `2-Proposal`
- `3-BlogsPosted`
- `4-EventParticipated`
- `5-Workshop`
- `6-Self-evaluation`
- `7-Feedback`

Hugo section landing pages use `_index.md`. Vietnamese variants use `_index.vi.md`. This pattern appears at the homepage and throughout the section hierarchy, which is how the repo supports both English and Vietnamese content.

The workshop content is nested more deeply than the rest of the report. For example:

- `content/5-Workshop/_index.md` defines the workshop landing page.
- `content/5-Workshop/5.3-S3-vpc/_index.md` defines a subsection.
- `content/5-Workshop/5.3-S3-vpc/5.3.1-create-gwe/_index.md` defines a specific step page.

Images referenced by Markdown pages are stored under `static/images/...`, which Hugo publishes as `/images/...` in the final site.

## Where To Edit Common Things

- Homepage/student profile content: `content/_index.md` and `content/_index.vi.md`
- Section text and page order: the relevant files under `content/`
- Sidebar logo: `layouts/partials/logo.html`
- Sidebar footer content: `layouts/partials/menu-footer.html`
- Analytics snippet: `layouts/partials/custom-footer.html`
- Tab shortcode behavior: `layouts/shortcodes/tabs.html` and `layouts/shortcodes/tab.html`
- Site theme colors and workshop styling: `static/css/theme-workshop.css`
- Site-wide language, theme, and output settings: `config.toml`

## Source Vs Generated Output

Avoid making long-term manual edits directly in `public/`. Files there are generated artifacts and can be replaced the next time Hugo builds the site.

For durable changes, prefer:

- `content/` for text and page structure
- `layouts/` for template and rendering changes
- `static/` for assets and custom CSS
- `config.toml` for site configuration

Use `public/` mainly to inspect the rendered result and verify what Hugo produced.

## Validation Notes

This readout was cross-checked against:

- `config.toml`
- Local overrides in `layouts/partials/` and `layouts/shortcodes/`
- Representative content pages in `content/`
- Generated output in `public/index.html`

One environment note: `git status` could not be used in the current shell because Git reported this repo as a dubious ownership directory, so working tree validation through Git is currently blocked until the repo is marked as a safe directory in the local Git config.
