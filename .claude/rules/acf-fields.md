# ACF Field Types — Decision Reference

> Load this file when you need to decide which ACF field type to use for a given piece of HTML content.

---

## Field type decision table

| HTML content | ACF type | JSON `"type"` | PHP access | Notes |
|---|---|---|---|---|
| Heading, label, short phrase | Textarea | `textarea` | `string` | `rows: 2`, `new_lines: ""` |
| Subheading, 1–2 sentence desc in a classed element | Textarea | `textarea` | `string` | `rows: 3`, `new_lines: ""` |
| Body copy in a classed `<p>` (3+ sentences) | Textarea | `textarea` | `string` | `rows: 5`, `new_lines: ""` |
| `<p>` **without a class** | WYSIWYG | `wysiwyg` | HTML string | Use `wp_kses_post()` |
| Multiple classless `<p>` in same container | WYSIWYG | `wysiwyg` | HTML string | Editor needs formatting control |
| Any image | Image | `image` | `int` (attachment ID) | Always `return_format: "id"`, always `'full'` size |
| Icon in repeater card | Image | `image` | `int` | `preview_size: "medium"`, name: `icon` |
| Background section image | Image | `image` | `int` | Render with `wp_get_attachment_image()`, name: `background_image` |
| Lightbox gallery (multiple images with gallery/lightbox behaviour) | Gallery | `gallery` | `array` of IDs | Use instead of Repeater; `return_format: "id"` |
| Decorative inline `<svg>` (icon, checkmark, phone, location, etc.) | None | — | — | Keep SVG markup as static code in PHP template — no ACF field |
| Single CTA button / nav link | Link | `link` | `array` | Always `return_format: "array"`, name: `cta_link` |
| Multiple CTA buttons (primary + secondary + etc.) | Repeater | `repeater` | `array` | Sub-fields: `link` (Link) + `button_type` (Select: Primary/Secondary/White) |
| Phone number | Link | `link` | `array` | URL = `tel:+...`, title = display text |
| Email address | Link | `link` | `array` | URL = `mailto:...`, title = display text |
| iframe / embed script (JotForm, YouTube, Vimeo, map, etc.) | Textarea | `textarea` | `string` (raw HTML) | `rows: 5`, `new_lines: ""` — output **without esc** — see embed pattern |
| Show/hide section toggle | True/False | `true_false` | `bool` | Name: `enabled`, default: `1` |
| Layout variant / colour theme | Select | `select` | `string` | Define choices in `"choices"` object |
| Repeating cards / rows / items | Repeater | `repeater` | `array` | Loop: `foreach ($items as $i => $item)` |
| Social links / icon+URL pairs | Repeater | `repeater` | `array` | Sub-fields: `icon` (Image) + `link` (Link) |
| Number, count, year | Number | `number` | `int`/`float` | |
| External URL only (no label) | URL | `url` | `string` | |
| Section HTML id (every layout) | Text | `text` | `string` | Name: `section_id`, allows editor to set unique `id` per section |

---

## Textarea — field JSON

```json
{
  "key": "field_[layout]_[name]",
  "label": "Human Label",
  "name": "snake_case_name",
  "type": "textarea",
  "rows": 2,
  "new_lines": ""
}
```

`"new_lines": ""` — **always present on every textarea, no exceptions.**

`rows` guide:
- Heading / label / short phrase → `2`
- Subheading / 1–2 sentence description → `3`
- Body copy in classed element (3+ sentences) → `5`

---

## WYSIWYG — triggered by classless `<p>` tags

```json
{
  "key": "field_[layout]_[name]",
  "label": "Content",
  "name": "content",
  "type": "wysiwyg",
  "toolbar": "full",
  "media_upload": 0
}
```

Decision rule — source HTML:

| Pattern | Use |
|---|---|
| `<p class="text-lg ...">` | Textarea |
| `<p>` (no class) | **WYSIWYG** |
| Two or more `<p>` (no class) in same div | **WYSIWYG** |

---

## Image — field JSON

```json
{
  "key": "field_[layout]_[name]",
  "label": "Image",
  "name": "image",
  "type": "image",
  "return_format": "id",
  "preview_size": "medium"
}
```

All images rendered with `wp_get_attachment_image($id, 'full', ...)`.
Background image → name `background_image`.

---

## Gallery — field JSON

Use when a section is a dedicated lightbox gallery. Prefer over a Repeater of Image fields.

```json
{
  "key": "field_[layout]_gallery",
  "label": "Gallery",
  "name": "gallery",
  "type": "gallery",
  "return_format": "id",
  "preview_size": "medium",
  "insert": "append",
  "min": "",
  "max": ""
}
```

PHP output:
```php
<?php $gallery = get_sub_field('gallery'); ?>
<?php if (!empty($gallery)): ?>
  <div class="gallery-grid">
    <?php foreach ($gallery as $image_id): ?>
      <?= wp_get_attachment_image($image_id, 'full', false, ['class' => 'gallery-item', 'loading' => 'lazy']) ?>
    <?php endforeach; ?>
  </div>
<?php endif; ?>
```

The lightbox JS is wired via data attributes or classes — keep that markup from source HTML as-is.

---

## Link — field JSON

```json
{
  "key": "field_[layout]_[name]",
  "label": "CTA Button",
  "name": "cta_link",
  "type": "link",
  "return_format": "array"
}
```

Phone and email use the same structure — the editor sets the `tel:` / `mailto:` URL.

---

## Multiple CTA buttons — Repeater pattern

When a section has more than one button (e.g. "Get Started" + "Learn More"):

```json
{
  "key": "field_[layout]_buttons",
  "label": "Buttons",
  "name": "buttons",
  "type": "repeater",
  "layout": "block",
  "button_label": "Add button",
  "sub_fields": [
    {
      "key": "field_[layout]_buttons_link",
      "label": "Link",
      "name": "link",
      "type": "link",
      "return_format": "array",
      "parent_repeater": "field_[layout]_buttons"
    },
    {
      "key": "field_[layout]_buttons_button_type",
      "label": "Button Type",
      "name": "button_type",
      "type": "select",
      "choices": {
        "primary": "Primary",
        "secondary": "Secondary",
        "white": "White"
      },
      "default_value": "primary",
      "allow_null": 0,
      "multiple": 0,
      "ui": 0,
      "return_format": "value",
      "parent_repeater": "field_[layout]_buttons"
    }
  ]
}
```

PHP loop:
```php
<?php $buttons = get_sub_field('buttons'); ?>
<?php if (!empty($buttons)): ?>
  <?php foreach ($buttons as $i => $btn): ?>
    <?php $link = $btn['link']; $type = $btn['button_type']; ?>
    <?php if (!empty($link)): ?>
      <a
        href="<?= esc_url($link['url']) ?>"
        <?= !empty($link['target']) ? 'target="' . esc_attr($link['target']) . '"' : '' ?>
        class="btn btn--<?= esc_attr($type) ?>"
      >
        <?= esc_html($link['title']) ?>
      </a>
    <?php endif; ?>
  <?php endforeach; ?>
<?php endif; ?>
```

Replace `btn btn--<?= esc_attr($type) ?>` with the actual classes from the source HTML mapped to each button type.

---

## Repeater — field JSON

```json
{
  "key": "field_[layout]_items",
  "label": "Items",
  "name": "items",
  "type": "repeater",
  "layout": "block",
  "button_label": "Add item",
  "sub_fields": [
    {
      "key": "field_[layout]_items_title",
      "label": "Title",
      "name": "title",
      "type": "textarea",
      "rows": 2,
      "new_lines": "",
      "parent_repeater": "field_[layout]_items"
    }
  ]
}
```

**Every repeater sub-field must include `"parent_repeater"` set to the repeater's `"key"`.**

---

## True/False — field JSON

```json
{
  "key": "field_[layout]_enabled",
  "label": "Enabled",
  "name": "enabled",
  "type": "true_false",
  "default_value": 1,
  "ui": 1,
  "ui_on_text": "Yes",
  "ui_off_text": "No"
}
```

---

## Select — field JSON

```json
{
  "key": "field_[layout]_[name]",
  "label": "Theme",
  "name": "theme",
  "type": "select",
  "choices": {
    "light": "Light",
    "dark": "Dark"
  },
  "default_value": "light",
  "allow_null": 0,
  "multiple": 0,
  "ui": 0,
  "return_format": "value"
}
```

PHP usage example:
```php
$theme = get_sub_field('theme'); // returns string: "light" | "dark"
```
```html
<section class="section--<?= esc_attr($theme) ?> ...">
```

---

## Number — field JSON

```json
{
  "key": "field_[layout]_[name]",
  "label": "Count",
  "name": "count",
  "type": "number",
  "min": "",
  "max": "",
  "step": ""
}
```

---

## URL — field JSON

```json
{
  "key": "field_[layout]_[name]",
  "label": "URL",
  "name": "url",
  "type": "url"
}
```

---

## Embed / iframe — field JSON

For any `<iframe>` or embed script (JotForm, YouTube, Vimeo, Google Maps, etc.):

```json
{
  "key": "field_[layout]_embed_code",
  "label": "Embed Code",
  "name": "embed_code",
  "type": "textarea",
  "rows": 5,
  "new_lines": ""
}
```

PHP output — **no escaping**, raw echo:
```php
<?php $embed_code = get_sub_field('embed_code'); ?>
<?php if (!empty($embed_code)): ?>
  <div class="...">
    <?= $embed_code // phpcs:ignore WordPress.Security.EscapeOutput ?>
  </div>
<?php endif; ?>
```

This field is admin-only input — trusted content, intentionally unescaped.
**Never** apply `esc_html()`, `esc_attr()`, or `wp_kses_post()` to embed fields.

---

## Section ID — field JSON (add to every layout)

```json
{
  "key": "field_[layout]_section_id",
  "label": "Section ID",
  "name": "section_id",
  "type": "text"
}
```

PHP usage in every template part:
```php
$section_id = get_sub_field('section_id');
// ...
<section <?= !empty($section_id) ? 'id="' . esc_attr($section_id) . '"' : '' ?> class="...">
```

---

## Field naming conventions

- Always `snake_case`
- Short and descriptive: `heading` not `main_section_heading_text`
- Sub-fields named relative to the row: repeater `services` → sub-field `title`, not `service_title`
- Background image → always `background_image`
- Single CTA button → always `cta_link`
- Multiple CTA buttons → always `buttons` (Repeater with `link` + `button_type`)
- Section toggle → always `enabled`
- Icon in repeater → always `icon`
- Section HTML id → always `section_id`
- Embed/iframe field → always `embed_code`
