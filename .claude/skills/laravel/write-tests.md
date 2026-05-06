# Skill: Write Tests (Laravel — Pest)

## Checklist
- [ ] 1.  `uses(RefreshDatabase::class)` on all feature tests
- [ ] 2.  `Queue::fake()`, `Storage::fake()`, `Http::fake()` — never real services
- [ ] 3.  `actingAs($user, 'sanctum')` for authenticated routes
- [ ] 4.  Cover: success, 422, 401, 403, 404, 502 as applicable
- [ ] 5.  `php artisan test --coverage`

## Feature test template
```php
<?php declare(strict_types=1);

uses(\Illuminate\Foundation\Testing\RefreshDatabase::class);

describe('POST /api/v1/{resources}', function () {
    beforeEach(function () {
        $this->user = \App\Models\User::factory()->create();
        $this->actingAs($this->user, 'sanctum');
        \Illuminate\Support\Facades\Queue::fake();
        \Illuminate\Support\Facades\Storage::fake('private');
        \Illuminate\Support\Facades\Http::fake([
            // 'external-api.com/*' => Http::response([...], 200),
        ]);
    });

    it('creates resource and returns 201', function () {
        $this->postJson('/api/v1/{resources}', [/* valid payload */])
             ->assertStatus(201)
             ->assertJsonStructure(['data' => ['id', 'status']]);
        $this->assertDatabaseHas('{table}', ['user_id' => $this->user->id]);
    });

    it('returns 422 when payload invalid', function () {
        $this->postJson('/api/v1/{resources}', [])
             ->assertStatus(422)
             ->assertJsonValidationErrors([/* required fields */]);
    });

    it('returns 401 when unauthenticated', function () {
        $this->withoutMiddleware()->postJson('/api/v1/{resources}', [])->assertStatus(401);
    });
});
```

## Unit test template
```php
<?php declare(strict_types=1);
use App\Services\{Name}Service;
use Illuminate\Support\Facades\Http;

describe('{Name}Service', function () {
    it('processes successfully', function () {
        Http::fake(['api.example.com/*' => Http::response([/* expected response */], 200)]);
        $result = app({Name}Service::class)->process(/* input */);
        expect($result)->toBeArray()->toHaveKeys([/* expected keys */]);
    });

    it('throws {Name}Exception on provider failure', function () {
        Http::fake(['api.example.com/*' => Http::response([], 503)]);
        expect(fn () => app({Name}Service::class)->process(/* input */))
            ->toThrow(\App\Exceptions\{Name}Exception::class);
    });
});
```
