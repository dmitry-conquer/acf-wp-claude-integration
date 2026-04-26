# /section — Convert a Single HTML Section

## What this command does

Converts one named section from the static HTML source into:
1. A PHP flexible layout template part
2. A JSON layout block ready to merge into `group_flexible_content.json`

## Usage

```
/section [layout-name]
```

Example:
```
/section hero
/section trust_strip
/section testimonials
```

## Steps Claude will follow

1. Load `acf-fields.md` and `php-patterns.md` and `acf-json.md`
2. Locate the section in `pages/index.html` by HTML comment marker
3. List all hardcoded values → map to ACF field types
4. Output PHP: `wp-theme/template-parts/flexible/[name].php`
5. Output JSON layout block to merge into `group_flexible_content.json`
6. Verify before confirming: no `nl2br()`, no `aria-label` on `<section>`, no `@package`, no HTML comments in template body, all textarea fields have `"new_lines": ""`, `section_id` is the first sub_field

## Notes

- Layout name must be `snake_case` (e.g. `trust_strip`, not `trust-strip`)
- If the section does not exist in source HTML, report the blocker and stop
