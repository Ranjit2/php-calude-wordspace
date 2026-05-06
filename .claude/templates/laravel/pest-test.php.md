# Template: Pest Feature Test (Laravel)

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
            // 'api.provider.com/*' => Http::response([...], 200),
        ]);
    });

    it('creates resource and returns 201', function () {
        $this->postJson('/api/v1/{resources}', [/* valid payload */])
             ->assertStatus(201)
             ->assertJsonStructure(['data' => ['id', 'status', 'created_at']]);
        $this->assertDatabaseHas('{table}', ['user_id' => $this->user->id]);
        \Illuminate\Support\Facades\Queue::assertPushed(\App\Jobs\Process{Name}Job::class);
    });

    it('returns 422 when payload invalid', function () {
        $this->postJson('/api/v1/{resources}', [])
             ->assertStatus(422)
             ->assertJsonValidationErrors([/* required field names */]);
    });

    it('returns 401 when unauthenticated', function () {
        $this->withoutMiddleware()->postJson('/api/v1/{resources}', [])->assertStatus(401);
    });

    it('returns 403 when not owner', function () {
        $other    = \App\Models\User::factory()->create();
        $resource = \App\Models\{Name}::factory()->for($other)->create();
        $this->deleteJson("/api/v1/{resources}/{$resource->id}")->assertStatus(403);
    });
});
```
