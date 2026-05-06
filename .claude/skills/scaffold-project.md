# Skill: Scaffold a New PHP Project

> Framework-specific checklists below. Choose the section that matches your project.


## When to use
Starting any new PHP project from scratch.

## Step 1 — Detect or confirm framework
Ask if not provided: Laravel / Symfony / Core PHP?
Then load the appropriate rules file.

## Laravel scaffold checklist
- [ ] `composer create-project laravel/laravel {name}`
- [ ] Set `.env`: APP_NAME, APP_URL, DB_*, QUEUE_CONNECTION, CACHE_STORE
- [ ] `php artisan key:generate`
- [ ] `composer require laravel/sanctum spatie/pdf-to-text pestphp/pest --dev`
- [ ] `php artisan install:api` (Sanctum)
- [ ] `php artisan pest:install`
- [ ] Create folder structure: app/Data/ app/Enums/ app/Exceptions/ app/Repositories/ app/Services/Contracts/ app/Values/
- [ ] Create base exception: app/Exceptions/DomainException.php
- [ ] Register interface bindings in AppServiceProvider::register()
- [ ] Configure rate limiters in AppServiceProvider::boot()
- [ ] Update config/services.php with all external API keys
- [ ] Create .env.example with all required keys (dummy values)
- [ ] Install Laravel Pint: `composer require laravel/pint --dev`
- [ ] Create pint.json with preset: laravel
- [ ] `php artisan migrate`
- [ ] Run tests: `php artisan test` (Laravel) / `php bin/phpunit` (Symfony) / `./vendor/bin/pest` (Core PHP)

## Symfony scaffold checklist
- [ ] `composer create-project symfony/skeleton {name}`
- [ ] `composer require webapp` (full stack)
- [ ] Set `.env`: DATABASE_URL, MESSENGER_TRANSPORT_DSN
- [ ] `composer require lexik/jwt-authentication-bundle` (API auth)
- [ ] Configure services.yaml autowiring and interface bindings
- [ ] Create domain folder structure
- [ ] `php bin/console doctrine:database:create`
- [ ] `php bin/console doctrine:migrations:migrate`
- [ ] `php bin/phpunit` — confirm green

## Core PHP scaffold checklist
- [ ] Create folder structure: src/ public/ config/ tests/ resources/
- [ ] `composer init` — set PSR-4 autoloading for src/ and tests/
- [ ] `composer require php-di/php-di vlucas/phpdotenv guzzlehttp/guzzle monolog/monolog ramsey/uuid`
- [ ] `composer require pestphp/pest phpstan/phpstan --dev`
- [ ] Create public/index.php (front controller)
- [ ] Create src/bootstrap.php (container setup)
- [ ] Create .env and .env.example
- [ ] Configure DI container
- [ ] `./vendor/bin/pest` — confirm green

## Universal post-scaffold tasks
- [ ] Create .gitignore (vendor/, .env, storage/, node_modules/, .phpunit.cache)
- [ ] Create README.md with: project description, setup steps, all required env keys
- [ ] Initialize git: `git init && git add . && git commit -m "Initial scaffold"`
- [ ] Create GitHub repo and push
- [ ] Set up GitHub Actions CI (see templates/github-actions-ci.yml)
