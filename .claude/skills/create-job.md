# Skill: Create a Queued Job

> **Laravel:** `ShouldQueue` + `php artisan make:job` + queue worker
> **Symfony:** Messenger `Message` + `Handler` + transport config
> **Core PHP:** any queue library (e.g. `enqueue/enqueue`) or simple database-backed worker


## When to use
Any work taking more than 2 seconds: external API calls, file processing,
email batches, report generation, data imports, notifications.

---

## Checklist

- [ ] 1.  Create the job/message class:
          - **Laravel:** `php artisan make:job {VerbPhrase}Job`
          - **Symfony:** create a `{Name}Message` + `{Name}Handler` class
          - **Core PHP:** create a worker class for your queue library
- [ ] 2.  **Laravel:** implement `ShouldQueue` · **Symfony:** tag handler with `#[AsMessageHandler]`
- [ ] 3.  **Laravel:** consider `ShouldBeUnique` — prevents duplicate processing
- [ ] 4.  Set `$queue`, `$tries`, `$backoff`, `$timeout`
- [ ] 5.  Inject the model via constructor — it will be serialised cleanly by Eloquent
- [ ] 6.  `handle()` must delegate to a service — no business logic in the job itself
- [ ] 7.  Implement `failed(Throwable $e)` — update record status, log with context
- [ ] 8.  Write test for dispatch:
          - **Laravel:** `Queue::fake()` then `Queue::assertPushed(...)`
          - **Symfony:** `$transport->get()` on in-memory transport
          - **Core PHP:** assert your queue table/store has the job record
- [ ] 9.  Write test for the handler with all external dependencies mocked

---

## Job template

```php
<?php declare(strict_types=1);

namespace App\Jobs;

use App\Enums\{Name}Status;
use App\Models\{Name};
use App\Services\{Name}Service;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Log;

final class Process{Name}Job implements ShouldQueue, ShouldBeUnique {
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    // ─── Queue config ───────────────────────────────────────
    public int    $tries   = 3;
    public array  $backoff = [30, 120, 300];  // seconds between retries
    public int    $timeout = 120;             // max seconds per attempt
    public string $queue   = 'default';       // change to named queue if needed

    public function __construct(
        private readonly {Name} $model,
    ) {}

    /**
     * Unique key prevents duplicate jobs for the same record.
     */
    public function uniqueId(): string {
        return static::class . '_' . $this->model->id;
    }

    /**
     * How long the unique lock is held (seconds).
     * Prevents duplicate dispatch while job is queued.
     */
    public function uniqueFor(): int {
        return 60;
    }

    public function handle({Name}Service $service): void {
        // Update status to running before delegating
        $this->model->update(['status' => {Name}Status::Running]);

        // Delegate ALL logic to the service — never put business logic here
        $service->process($this->model);
    }

    public function failed(\Throwable $e): void {
        // Always update the record status on failure
        $this->model->update([
            'status'       => {Name}Status::Failed,
            'processed_at' => now(),
        ]);

        // Always log with enough context to debug
        Log::error('Process{Name}Job failed', [
            'model_id' => $this->model->id,
            'attempt'  => $this->attempts(),
            'error'    => $e->getMessage(),
            'class'    => $e::class,
        ]);
    }

    /**
     * Stop retrying after this time, regardless of $tries.
     */
    public function retryUntil(): \DateTime {
        return now()->addMinutes(30);
    }
}
```

---

## Dispatch examples

```php
// Basic dispatch
Process{Name}Job::dispatch($model);

// With queue and delay
Process{Name}Job::dispatch($model)
    ->onQueue('heavy')
    ->delay(now()->addSeconds(5));

// Batch processing
Bus::batch([
    new Process{Name}Job($model1),
    new Process{Name}Job($model2),
])->then(function (Batch $batch) {
    // All jobs completed
})->catch(function (Batch $batch, Throwable $e) {
    // First failure
})->dispatch();
```
