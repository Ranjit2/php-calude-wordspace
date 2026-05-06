# Skill: Debug Issue (Symfony)

## Checklist
- [ ] 1.  `tail -f var/log/dev.log` (or `prod.log`)
- [ ] 2.  `php bin/console cache:clear`
- [ ] 3.  Check Messenger failures: `php bin/console messenger:failed:show`
- [ ] 4.  Reproduce locally with same input
- [ ] 5.  State hypothesis — one specific cause
- [ ] 6.  Verify with Symfony Profiler or targeted log statement
- [ ] 7.  Fix root cause — not a catch-and-ignore
- [ ] 8.  Write regression test

## Symfony toolkit
```bash
php bin/console cache:clear
php bin/console cache:warmup
php bin/console messenger:failed:show
php bin/console messenger:failed:retry
php bin/console messenger:consume async --time-limit=60
php bin/console debug:container {ClassName}
php bin/console debug:autowiring | grep {Interface}
php bin/console router:match /api/v1/your-path
php bin/console doctrine:query:sql "SELECT * FROM {table} WHERE id = 1"
php bin/console doctrine:migrations:status
```

## Common Symfony bugs
| Symptom | Cause | Fix |
|---|---|---|
| Autowiring fails | Interface not bound in services.yaml | Add explicit binding |
| Message never handled | Consumer not running | `messenger:consume async` |
| Entity not persisted | Missing `$em->flush()` | Add flush after persist |
| Config null in prod | .env.local not on server | Set env vars on server |
| 401 on all routes | Firewall misconfigured | Check security.yaml firewall order |
