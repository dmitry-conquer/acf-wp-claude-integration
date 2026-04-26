# acf-wp-integration

A Claude Code ruleset for converting static HTML layouts into WordPress theme files with **ACF Pro Flexible Content** field groups and a Theme Settings options page.

---

## What this does

Drop this ruleset into any WordPress project. Point Claude Code at your static HTML source, and it will:

- Detect flexible sections from HTML comment markers in `pages/index.html` and individual files in `components/`
- Map every hardcoded value to the correct ACF field type
- Write PHP template parts with proper escaping and WordPress patterns
- Build `group_flexible_content.json` with all flexible layouts
- Integrate header, footer, and nav into WordPress via existing walker functions
- Create a `group_theme_settings.json` (ACF options page) for editable header/footer content

---

## Requirements

| Requirement | Version |
|---|---|
| ACF Pro | 6.x |
| Tailwind CSS | v4 |
| WordPress | 6.4+ |
| PHP | 8.1+ |

---

## Repository structure

```
acf-wp-integration/
├── CLAUDE.md                        ← Claude Code entry point (always loaded)
├── PROMPT.md                        ← Copy-paste prompt for starting a session
└── .claude/
    ├── rules/
    │   ├── workflow.md              ← Step-by-step integration process
    │   ├── acf-fields.md            ← Field type decision table
    │   ├── acf-json.md              ← JSON structure & key naming rules
    │   └── php-patterns.md          ← PHP escaping, loops, nav patterns
    └── commands/
        ├── integrate.md             ← /integrate — full run
        ├── section.md               ← /section [name] — single section
        └── audit-json.md            ← /audit-json — validate JSON
```

---

## How to use

### 1. Add to your project

Copy or symlink this ruleset into your WordPress project root, or clone it as a git submodule:

```bash
git submodule add https://github.com/your-org/acf-wp-integration .acf-rules
```

Then copy or symlink `CLAUDE.md` to your project root and `.claude/` to `.claude/`.

### 2. Prepare your source files

Your project must have:

```
pages/
  index.html             ← sections inside <main>, each with <!-- Comment Marker -->

components/              ← optional; may contain:
  header.html            ←   structural files (not flexible sections)
  footer.html
  hero.html              ←   individual section files (one file = one layout)
  services.html

wp-theme/
  templates/flexible.php
  acf-json/              ← group_flexible_content.json + group_theme_settings.json
  template-parts/flexible/
  inc/                   ← nav walker class(es)
```

### 3. Start Claude Code

Open Claude Code in your project root and use the prompt from `PROMPT.md`:

```
Read CLAUDE.md, then read .claude/rules/workflow.md.
Perform a full integration...
```

Or use slash commands:

```
/integrate           ← full run
/section hero        ← single section
/audit-json          ← validate JSON
```

---

## Design principles

**Token-efficient** — rule files are loaded on demand, not all upfront. Claude reads only what it needs for the current step.

**Two JSON files** — flexible layouts in `group_flexible_content.json`, theme-wide settings in `group_theme_settings.json`. No per-layout files, no sync conflicts.

**Strict conventions** — layout names in `snake_case`, `"required"` never set, `return_format` always explicit. Auditable and reproducible across projects and contributors.

**No custom CSS** — Tailwind v4 utility classes only, copied verbatim from source HTML.

**Safety by default** — all PHP output is escaped. Forms are never generated. `get_sub_field()` is enforced inside layout parts.

---

## Contributing

Rule files live in `.claude/rules/`. Each file covers one concern and is loaded independently. Keep rules focused — if a file grows beyond ~150 lines, consider splitting it.
