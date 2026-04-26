# /integrate — Full HTML to ACF WP Integration

## What this command does

Runs a complete conversion of static HTML source files into WordPress ACF Pro Flexible Content layouts.

## Steps Claude will follow

1. Read `.claude/rules/workflow.md` for the full step-by-step process
2. Audit `pages/index.html` — scan `<main>` for section comment markers
3. Report detected sections
4. Check `wp-theme/template-parts/flexible/` for already-implemented sections
5. Process each pending section one at a time:
   - PHP template part → `wp-theme/template-parts/flexible/[name].php`
   - JSON layout block → into `wp-theme/acf-json/group_flexible_content.json`
6. After all sections are done — run `/audit-json` and fix any issues before reporting completion

## Rule files loaded on demand (not all at once)

- `acf-fields.md` — when deciding field types for a section
- `acf-json.md` — when writing JSON layout blocks
- `php-patterns.md` — when writing PHP template parts

## How to trigger

Type in Claude Code:
```
/integrate
```

Or with a specific section:
```
/integrate hero
/integrate services
```
