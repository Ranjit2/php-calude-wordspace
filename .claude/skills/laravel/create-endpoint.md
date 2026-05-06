# Skill: Create Endpoint (Laravel)

## Checklist
- [ ] 1.  Define route in `routes/api.php` inside auth + throttle middleware group
- [ ] 2.  `php artisan make:request Store{Name}Request` — define `rules()` + `authorize()`
- [ ] 3.  Create DTO: `app/Data/{Name}Data.php` — `readonly` class with `fromRequest()`
- [ ] 4.  Create or extend service: `app/Services/{Name}Service.php`
- [ ] 5.  Bind interface in `AppServiceProvider::register()` if interface was created
- [ ] 6.  `php artisan make:controller Api/V1/{Name}Controller` — thin, delegate to service
- [ ] 7.  `php artisan make:resource {Name}Resource` — shape response, never `toArray()`
- [ ] 8.  Add Policy gate if action touches owned or restricted data
- [ ] 9.  Write Pest feature test: 201, 422, 401, 403, 404, 502 as applicable
- [ ] 10. `./vendor/bin/pint && php artisan test`

## Controller template
```php
<?php declare(strict_types=1);
namespace App\Http\Controllers\Api\V1;

use App\Data\{Name}Data;
use App\Http\Controllers\Controller;
use App\Http\Requests\Store{Name}Request;
use App\Http\Resources\{Name}Resource;
use App\Models\{Name};
use App\Services\{Name}Service;
use Illuminate\Http\JsonResponse;

final class {Name}Controller extends Controller {
    public function __construct(private readonly {Name}Service $service) {}

    public function store(Store{Name}Request $request): JsonResponse {
        $result = $this->service->create({Name}Data::fromRequest($request));
        return (new {Name}Resource($result))->response()->setStatusCode(201);
    }

    public function show({Name} $model): JsonResponse {
        $this->authorize('view', $model);
        return new {Name}Resource($model);
    }

    public function destroy({Name} $model): JsonResponse {
        $this->authorize('delete', $model);
        $this->service->delete($model);
        return response()->json(null, 204);
    }
}
```

## FormRequest template
```php
<?php declare(strict_types=1);
namespace App\Http\Requests;
use Illuminate\Foundation\Http\FormRequest;

final class Store{Name}Request extends FormRequest {
    public function authorize(): bool { return auth()->check(); }
    public function rules(): array {
        return [
            // 'title' => ['required', 'string', 'max:255'],
            // 'file'  => ['required', 'file', 'mimes:pdf', 'max:10240'],
        ];
    }
}
```

## API Resource template
```php
<?php declare(strict_types=1);
namespace App\Http\Resources;
use Illuminate\Http\Resources\Json\JsonResource;
use Illuminate\Http\Request;

final class {Name}Resource extends JsonResource {
    public function toArray(Request $request): array {
        return [
            'id'         => $this->id,
            // map your fields here
            // rename internal columns to API-facing names if needed
            'status'     => $this->status,
            'created_at' => $this->created_at?->toIso8601String(),
        ];
    }
}
```
