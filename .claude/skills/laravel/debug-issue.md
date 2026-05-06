# Skill: Debug Issue (Laravel)

## Checklist
- [ ] 1.  `tail -f storage/logs/laravel.log`
- [ ] 2.  `php artisan config:clear && php artisan cache:clear` (after .env changes)
- [ ] 3.  Check queue: `php artisan queue:failed`
- [ ] 4.  Reproduce locally with same input
- [ ] 5.  State hypothesis — one specific cause
- [ ] 6.  Verify with targeted test or `Log::debug()`
- [ ] 7.  Fix root cause — not a try/catch wrapper
- [ ] 8.  Write regression test

## Artisan toolkit
```bash
php artisan queue:failed
php artisan queue:retry all
php artisan horizon:status
php artisan config:clear
php artisan cache:clear
php artisan route:list --path=api/v1
php artisan tinker
>>> DB::listen(fn($q) => dump($q->sql, $q->bindings));
>>> \App\Models\{Name}::find(1)->toArray();
```

## Common Laravel bugs
| Symptom | Cause | Fix |
|---|---|---|
| Config returns null in prod | Cached config with old values | `php artisan config:clear` |
| Job never runs | Queue worker stopped | Restart Supervisor / Horizon |
| Duplicate job processing | Missing `ShouldBeUnique` | Add interface + `uniqueId()` |
| JSON column returns string | Missing `$casts` entry or wrong migration type | Add `'array'` cast + use `$table->json()` |
| N+1 (100+ queries) | Missing `->with()` | Eager-load relationships |
