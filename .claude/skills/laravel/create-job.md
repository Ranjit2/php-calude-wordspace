# Skill: Create Queued Job (Laravel)

## Checklist
- [ ] 1.  `php artisan make:job Process{Name}Job`
- [ ] 2.  Implement `ShouldQueue` + consider `ShouldBeUnique`
- [ ] 3.  Set `$queue`, `$tries`, `$backoff`, `$timeout`
- [ ] 4.  Inject model via constructor — serialised cleanly by Eloquent
- [ ] 5.  `handle()` delegates entirely to service — no logic in job
- [ ] 6.  Implement `failed()` — update status, log with context
- [ ] 7.  Pest test with `Queue::fake()` for dispatch
- [ ] 8.  Pest test for `handle()` with mocked service

## Job template
```php
<?php declare(strict_types=1);
namespace App\Jobs;
use App\Models\{Name};
use App\Services\{Name}Service;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\{ShouldBeUnique, ShouldQueue};
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\{InteractsWithQueue, SerializesModels};
use Illuminate\Support\Facades\Log;

final class Process{Name}Job implements ShouldQueue, ShouldBeUnique {
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int    $tries   = 3;
    public array  $backoff = [30, 120, 300];
    public int    $timeout = 120;
    public string $queue   = 'default';

    public function __construct(private readonly {Name} $model) {}

    public function uniqueId(): string { return static::class . '_' . $this->model->id; }

    public function handle({Name}Service $service): void {
        $service->process($this->model);
    }

    public function failed(\Throwable $e): void {
        $this->model->update(['status' => \App\Enums\{Name}Status::Failed]);
        Log::error('Process{Name}Job failed', [
            'id'    => $this->model->id,
            'error' => $e->getMessage(),
            'class' => $e::class,
        ]);
    }
}
```
