# Skill: Write Tests

> **Laravel:** Pest + `RefreshDatabase` + `Http::fake()` + `Queue::fake()`
> **Symfony:** PHPUnit + `KernelTestCase` + mock HTTP client
> **Core PHP:** Pest or PHPUnit + Mockery + Guzzle mock handler


## Rule: every task ends with a test
No feature, endpoint, service method, or job is complete until a test exists.

---

## Test types

| Type    | Covers                                     | Mocks external? |
|---------|--------------------------------------------|-----------------|
| Feature | Full HTTP request → database → response    | Yes — Http::fake |
| Unit    | Single class in isolation                  | Yes — always    |
| Browser | JS-rendered UI (Laravel Dusk)              | No              |

---

## Coverage targets

- Controllers: 100% of public action methods
- Services: 80% minimum — happy path + main failure paths
- Jobs: `handle()` and `failed()` both covered
- Models: scopes, accessors, key relationships

---

## Feature test template

```php
<?php declare(strict_types=1);

use App\Enums\{Name}Status;
use App\Jobs\Process{Name}Job;
use App\Models\{Name};
use App\Models\User;
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Queue;
use Illuminate\Support\Facades\Storage;

uses(\Illuminate\Foundation\Testing\RefreshDatabase::class);

describe('POST /api/v1/{resources}', function () {

    beforeEach(function () {
        // Authenticate a test user
        $this->user = User::factory()->create();
        $this->actingAs($this->user, 'sanctum');

        // Fake external services — never call real APIs in tests
        Storage::fake('private');
        Queue::fake();
        Http::fake([
            // Replace with the external API your service calls
            // e.g. 'api.stripe.com/*' => Http::response(['id' => 'ch_test'], 200),
        ]);
    });

    it('creates a {name} and returns 201', function () {
        $response = $this->postJson('/api/v1/{resources}', [
            // Add valid request data here
        ]);

        $response->assertStatus(201)
                 ->assertJsonStructure(['data' => ['id', 'status']]);

        $this->assertDatabaseHas('{table}', [
            'user_id' => $this->user->id,
            // Assert key fields were saved
        ]);
    });

    it('dispatches a job after creation', function () {
        $this->postJson('/api/v1/{resources}', [
            // Valid request data
        ])->assertStatus(201);

        Queue::assertPushed(Process{Name}Job::class);
    });

    it('returns 422 when required fields are missing', function () {
        $this->postJson('/api/v1/{resources}', [])
             ->assertStatus(422)
             ->assertJsonPath('error.code', 'VALIDATION_FAILED')
             ->assertJsonValidationErrors([
                 // List the required fields
                 // e.g. 'name', 'email'
             ]);
    });

    it('returns 401 when unauthenticated', function () {
        $this->withoutMiddleware()
             ->postJson('/api/v1/{resources}', [])
             ->assertStatus(401);
    });

    // Add more cases:
    // it('returns 403 when user does not own the resource', ...)
    // it('returns 404 when resource not found', ...)
    // it('returns 429 when rate limit exceeded', ...)
    // it('returns 502 when external service fails', ...)
});

describe('GET /api/v1/{resources}', function () {

    beforeEach(function () {
        $this->user = User::factory()->create();
        $this->actingAs($this->user, 'sanctum');
    });

    it('returns paginated list', function () {
        {Name}::factory(5)->for($this->user)->create();

        $this->getJson('/api/v1/{resources}')
             ->assertStatus(200)
             ->assertJsonStructure([
                 'data' => [['id', 'status', 'created_at']],
                 'meta' => ['current_page', 'per_page', 'total'],
             ]);
    });

    it('does not return other users records', function () {
        $otherUser = User::factory()->create();
        {Name}::factory(3)->for($otherUser)->create();

        $response = $this->getJson('/api/v1/{resources}');
        $response->assertStatus(200);
        expect($response->json('meta.total'))->toBe(0);
    });
});
```

---

## Unit test template

```php
<?php declare(strict_types=1);

use App\Services\{Name}Service;
use Illuminate\Support\Facades\Http;

describe('{Name}Service', function () {

    it('processes successfully with valid input', function () {
        // Arrange — mock external dependencies
        Http::fake([
            'your-external-api.com/*' => Http::response([
                // Mock successful response
            ], 200),
        ]);

        // Act
        $result = app({Name}Service::class)->process(/* input */);

        // Assert
        expect($result)->toBeArray()->toHaveKeys([
            // Expected keys in result
        ]);
    });

    it('throws {Name}Exception when external service fails', function () {
        Http::fake([
            'your-external-api.com/*' => Http::response([], 503),
        ]);

        expect(fn () => app({Name}Service::class)->process(/* input */))
            ->toThrow(\App\Exceptions\{Name}Exception::class);
    });

    it('returns empty result on malformed response', function () {
        Http::fake([
            'your-external-api.com/*' => Http::response([
                'unexpected' => 'format',
            ], 200),
        ]);

        $result = app({Name}Service::class)->process(/* input */);
        expect($result)->toBe([]);
    });
});
```

---

## Anti-patterns to avoid

```php
// Never call real external APIs in tests
// Wrong:
Http::post('https://api.openai.com/...', [...]);
// Right: Http::fake(['api.openai.com/*' => Http::response([...], 200)])

// Never use sleep() in tests
// Wrong: sleep(2);
// Right: Carbon::setTestNow(now()->addSeconds(2));

// Never test private methods directly
// Test behaviour through the public API instead

// Never rely on test execution order
// Each test must be independent (use RefreshDatabase)

// Never use $this->assertTrue(true) as a placeholder
// Write a real assertion before committing
```
