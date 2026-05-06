# Template: Service Class

Replace `{Name}` with your domain name (e.g. `Order`, `Invoice`, `Report`, `Notification`).
Replace `{ExternalService}` with the external dependency (e.g. `PaymentGateway`, `AiProvider`, `StorageDriver`).
Replace `{queue-name}` with your queue name (e.g. `default`, `heavy`, `emails`).

---

```php
<?php declare(strict_types=1);

namespace App\Services;

use App\Data\{Name}Data;
use App\Events\{Name}Completed;
use App\Exceptions\{Name}Exception;
use App\Models\{Name};
use App\Repositories\{Name}Repository;
use App\Services\Contracts\{ExternalService}Interface;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;

final class {Name}Service {
    public function __construct(
        private readonly {Name}Repository              $repository,
        private readonly {ExternalService}Interface    $externalService,
    ) {}

    /**
     * Create a new {name} record and dispatch processing.
     *
     * @throws {Name}Exception
     */
    public function create({Name}Data $data): {Name} {
        return DB::transaction(function () use ($data): {Name} {
            $model = $this->repository->create($data->toArray());

            // Dispatch async job if this triggers long-running work
            // Process{Name}Job::dispatch($model)->onQueue('{queue-name}');

            // Fire event if other parts of the system need to react
            // {Name}Created::dispatch($model);

            return $model;
        });
    }

    /**
     * Process an existing {name} (called from Job or synchronously).
     *
     * @throws {Name}Exception
     */
    public function process({Name} $model): void {
        $cacheKey = "{name}_{$model->id}_result";

        if ($cached = Cache::get($cacheKey)) {
            $this->applyResult($model, $cached);
            return;
        }

        try {
            $result = $this->externalService->execute(
                // Pass the relevant data to the external service
                // e.g. $model->content, $model->toPayload()
            );
        } catch (\Throwable $e) {
            Log::error('{Name} processing failed', [
                'id'    => $model->id,
                'error' => $e->getMessage(),
            ]);
            throw new {Name}Exception(
                "Processing failed: {$e->getMessage()}",
                previous: $e
            );
        }

        $this->applyResult($model, $result);
        Cache::put($cacheKey, $result, now()->addDay());
        {Name}Completed::dispatch($model);
    }

    /**
     * Apply an external result to the model.
     */
    private function applyResult({Name} $model, array $data): void {
        $model->update([
            // Map result data to model columns
            // e.g. 'output'       => $data['output'],
            //      'status'       => {Name}Status::Completed,
            //      'processed_at' => now(),
        ]);
    }
}
```
