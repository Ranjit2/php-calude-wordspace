# Symfony Rules
# Loaded automatically when symfony/framework-bundle is detected in composer.json.

## Version
Symfony 6.4 / 7.x. Use attributes over YAML/XML configuration where possible.

## Controllers
```php
#[Route('/api/v1/{resources}', name: 'api_{resources}_')]
final class {Name}Controller extends AbstractController {
    public function __construct(private readonly {Name}Service $service) {}

    #[Route('', name: 'index', methods: ['GET'])]
    public function index(Request $request): JsonResponse {
        $page  = $request->query->getInt('page', 1);
        $items = $this->service->paginate($page);
        return $this->json(['data' => $items]);
    }

    #[Route('', name: 'create', methods: ['POST'])]
    public function create(#[MapRequestPayload] {Name}Data $data): JsonResponse {
        $this->denyAccessUnlessGranted('ROLE_USER');
        $result = $this->service->create($data);
        return $this->json(['data' => $result], Response::HTTP_CREATED);
    }

    #[Route('/{id}', name: 'show', methods: ['GET'])]
    public function show(int $id): JsonResponse {
        $entity = $this->service->findOrFail($id);
        $this->denyAccessUnlessGranted({Name}Voter::VIEW, $entity);
        return $this->json(['data' => $entity]);
    }
}
```

## DTO with validation attributes
```php
final readonly class {Name}Data {
    public function __construct(
        #[Assert\NotBlank]
        #[Assert\Length(min: 3, max: 255)]
        public readonly string $title,

        #[Assert\NotBlank]
        public readonly string $body,

        // Add your fields with Assert constraints
    ) {}
}
```

## Doctrine entity
```php
#[Entity]
#[Table(name: '{table}')]
class {Name} {
    #[Id]
    #[GeneratedValue]
    #[Column(type: 'integer')]
    private int $id;

    #[Column(type: 'string', length: 255)]
    private string $title;

    #[Column(type: 'string', length: 20)]
    private string $status = 'draft';

    #[Column(type: 'json', nullable: true)]
    private ?array $metadata = null;

    #[ManyToOne(targetEntity: User::class)]
    #[JoinColumn(nullable: false, onDelete: 'CASCADE')]
    private User $author;

    #[Column(type: 'datetime_immutable')]
    private \DateTimeImmutable $createdAt;
}
```

## Repository
```php
#[AsEntityRepository]
class {Name}Repository extends ServiceEntityRepository {
    public function __construct(ManagerRegistry $registry) {
        parent::__construct($registry, {Name}::class);
    }

    public function findPublishedByUser(int $userId): array {
        return $this->createQueryBuilder('n')
            ->where('n.author = :userId AND n.status = :status')
            ->setParameter('userId', $userId)
            ->setParameter('status', 'active')
            ->orderBy('n.createdAt', 'DESC')
            ->getQuery()
            ->getResult();
    }
}
```

## Dependency injection (services.yaml)
```yaml
services:
    _defaults:
        autowire: true
        autoconfigure: true

    App\Services\Contracts\{Name}Interface:
        class: App\Services\{Name}Service

    App\Services\Contracts\ExternalServiceInterface:
        class: App\Services\{Provider}Service
        arguments:
            $apiKey: '%env(EXTERNAL_API_KEY)%'
```

## Messenger (async jobs)
```php
// Message
final readonly class Process{Name}Message {
    public function __construct(public readonly int $id) {}
}

// Handler
#[AsMessageHandler]
final class Process{Name}Handler {
    public function __construct(private readonly {Name}Service $service) {}
    public function __invoke(Process{Name}Message $message): void {
        $this->service->process($message->id);
    }
}

// Dispatch
$this->messageBus->dispatch(new Process{Name}Message($entity->getId()));
```

## messenger.yaml
```yaml
framework:
    messenger:
        failure_transport: failed
        transports:
            async: '%env(MESSENGER_TRANSPORT_DSN)%'
            failed: 'doctrine://default?queue_name=failed'
        routing:
            App\Message\Process{Name}Message: async
```

## Security (Voter pattern)
```php
#[AsVoter]
final class {Name}Voter extends Voter {
    protected function supports(string $attribute, mixed $subject): bool {
        return in_array($attribute, ['VIEW', 'EDIT', 'DELETE'])
            && $subject instanceof {Name};
    }
    protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool {
        $user = $token->getUser();
        return match($attribute) {
            'VIEW'   => true,
            'EDIT', 'DELETE' => $subject->getAuthor() === $user,
            default  => false,
        };
    }
}
```

## Run commands
```bash
php bin/console make:entity {Name}
php bin/console doctrine:migrations:diff
php bin/console doctrine:migrations:migrate
php bin/console make:message Process{Name}
php bin/console messenger:consume async --time-limit=60
./vendor/bin/php-cs-fixer fix && php bin/phpunit
```
