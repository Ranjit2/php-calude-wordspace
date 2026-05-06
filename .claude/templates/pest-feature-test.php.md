# Template: Pest Feature Test

Replace all `{Name}` / `{resource}` / `{table}` placeholders with your domain names.
Delete any test cases that do not apply to your endpoint.
Add test cases specific to your business rules.

---

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

// ─── POST (create) ────────────────────────────────────────────────────────────

describe('POST /api/v1/{resources}', function () {

    beforeEach(function () {
        $this->user = User::factory()->create();
        $this->actingAs($this->user, 'sanctum');
        Queue::fake();
        Storage::fake('private');

        // Fake any external HTTP calls your service makes
        // Http::fake([
        //     'external-api.com/*' => Http::response([...], 200),
        // ]);
    });

    it('creates a {name} with valid data', function () {
        $response = $this->postJson('/api/v1/{resources}', [
            // Valid request payload — fill in your fields
            // e.g. 'name' => 'Test item', 'description' => 'A description'
        ]);

        $response->assertStatus(201)
                 ->assertJsonStructure(['data' => [
                     'id',
                     'status',
                     'created_at',
                     // Add expected response fields
                 ]]);

        $this->assertDatabaseHas('{table}', [
            'user_id' => $this->user->id,
            // Assert key fields were persisted correctly
        ]);
    });

    it('dispatches processing job after creation', function () {
        $this->postJson('/api/v1/{resources}', [
            // Valid request payload
        ])->assertStatus(201);

        Queue::assertPushed(Process{Name}Job::class);
    });

    it('fires an event after creation', function () {
        Event::fake();

        $this->postJson('/api/v1/{resources}', [
            // Valid request payload
        ])->assertStatus(201);

        // Uncomment if your service dispatches an event
        // Event::assertDispatched({Name}Created::class);
    });

    it('returns 422 when required fields are missing', function () {
        $this->postJson('/api/v1/{resources}', [])
             ->assertStatus(422)
             ->assertJsonPath('error.code', 'VALIDATION_FAILED')
             ->assertJsonValidationErrors([
                 // List the required field names
                 // e.g. 'name', 'type'
             ]);
    });

    it('returns 422 when field format is invalid', function () {
        $this->postJson('/api/v1/{resources}', [
            // Intentionally invalid data — e.g. wrong type, too long
        ])->assertStatus(422);
    });

    it('returns 401 when not authenticated', function () {
        $this->withoutMiddleware()
             ->postJson('/api/v1/{resources}', [])
             ->assertStatus(401);
    });

    // Uncomment if your endpoint calls an external service
    // it('returns 502 when external service fails', function () {
    //     Http::fake(['external-api.com/*' => Http::response([], 503)]);
    //
    //     $this->postJson('/api/v1/{resources}', [/* valid data */])
    //          ->assertStatus(502)
    //          ->assertJsonPath('error.code', 'PROVIDER_ERROR');
    // });
});

// ─── GET (list) ───────────────────────────────────────────────────────────────

describe('GET /api/v1/{resources}', function () {

    beforeEach(function () {
        $this->user = User::factory()->create();
        $this->actingAs($this->user, 'sanctum');
    });

    it('returns paginated list of {resources}', function () {
        {Name}::factory(5)->for($this->user)->create();

        $this->getJson('/api/v1/{resources}')
             ->assertStatus(200)
             ->assertJsonStructure([
                 'data' => [['id', 'status', 'created_at']],
                 'meta' => ['current_page', 'per_page', 'total', 'last_page'],
             ]);
    });

    it('does not return other users {resources}', function () {
        $otherUser = User::factory()->create();
        {Name}::factory(3)->for($otherUser)->create();

        $response = $this->getJson('/api/v1/{resources}');
        $response->assertStatus(200);
        expect($response->json('meta.total'))->toBe(0);
    });

    it('returns 401 when not authenticated', function () {
        $this->withoutMiddleware()
             ->getJson('/api/v1/{resources}')
             ->assertStatus(401);
    });
});

// ─── GET (single) ─────────────────────────────────────────────────────────────

describe('GET /api/v1/{resources}/{id}', function () {

    beforeEach(function () {
        $this->user = User::factory()->create();
        $this->actingAs($this->user, 'sanctum');
    });

    it('returns a single {name}', function () {
        $model = {Name}::factory()->for($this->user)->create();

        $this->getJson("/api/v1/{resources}/{$model->id}")
             ->assertStatus(200)
             ->assertJsonPath('data.id', $model->id);
    });

    it('returns 404 for unknown id', function () {
        $this->getJson('/api/v1/{resources}/99999')
             ->assertStatus(404);
    });

    it('returns 403 when resource belongs to another user', function () {
        $otherUser = User::factory()->create();
        $model     = {Name}::factory()->for($otherUser)->create();

        $this->getJson("/api/v1/{resources}/{$model->id}")
             ->assertStatus(403);
    });
});
```
