# Catalogue

The index of everything reusable in FixThisPlz: functions and helpers, services, UI components, constants and config keys, hooks and routes, CSS variables and design tokens, shared types and schemas, test helpers.

**Before writing code, check the catalogue of the package you work in, then grep.** If what you need exists, reuse it. If you add something reusable, add its entry to that catalogue in the same commit.

## Catalogues

| Part | Catalogue | Status |
|---|---|---|
| WordPress adapter | [`fixthisplz-wordpress/CATALOGUE.md`](https://github.com/fixthisplz/fixthisplz-wordpress/blob/main/CATALOGUE.md) (separate repository, cloned in `adapters/wordpress/`) | Available |
| `@fixthisplz/schema` | `packages/schema/CATALOGUE.md` | Created with the package (V0) |
| `@fixthisplz/widget` | `packages/widget/CATALOGUE.md` | Created with the package (V0) |

## Format

Each package has its own `CATALOGUE.md`, one table per category. One row per entry:

| Name | Location | What it does | When to use (or not) |
|---|---|---|---|

List only what is meant to be reused, not every private function. Keep each description to one line.

## Rules

1. **Catalogue first**: read the relevant catalogue and grep before adding a helper, constant, component or token.
2. **Same commit**: a new reusable element and its catalogue entry land together.
3. **Checked in CI**: a script compares each package's public exports with its catalogue and fails when an entry is missing (added with the V0 tooling).
4. **Duplicates**: when you spot two things doing the same job, open an issue to consolidate them instead of adding a third.
