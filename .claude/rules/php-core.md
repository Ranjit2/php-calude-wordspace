# Core PHP Rules
# Loaded when no framework is detected in composer.json.
# No Eloquent. No Symfony components. Pure PHP 8.x + PSR standards.

## Required packages (composer.json)
```json
{
  "require": {
    "php":                "^8.3",
    "php-di/php-di":      "^7.0",
    "vlucas/phpdotenv":   "^5.6",
    "guzzlehttp/guzzle":  "^7.9",
    "monolog/monolog":    "^3.0",
    "ramsey/uuid":        "^4.0",
    "nikic/fast-route":   "^1.3"
  },
  "require-dev": {
    "pestphp/pest":           "^3.0",
    "mockery/mockery":        "^1.6",
    "phpstan/phpstan":        "^1.0",
    "friendsofphp/php-cs-fixer": "^3.0"
  },
  "autoload": {
    "psr-4": { "App\\": "src/" }
  }
}
```

## Project structure
```
src/
├── Bootstrap/
│   ├── app.php          ← DI container setup
│   └── router.php       ← route definitions
├── Config/              ← config files (read from $_ENV)
├── Console/             ← CLI entry points
├── Controllers/         ← thin HTTP entry points
├── Data/                ← DTOs and Value Objects
├── Enums/
├── Events/
├── Exceptions/
├── Middleware/          ← PSR-15 middleware
├── Models/              ← plain PHP models (not ORM)
├── Repositories/
│   └── Contracts/
├── Services/
│   └── Contracts/
└── Workers/             ← background job processors

public/
└── index.php            ← front controller

config/
├── app.php
├── database.php
└── services.php

migrations/
└── 001_create_users_table.sql
```

## Front controller (public/index.php)
```php
<?php declare(strict_types=1);

require __DIR__ . '/../vendor/autoload.php';

$dotenv = Dotenv\Dotenv::createImmutable(__DIR__ . '/..');
$dotenv->load();

$container = require __DIR__ . '/../src/Bootstrap/app.php';
$router    = require __DIR__ . '/../src/Bootstrap/router.php';

$dispatcher = FastRoute\simpleDispatcher($router);
$httpMethod = $_SERVER['REQUEST_METHOD'];
$uri        = rawurldecode(parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH));

$routeInfo = $dispatcher->dispatch($httpMethod, $uri);

match($routeInfo[0]) {
    FastRoute\Dispatcher::NOT_FOUND         => http_response_code(404),
    FastRoute\Dispatcher::METHOD_NOT_ALLOWED => http_response_code(405),
    FastRoute\Dispatcher::FOUND             => (function () use ($routeInfo, $container): void {
        [$handler, $vars] = [$routeInfo[1], $routeInfo[2]];
        [$class, $method] = $handler;
        $controller = $container->get($class);
        $controller->$method(...$vars);
    })(),
};
```

## DI container (src/Bootstrap/app.php)
```php
<?php declare(strict_types=1);

use DI\ContainerBuilder;

$builder = new ContainerBuilder();
$builder->addDefinitions([
    PDO::class => function () {
        $pdo = new PDO(
            dsn:      $_ENV['DB_DSN'],
            username: $_ENV['DB_USER'],
            password: $_ENV['DB_PASS'],
            options:  [
                PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                PDO::ATTR_EMULATE_PREPARES   => false,
            ],
        );
        return $pdo;
    },

    // Bind interfaces to implementations
    \App\Repositories\Contracts\{Name}RepositoryInterface::class
        => DI\autowire(\App\Repositories\{Name}Repository::class),

    \App\Services\Contracts\{External}Interface::class
        => DI\autowire(\App\Services\{Provider}Service::class),
]);

return $builder->build();
```

## Config pattern (config/services.php)
```php
<?php declare(strict_types=1);

return [
    // Add one section per external service
    'stripe' => [
        'key'     => $_ENV['STRIPE_API_KEY']
                     ?? throw new \RuntimeException('STRIPE_API_KEY not set'),
        'base_url'=> $_ENV['STRIPE_BASE_URL'] ?? 'https://api.stripe.com/v1',
    ],
    // 'sendgrid' => [...],
    // 'openai'   => [...],
];
```

## Repository with PDO
```php
<?php declare(strict_types=1);

namespace App\Repositories;

use App\Repositories\Contracts\{Name}RepositoryInterface;

final class {Name}Repository implements {Name}RepositoryInterface {
    public function __construct(private readonly \PDO $pdo) {}

    public function findById(int $id): ?array {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM {table} WHERE id = :id AND deleted_at IS NULL LIMIT 1'
        );
        $stmt->execute(['id' => $id]);
        return $stmt->fetch() ?: null;
    }

    public function findAllByUser(int $userId, int $page = 1, int $perPage = 15): array {
        $offset = ($page - 1) * $perPage;
        $stmt   = $this->pdo->prepare(
            'SELECT * FROM {table} WHERE user_id = :uid AND deleted_at IS NULL
             ORDER BY created_at DESC LIMIT :limit OFFSET :offset'
        );
        $stmt->bindValue(':uid',    $userId,  \PDO::PARAM_INT);
        $stmt->bindValue(':limit',  $perPage, \PDO::PARAM_INT);
        $stmt->bindValue(':offset', $offset,  \PDO::PARAM_INT);
        $stmt->execute();
        return $stmt->fetchAll();
    }

    public function create(array $data): int {
        $columns = implode(', ', array_keys($data));
        $placeholders = ':' . implode(', :', array_keys($data));
        $stmt = $this->pdo->prepare(
            "INSERT INTO {table} ({$columns}) VALUES ({$placeholders})"
        );
        $stmt->execute($data);
        return (int) $this->pdo->lastInsertId();
    }

    public function update(int $id, array $data): bool {
        $sets = implode(', ', array_map(fn($k) => "{$k} = :{$k}", array_keys($data)));
        $data['id'] = $id;
        $stmt = $this->pdo->prepare("UPDATE {table} SET {$sets} WHERE id = :id");
        $stmt->execute($data);
        return $stmt->rowCount() > 0;
    }

    public function delete(int $id): bool {
        $stmt = $this->pdo->prepare(
            'UPDATE {table} SET deleted_at = NOW() WHERE id = :id'
        );
        $stmt->execute(['id' => $id]);
        return $stmt->rowCount() > 0;
    }
}
```

## HTTP client (Guzzle)
```php
// All external HTTP calls go through a service that implements an interface
// Never use Guzzle directly in a controller or service constructor

final class {Provider}Service implements {External}Interface {
    private readonly \GuzzleHttp\Client $client;

    public function __construct(array $config) {
        $this->client = new \GuzzleHttp\Client([
            'base_uri' => $config['base_url'],
            'timeout'  => $config['timeout'] ?? 30,
            'headers'  => [
                'Authorization' => 'Bearer ' . $config['key'],
                'Content-Type'  => 'application/json',
                'Accept'        => 'application/json',
            ],
        ]);
    }

    public function post(string $endpoint, array $payload): array {
        try {
            $response = $this->client->post($endpoint, ['json' => $payload]);
        } catch (\GuzzleHttp\Exception\ConnectException $e) {
            throw new \App\Exceptions\ExternalServiceException(
                'Connection failed: ' . $e->getMessage(), previous: $e
            );
        } catch (\GuzzleHttp\Exception\ClientException $e) {
            throw new \App\Exceptions\ExternalServiceException(
                'Client error: ' . $e->getResponse()->getStatusCode(), previous: $e
            );
        }

        return json_decode($response->getBody()->getContents(), true, 512, JSON_THROW_ON_ERROR);
    }
}
```

## Error handling bootstrap
```php
// public/index.php — before routing
set_exception_handler(function (\Throwable $e): void {
    $code    = $e instanceof \App\Exceptions\HttpException ? $e->getCode() : 500;
    $message = app()->isProduction() ? 'An error occurred.' : $e->getMessage();
    http_response_code($code);
    header('Content-Type: application/json');
    echo json_encode(['error' => ['code' => 'SERVER_ERROR', 'message' => $message]]);
    error_log($e->getMessage() . ' in ' . $e->getFile() . ':' . $e->getLine());
});

set_error_handler(function (int $errno, string $errstr, string $errfile, int $errline): bool {
    throw new \ErrorException($errstr, 0, $errno, $errfile, $errline);
});
```

## Run commands
```bash
composer install
php -S localhost:8000 -t public/    # development server
./vendor/bin/php-cs-fixer fix       # code style
./vendor/bin/phpstan analyse src/   # static analysis
./vendor/bin/pest                   # tests
```
