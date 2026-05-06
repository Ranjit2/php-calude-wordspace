# Skill: Write Tests (Core PHP — Pest + Mockery)

## Checklist
- [ ] 1.  Pest for test runner — PHPUnit for assertions when preferred
- [ ] 2.  Mockery for mocking interfaces — `Mockery::close()` in afterEach
- [ ] 3.  Guzzle `MockHandler` for external HTTP calls — never real HTTP
- [ ] 4.  In-memory array for cache in tests — never real Redis
- [ ] 5.  Cover: service happy path, failure paths, edge cases
- [ ] 6.  `./vendor/bin/pest --coverage`

## Service unit test
```php
<?php declare(strict_types=1);

use App\Services\{Name}Service;
use App\Repositories\Contracts\{Name}RepositoryInterface;
use App\Services\Contracts\{External}Interface;
use App\Exceptions\{Name}Exception;

describe('{Name}Service', function () {

    beforeEach(function () {
        $this->repo     = Mockery::mock({Name}RepositoryInterface::class);
        $this->external = Mockery::mock({External}Interface::class);
        $this->cache    = new \Symfony\Component\Cache\Adapter\ArrayAdapter();
        $this->logger   = new \Psr\Log\NullLogger();
        $this->service  = new {Name}Service($this->repo, $this->external, $this->cache, $this->logger);
    });

    afterEach(fn () => Mockery::close());

    it('creates a resource and returns the new ID', function () {
        $this->repo->shouldReceive('create')->once()->andReturn(42);

        $data   = new \App\Data\{Name}Data(userId: 1, title: 'Test');
        $result = $this->service->create($data);

        expect($result)->toBe(42);
    });

    it('processes using cached result on second call', function () {
        $this->external->shouldReceive('execute')->once()->andReturn(['status' => 'ok']);
        $this->repo->shouldReceive('update')->twice();

        $this->service->process(1);
        $this->service->process(1); // second call uses cache — external called once
    });

    it('throws {Name}Exception when external service fails', function () {
        $this->external->shouldReceive('execute')
            ->andThrow(new \App\Exceptions\ExternalServiceException('timeout'));

        expect(fn () => $this->service->process(1))
            ->toThrow({Name}Exception::class);
    });
});
```

## HTTP test helper (tests/Helpers/TestRequest.php)
```php
<?php declare(strict_types=1);
namespace Tests\Helpers;

final class TestRequest {
    public static function post(string $path, array $body = [], array $headers = []): TestResponse {
        $_SERVER['REQUEST_METHOD'] = 'POST';
        $_SERVER['REQUEST_URI']    = $path;
        foreach ($headers as $k => $v) $_SERVER['HTTP_'.strtoupper(str_replace('-','_',$k))] = $v;

        // Capture output
        ob_start();
        // Bootstrap app with test container
        $app = require __DIR__ . '/../../src/Bootstrap/app.php';
        // ... run dispatcher
        $output = ob_get_clean();

        return new TestResponse(http_response_code(), json_decode($output, true));
    }
}
```
