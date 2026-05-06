# Post-generation Hook
# Self-review after generating code.

## 1 — strict_types
Every PHP file starts with `<?php declare(strict_types=1);` — if not, add it.

## 2 — Return types
Every public method has an explicit return type. Scan every signature.

## 3 — Security scan
- User content wrapped in delimiters before AI prompts?
- Secrets accessed via config layer — not env() in services?
- File stored with UUID filename on private storage?
- SQL queries using parameterised bindings?

## 4 — Architecture
- Any business logic in controller/handler/command? → Extract to service
- Any framework HTTP objects inside service? → Wrong layer
- Any logic in job/worker handle()? → Delegate to service

## 5 — Type quality
- Plain array returned where DTO could be used?
- Magic string in status/type comparison? → Use Enum
- Nullable param used without nullsafe operator or null check?

## 6 — Tests
- Test included for every service method and entry point?
- External HTTP/storage/queue properly faked — not real?
- Covers 422, 401, and at least one failure path?

## 7 — Migration / entity
- New column? → migration file created (never existing migration modified)
- down() reverses up() exactly?
- FK indexes added explicitly?
- Model $casts / entity type mappings updated?

## 8 — Run command
Always output this at the end of any code generation response:

```
# Laravel
./vendor/bin/pint && php artisan test

# Symfony
./vendor/bin/php-cs-fixer fix && php bin/phpunit

# Core PHP
./vendor/bin/php-cs-fixer fix && ./vendor/bin/pest
```

## 9 — Summary
After a complete feature, output:
```
Files created:
- [list every file]

Interfaces bound:
- [interface] → [implementation] ([where it's registered])

Next steps:
- [run migrations]
- [run tests]
- [run code style fixer]
```
