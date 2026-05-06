# Skill: Create Endpoint (Core PHP)

## Checklist
- [ ] 1.  Register route in `src/Bootstrap/router.php`
- [ ] 2.  Create DTO with typed constructor + static `fromInput()` factory
- [ ] 3.  Create Validator class — whitelist fields, validate types, return errors
- [ ] 4.  Create or extend service in `src/Services/{Name}Service.php`
- [ ] 5.  Bind interface in `src/Bootstrap/app.php` DI container
- [ ] 6.  Create Controller — thin, validate → service → respond
- [ ] 7.  Create Presenter — shapes response array, never expose raw DB row
- [ ] 8.  Write Pest test using Mockery for all dependencies
- [ ] 9.  `./vendor/bin/php-cs-fixer fix && ./vendor/bin/pest`

## Router registration
```php
// src/Bootstrap/router.php
return function (\FastRoute\RouteCollector $r): void {
    $r->addGroup('/api/v1', function (\FastRoute\RouteCollector $r) {
        $r->post('/{resources}',      [\App\Controllers\{Name}Controller::class, 'store']);
        $r->get('/{resources}',       [\App\Controllers\{Name}Controller::class, 'index']);
        $r->get('/{resources}/{id:\d+}', [\App\Controllers\{Name}Controller::class, 'show']);
        $r->delete('/{resources}/{id:\d+}', [\App\Controllers\{Name}Controller::class, 'destroy']);
    });
};
```

## Controller template
```php
<?php declare(strict_types=1);
namespace App\Controllers;

use App\Data\{Name}Data;
use App\Services\{Name}Service;
use App\Presenters\{Name}Presenter;
use App\Http\Request;
use App\Http\JsonResponse;

final class {Name}Controller {
    public function __construct(
        private readonly {Name}Service   $service,
        private readonly {Name}Presenter $presenter,
    ) {}

    public function store(): void {
        $input = json_decode(file_get_contents('php://input'), true, 512, JSON_THROW_ON_ERROR);

        $errors = {Name}Validator::validate($input);
        if ($errors) {
            JsonResponse::send(['error' => ['code' => 'VALIDATION_FAILED', 'details' => $errors]], 422);
            return;
        }

        $data   = {Name}Data::fromInput($input, $this->auth->userId());
        $result = $this->service->create($data);

        JsonResponse::send(['data' => $this->presenter->format($result)], 201);
    }
}
```

## DTO template
```php
<?php declare(strict_types=1);
namespace App\Data;

final readonly class {Name}Data {
    public function __construct(
        public readonly int    $userId,
        public readonly string $title,
        // add your fields
    ) {}

    public static function fromInput(array $input, int $userId): self {
        return new self(
            userId: $userId,
            title:  trim((string) ($input['title'] ?? '')),
            // map validated input fields
        );
    }
}
```

## Presenter template (shapes API response)
```php
<?php declare(strict_types=1);
namespace App\Presenters;

final class {Name}Presenter {
    public function format(array $row): array {
        return [
            'id'         => $row['id'],
            // map internal column names to API-facing names
            'status'     => $row['status'],
            'created_at' => (new \DateTimeImmutable($row['created_at']))->format(\DateTimeInterface::ATOM),
        ];
    }

    public function formatMany(array $rows): array {
        return array_map($this->format(...), $rows);
    }
}
```

## JsonResponse helper
```php
<?php declare(strict_types=1);
namespace App\Http;

final class JsonResponse {
    public static function send(array $data, int $status = 200): void {
        http_response_code($status);
        header('Content-Type: application/json; charset=utf-8');
        echo json_encode($data, JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR);
    }
}
```
