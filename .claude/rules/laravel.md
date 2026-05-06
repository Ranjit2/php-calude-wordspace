# Laravel Rules
# Loaded automatically when laravel/framework is detected in composer.json.

## Version
Laravel 10 / 11. Do not suggest pre-10 patterns.

## Routing
```php
Route::middleware(['auth:sanctum', 'throttle:api'])->prefix('v1')->group(function () {
    Route::apiResource('{resources}', Api\V1\{Name}Controller::class);
    Route::post('{resources}/{resource}/publish', [Api\V1\{Name}Controller::class, 'publish']);
});
```

## Controllers — thin
```php
final class {Name}Controller extends Controller {
    public function __construct(private readonly {Name}Service $service) {}

    public function store(Store{Name}Request $request): JsonResponse {
        $result = $this->service->create({Name}Data::fromRequest($request));
        return (new {Name}Resource($result))->response()->setStatusCode(201);
    }

    public function show({Name} $model): JsonResponse {
        $this->authorize('view', $model);
        return new {Name}Resource($model->load('relation'));
    }
}
```

## FormRequest — always, never $request->validate() in controllers
```php
final class Store{Name}Request extends FormRequest {
    public function authorize(): bool { return auth()->check(); }
    public function rules(): array {
        return [
            // 'field' => ['required', 'string', 'max:255'],
        ];
    }
}
```

## Eloquent model
```php
final class {Name} extends Model {
    use HasFactory, SoftDeletes;
    protected $fillable = [/* explicit list */];
    protected $hidden   = ['password', 'remember_token'];
    protected $casts    = [
        // 'status'       => {Name}Status::class,
        // 'metadata'     => 'array',
        // 'is_published' => 'boolean',
        // 'price'        => 'decimal:2',
        // 'published_at' => 'datetime',
    ];
}
```

## Service binding
```php
// app/Providers/AppServiceProvider.php — register()
$this->app->bind({Name}Interface::class, {Name}Service::class);
```

## Queues
```php
// Dispatch
Process{Name}Job::dispatch($model)->onQueue('default')->delay(now()->addSeconds(5));

// Job
final class Process{Name}Job implements ShouldQueue, ShouldBeUnique {
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    public int $tries = 3;
    public array $backoff = [30, 120, 300];
    public function uniqueId(): string { return static::class.'_'.$this->model->id; }
    public function handle({Name}Service $service): void { $service->process($this->model); }
    public function failed(\Throwable $e): void {
        $this->model->update(['status' => {Name}Status::Failed]);
        Log::error('Job failed', ['id' => $this->model->id, 'error' => $e->getMessage()]);
    }
}
```

## Caching
```php
Cache::remember("{resource}_{$id}", now()->addHour(), fn () => $this->repository->find($id));
Cache::tags(['{resource}', "user_{$userId}"])->flush();
```

## Events
```php
// Past-tense name, readonly constructor
final class {Name}Published {
    public function __construct(public readonly {Name} $model) {}
}
// Listener implements ShouldQueue for async side effects
```

## Sanctum auth
```php
$token = $user->createToken('api', ['*'], now()->addDays(30));
return response()->json(['token' => $token->plainTextToken]);
```

## Rate limiting (AppServiceProvider::boot)
```php
RateLimiter::for('{action}', function (Request $request) {
    return Limit::perMinute(10)->by($request->user()?->id ?: $request->ip());
});
```

## Run commands
```bash
php artisan make:model {Name} --migration --factory --policy
php artisan make:request Store{Name}Request
php artisan make:resource {Name}Resource
php artisan make:job Process{Name}Job
php artisan migrate
./vendor/bin/pint && php artisan test
```
