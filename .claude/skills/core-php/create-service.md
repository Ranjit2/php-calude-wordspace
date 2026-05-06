# Skill: Create Service (Core PHP)

## Checklist
- [ ] 1.  Define the single responsibility — what does this service own?
- [ ] 2.  Create interface: `src/Services/Contracts/{Name}Interface.php`
- [ ] 3.  Create service: `src/Services/{Name}Service.php` — constructor injection only
- [ ] 4.  Bind in `src/Bootstrap/app.php` DI container
- [ ] 5.  Define DTOs for inter-layer data
- [ ] 6.  Define custom exceptions in `src/Exceptions/`
- [ ] 7.  Unit test with Mockery for all dependencies
- [ ] 8.  `./vendor/bin/php-cs-fixer fix && ./vendor/bin/pest`

## Interface template
```php
<?php declare(strict_types=1);
namespace App\Services\Contracts;

interface {Name}Interface {
    /**
     * @throws \App\Exceptions\{Name}Exception
     */
    public function create({Name}Data $data): int;
    public function process(int $id): void;
}
```

## Service template
```php
<?php declare(strict_types=1);
namespace App\Services;

use App\Data\{Name}Data;
use App\Exceptions\{Name}Exception;
use App\Repositories\Contracts\{Name}RepositoryInterface;
use App\Services\Contracts\{External}Interface;
use Psr\Log\LoggerInterface;
use Psr\SimpleCache\CacheInterface;

final class {Name}Service implements {Name}Interface {
    public function __construct(
        private readonly {Name}RepositoryInterface $repository,
        private readonly {External}Interface        $external,
        private readonly CacheInterface             $cache,
        private readonly LoggerInterface            $logger,
    ) {}

    public function create({Name}Data $data): int {
        return $this->repository->create($data->toArray());
    }

    public function process(int $id): void {
        $cacheKey = "{name}_{$id}";

        if ($cached = $this->cache->get($cacheKey)) {
            $this->repository->update($id, $cached);
            return;
        }

        try {
            $result = $this->external->execute($id);
        } catch (\Throwable $e) {
            $this->logger->error('{Name} processing failed', ['id' => $id, 'error' => $e->getMessage()]);
            throw new {Name}Exception("Processing failed: {$e->getMessage()}", previous: $e);
        }

        $this->repository->update($id, $result);
        $this->cache->set($cacheKey, $result, 86400);
    }
}
```

## DI container binding (src/Bootstrap/app.php)
```php
\App\Services\Contracts\{Name}Interface::class
    => \DI\autowire(\App\Services\{Name}Service::class),

\App\Services\Contracts\{External}Interface::class
    => \DI\autowire(\App\Services\{Provider}Service::class),

\Psr\Log\LoggerInterface::class
    => \DI\factory(function () {
        $logger  = new \Monolog\Logger('app');
        $handler = new \Monolog\Handler\StreamHandler(
            __DIR__ . '/../../storage/logs/app.log', \Monolog\Level::Debug
        );
        $logger->pushHandler($handler);
        return $logger;
    }),
```
