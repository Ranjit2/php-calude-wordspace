# Skill: Create Background Worker (Core PHP)

## Checklist
- [ ] 1.  Create job table in database for queue persistence
- [ ] 2.  Create `src/Workers/{Name}Worker.php` — fetches and processes jobs
- [ ] 3.  Create job dispatcher: `src/Queue/Queue.php`
- [ ] 4.  Worker delegates entirely to service — no logic in worker
- [ ] 5.  Implement retry logic with exponential backoff
- [ ] 6.  Write Pest test for worker with mocked service
- [ ] 7.  Set up Supervisor to run worker in production

## Jobs table SQL
```sql
CREATE TABLE jobs (
    id           BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    queue        VARCHAR(100) NOT NULL DEFAULT 'default',
    payload      JSON NOT NULL,
    attempts     TINYINT UNSIGNED NOT NULL DEFAULT 0,
    status       VARCHAR(20) NOT NULL DEFAULT 'pending',
    available_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    failed_at    TIMESTAMP DEFAULT NULL,
    error        TEXT DEFAULT NULL,
    INDEX idx_jobs_queue_status (queue, status),
    INDEX idx_jobs_available_at (available_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## Queue dispatcher
```php
<?php declare(strict_types=1);
namespace App\Queue;

final class Queue {
    public function __construct(private readonly \PDO $pdo) {}

    public function push(string $worker, array $payload, string $queue = 'default', int $delaySeconds = 0): void {
        $stmt = $this->pdo->prepare(
            'INSERT INTO jobs (queue, payload, available_at) VALUES (:queue, :payload, :available_at)'
        );
        $stmt->execute([
            'queue'        => $queue,
            'payload'      => json_encode(['worker' => $worker, 'data' => $payload], JSON_THROW_ON_ERROR),
            'available_at' => date('Y-m-d H:i:s', time() + $delaySeconds),
        ]);
    }
}
```

## Worker process (bin/worker.php)
```php
<?php declare(strict_types=1);

require __DIR__ . '/../vendor/autoload.php';

$container = require __DIR__ . '/../src/Bootstrap/app.php';
$pdo       = $container->get(\PDO::class);
$queue     = $_argv[1] ?? 'default';

echo "Worker started on queue: {$queue}\n";

while (true) {
    $stmt = $pdo->prepare(
        'SELECT * FROM jobs
         WHERE queue = :queue AND status = "pending" AND available_at <= NOW()
         ORDER BY available_at ASC LIMIT 1 FOR UPDATE SKIP LOCKED'
    );
    $stmt->execute(['queue' => $queue]);
    $job = $stmt->fetch(\PDO::FETCH_ASSOC);

    if (!$job) { sleep(2); continue; }

    $pdo->prepare('UPDATE jobs SET status = "running", attempts = attempts + 1 WHERE id = :id')
        ->execute(['id' => $job['id']]);

    try {
        $payload = json_decode($job['payload'], true, 512, \JSON_THROW_ON_ERROR);
        $worker  = $container->get($payload['worker']);
        $worker->handle($payload['data']);
        $pdo->prepare('UPDATE jobs SET status = "completed" WHERE id = :id')
            ->execute(['id' => $job['id']]);
    } catch (\Throwable $e) {
        $failed = $job['attempts'] >= 3;
        $pdo->prepare(
            'UPDATE jobs SET status = :status, error = :error, failed_at = :failed_at,
             available_at = DATE_ADD(NOW(), INTERVAL :delay SECOND) WHERE id = :id'
        )->execute([
            'status'    => $failed ? 'failed' : 'pending',
            'error'     => $e->getMessage(),
            'failed_at' => $failed ? date('Y-m-d H:i:s') : null,
            'delay'     => min(30 * (2 ** $job['attempts']), 300), // exponential backoff
            'id'        => $job['id'],
        ]);
        error_log("Job {$job['id']} failed: " . $e->getMessage());
    }
}
```

## Supervisor config (production)
```ini
[program:php-worker]
command=php /var/www/html/bin/worker.php default
directory=/var/www/html
autostart=true
autorestart=true
numprocs=2
process_name=%(program_name)s_%(process_num)02d
stdout_logfile=/var/log/supervisor/worker.log
stderr_logfile=/var/log/supervisor/worker_err.log
```
