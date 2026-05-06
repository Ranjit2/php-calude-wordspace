# Skill: Debug an Issue

> Artisan commands below are Laravel-specific. Symfony and core PHP equivalents noted where different.


## When to use
A bug report, error in production, test failure, or unexpected behaviour.

---

## Debugging checklist

- [ ] 1.  Read the FULL stack trace — find the originating exception line
- [ ] 2.  Identify the exception type — what category of problem is this?
- [ ] 3.  Reproduce locally with the same input
- [ ] 4.  Check the logs:
          - Laravel: `storage/logs/laravel.log` or `php artisan telescope`
          - Symfony: `var/log/dev.log` or `var/log/prod.log`
          - Core PHP: wherever your logger writes (check your `Monolog` config)
- [ ] 5.  Form a hypothesis — one specific cause, not a list of guesses
- [ ] 6.  Verify hypothesis with a targeted test or `Log::debug()`
- [ ] 7.  Fix the ROOT CAUSE — not a try/catch that swallows the error
- [ ] 8.  Write a regression test that would have caught this
- [ ] 9.  Run: `./vendor/bin/pest`
- [ ] 10. Remove all debug statements before committing

---

## Output format for reporting a finding

```
## Error
[Full exception message and type]

## Root cause
[One paragraph tracing exactly what caused this]

## Evidence
- Stack trace line X shows...
- Log entry shows...
- The model's $casts is missing...

## Fix
[Exact code change required]

## Regression test
[Test that prevents this from happening again]
```

---

## Diagnostic commands

```bash
# ── Laravel ──────────────────────────────────────────────
tail -f storage/logs/laravel.log
php artisan queue:failed
php artisan queue:retry all
php artisan config:clear && php artisan cache:clear
php artisan route:list --path=api/v1
php artisan tinker
>>> DB::listen(fn($q) => dump($q->sql, $q->bindings));

# ── Symfony ──────────────────────────────────────────────
tail -f var/log/dev.log
php bin/console messenger:failed:show
php bin/console messenger:failed:retry
php bin/console cache:clear
php bin/console router:match /api/v1/your-route
php bin/console debug:container YourServiceClass

# ── Core PHP ─────────────────────────────────────────────
tail -f /path/to/your/app.log       # wherever Monolog writes
# No built-in CLI tools — use Xdebug or var_dump() + die()
# Use DB::listen equivalent via PDO query logging

# ── All frameworks ────────────────────────────────────────
php -m | grep -i redis              # verify extension loaded
php --ri xdebug                     # xdebug config
php --ri opcache                    # opcache status
composer diagnose                   # composer environment
composer validate                   # composer.json validity
```

---

## Common bugs to check first

| Symptom | Most likely cause |
|---|---|
| Works locally, fails in prod | Missing `.env` key — run `config:clear` |
| Job never processes | Queue worker not running / Supervisor down |
| Job runs twice | Missing `ShouldBeUnique` |
| Slow page | N+1 queries — check Debugbar or Telescope |
| JSON column returns string | Wrong migration type (`text` not `json`) or missing `$cast` |
| File not found after upload | Mixing `storage_path()` vs `Storage::path()` |
| 500 with no log entry | `APP_DEBUG=false` hiding errors — check `storage/logs/` |
| CORS works locally, fails prod | Cached config — run `config:clear` on server |
