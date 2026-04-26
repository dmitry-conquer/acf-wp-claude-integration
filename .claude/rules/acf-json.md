# ACF JSON — Structure & Key Naming

> Load this file when creating or updating `wp-theme/acf-json/group_flexible_content.json`.

---

## How ACF JSON sync works

- ACF reads `.json` files from `acf-json/` on theme activation and field group save
- File name must equal the field group `"key"` value: `group_flexible_content.json`
- One file holds all layouts — never split into `layout_*.json` files

---

## Master file structure

File: `wp-theme/acf-json/group_flexible_content.json`

```json
{
  "key": "group_flexible_content",
  "title": "Page Sections",
  "fields": [
    {
      "key": "field_content",
      "label": "Content",
      "name": "content",
      "type": "flexible_content",
      "button_label": "Add section",
      "layouts": {
        "layout_hero": {
          "key": "layout_hero",
          "name": "hero",
          "label": "Hero",
          "display": "block",
          "sub_fields": []
        }
      }
    }
  ],
  "location": [
    [
      {
        "param": "page_template",
        "operator": "==",
        "value": "templates/flexible.php"
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

---

## Key naming rules — globally unique, no exceptions

| What | Pattern | Example |
|---|---|---|
| Field group | `group_[slug]` | `group_flexible_content` |
| Flexible content field | `field_content` | `field_content` |
| Layout | `layout_[section]` | `layout_hero` |
| Sub-field | `field_[section]_[name]` | `field_hero_heading` |
| Repeater field | `field_[section]_[name]` | `field_services_items` |
| Repeater sub-field | `field_[section]_[repeater]_[name]` | `field_services_items_title` |

**Never reuse the same key across different layouts.**

---

## Layout name format rules

| Context | Format | Example |
|---|---|---|
| ACF layout `"name"` | `snake_case` | `trust_strip` |
| PHP template filename | `snake_case` | `template-parts/flexible/trust_strip.php` |
| HTML `id` attribute | `kebab-case` | `id="trust-strip"` |

The layout `"name"` must use **underscores only** — it is used in `get_template_part()` as a PHP string and as a filename. Hyphens will break the lookup.

**The object key in `"layouts"` must exactly match the `"key"` field inside that layout object:**
```json
"layouts": {
  "layout_hero": {
    "key": "layout_hero",
    ...
  }
}
```

---

## Minimum required fields on every sub_field object

```json
{
  "key": "field_[layout]_[name]",
  "label": "Human Label",
  "name": "snake_case_name",
  "type": "textarea"
}
```

Additional required properties by type:

| Type | Extra required |
|---|---|
| `textarea` | `"rows": 2`, `"new_lines": ""` |
| `image` | `"return_format": "id"`, `"preview_size": "medium"` |
| `link` | `"return_format": "array"` |
| `wysiwyg` | `"toolbar": "full"`, `"media_upload": 0` |
| `repeater` | `"layout": "block"`, `"button_label": "Add item"` |
| `true_false` | `"default_value": 1`, `"ui": 1` |
| `select` | `"choices": {}`, `"default_value": ""`, `"allow_null": 0`, `"multiple": 0`, `"ui": 0`, `"return_format": "value"` |
| `gallery` | `"return_format": "id"`, `"preview_size": "medium"`, `"insert": "append"`, `"min": ""`, `"max": ""` |

**NEVER include `"required": 0` or `"required": 1`** — all fields are optional by default.

---

## Repeater layout example

```json
{
  "key": "layout_services",
  "name": "services",
  "label": "Services",
  "display": "block",
  "sub_fields": [
    {
      "key": "field_services_section_id",
      "label": "Section ID",
      "name": "section_id",
      "type": "text"
    },
    {
      "key": "field_services_heading",
      "label": "Heading",
      "name": "heading",
      "type": "textarea",
      "rows": 2,
      "new_lines": ""
    },
    {
      "key": "field_services_items",
      "label": "Service Items",
      "name": "items",
      "type": "repeater",
      "layout": "block",
      "button_label": "Add item",
      "sub_fields": [
        {
          "key": "field_services_items_title",
          "label": "Title",
          "name": "title",
          "type": "textarea",
          "rows": 2,
          "new_lines": "",
          "parent_repeater": "field_services_items"
        },
        {
          "key": "field_services_items_description",
          "label": "Description",
          "name": "description",
          "type": "textarea",
          "rows": 3,
          "new_lines": "",
          "parent_repeater": "field_services_items"
        },
        {
          "key": "field_services_items_icon",
          "label": "Icon",
          "name": "icon",
          "type": "image",
          "return_format": "id",
          "preview_size": "medium",
          "parent_repeater": "field_services_items"
        }
      ]
    }
  ]
}
```

---

## section_id — standard field on every layout

Every layout must include `section_id` as the first sub_field:

```json
{
  "key": "field_[layout]_section_id",
  "label": "Section ID",
  "name": "section_id",
  "type": "text"
}
```

Never add `"instructions"` or `"placeholder"` to any field JSON unless explicitly asked.

---

## Adding a new layout — checklist

1. Check if `group_flexible_content.json` exists:
   - **Not exists** → create file with master structure above, add layout inside `"layouts": { }`
   - **Exists** → read file, insert new layout object into `"layouts": { }`, leave all else unchanged
2. Object key = layout `"key"` value (e.g. `"layout_hero": { "key": "layout_hero", ... }`)
3. Layout `"name"` = PHP filename without `.php` (e.g. `"name": "hero"`)
4. First sub_field = `section_id` (Text field)
5. Create matching PHP: `wp-theme/template-parts/flexible/[name].php`
6. Verify no key collisions with existing layouts
