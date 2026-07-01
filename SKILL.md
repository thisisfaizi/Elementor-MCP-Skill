---
name: elementor-mcp
description: Rebuild a design (HTML mockup, screenshot, or brief) as a real Elementor page on the connected WordPress site using native Elementor widgets — pixel-perfect, SEO-optimized, and mobile-responsive. Use when the user asks to create, port, lay out, restyle, or modify Elementor pages/sections/widgets via the EMCP Tools (emcp-tools) MCP server, formerly elementor-mcp. Not for editing static HTML files directly, and not for general WordPress/PHP work.
---

# EMCP Tools — Elementor page-building driver

Orchestrates the **EMCP Tools** MCP server (server route `emcp-tools`, formerly `elementor-mcp`) to turn a source design into a real Elementor page built from **native Elementor widgets**. The server already injects every tool's name and description into context, so this file does not re-list tools — it covers what the tool list can't: the build philosophy, the call sequence, and the rules that make output pixel-perfect, semantic, and responsive.

> **v3.0.0 rename.** The server namespace changed from `elementor-mcp` to `emcp-tools`. MCP tool names are now `emcp-tools-<tool>` (e.g. `emcp-tools-list-widgets`) and the server route is `/wp-json/mcp/emcp-tools-server`. If tools appear missing after an upgrade, the client is still pointed at the old route — see **Connection notes**.
>
> **Scope.** As of v3.0.0 EMCP Tools also exposes general WordPress management (content, settings, plugins & themes, media, users, filesystem, database, performance, security) and Elementor is now optional. Those WordPress-domain tools are self-describing in the tool list — **this skill stays focused on the Elementor page-building workflow.**

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
2. **Elementor Pro:** may I use **Pro widgets**, or **free widgets only**? Check with `list-widgets` using `tier:pro` — if it returns Pro widgets, Pro is active. If the user wants Pro but it isn't active, say so and fall back.
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

1. **Build with the family the user chose in Step 1** (atomic vs classic) — and never mix the two in one layout. Atomic (4.0+) uses `add-flexbox` / `add-div-block` + `add-atomic-widget`; classic uses `add-container` + the catalog widget tools below.
2. **Discover IDs before editing existing pages:** `list-pages` / `get-page-structure` / `find-element` to get real element IDs. You need them before any update/move/remove.
3. **Classic widgets are a 3-step catalog flow — discover → inspect → act** (the 62 per-widget shortcuts like `add-heading`/`add-button` are gone as of v3.0.0; nothing is lost):
   - **Discover:** `list-widgets` (filter with `tier` / `category` / `search`) to find the `widget_type`.
   - **Inspect:** `get-widget-schema` for that type — returns curated params by default; pass `types:[…]` to batch several widgets in one call, or `full:true` for the complete raw control set. Setting keys follow Elementor's control system and are **not guessable** — guessing is the #1 cause of edits that appear to do nothing.
   - **Act:** `add-free-widget` (any free/core widget) or `add-pro-widget` (Pro/WooCommerce, only when Pro is active), passing `widget_type` + `post_id` + `parent_id` + `settings`. Catalog defaults merge in automatically. Edit with `update-widget`. Read container schemas with `get-container-schema` before adding/updating containers.
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
- Keep content as real text widgets (crawlable), never baked into images; use semantic containers and `list-dynamic-tags` (Pro) where the plugin supports them.
- v3.0.0 adds dedicated SEO tools — `audit-page-seo`, `generate-meta-tags`, `generate-schema-markup`, `extract-keywords-from-content` — but the **SEO & Accessibility family ships disabled-by-default**. If they aren't in the tool list, tell the user to enable them under **EMCP Tools → Tools** rather than assuming they're broken.

## Mobile responsive

- Elementor exposes **per-device controls** (desktop / tablet / mobile). Set typography, padding/margins, and layout direction per breakpoint — don't rely on desktop values cascading.
- Convert multi-column grids to stack on mobile (flex `wrap`, or change flex-direction to column per device); mirror the source's `@media` cutovers from Step 1.
- Prefer **fluid type** (`clamp`-style min/preferred/max) where the source uses it.
- After building, re-read structure and reason through tablet/mobile to confirm nothing overflows or stays in a desktop-only grid.

## Tool-call economy

The tool set is large (~106 free, up to ~130 with Pro + atomic; ~77–96 active by default since 39 ship disabled-by-default), but the v3.0.0 widget consolidation cut the per-turn tool-list cost ~10×:

- Prefer **`build-page`** (whole page from one declarative JSON) and **`batch-update`** over many single calls.
- Add classic widgets with `add-free-widget` / `add-pro-widget` and edit with `update-widget` — always after `list-widgets` → `get-widget-schema` (see Step 3). There are no longer per-widget shortcut tools.
- Reuse repeated sections via `save-as-template` / `apply-template`.
- Tool caps are rarely a problem now, but if a client still reports too many tools, point the user to **low-tools mode** in the **EMCP Tools → Tools** admin tab (or have them disable tool families they don't need).

## Connection notes (only when tools misbehave)

- Assume the server is already configured; **never put WP credentials in files or commands**. If tools are missing, tell the user to check `claude mcp list` / `/mcp`.
- Tools are invoked as `mcp__<server>__emcp-tools-<tool>`; the `<server>` segment is the registered client name — match on the stable `emcp-tools-<tool>` part. **On a v3.0.0 upgrade the whole toolset can vanish** because the server route moved from `/wp-json/mcp/elementor-mcp-server` to `/wp-json/mcp/emcp-tools-server` (WP-CLI `--server=emcp-tools-server`): the client must be **reconnected with the new route** — have the user regenerate the config from **EMCP Tools → Connection**. Per-tool enable/disable toggles migrate automatically.
- `MCP_PROTOCOL_VERSION=2024-11-05` is **Node-proxy-only**; `Mcp-Session-Id` resend is a **direct-HTTP** concern (proxy handles it automatically).
- Permission errors mean the connected user lacks the WP capability (e.g. `manage_options` for global settings / code snippets, `upload_files` for stock images, `publish_pages` for page creation) — not a broken tool.
- If Elementor tools are absent but WordPress tools are present, **Elementor isn't active** on the connected site (Elementor is optional as of v3.0.0). Elementor widgets/layout/templates/globals/brand-kits/SEO tools register only when Elementor is active.
- Site-wide guidance (business identity, brand voice, guardrails) can be set once under **EMCP Tools → Context** and is delivered to every connecting agent as the server's `instructions` — prefer that over repeating standing rules each prompt.
- Confirm the **target site/page** with the user when ambiguous; the connected server may point at a different host than the current project.
