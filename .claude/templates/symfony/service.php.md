# Template: Service Class (Symfony)

```php
<?php declare(strict_types=1);

namespace App\Service;

use App\Entity\{Name};
use App\Exception\{Name}Exception;
use App\Repository\{Name}Repository;
use App\Service\Contracts\{External}Interface;
use Doctrine\ORM\EntityManagerInterface;
use Psr\Cache\CacheItemPoolInterface;
use Psr\Log\LoggerInterface;

final class {Name}Service {
    public function __construct(
        private readonly EntityManagerInterface $em,
        private readonly {Name}Repository       $repository,
        private readonly {External}Interface    $external,
        private readonly CacheItemPoolInterface $cache,
        private readonly LoggerInterface        $logger,
    ) {}

    /** @throws {Name}Exception */
    public function create({Name}Data $data): {Name} {
        $entity = new {Name}();
        $entity->setTitle($data->title);
        // map data to entity setters

        $this->em->persist($entity);
        $this->em->flush();

        // $this->messageBus->dispatch(new Process{Name}Message($entity->getId()));

        return $entity;
    }

    /** @throws {Name}Exception */
    public function process(int $id): void {
        $item = $this->cache->getItem("{name}_{$id}");
        if ($item->isHit()) { $this->applyResult($id, $item->get()); return; }

        try {
            $result = $this->external->execute($id);
        } catch (\Throwable $e) {
            $this->logger->error('{Name} failed', ['id' => $id, 'error' => $e->getMessage()]);
            throw new {Name}Exception("Processing failed: {$e->getMessage()}", previous: $e);
        }

        $this->applyResult($id, $result);
        $item->set($result)->expiresAfter(86400);
        $this->cache->save($item);
    }

    private function applyResult(int $id, array $data): void {
        $entity = $this->repository->find($id);
        if (!$entity) throw {Name}Exception::notFound($id);
        // map data to entity setters
        $this->em->flush();
    }
}
```
