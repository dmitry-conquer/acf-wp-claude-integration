# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This ruleset converts static HTML layouts (sections) into WordPress theme files with ACF Pro Flexible Content field groups.

> Read the relevant rule file **only when you need it**. Do not pre-load all rules upfront — load on demand to preserve context tokens.

---

## Workflow overview

1. **Audit** all `.html` files in `pages/` and `components/` — identify flexible sections and structural files (header, footer, nav)
2. **Scope** — check `wp-theme/template-parts/flexible/` for already-done sections
3. **Scan for structural duplicates** — before processing, identify identical sections; use ONE layout + Select modifier instead of two separate layouts
4. **Process** each pending section one at a time: PHP part → JSON block → verify
5. **Structural components** (any header/footer/nav files in `components/`) — integrate via WP walkers in `wp-theme/inc/`, not as flexible sections

Full step details: `.claude/rules/workflow.md`

---

## Project file structure (target WP theme)

| What | Path |
|---|---|
| Flexible page template | `wp-theme/templates/flexible.php` |
| Flexible section parts | `wp-theme/template-parts/flexible/[layout-name].php` |
| ACF JSON (single file) | `wp-theme/acf-json/group_flexible_content.json` |

Static source HTML lives in:
- `pages/*.html` — flexible sections inside `<main>`, identified by HTML comment markers
- `components/*.html` — may contain additional section files (one file = one section) and/or structural files (header, footer, nav)

---

## Slash commands

| Command | What it does |
|---|---|
| `/integrate` | Full run — audit, process all pending sections, audit-json at the end |
| `/section [name]` | Convert one named section (e.g. `/section hero`) |
| `/audit-json` | Validate `group_flexible_content.json` for all known issues |

---

## Rule files (load on demand)

| Task | Rule file |
|---|---|
| Full integration workflow & step order | `.claude/rules/workflow.md` |
| ACF field type decisions | `.claude/rules/acf-fields.md` |
| ACF JSON structure & key naming | `.claude/rules/acf-json.md` |
| PHP template patterns & escaping | `.claude/rules/php-patterns.md` |

---

## JSON validation (quick check)

```bash
python3 -m json.tool wp-theme/acf-json/group_flexible_content.json
```

Run this whenever the JSON file is touched. The `/audit-json` command checks semantic issues; this catches syntax errors.

---

## Hard constraints (always active — no exceptions)

- **NEVER** hardcode text, images, or links that were dynamic in the static HTML
- **NEVER** use `the_field()` inside flexible layout parts — always `get_sub_field()`
- **NEVER** create a separate `layout_*.json` file — all layouts go into `group_flexible_content.json`
- **NEVER** include `"required": 0` or `"required": 1` on any ACF field
- **NEVER** convert decorative inline `<svg>` icons into ACF Image fields — keep them as static markup in the PHP template
- **NEVER** write `<img>` tags manually — always use `wp_get_attachment_image()`
- **Always** escape output: `esc_html()`, `esc_url()`, `esc_attr()`, `wp_kses_post()` — **exception:** embed/iframe fields output raw (see `acf-fields.md`)
- **Always** use `return_format: "id"` for Image fields
- **Always** use `return_format: "array"` for Link fields
- **Always** use `foreach ($items as $i => $item)` for Repeater loops — never `have_rows`
- **Always** add a `section_id` Text field to every layout for editor-controlled HTML `id` attribute
- **Always** use `'full'` image size in `wp_get_attachment_image()`
- **NEVER** create two separate layouts for structurally identical sections — use ONE layout with a `Select` modifier field (e.g. `theme`, `variant`) instead
- **NEVER** use `__()`, `_e()`, or any text-domain translation functions — the theme is single-language
- **NEVER** add `instructions` or `placeholder` to ACF field JSON unless explicitly asked
- **NEVER** add `aria-label` to a `<section>` element unless it was present in the source HTML
- **NEVER** wrap textarea output in `nl2br()` — output `esc_html()` only, no line-break formatting
- **NEVER** use CSS `background-image: url()` for images — always render with `wp_get_attachment_image()`
- **NEVER** copy HTML comments from source markup into flexible section PHP templates — only the top docblock comment is allowed
- **NEVER** use `@package` or text domain in PHP docblocks
- **ALWAYS** use ACF Gallery field (not Repeater) when a section functions as a lightbox gallery
- **ALWAYS** integrate any structural `components/*.html` files (header, footer, nav) into WordPress — create PHP partials, wire navigation via existing walkers in `wp-theme/inc/`, and create a separate `group_theme_settings` ACF field group (options page) for all editable header/footer content — full steps in `workflow.md` — these are not flexible sections
- **NEVER** add `aria-hidden` to `<section>` wrappers or background image containers — `aria-hidden` is only for inline SVGs and explicitly decorative `<img>` elements
- **ALWAYS** add `"new_lines": ""` to every `textarea` field in ACF JSON — without it ACF injects `<br>` tags into output
- **ALWAYS** add `"parent_repeater": "field_[repeater_key]"` to every sub-field inside a Repeater — without it ACF cannot associate the field on JSON sync
- **NEVER** use hyphens in a layout `"name"` — underscores only (`trust_strip` not `trust-strip`) — hyphens break `get_template_part()` file lookup
- **ALWAYS** make the object key in `"layouts": {}` exactly match the `"key"` value inside that layout object — mismatch silently breaks ACF JSON sync
- **After completing each section or batch**, perform 1–2 verification checks before announcing completion — do not stop until checks pass or the user says to

---

## Token-efficient working mode

When performing a full integration, work **one section at a time**:
1. Announce which section you are working on
2. Output the PHP template part
3. Output the JSON layout block (to merge into `group_flexible_content.json`)
4. Verify and confirm before moving to the next section (see `workflow.md` step 3e)

Do not load all source HTML into context at once unless explicitly asked.
