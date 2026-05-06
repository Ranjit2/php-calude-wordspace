# Skill: Debug Issue (Core PHP)

## Checklist
- [ ] 1.  Check your log file — wherever Monolog writes
- [ ] 2.  Enable detailed error reporting in development:
          `ini_set('display_errors', '1'); error_reporting(E_ALL);`
- [ ] 3.  Reproduce locally with same input
- [ ] 4.  State one hypothesis — not a list of guesses
- [ ] 5.  Verify with targeted `error_log()` or Xdebug breakpoint
- [ ] 6.  Fix root cause — not a catch-and-swallow
- [ ] 7.  Write regression test

## Diagnostic toolkit
```bash
# Logs
tail -f storage/logs/app.log

# PHP environment
php -m | grep -i redis
php --ri xdebug
php --ri pdo_mysql

# Database
php -r "
require 'vendor/autoload.php';
\$pdo = require 'config/database.php';
\$rows = \$pdo->query('SELECT * FROM {table} LIMIT 5')->fetchAll(PDO::FETCH_ASSOC);
var_dump(\$rows);
"

# Composer
composer diagnose
composer validate

# Development server with error display
php -S localhost:8000 -t public/ 2>&1 | tee server.log
```

## Common core PHP bugs
| Symptom | Cause | Fix |
|---|---|---|
| PDO returns false | Wrong column name or null fetch | Check column names, use `FETCH_ASSOC` |
| JSON decode returns null | Malformed JSON or wrong encoding | Use `JSON_THROW_ON_ERROR` |
| File upload fails silently | `upload_max_filesize` too small | Check `php.ini` + `post_max_size` |
| DI not injecting | Container binding missing | Add binding in Bootstrap/app.php |
| Query returns no rows | Missing index or wrong WHERE | Run EXPLAIN, check column types |
| Headers already sent | Output before header() call | Check for whitespace before `<?php` |
