# Skill: Create a Service Class

> **All frameworks:** same pattern — plain PHP class, constructor injection, interface binding
> **Laravel:** bind in `AppServiceProvider::register()`
> **Symfony:** autowired via `services.yaml`
> **Core PHP:** bind in your DI container (e.g. `php-di`)


## When to use
Extracting business logic out of a controller, or building a new
service for any domain concern.

---

## Checklist

- [ ] 1.  Define the domain concern — what single responsibility does this service own?
- [ ] 2.  Create an interface if this service could be swapped (e.g. payment gateway,
          AI provider, storage driver, email provider)
          Place in: `app/Services/Contracts/{Name}Interface.php`
- [ ] 3.  Create the service: `app/Services/{Name}Service.php`
          All dependencies injected via constructor
- [ ] 4.  Define input/output DTOs if data crosses layer boundaries
- [ ] 5.  Define custom exceptions for predictable failure modes
          Place in: `app/Exceptions/{Name}Exception.php`
- [ ] 6.  Bind the interface in `AppServiceProvider::register()` if one was created
- [ ] 7.  Write unit tests with all dependencies mocked
- [ ] 8.  Run: `./vendor/bin/pint`

---

## Interface template

```php
<?php declare(strict_types=1);

namespace App\Services\Contracts;

interface {Name}Interface {
    /**
     * [Describe what this method does]
     *
     * @throws \App\Exceptions\{Name}Exception
     */
    public function execute({InputType} $input): {OutputType};
}
```

---

## Service template

```php
<?php declare(strict_types=1);

namespace App\Services;

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
     * [Describe what this method does]
     *
     * @throws {Name}Exception
     */
    public function create({Name}Data $data): {Name} {
        return DB::transaction(function () use ($data): {Name} {
            $model = $this->repository->create($data->toArray());

            // Dispatch async job if this triggers long-running work
            // Process{Name}Job::dispatch($model)->onQueue('{queue-name}');

            // Fire event if other parts of the system care about this
            // {Name}Created::dispatch($model);

            return $model;
        });
    }

    /**
     * [Describe what this method does — called from Job if async]
     *
     * @throws {Name}Exception
     */
    public function process({Name} $model): void {
        // Check cache to avoid duplicate processing
        $cacheKey = "{name}_{$model->id}_result";

        if ($cached = Cache::get($cacheKey)) {
            $this->applyResult($model, $cached);
            return;
        }

        try {
            $result = $this->externalService->execute($model->getData());
        } catch (\Throwable $e) {
            Log::error('{Name} processing failed', [
                'id'    => $model->id,
                'error' => $e->getMessage(),
            ]);
            throw new {Name}Exception("Processing failed: {$e->getMessage()}", previous: $e);
        }

        $this->applyResult($model, $result);
        Cache::put($cacheKey, $result, now()->addDay());
        {Name}Completed::dispatch($model);
    }

    /**
     * Apply a processed result to the model.
     */
    private function applyResult({Name} $model, array $data): void {
        $model->update([
            // Map result data to model columns
            // e.g. 'status' => {Name}Status::Completed,
            //      'output' => $data['output'],
        ]);
    }
}
```

---

## AppServiceProvider binding

```php
// app/Providers/AppServiceProvider.php — register() method
public function register(): void {
    $this->app->bind(
        \App\Services\Contracts\{Name}Interface::class,
        \App\Services\{Name}Service::class,
    );
}
```

---

## Custom exception template

```php
<?php declare(strict_types=1);

namespace App\Exceptions;

final class {Name}Exception extends \RuntimeException {
    public static function processingFailed(string $reason): self {
        return new self("Processing failed: {$reason}");
    }

    public static function notFound(int $id): self {
        return new self("{Name} #{$id} not found.");
    }
}
```
