---
name: elementor-mcp
description: Rebuild a design (HTML mockup, screenshot, or brief) as a real Elementor page on the connected WordPress site using native Elementor widgets — pixel-perfect, SEO-optimized, and mobile-responsive. Use when the user asks to create, port, lay out, restyle, or modify Elementor pages/sections/widgets via the elementor-mcp server. Not for editing static HTML files directly, and not for general WordPress/PHP work.
---

# Elementor MCP driver

Orchestrates the **elementor-mcp** server to turn a source design into a real Elementor page built from **native Elementor widgets**. The server already injects every tool's name and description into context, so this file does not re-list tools — it covers what the tool list can't: the build philosophy, the call sequence, and the rules that make output pixel-perfect, semantic, and responsive.

## The one hard rule: native widgets only — never the HTML widget

**Do not use the HTML widget, the Shortcode widget, or `add-custom-js`/custom-HTML injection to lay out or style sections.** Pasting markup in defeats the entire point: it isn't editable in the Elementor UI, breaks responsive controls, harms SEO, and isn't portable. Every visual element must map to a **real, registered Elementor widget or container** with its settings configured through the schema.

Map source markup to the nearest native widget:

| Source element | Native Elementor widget |
| --- | --- |
| Section / row / column wrapper | Container (legacy) or **Flexbox / Div-block** (atomic 4.0+) |
| `<h1>`–`<h6>` | Heading (set the HTML tag for correct hierarchy) / atomic Heading |
| Paragraph / rich text | Text Editor / atomic Paragraph |
| Button, CTA link | Button / atomic Button |
| `<img>` | Image / atomic Image (always set alt text) |
| Inline SVG / icon | Icon widget (upload via `upload-svg-icon` first if custom) / atomic SVG |
| Bullet / feature list | Icon List |
| Accordion / FAQ | Accordion or Toggle |
| Tabbed content | Tabs |
| Testimonial / quote | Testimonial |
| Star rating | Star Rating |
| Video embed | Video / atomic Video / YouTube |
| Spacing gap | Spacer / Divider (or container padding/gap) |
| Nav menu, forms, pricing | Native/Pro widget if present; otherwise compose from containers + the widgets above |

If no single widget fits, **compose** the look from containers + base widgets — never fall back to raw HTML.

## Step 1 — Confirm build scope with the user (ask before building)

**Always ask before you start building.** Different widget families produce very different pages, so don't assume. First run `detect-elementor-version` and `list-widgets` to see what's actually installed (so you only offer real options), then ask the user — use the AskUserQuestion tool when interactive:

1. **Widget family:** **Atomic widgets (Elementor v4)** vs **classic/simple Elementor widgets (v3-style)**. Only offer atomic if `detect-elementor-version` reports 4.0+. These are different tool families and must not be mixed in one layout.
2. **Elementor Pro:** may I use **Pro widgets**, or **free widgets only**? Only offer Pro if `list-widgets` shows Pro widgets registered (Pro active). If the user wants Pro but it isn't active, say so and fall back.
3. **Theme Builder:** is this a normal **page**, or a **Theme Builder part** (header / footer / single / archive) or a **popup**? Theme parts use `create-theme-template` / `set-template-conditions` / `create-popup` and are Pro-gated.

Reconcile the answers with what's installed before proceeding — never promise a family or Pro feature the connected site doesn't have. Once confirmed, carry the chosen widget family through the whole build.

## Step 2 — Derive the design system from the source

Do this **before** building anything. Analyze the provided HTML/CSS (or screenshot/brief) and extract a reusable system rather than eyeballing each section:

- **Color palette + roles:** pull every distinct color (read `:root` custom properties / CSS vars if present). Identify *semantic roles* — primary, surface/background, ink/text, muted, hairline/border, and especially the **action/CTA color** (often used sparingly). Preserve those roles; don't flatten them into one accent.
- **Typography:** font families (display vs body vs mono), the weight set, and the **type scale** — capture exact `font-size`, `line-height`, `letter-spacing` per level, including any `clamp()` fluid sizing.
- **Spacing & shape:** the spacing scale (section padding, gaps), border-radius scale, and shadow definitions.
- **Breakpoints:** read the `@media` queries to learn the source's exact responsive cutovers and what reflows at each (grids collapsing to one column, nav → hamburger, font shrink).
- **Structure:** the section order and each section's layout intent (asymmetric hero, even grid, etc.).

Then **apply the system to Elementor globals** with `update-global-colors` and `update-global-typography` once, and reference globals when building so restyles are one-touch. The system is derived per project — there is no built-in brand.

## Step 3 — Build, in this order

Calls fail *silently* (no error, wrong result) when run out of order:

1. **Build with the family the user chose in Step 1** (atomic vs classic) — and never mix the two in one layout. Atomic (4.0+) uses Flexbox/Div-block + atomic Heading/Paragraph/Button/Image/SVG; classic uses Container + the legacy widgets.
2. **Discover IDs before editing existing pages:** `list-pages` / `get-page-structure` / `find-element` to get real element IDs. You need them before any update/move/remove.
3. **Read the schema before every add/update:** `get-widget-schema` / `get-container-schema`. Setting keys follow Elementor's control system and are **not guessable** — guessing is the #1 cause of edits that appear to do nothing.
4. **Verify by re-reading.** Changes save immediately to post meta (no separate publish). Confirm with `get-page-structure` instead of assuming.

## Pixel-perfect

- Transfer **exact values**, not approximations: hex colors, px sizes, font weights, line-height, letter-spacing, radii, and shadow specs from Step 1 into widget/container settings.
- Match **layout geometry**: container `max-width`, section padding, flex `gap`, column ratios, and alignment — these carry most of the "feel."
- Reproduce **backgrounds and effects** with Elementor's native background controls (solid, gradient, image, overlay) — recreate gradients/dotted textures via background settings, **not** an HTML/CSS-injection widget.
- Use global colors/typography so values stay consistent and a single change propagates.

## SEO

Building with native widgets is itself the biggest SEO win (semantic, crawlable markup vs an HTML-widget blob). Also:

- **One `<h1>` per page**, then a logical heading order — set each Heading widget's **HTML tag** explicitly; don't pick tags by visual size.
- **Alt text on every Image**; descriptive link/button text (no "click here").
- Set **page title and meta description** via `update-page-settings`.
- Use semantic containers and `list-dynamic-tags`/structured-data tools where the plugin supports them; keep content as real text widgets (crawlable), never baked into images.

## Mobile responsive

- Elementor exposes **per-device controls** (desktop / tablet / mobile). Set typography, padding/margins, and layout direction per breakpoint — don't rely on desktop values cascading.
- Convert multi-column grids to stack on mobile (flex `wrap`, or change flex-direction to column per device); mirror the source's `@media` cutovers from Step 1.
- Prefer **fluid type** (`clamp`-style min/preferred/max) where the source uses it.
- After building, re-read structure and reason through tablet/mobile to confirm nothing overflows or stays in a desktop-only grid.

## Tool-call economy

The tool set is large (~61 free, up to ~118 with Pro) and clients can hit tool caps:

- Prefer **`build-page`** (whole page from one declarative JSON) and **`batch-update`** over many single calls.
- Use universal `add-widget` / `update-widget` when you know the type + schema; reach for convenience shortcuts (`add-heading`, `add-button`…) only when they save real setup.
- Reuse repeated sections via `save-as-template` / `apply-template`.
- If the client reports too many tools, point the user to **low-tools mode** (~50 essentials) in the EMCP Tools admin menu.

## Connection notes (only when tools misbehave)

- Assume the server is already configured; **never put WP credentials in files or commands**. If tools are missing, tell the user to check `claude mcp list` / `/mcp`.
- Tools are invoked as `mcp__<server>__elementor-mcp-<tool>`; the `<server>` segment is the registered name (e.g. `elementor-mcp`) — match on the stable `elementor-mcp-<tool>` part if it was added under a different name.
- `MCP_PROTOCOL_VERSION=2024-11-05` is **Node-proxy-only**; `Mcp-Session-Id` resend is a **direct-HTTP** concern (proxy handles it automatically).
- Permission errors mean the connected user lacks the WP capability (e.g. `manage_options` for global settings / code snippets, `upload_files` for stock images, `publish_pages` for page creation) — not a broken tool.
- Confirm the **target site/page** with the user when ambiguous; the connected server may point at a different host than the current project.
