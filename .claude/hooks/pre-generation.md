# Pre-generation Hook
# Run silently before writing any code.

## 1 — Framework detection
Read composer.json.
- laravel/framework        → LARAVEL mode → load rules/laravel.md
- symfony/framework-bundle → SYMFONY mode → load rules/symfony.md
- neither                  → CORE PHP mode → load rules/php-core.md

Confirm at the top of the response:
"Detected: [Framework]. Using skills/[framework]/ and templates/[framework]/."

## 2 — Security check
Does this task involve any of:
- [ ] File uploads → re-read rules/security.md file upload section before writing
- [ ] User input in a database query → confirm parameterised bindings
- [ ] User input in an AI prompt → confirm XML delimiter wrapping
- [ ] API keys or secrets → confirm config layer, not env() in business code
- [ ] HTML output of user content → confirm escaping

## 3 — Layer check (architecture.md)
- Business logic about to go in a controller/handler? → Extract to service
- HTTP/Request/Response objects inside a service? → Wrong layer
- Database query in a controller? → Move to repository
- Business logic in a job/worker handle()? → Delegate to service

## 4 — Skill check — use the right folder
| Task                     | Laravel skill             | Symfony skill              | Core PHP skill              |
|--------------------------|---------------------------|----------------------------|-----------------------------|
| New endpoint             | skills/laravel/create-endpoint.md | skills/symfony/create-endpoint.md | skills/core-php/create-endpoint.md |
| New table/entity         | skills/laravel/create-migration.md | skills/symfony/create-migration.md | skills/core-php/create-migration.md |
| New service              | skills/laravel/create-service.md | skills/symfony/create-service.md | skills/core-php/create-service.md |
| Async job/worker         | skills/laravel/create-job.md | skills/symfony/create-message.md | skills/core-php/create-worker.md |
| Tests                    | skills/laravel/write-tests.md | skills/symfony/write-tests.md | skills/core-php/write-tests.md |
| Debugging                | skills/laravel/debug-issue.md | skills/symfony/debug-issue.md | skills/core-php/debug-issue.md |
| Refactoring              | skills/shared/refactor-code.md | skills/shared/refactor-code.md | skills/shared/refactor-code.md |

## 5 — PHP quality check
Before generating any file:
- [ ] `declare(strict_types=1)` on line 1
- [ ] All public methods will have return types
- [ ] Constructor injection — no `new ClassName()` inside methods
- [ ] Enums for status/type fields — no magic strings
- [ ] DTOs for inter-layer data — no plain arrays

## 6 — Test reminder
Every generated service method and every entry point needs a test.
- Laravel:   Http::fake(), Storage::fake(), Queue::fake()
- Symfony:   MockHttpClient, MockResponse, in-memory transport
- Core PHP:  Mockery, Guzzle MockHandler, array cache
Never call real external APIs, real storage, or real queues in tests.
