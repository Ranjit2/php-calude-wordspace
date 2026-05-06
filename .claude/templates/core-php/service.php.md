# Template: Service Class (Core PHP)

```php
<?php declare(strict_types=1);

namespace App\Services;

use App\Data\{Name}Data;
use App\Exceptions\{Name}Exception;
use App\Repositories\Contracts\{Name}RepositoryInterface;
use App\Services\Contracts\{External}Interface;
use Psr\Log\LoggerInterface;
use Psr\SimpleCache\CacheInterface;

final class {Name}Service implements \App\Services\Contracts\{Name}Interface {
    public function __construct(
        private readonly {Name}RepositoryInterface $repository,
        private readonly {External}Interface        $external,
        private readonly CacheInterface             $cache,
        private readonly LoggerInterface            $logger,
    ) {}

    public function create({Name}Data $data): int {
        return $this->repository->create($data->toArray());
    }

    /** @throws {Name}Exception */
    public function process(int $id): void {
        $key = "{name}_{$id}";
        if ($cached = $this->cache->get($key)) {
            $this->repository->update($id, $cached);
            return;
        }

        try {
            $result = $this->external->execute($id);
        } catch (\Throwable $e) {
            $this->logger->error('{Name} failed', ['id' => $id, 'error' => $e->getMessage()]);
            throw new {Name}Exception("Processing failed: {$e->getMessage()}", previous: $e);
        }

        $this->repository->update($id, $result);
        $this->cache->set($key, $result, 86400);
    }
}
```
