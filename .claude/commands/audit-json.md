# /audit-json — Validate ACF JSON Field Group

## What this command does

Reads `wp-theme/acf-json/group_flexible_content.json` and checks for:

1. **Duplicate keys** — any `"key"` value used more than once
2. **Layout name format** — all layout `"name"` values must be `snake_case` (no hyphens)
3. **Layout key consistency** — the object key in `"layouts"` must match the `"key"` inside each layout
4. **Missing `parent_repeater`** — every repeater sub-field must declare `"parent_repeater"`
5. **Forbidden `"required"` field** — report any field that includes `"required": 0` or `"required": 1`
6. **Image/Gallery return_format** — every `image` and `gallery` field must have `"return_format": "id"`
7. **Link return_format** — every `link` field must have `"return_format": "array"`
8. **Textarea new_lines** — every `textarea` field must have `"new_lines": ""`
9. **WYSIWYG properties** — every `wysiwyg` field must have `"toolbar": "full"` and `"media_upload": 0`
10. **True/False properties** — every `true_false` field must have `"default_value": 1` and `"ui": 1`
11. **Select properties** — every `select` field must have `"allow_null": 0` and `"return_format": "value"`
12. **Missing section_id** — every layout must have a sub_field with `"name": "section_id"` and `"type": "text"`
13. **Orphan PHP files** — list any `template-parts/flexible/*.php` files without a matching layout in JSON

## Output format

Report each issue as:
```
[ISSUE TYPE] layout: [layout_name] / field: [field_key]
Description of the problem and the fix required.
```

If no issues found:
```
✓ group_flexible_content.json — no issues found
```

## How to trigger

```
/audit-json
```
