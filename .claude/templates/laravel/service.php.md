# Template: Service Class (Laravel)

```php
<?php declare(strict_types=1);

namespace App\Services;

use App\Events\{Name}Completed;
use App\Exceptions\{Name}Exception;
use App\Models\{Name};
use App\Repositories\{Name}Repository;
use App\Services\Contracts\{External}Interface;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;

final class {Name}Service {
    public function __construct(
        private readonly {Name}Repository  $repository,
        private readonly {External}Interface $external,
    ) {}

    /** @throws {Name}Exception */
    public function create({Name}Data $data): {Name} {
        return DB::transaction(function () use ($data): {Name} {
            $model = $this->repository->create($data->toArray());
            // Process{Name}Job::dispatch($model)->onQueue('default');
            // {Name}Created::dispatch($model);
            return $model;
        });
    }

    /** @throws {Name}Exception */
    public function process({Name} $model): void {
        $key = "{name}_{$model->id}";
        if ($cached = Cache::get($key)) { $this->apply($model, $cached); return; }

        try {
            $result = $this->external->execute($model->getData());
        } catch (\Throwable $e) {
            Log::error('{Name} failed', ['id' => $model->id, 'error' => $e->getMessage()]);
            throw new {Name}Exception("Processing failed: {$e->getMessage()}", previous: $e);
        }

        $this->apply($model, $result);
        Cache::put($key, $result, now()->addDay());
        {Name}Completed::dispatch($model);
    }

    private function apply({Name} $model, array $data): void {
        $model->update([/* map result to columns */]);
    }
}
```
