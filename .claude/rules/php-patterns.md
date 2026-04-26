# PHP Template Patterns

> Load this file when writing or reviewing `wp-theme/template-parts/flexible/[name].php`.

---

## Template part skeleton

```php
<?php
/**
 * Flexible layout: [Layout Label]
 *
 * Fields:
 *  - heading (textarea)
 *  - body_text (textarea)
 *  - cta_link (link)
 */

// Fetch all fields at the top — never call get_sub_field() inline inside attributes
$section_id = get_sub_field('section_id');
$heading    = get_sub_field('heading');
$body_text  = get_sub_field('body_text');
$cta_link   = get_sub_field('cta_link');
?>

<section <?= !empty($section_id) ? 'id="' . esc_attr($section_id) . '"' : '' ?> class="...">
  <?php // section content goes here ?>
</section>
```

---

## Escaping — mandatory, with one exception

| Context | Function |
|---|---|
| Visible text content | `esc_html($value)` |
| HTML attribute value | `esc_attr($value)` |
| `href`, `src`, `action` URL | `esc_url($value)` |
| WYSIWYG / HTML field | `wp_kses_post($value)` |
| Embed / iframe field | **raw echo** — `<?= $embed_code ?>` — no escaping |

Embed fields contain trusted admin-entered HTML (iframes, scripts). Escaping would break them.

---

## Emptiness checks — always `!empty()`, never bare `if ($var)`

```php
if (!empty($heading)) { ... }
if (!empty($image))   { ... }
if (!empty($cta_link)) { ... }
```

---

## Textarea output

```php
<?php if (!empty($heading)): ?>
  <h2 class="..."><?= esc_html($heading) ?></h2>
<?php endif; ?>

<?php if (!empty($text)): ?>
  <p class="text-lg ..."><?= esc_html($text) ?></p>
<?php endif; ?>
```

Textarea fields output plain escaped text — never `nl2br()`, no HTML formatting.

---

## WYSIWYG output — classless `<p>` tags only

```php
<?php if (!empty($content)): ?>
  <div class="prose"><?= wp_kses_post($content) ?></div>
<?php endif; ?>
```

---

## Image field (`return_format: "id"`)

Always use `'full'` as the image size.

Standard (below fold):
```php
<?php if (!empty($image)): ?>
  <?= wp_get_attachment_image($image, 'full', false, [
    'class'   => 'w-full object-cover',
    'loading' => 'lazy',
  ]) ?>
<?php endif; ?>
```

Hero / above fold — override loading:
```php
<?php if (!empty($image)): ?>
  <?= wp_get_attachment_image($image, 'full', false, [
    'class'         => 'w-full h-full object-cover',
    'loading'       => 'eager',
    'fetchpriority' => 'high',
  ]) ?>
<?php endif; ?>
```

Background image — render as a positioned `<img>` instead of inline style:
```php
<?php $background_image = get_sub_field('background_image'); ?>
<section class="relative ...">
  <?php if (!empty($background_image)): ?>
    <div class="absolute inset-0 overflow-hidden">
      <?= wp_get_attachment_image($background_image, 'full', false, [
        'class'   => 'w-full h-full object-cover',
        'loading' => 'lazy',
      ]) ?>
    </div>
  <?php endif; ?>
  <div class="relative ...">
    <?php // section content goes here ?>
  </div>
</section>
```

`wp_get_attachment_image()` auto-generates `alt`, `width`, `height`, `srcset`, `sizes`.
**Never write `<img>` tags manually.**

---

## Embed / iframe field (no escaping)

```php
<?php $embed_code = get_sub_field('embed_code'); ?>
<?php if (!empty($embed_code)): ?>
  <div class="...">
    <?= $embed_code // phpcs:ignore WordPress.Security.EscapeOutput ?>
  </div>
<?php endif; ?>
```

Never apply any escaping function to embed fields — they contain trusted admin-entered HTML.

---

## Link field (`return_format: "array"`)

```php
<?php if (!empty($cta_link)): ?>
  <a
    href="<?= esc_url($cta_link['url']) ?>"
    <?= !empty($cta_link['target']) ? 'target="' . esc_attr($cta_link['target']) . '"' : '' ?>
    class="..."
  >
    <?= esc_html($cta_link['title']) ?>
  </a>
<?php endif; ?>
```

Phone and email follow the same pattern — the editor sets `tel:` / `mailto:` in the URL field.

---

## Repeater field — loop pattern

**Always `foreach ($items as $i => $item)`. Never `have_rows` / `the_row` inside a repeater.**

```php
<?php $items = get_sub_field('items'); ?>
<?php if (!empty($items)): ?>
  <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
    <?php foreach ($items as $i => $item): ?>
      <?php
        $title = $item['title'];
        $desc  = $item['description'];
        $icon  = $item['icon']; // int (attachment ID)
      ?>
      <div data-aos="fade-up" data-aos-delay="<?= esc_attr($i * 100) ?>" class="...">
        <?php if (!empty($icon)): ?>
          <?= wp_get_attachment_image($icon, 'full', false, [
            'class'       => 'size-10',
            'aria-hidden' => 'true',
          ]) ?>
        <?php endif; ?>
        <?php if (!empty($title)): ?>
          <h3 class="..."><?= esc_html($title) ?></h3>
        <?php endif; ?>
        <?php if (!empty($desc)): ?>
          <p class="..."><?= esc_html($desc) ?></p>
        <?php endif; ?>
      </div>
    <?php endforeach; ?>
  </div>
<?php endif; ?>
```

Rules:
- `$i` is 0-based; display order number as `$i + 1`
- `$i` **always** declared in `foreach` signature even if not printed (needed for `data-aos-delay`)
- Sub-fields accessed as array keys: `$item['field_name']`

---

## Social links / repeating icon+URL sets

```php
<?php $socials = get_sub_field('socials'); ?>
<?php if (!empty($socials)): ?>
  <div class="flex gap-4">
    <?php foreach ($socials as $i => $social): ?>
      <?php $link = $social['link']; $icon = $social['icon']; ?>
      <?php if (!empty($link)): ?>
        <a
          href="<?= esc_url($link['url']) ?>"
          <?= !empty($link['target']) ? 'target="' . esc_attr($link['target']) . '"' : '' ?>
          class="..."
        >
          <?php if (!empty($icon)): ?>
            <?= wp_get_attachment_image($icon, 'full', false, [
              'class'       => 'size-5',
              'aria-hidden' => 'true',
            ]) ?>
          <?php endif; ?>
          <span class="sr-only"><?= esc_html($link['title']) ?></span>
        </a>
      <?php endif; ?>
    <?php endforeach; ?>
  </div>
<?php endif; ?>
```

---

## Conditional section visibility (`enabled` field)

```php
$enabled = get_sub_field('enabled');
if (!$enabled) return;
```

Place at the very top of the template part, before any output.

---

## Inline SVG

Copy inline `<svg>` elements from source HTML verbatim into the PHP template — no ACF field needed.

```php
<svg xmlns="http://www.w3.org/2000/svg" class="size-6" fill="none" viewBox="0 0 24 24" aria-hidden="true">
  <path stroke-linecap="round" .../>
</svg>
```

---

## No HTML comments in templates

Never transfer HTML comments from source HTML into PHP template parts. The only allowed comment is the top docblock:

```php
<?php
/**
 * Flexible layout: [Layout Label]
 * ...
 */
```

Strip all other comments (`<!-- Section Header -->`, `<!-- /end hero -->`, etc.) when converting HTML to PHP.

---

## Section ID

Fetch `section_id` at the top of every template part and use it on the `<section>` element:

```php
$section_id = get_sub_field('section_id');
// ...
<section <?= !empty($section_id) ? 'id="' . esc_attr($section_id) . '"' : '' ?> class="...">
```

---

## AOS attributes

Copy `data-aos` and `data-aos-delay` from source HTML exactly as-is.

```php
<h2 data-aos="fade-up" class="..."><?= esc_html($heading) ?></h2>
```

AOS is initialised globally — no PHP changes required.

---

## Alpine.js in PHP templates

Alpine directives work as-is in PHP — never modify them:

```php
<nav x-data="{ open: false }" @keydown.escape.window="open = false">
  <button @click="open = !open" :aria-expanded="open">...</button>
  <div x-show="open" x-transition>...</div>
</nav>
```

---

## Forms

Never build HTML forms. Where a form exists in the source, leave a PHP comment placeholder and keep the surrounding section structure intact:

```php
<?php // [FORM HERE] ?>
```
