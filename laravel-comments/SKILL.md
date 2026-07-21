---
name: laravel-comments
description: "Format Laravel config file comments using the cascading style. Use when writing or fixing comments in config/*.php files."
license: MIT
metadata:
  author: pushpak1300
  url: https://gist.github.com/pushpak1300/09cb39de9616d32ca8efe97996844b61
---

# Laravel Cascading Comment Style

## When to Use This Skill

Activate this skill when:
- Writing new config file comments in Laravel
- Fixing or reformatting existing config comments
- Creating package config files that should match Laravel's style

---

## The Pattern

Laravel uses a distinctive three-line cascading comment style where each line is progressively shorter:

| Line | Target Length | Difference |
|------|---------------|------------|
| 1st  | ~74 chars     | baseline   |
| 2nd  | ~70 chars     | 4 fewer    |
| 3rd  | ~67 chars     | 3 fewer    |

This creates a subtle visual rhythm that contributes to Laravel's polished feel.

---

## Format Structure

```php
/*
|--------------------------------------------------------------------------
| Section Title Here
|--------------------------------------------------------------------------
|
| First line of description should be approximately seventy-four characters.
| Second line should be about four characters shorter than the first.
| Third line three fewer than second, ending the cascade here.
|
*/
```

---

## Examples

**Good (follows cascade):**

```php
/*
|--------------------------------------------------------------------------
| Application Timezone
|--------------------------------------------------------------------------
|
| Here you may specify the default timezone for your application, which
| will be used by the PHP date and date-time functions. We have gone
| ahead and set this to a sensible default for you out of the box.
|
*/
```

Line lengths: 72 → 68 → 65 ✓

**Bad (breaks cascade):**

```php
/*
|--------------------------------------------------------------------------
| Executables Config
|--------------------------------------------------------------------------
|
| The following options allow you to configure custom paths for the
| executables used by Boost. Leave empty to use defaults.
| When configured, these take precedence over automatic detection.
|
*/
```

Line lengths: 67 → 57 → 66 ✗ (second line too short, third jumps back up)

---

## Writing Process

1. **Draft your content** - Write what needs to be said
2. **Count characters** - Use an editor or `wc -c` to measure each line
3. **Adjust for cascade** - Reword to hit the ~74 → ~70 → ~67 pattern
4. **Read aloud** - Ensure it still reads naturally

---

## Adjustment Techniques

When a line is too short:
- Add clarifying adjectives: "the" → "the automatic"
- Expand contractions: "don't" → "do not"
- Use fuller phrases: "use defaults" → "use defaults from your $PATH"

When a line is too long:
- Remove filler words: "in order to" → "to"
- Use shorter synonyms: "configuration" → "config"
- Move content to another line

---

## Quick Reference

Count characters excluding the `| ` prefix (2 chars).

Target line content lengths:
- **Line 1:** 72-76 characters
- **Line 2:** 68-72 characters (4 fewer than line 1)
- **Line 3:** 65-69 characters (3 fewer than line 2)

---

## Source

This pattern was discovered by Caleb Porzio who analyzed 598 three-line comments in Laravel's codebase. It's an undocumented but consistent stylistic choice that gives Laravel configs their characteristic polish.

Reference: https://calebporzio.com/laravel-comments