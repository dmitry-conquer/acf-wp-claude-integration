# acf-wp-integration

A Claude Code ruleset for converting static HTML layouts (sections, header, footer) into WordPress theme files with **ACF Pro Flexible Content** field groups.

---

## What this does

Drop this ruleset into any WordPress project. Point Claude Code at your static HTML source, and it will:

- Detect all flexible sections from HTML comment markers
- Map every hardcoded value to the correct ACF field type
- Write PHP template parts with proper escaping and WordPress patterns
- Build a single valid `group_flexible_content.json` with all layouts
- Convert the header and footer to WordPress partials with `wp_nav_menu()`

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
├── .claude/
│   ├── rules/
│   │   ├── workflow.md              ← Step-by-step integration process
│   │   ├── acf-fields.md            ← Field type decision table
│   │   ├── acf-json.md              ← JSON structure & key naming rules
│   │   └── php-patterns.md          ← PHP escaping, loops, nav patterns
│   └── commands/
│       ├── integrate.md             ← /integrate — full run
│       ├── section.md               ← /section [name] — single section
│       └── audit-json.md            ← /audit-json — validate JSON
└── docs/
    ├── source-structure.md          ← Required project file layout
    └── troubleshooting.md           ← Common issues and fixes
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
source/
  pages/index.html       ← sections inside <main>, each with <!-- Comment Marker -->
  components/header.html
  components/footer.html

wp-theme/
  templates/flexible.php
  acf-json/
  template-parts/flexible/
```

See `docs/source-structure.md` for full details.

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

**One JSON file** — all ACF layouts live in `group_flexible_content.json`. No split files, no sync conflicts.

**Strict conventions** — layout names in `snake_case`, `"required"` never set, `return_format` always explicit. Auditable and reproducible across projects and contributors.

**No custom CSS** — Tailwind v4 utility classes only, copied verbatim from source HTML.

**Safety by default** — all PHP output is escaped. Forms are never generated. `get_sub_field()` is enforced inside layout parts.

---

## Contributing

Rule files live in `.claude/rules/`. Each file covers one concern and is loaded independently. Keep rules focused — if a file grows beyond ~150 lines, consider splitting it.
