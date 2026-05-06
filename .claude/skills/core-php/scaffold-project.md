# Skill: scaffold-project (Core PHP)
# See rules/php-core.md for core PHP patterns.
# See skills/shared/ for universal patterns that apply here.

## When to use
Scaffold Project in a core PHP project (no framework).

## Checklist
- [ ] Follow patterns in rules/php-core.md
- [ ] Use php-di container — all dependencies injected via constructor
- [ ] Validate via whitelist + typed DTO — never trust raw input
- [ ] Test with Pest + Mockery — no real HTTP, no real DB in unit tests
- [ ] Run: ./vendor/bin/php-cs-fixer fix && ./vendor/bin/pest
