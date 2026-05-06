# Skill: Create Messenger Message + Handler (Symfony)

## Checklist
- [ ] 1.  Create `src/Message/Process{Name}Message.php` — readonly, carries only the ID
- [ ] 2.  Create `src/MessageHandler/Process{Name}Handler.php` with `#[AsMessageHandler]`
- [ ] 3.  Handler delegates entirely to service — no logic in handler
- [ ] 4.  Configure transport in `config/packages/messenger.yaml`
- [ ] 5.  Set up failure transport for failed messages
- [ ] 6.  Write PHPUnit test with in-memory transport
- [ ] 7.  `php bin/phpunit`

## Message template
```php
<?php declare(strict_types=1);
namespace App\Message;

final readonly class Process{Name}Message {
    public function __construct(
        public readonly int $id,
    ) {}
}
```

## Handler template
```php
<?php declare(strict_types=1);
namespace App\MessageHandler;
use App\Message\Process{Name}Message;
use App\Service\{Name}Service;
use Psr\Log\LoggerInterface;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;

#[AsMessageHandler]
final class Process{Name}Handler {
    public function __construct(
        private readonly {Name}Service  $service,
        private readonly LoggerInterface $logger,
    ) {}

    public function __invoke(Process{Name}Message $message): void {
        try {
            $this->service->process($message->id);
        } catch (\Throwable $e) {
            $this->logger->error('Process{Name}Handler failed', [
                'id'    => $message->id,
                'error' => $e->getMessage(),
            ]);
            throw $e; // re-throw so Messenger retries / routes to failure transport
        }
    }
}
```

## messenger.yaml
```yaml
framework:
    messenger:
        failure_transport: failed
        transports:
            async:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                retry_strategy:
                    max_retries: 3
                    delay: 1000
                    multiplier: 2
            failed: 'doctrine://default?queue_name=failed'
        routing:
            App\Message\Process{Name}Message: async
```
