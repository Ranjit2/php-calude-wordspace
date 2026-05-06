# Testing Rules
# Covers Laravel (Pest), Symfony (PHPUnit), and Core PHP (Pest/PHPUnit).

## The rule: every task ends with a test
No feature, endpoint, service, or worker is done until a test exists.
Tests are not optional. They are part of the definition of done.

---

## Coverage targets (all frameworks)
- Entry points (controllers, handlers, commands): 100% of public actions
- Services: 80% minimum — happy path + main failure paths
- Jobs/workers/handlers: `handle()` and failure path both covered
- Repositories: key query methods — especially filtered/sorted queries
- Skip: config files, migration files, DI bootstrap

---

## Test anatomy

```
Arrange  → set up the state (models, mocks, fakes)
Act      → call the method or make the HTTP request
Assert   → verify the outcome (response, DB state, events, jobs)
```

---

## Laravel — Pest

```php
<?php declare(strict_types=1);

uses(\Illuminate\Foundation\Testing\RefreshDatabase::class);

describe('POST /api/v1/{resources}', function () {

    beforeEach(function () {
        $this->user = \App\Models\User::factory()->create();
        $this->actingAs($this->user, 'sanctum');

        // Fake external dependencies — never call real services in tests
        \Illuminate\Support\Facades\Queue::fake();
        \Illuminate\Support\Facades\Storage::fake('private');
        \Illuminate\Support\Facades\Http::fake([
            'external-api.com/*' => \Illuminate\Support\Facades\Http::response(
                ['id' => 'test_123', 'status' => 'ok'], 200
            ),
        ]);
    });

    it('creates a resource and returns 201', function () {
        $response = $this->postJson('/api/v1/{resources}', [
            // valid payload
        ]);

        $response->assertStatus(201)
                 ->assertJsonStructure(['data' => ['id', 'status']]);

        $this->assertDatabaseHas('{table}', [
            'user_id' => $this->user->id,
        ]);
    });

    it('returns 422 when required fields are missing', function () {
        $this->postJson('/api/v1/{resources}', [])
             ->assertStatus(422)
             ->assertJsonValidationErrors([/* required fields */]);
    });

    it('returns 401 when unauthenticated', function () {
        $this->withoutMiddleware()
             ->postJson('/api/v1/{resources}', [])
             ->assertStatus(401);
    });

    it('returns 403 when user does not own the resource', function () {
        $other    = \App\Models\User::factory()->create();
        $resource = \App\Models\{Name}::factory()->for($other)->create();

        $this->deleteJson("/api/v1/{resources}/{$resource->id}")
             ->assertStatus(403);
    });
});
```

---

## Symfony — PHPUnit + WebTestCase

```php
<?php declare(strict_types=1);

namespace App\Tests\Controller;

use App\Entity\User;
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Symfony\Component\HttpFoundation\Response;

final class {Name}ControllerTest extends WebTestCase {
    private \Symfony\Bundle\FrameworkBundle\KernelBrowser $client;

    protected function setUp(): void {
        $this->client = static::createClient();
        // Authenticate user for protected routes
        // $this->client->loginUser($this->createUser());
    }

    public function testCreate{Name}Returns201(): void {
        $this->client->request('POST', '/api/v1/{resources}', [], [], [
            'CONTENT_TYPE' => 'application/json',
        ], json_encode([
            // valid payload
        ]));

        $this->assertResponseStatusCodeSame(Response::HTTP_CREATED);
        $data = json_decode($this->client->getResponse()->getContent(), true);
        $this->assertArrayHasKey('id', $data['data']);
    }

    public function testCreate{Name}Returns422WhenInvalid(): void {
        $this->client->request('POST', '/api/v1/{resources}', [], [], [
            'CONTENT_TYPE' => 'application/json',
        ], json_encode([]));

        $this->assertResponseStatusCodeSame(Response::HTTP_UNPROCESSABLE_ENTITY);
    }

    public function testCreate{Name}Returns401WhenUnauthenticated(): void {
        // Use anonymous client
        $client = static::createClient();
        $client->request('POST', '/api/v1/{resources}');

        $this->assertResponseStatusCodeSame(Response::HTTP_UNAUTHORIZED);
    }
}
```

### Symfony service unit test
```php
<?php declare(strict_types=1);

namespace App\Tests\Service;

use App\Service\{Name}Service;
use PHPUnit\Framework\TestCase;
use PHPUnit\Framework\MockObject\MockObject;

final class {Name}ServiceTest extends TestCase {
    private {Name}Service $service;
    private MockObject $repository;
    private MockObject $externalClient;

    protected function setUp(): void {
        $this->repository     = $this->createMock({Name}RepositoryInterface::class);
        $this->externalClient = $this->createMock(HttpClientInterface::class);
        $this->service        = new {Name}Service($this->repository, $this->externalClient);
    }

    public function testProcessSuccessfully(): void {
        $this->externalClient
            ->method('request')
            ->willReturn(new MockResponse(json_encode(['status' => 'ok'])));

        $result = $this->service->process(/* input */);

        $this->assertSame('ok', $result->status);
    }

    public function testThrowsExceptionWhenProviderFails(): void {
        $this->externalClient
            ->method('request')
            ->willThrowException(new TransportExceptionInterface('timeout'));

        $this->expectException(\App\Exception\ExternalServiceException::class);
        $this->service->process(/* input */);
    }
}
```

---

## Core PHP — Pest + Mockery

```php
<?php declare(strict_types=1);

use App\Services\{Name}Service;
use App\Repositories\Contracts\{Name}RepositoryInterface;
use App\Http\Clients\Contracts\HttpClientInterface;

describe('{Name}Service', function () {

    beforeEach(function () {
        $this->repository = Mockery::mock({Name}RepositoryInterface::class);
        $this->httpClient = Mockery::mock(HttpClientInterface::class);
        $this->service    = new {Name}Service($this->repository, $this->httpClient);
    });

    afterEach(fn () => Mockery::close());

    it('processes successfully with valid input', function () {
        $this->httpClient
            ->shouldReceive('post')
            ->once()
            ->andReturn(['status' => 'ok', 'id' => 'ext_123']);

        $this->repository
            ->shouldReceive('save')
            ->once();

        $result = $this->service->process(/* input */);

        expect($result->status)->toBe('ok');
    });

    it('throws ExternalServiceException when HTTP client fails', function () {
        $this->httpClient
            ->shouldReceive('post')
            ->andThrow(new \App\Exceptions\HttpTimeoutException('timeout'));

        expect(fn () => $this->service->process(/* input */))
            ->toThrow(\App\Exceptions\ExternalServiceException::class);
    });

    it('returns cached result on second call', function () {
        $this->httpClient->shouldReceive('post')->once()->andReturn(['status' => 'ok']);

        $this->service->process(/* input */);
        $this->service->process(/* input */); // should not call HTTP again
    });
});

// ── HTTP endpoint test (Core PHP with custom router) ─────────────────────
describe('POST /api/v1/{resources}', function () {

    it('returns 201 with valid payload', function () {
        $response = testRequest('POST', '/api/v1/{resources}', [
            // valid payload
        ]);

        expect($response->status)->toBe(201)
            ->and($response->json('data.id'))->not->toBeNull();
    });

    it('returns 422 when payload is invalid', function () {
        $response = testRequest('POST', '/api/v1/{resources}', []);
        expect($response->status)->toBe(422);
    });

    it('returns 401 without auth token', function () {
        $response = testRequest('POST', '/api/v1/{resources}', [], auth: false);
        expect($response->status)->toBe(401);
    });
});
```

---

## Anti-patterns (all frameworks)

```
Never call real external APIs in tests    → always mock/fake HTTP
Never use sleep()                         → use time mocking
Never test private methods directly       → test through the public API
Never rely on test execution order        → every test is independent
Never leave assertTrue(true) placeholder  → write a real assertion
Never share mutable state between tests   → reset in setUp/beforeEach
```
