# php-claude-workspace

> A structured Claude AI workspace for PHP developers.
> Works for Laravel, Symfony, and core PHP — nothing bleeds across.
> Drop it in. Claude immediately works like a senior PHP engineer.

---

## What is this?

A `.claude/` folder you copy into any PHP project. Claude reads it, detects your framework, and loads exactly the right rules, skills, and templates for your stack — without you explaining anything.

A core PHP developer sees core PHP patterns. A Laravel developer sees Eloquent and Artisan. A Symfony developer sees Doctrine and Messenger. No framework bleed.

---

## Quick start

```bash
# 1. Copy into your project root
cp -r .claude/ /path/to/your/project/

# 2. Open Claude Code
cd /path/to/your/project
claude

# 3. Start every conversation with
```
```
Read CLAUDE.md, detect the framework from composer.json,
and confirm which rules and skills are loaded.
```

---

## Structure

```
.claude/
│
├── CLAUDE.md                  ← master context; fill in your project details
├── settings.json              ← model, context files, hooks
│
├── agents/                    ← role personas — work for all frameworks
│   ├── php-architect.md       ← system design, boundaries, interfaces
│   ├── backend-engineer.md    ← writes production PHP code
│   ├── api-designer.md        ← endpoint and response contract design
│   ├── database-engineer.md   ← schema, migrations, query optimisation
│   ├── code-reviewer.md       ← PR review: CRITICAL / WARNING / STYLE
│   └── debugger.md            ← root cause analysis, all three frameworks
│
├── rules/                     ← always loaded; split by concern
│   ├── architecture.md        ← universal layer pattern (all frameworks)
│   ├── coding-standards.md    ← PSR-12, strict_types, PHP 8.x — pure PHP
│   ├── security.md            ← uploads, SQL, prompt injection — all frameworks
│   ├── api-standards.md       ← response envelopes, status codes — universal
│   ├── database.md            ← column types, indexing — MySQL/PostgreSQL
│   ├── testing.md             ← Laravel/Symfony/Core PHP test patterns
│   ├── laravel.md             ← auto-loaded for Laravel projects
│   ├── symfony.md             ← auto-loaded for Symfony projects
│   └── php-core.md            ← auto-loaded for framework-free PHP
│
├── skills/
│   ├── laravel/               ← Artisan, Eloquent, Pest, Sanctum, Queues
│   │   ├── create-endpoint.md
│   │   ├── create-migration.md
│   │   ├── create-service.md
│   │   ├── create-job.md
│   │   ├── write-tests.md
│   │   ├── debug-issue.md
│   │   └── ...
│   ├── symfony/               ← Doctrine, Messenger, PHPUnit, Voter, DI
│   │   ├── create-endpoint.md
│   │   ├── create-migration.md
│   │   ├── create-service.md
│   │   ├── create-message.md
│   │   ├── write-tests.md
│   │   ├── debug-issue.md
│   │   └── ...
│   ├── core-php/              ← PDO, php-di, Guzzle, Pest, Mockery
│   │   ├── create-endpoint.md
│   │   ├── create-migration.md
│   │   ├── create-service.md
│   │   ├── create-worker.md
│   │   ├── write-tests.md
│   │   ├── debug-issue.md
│   │   └── ...
│   └── shared/                ← universal patterns, all frameworks
│       ├── refactor-code.md
│       └── debug-issue.md
│
├── templates/
│   ├── shared/                ← pure PHP 8.x — no framework imports
│   │   ├── dto.php.md
│   │   ├── enum.php.md
│   │   └── value-object.php.md
│   ├── laravel/               ← Laravel-specific skeletons
│   │   ├── service.php.md
│   │   └── pest-test.php.md
│   ├── symfony/               ← Symfony-specific skeletons
│   │   └── service.php.md
│   └── core-php/              ← Core PHP skeletons
│       └── service.php.md
│
└── hooks/
    ├── pre-generation.md      ← 6 checks before writing code (framework-aware)
    └── post-generation.md     ← 9 checks after writing code (all frameworks)
```

---

## Prompt reference

### Start any conversation
```
Read CLAUDE.md, detect the framework from composer.json, and confirm which rules are loaded.
```

### Invoke an agent
```
Read agents/backend-engineer.md then build [feature].
Read agents/php-architect.md and design the structure for [feature].
Read agents/code-reviewer.md and review: [paste code]
Read agents/debugger.md — stack trace: [paste trace]
Read agents/database-engineer.md and design the schema for [feature].
```

### Use a skill (framework-specific)
```
# Laravel
Read agents/backend-engineer.md and skills/laravel/create-endpoint.md
then build POST /api/v1/[resource].

# Symfony
Read agents/backend-engineer.md and skills/symfony/create-endpoint.md
then build the [resource] endpoint.

# Core PHP
Read agents/backend-engineer.md and skills/core-php/create-endpoint.md
then build POST /api/v1/[resource].
```

### Full feature
```
Read agents/backend-engineer.md
+ skills/[framework]/create-migration.md
+ skills/[framework]/create-service.md
+ skills/[framework]/create-endpoint.md
and build the [feature] feature end to end.
```

---

## Customise for your project

### Step 1 — Fill in `CLAUDE.md`
Open `.claude/CLAUDE.md` and complete the Project Context section at the bottom.
This is the most important step.

### Step 2 — Override any rule file
Add your team conventions to the relevant `rules/` file.
For example, add approved packages to `rules/coding-standards.md`.

### Step 3 — Add domain-specific agents
```
agents/payments-specialist.md   ← fintech domain knowledge
agents/search-engineer.md       ← Elasticsearch / search patterns
```

### Step 4 — Add project-specific skills
```
skills/laravel/create-report.md
skills/core-php/import-csv.md
```

### Step 5 — Local override (not committed)
Create `.claude/CLAUDE.local.md` for sensitive context (listed in `.gitignore`).

---

## What is enforced automatically

| Concern        | Enforced behaviour                                                      |
|----------------|-------------------------------------------------------------------------|
| Code quality   | `strict_types=1`, full type hints, return types, method length limits   |
| Architecture   | Entry point → Service → Repository, no logic in jobs or controllers     |
| Security       | UUID filenames, private disk, parameterised SQL, prompt injection guards |
| Database       | Correct column types, explicit FK indexes, soft deletes, JSON casts     |
| Testing        | Real external services always faked — HTTP, storage, queue              |
| API design     | Consistent envelopes, correct status codes, pagination always           |
| PHP 8.x        | Enums, readonly DTOs, match expressions, constructor promotion           |
| Consistency    | Post-generation hook checks every file before Claude finishes           |

---

## Requirements

- [Claude Code](https://claude.ai/code) or any Claude interface
- PHP 8.1+ (8.3 recommended)
- Composer

---

## Contributing

PRs welcome. To keep it useful for everyone:
- New framework support → add `rules/{framework}.md` + `skills/{framework}/`
- New agent → add `agents/{domain}.md`
- New shared pattern → add `skills/shared/{task}.md`
- New template → add `templates/shared/` or `templates/{framework}/`

Keep examples generic — use `{Name}`, `{Resource}`, `{table}` as placeholders.

---

## License

MIT
# php-calude-wordspace
