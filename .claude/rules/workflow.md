# Integration Workflow

> Load this file when starting a full or partial HTML → ACF WP integration.

---

## Step-by-step order (always follow exactly)

### Step 1 — Audit source files

Read only what you need:
- `pages/*.html` — scan `<main>` in each file for section HTML comment markers
- `components/` — list all `.html` files; classify each as a **section file** or a **structural component**

**Classifying `components/` files:**
- `header.html`, `footer.html`, `nav.html`, or any file whose name clearly indicates a site-wide structural element → structural component (handled separately, see "Header, footer, and nav integration" below)
- All other `.html` files (e.g. `hero.html`, `services.html`, `cta.html`) → section source files; treat each file as one section, named after the filename in `snake_case`

Identify sections from both sources. Convert comment labels and filenames to `snake_case` layout names (e.g. `hero`, `trust_strip`).

**Report before proceeding:**
```
Sections detected: hero, narrative, services, cta, ...
Structural components: header, footer, nav
```

---

### Step 2 — Determine scope

- List files in `wp-theme/template-parts/flexible/` — each existing `[name].php` = already done
- Remaining sections = pending work

**Report:**
```
Implemented: hero, services
Pending: narrative, cta, testimonials
```

---

### Step 3 — Process each section (one at a time)

**Before processing** — scan all pending sections as a group. Identify any structurally identical pairs (same fields, same layout, differing only in styling/colour). For each pair, use ONE layout with a `Select` modifier field. Do not create separate layouts for visual variants.

For each pending section:

**3a. Read the section HTML**
- If the section came from `pages/` — locate it in the relevant file by its HTML comment marker inside `<main>`
- If the section came from `components/[name].html` — read that file directly

Understand the full structure before writing anything.

**3b. Identify ACF fields**
List every hardcoded value that must become a field:
- Text nodes → Textarea or WYSIWYG (see `acf-fields.md`)
- Images → Image field (`return_format: "id"`)
- Links / buttons → Link field (`return_format: "array"`)
- Repeating rows of cards / items → Repeater
- Section that is a lightbox gallery → Gallery field (not Repeater)

**3c. Write the PHP template part**
File: `wp-theme/template-parts/flexible/[layout-name].php`
Follow all patterns in `php-patterns.md`.

**3d. Write the JSON layout block**

Before writing:
- Check if `wp-theme/acf-json/group_flexible_content.json` exists.
  - **NOT EXISTS** → create the file using the master structure from `acf-json.md`, placing the new layout inside `"layouts": { }`.
  - **EXISTS** → read the file, add the new layout object to `"layouts": { }`, do NOT touch any other part of the file.

Follow all key naming rules in `acf-json.md`.

**3e. Verify before confirming**

Check:
- No duplicate `"key"` values introduced in the JSON
- No forbidden patterns: no `nl2br()`, no `aria-label` on `<section>`, no `@package`, no HTML comments in template body, no `__()` / `_e()`
- All textarea fields have `"new_lines": ""`
- `section_id` is the first sub_field in the layout

**Confirm:** "Section `[name]` done — verification passed. Moving to next."

---

### Step 4 — Final audit

After all sections are processed, run `/audit-json`. Fix every reported issue before announcing the integration complete.

---

## ACF Flexible Content loop (reference — already in `flexible.php`)

```php
<?php if (have_rows('content')): ?>
  <?php while (have_rows('content')): the_row(); ?>
    <?php get_template_part('template-parts/flexible/' . get_row_layout()); ?>
  <?php endwhile; ?>
<?php endif; ?>
```

The field name `content` must match the `"name"` of the flexible_content field
in `group_flexible_content.json`.

---

## Header, footer, and nav integration

This is a full integration task on par with flexible section work — not an afterthought.

If `components/` contains structural files (header, footer, nav, or any site-wide element identified during Step 1):
- These are **not** flexible content sections — do not add them to `group_flexible_content.json`
- Report them during Step 1 as "structural components" separate from the section list
- Complete them as part of the same integration run

---

### Step A — PHP partials

Convert each structural component file to a PHP partial. Name the target after the source filename:

| Source (example) | Target |
|---|---|
| `components/header.html` | `wp-theme/template-parts/header.php` |
| `components/footer.html` | `wp-theme/template-parts/footer.php` |
| `components/nav.html` | Absorbed into `header.php` (or `wp-theme/template-parts/nav.php` if standalone) |

Any other structural files found during Step 1 follow the same pattern.

Include these from the theme root `header.php` / `footer.php` via `get_template_part()`.

---

### Step B — Navigation menus (walkers)

- Check `wp-theme/inc/` for an existing walker class (e.g. `class-nav-walker.php`) — use it, do not create a new one
- Register nav menus in `wp-theme/functions.php` if not already present:

```php
register_nav_menus([
    'primary' => 'Primary Navigation',
    'footer'  => 'Footer Navigation',
]);
```

- Replace static `<ul><li>` nav markup with `wp_nav_menu()` calls pointing to the existing walker
- Mobile menu toggle: leave Alpine.js directives (`x-data`, `@click`, `x-show`) as-is — no PHP changes needed

---

### Step C — Theme Settings ACF field group

Create a dedicated ACF field group for all **editable non-menu content** in the header and footer: logo, contact details, social links, copyright text, CTA buttons, etc.

**1. Register an ACF Options page** in `wp-theme/functions.php` (skip if already present):

```php
if (function_exists('acf_add_options_page')) {
    acf_add_options_page([
        'page_title' => 'Theme Settings',
        'menu_title' => 'Theme Settings',
        'menu_slug'  => 'theme-settings',
        'capability' => 'edit_posts',
        'redirect'   => false,
    ]);
}
```

**2. Create** `wp-theme/acf-json/group_theme_settings.json`.

Master structure — add only fields that exist in the actual source HTML:

```json
{
  "key": "group_theme_settings",
  "title": "Theme Settings",
  "fields": [
    {
      "key": "field_ts_logo",
      "label": "Logo",
      "name": "logo",
      "type": "image",
      "return_format": "id",
      "preview_size": "medium"
    },
    {
      "key": "field_ts_phone",
      "label": "Phone",
      "name": "phone",
      "type": "link",
      "return_format": "array"
    },
    {
      "key": "field_ts_email",
      "label": "Email",
      "name": "email",
      "type": "link",
      "return_format": "array"
    },
    {
      "key": "field_ts_socials",
      "label": "Social Links",
      "name": "socials",
      "type": "repeater",
      "layout": "block",
      "button_label": "Add social link",
      "sub_fields": [
        {
          "key": "field_ts_socials_icon",
          "label": "Icon",
          "name": "icon",
          "type": "image",
          "return_format": "id",
          "preview_size": "medium",
          "parent_repeater": "field_ts_socials"
        },
        {
          "key": "field_ts_socials_link",
          "label": "Link",
          "name": "link",
          "type": "link",
          "return_format": "array",
          "parent_repeater": "field_ts_socials"
        }
      ]
    },
    {
      "key": "field_ts_copyright",
      "label": "Copyright",
      "name": "copyright",
      "type": "textarea",
      "rows": 2,
      "new_lines": ""
    }
  ],
  "location": [
    [
      {
        "param": "options_page",
        "operator": "==",
        "value": "theme-settings"
      }
    ]
  ],
  "menu_order": 0,
  "position": "normal",
  "style": "default",
  "label_placement": "top",
  "instruction_placement": "label",
  "active": true,
  "description": ""
}
```

**Key naming for Theme Settings:**

| What | Pattern | Example |
|---|---|---|
| Field group | `group_theme_settings` | `group_theme_settings` |
| Top-level field | `field_ts_[name]` | `field_ts_logo` |
| Repeater | `field_ts_[name]` | `field_ts_socials` |
| Repeater sub-field | `field_ts_[repeater]_[name]` | `field_ts_socials_icon` |

Never reuse keys from `group_flexible_content` in `group_theme_settings`.

**3. PHP access pattern** — always pass `'option'` as the second argument:

```php
$logo      = get_field('logo', 'option');
$phone     = get_field('phone', 'option');
$socials   = get_field('socials', 'option');
$copyright = get_field('copyright', 'option');
```

Repeater loops use the same `foreach` pattern as flexible sections.

---

### Step D — Verify and report

After completing structural components, report:

```
Header: done — logo, phone, nav walker wired
Footer: done — socials, copyright, footer nav wired
Theme Settings: done — group_theme_settings.json created, options page registered
```

Validate the JSON file manually before reporting done:

```bash
python3 -m json.tool wp-theme/acf-json/group_theme_settings.json
```

---

## Non-negotiables

All hard constraints are defined in `CLAUDE.md` — do not duplicate them here.
