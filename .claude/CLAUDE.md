# PHP Claude Workspace
# Built for principal and senior PHP developers.
# Supports Laravel · Symfony · Core PHP 8.x
# ─────────────────────────────────────────────────────────────

## How this workspace works
Claude reads this file first, detects your framework, loads the right
rules and skills, then writes production-quality PHP with no guessing.

Every folder is split by framework so a core PHP developer never sees
Eloquent code, and a Laravel developer never sees Doctrine annotations.

---

## Step 1 — Framework detection
Read composer.json at the start of every conversation.

| composer.json contains          | Mode       | Load                        |
|---------------------------------|------------|-----------------------------|
| `laravel/framework`             | Laravel    | rules/laravel.md            |
| `symfony/framework-bundle`      | Symfony    | rules/symfony.md            |
| neither / no composer.json      | Core PHP   | rules/php-core.md           |

Confirm at the start of every response:
> "Detected: Laravel 11. Loading laravel rules + universal rules."
> "Detected: Symfony 7. Loading symfony rules + universal rules."
> "Detected: Core PHP. Loading php-core rules + universal rules."

If framework cannot be determined, ask before writing a single line.

---

## Step 2 — Always-on rules (every framework, every conversation)

| File                      | What it governs                                        |
|---------------------------|--------------------------------------------------------|
| rules/architecture.md     | Layer pattern, request lifecycle, module structure     |
| rules/coding-standards.md | PSR-12, strict_types, PHP 8.x patterns, naming         |
| rules/security.md         | File uploads, SQL injection, secrets, prompt injection |
| rules/api-standards.md    | Response envelopes, HTTP codes, pagination             |
| rules/database.md         | Column types, indexing, migrations, query performance  |
| rules/testing.md          | PHPUnit/Pest, mocking, coverage — all three frameworks |

---

## Step 3 — Skills and templates (routed by framework)

### Laravel → skills/laravel/ + templates/laravel/
| Skill                | Use when                                        |
|----------------------|-------------------------------------------------|
| scaffold-project.md  | New Laravel project from scratch                |
| create-endpoint.md   | Route + FormRequest + Controller + Resource     |
| create-migration.md  | Migration + Eloquent model + casts + scopes     |
| create-service.md    | Service class + interface + AppServiceProvider  |
| create-job.md        | Queued job with ShouldQueue + ShouldBeUnique    |
| add-api-auth.md      | Sanctum token authentication                    |
| integrate-ai.md      | OpenRouter/Anthropic via Http facade            |
| add-file-upload.md   | Storage::disk private + signed URL              |
| write-tests.md       | Pest + RefreshDatabase + Http::fake             |
| refactor-code.md     | Clean up Laravel-specific anti-patterns         |
| debug-issue.md       | Artisan commands + Telescope + queue debugging  |

### Symfony → skills/symfony/
| Skill                | Use when                                        |
|----------------------|-------------------------------------------------|
| scaffold-project.md  | New Symfony project from scratch                |
| create-endpoint.md   | Controller + DTO + Serializer + route           |
| create-migration.md  | Doctrine entity + migration + repository        |
| create-service.md    | Service + interface + services.yaml binding     |
| create-message.md    | Messenger Message + Handler + transport         |
| add-api-auth.md      | LexikJWT or API token authentication            |
| integrate-ai.md      | HttpClientInterface + AI provider service       |
| add-file-upload.md   | Flysystem + private storage + signed URL        |
| write-tests.md       | PHPUnit + KernelTestCase + MockHttpClient       |
| refactor-code.md     | Clean up Symfony-specific anti-patterns         |
| debug-issue.md       | Console commands + Profiler + Messenger debug   |

### Core PHP → skills/core-php/
| Skill                | Use when                                        |
|----------------------|-------------------------------------------------|
| scaffold-project.md  | New core PHP project from scratch               |
| create-endpoint.md   | Router + DTO + Controller + Presenter           |
| create-migration.md  | SQL migration file + PDO execution              |
| create-service.md    | Service class + interface + DI container        |
| create-worker.md     | Background worker + queue without framework     |
| add-api-auth.md      | JWT or API key middleware (no framework)        |
| integrate-ai.md      | Guzzle + AI provider service                    |
| add-file-upload.md   | $_FILES + move_uploaded_file + private folder   |
| write-tests.md       | Pest/PHPUnit + Mockery + Guzzle MockHandler     |
| refactor-code.md     | Clean up framework-free anti-patterns           |
| debug-issue.md       | Xdebug + Monolog + manual query logging         |

### Shared → skills/shared/ (framework-agnostic)
| Skill                | Use when                                        |
|----------------------|-------------------------------------------------|
| integrate-ai.md      | AI integration principles (all frameworks)      |
| refactor-code.md     | Universal refactoring moves                     |
| debug-issue.md       | Universal debugging methodology                 |

---

## Agent roster (all frameworks)
Invoke one agent per task. Agents understand all three frameworks.

| Agent               | Invoke when                                        |
|---------------------|----------------------------------------------------|
| php-architect       | System design, folder structure, major decisions   |
| backend-engineer    | Writing any PHP code                               |
| api-designer        | Endpoint contracts, response design                |
| database-engineer   | Schema design, migrations, query optimisation      |
| code-reviewer       | Reviewing code, PR feedback                        |
| debugger            | Diagnosing bugs, stack traces, performance issues  |

---

## Code generation philosophy (all frameworks)
1. Thin entry points — controllers, handlers, commands delegate immediately
2. Services own business logic — framework-agnostic where possible
3. Repositories own data access — one place for all queries
4. DTOs for inter-layer data — typed, readonly, never plain arrays
5. Interfaces on external dependencies — swappable, testable
6. Jobs/workers for async work — anything over 2 seconds
7. Events for side effects — decouple listeners from triggers
8. Enums for all status/type fields — never magic strings
9. Every public method has a test — no exceptions

---

## What Claude never does
- Write business logic in a controller, handler, or command
- Use raw SQL without parameterised bindings
- Store API keys outside config/environment
- Return raw database results without a presenter/resource/DTO
- Generate code without `declare(strict_types=1)`
- Skip a test for any service method
- Use `new ClassName()` inside methods — always inject
- Ignore rules/security.md when handling file uploads or user input

---

## Project-specific context
# ─────────────────────────────────────────────────────────────
# FILL THIS IN FOR YOUR PROJECT
# ─────────────────────────────────────────────────────────────

## Project name
[Your project name]

## Framework + version
[Laravel 11 / Symfony 7 / Core PHP 8.3]

## Database
[MySQL 8 / PostgreSQL 15 / SQLite]

## Auth approach
[Sanctum / LexikJWT / Custom middleware / API key]

## Queue / async
[Redis+Horizon / Messenger / Custom worker / None]

## Cache
[Redis / File / APCu / None]

## External APIs
[List any: Stripe, SendGrid, OpenAI, S3, etc.]

## Deployment
[Forge / Vapor / Docker / VPS / Shared hosting]

## Custom conventions
[Any project-specific rules that override the defaults above]
