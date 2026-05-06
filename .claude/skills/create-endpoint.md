# Skill: Create a New Endpoint

> **Laravel:** `php artisan make:request` + `FormRequest` + `php artisan make:resource`
> **Symfony:** DTO + `#[MapRequestPayload]` + Symfony serializer
> **Core PHP:** DTO + manual validation + plain array/JSON response


## When to use
Adding any new route — JSON API endpoint, web page action, or async operation.

---

## Checklist (always in this order)

- [ ] 1.  Define the route in `routes/web.php` or `routes/api.php`
          Place inside the correct middleware group (auth, throttle)
- [ ] 2.  Create the request validator:
          - **Laravel:** `php artisan make:request {Action}{Resource}Request` → `FormRequest`
          - **Symfony:** DTO with `#[Assert\*]` attributes + `#[MapRequestPayload]`
          - **Core PHP:** DTO with manual validation in a `Validator` class
- [ ] 3.  Create DTO if data crosses layer boundaries: `app/Data/{Resource}Data.php`
          Use a `readonly` class with a static `fromRequest()` factory
- [ ] 4.  Create or extend the Service class: `app/Services/{Resource}Service.php`
          Bind its interface in `AppServiceProvider` if one exists
- [ ] 5.  Create the Controller action — thin, delegate everything to the service
- [ ] 6.  Shape the response — never return raw model/entity data:
          - **Laravel:** `php artisan make:resource {Resource}Resource` → `JsonResource`
          - **Symfony:** use Symfony Serializer groups or a dedicated Presenter class
          - **Core PHP:** a plain `{Resource}Presenter` class that returns a shaped array
- [ ] 7.  Add route model binding where the route uses `{model}`
- [ ] 8.  Add Policy gate if this action touches owned or restricted data
- [ ] 9.  Write Pest feature test covering:
          - Success case (200 / 201 / 202)
          - Validation failure (422)
          - Unauthenticated (401)
          - Forbidden (403) — if policy applies
          - Not found (404) — if route model binding used
          - External service failure (502) — if third-party API involved
- [ ] 10. Run quality checks:
          - **Laravel:** `./vendor/bin/pint && php artisan test`
          - **Symfony:** `./vendor/bin/php-cs-fixer fix && php bin/phpunit`
          - **Core PHP:** `./vendor/bin/php-cs-fixer fix && ./vendor/bin/pest`

---

## Controller template

```php
<?php declare(strict_types=1);

namespace App\Http\Controllers\Api\V1;

use App\Data\{Resource}Data;
use App\Http\Controllers\Controller;
use App\Http\Requests\Store{Resource}Request;
use App\Http\Resources\{Resource}Resource;
use App\Models\{Resource};
use App\Services\{Resource}Service;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Str;

final class {Resource}Controller extends Controller {
    public function __construct(
        private readonly {Resource}Service $service,
    ) {}

    public function index(): JsonResponse {
        $items = $this->service->paginate();
        return {Resource}Resource::collection($items)->response();
    }

    public function store(Store{Resource}Request $request): JsonResponse {
        $data   = {Resource}Data::fromRequest($request);
        $result = $this->service->create($data);

        return (new {Resource}Resource($result))
            ->response()
            ->setStatusCode(201);
    }

    public function show({Resource} $resource): JsonResponse {
        $this->authorize('view', $resource);
        return new {Resource}Resource($resource);
    }

    public function destroy({Resource} $resource): JsonResponse {
        $this->authorize('delete', $resource);
        $this->service->delete($resource);
        return response()->json(null, 204);
    }
}
```

---

## FormRequest template

```php
<?php declare(strict_types=1);

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

final class Store{Resource}Request extends FormRequest {
    public function authorize(): bool {
        return auth()->check();
    }

    public function rules(): array {
        return [
            // Add your validation rules here
            // e.g.
            // 'name'  => ['required', 'string', 'max:255'],
            // 'email' => ['required', 'email', 'unique:users,email'],
            // 'file'  => ['required', 'file', 'mimes:pdf', 'max:10240'],
        ];
    }

    public function messages(): array {
        return [
            // Override default validation messages if needed
            // e.g.
            // 'name.required' => 'Please provide a name.',
        ];
    }
}
```

---

## API Resource template

```php
<?php declare(strict_types=1);

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;
use Illuminate\Http\Request;

final class {Resource}Resource extends JsonResource {
    public function toArray(Request $request): array {
        return [
            'id'         => $this->id,
            // Add your fields here — rename internal columns to API-friendly names if needed
            // e.g. 'title' => $this->name,
            // e.g. 'author' => new UserResource($this->whenLoaded('user')),
            'status'     => $this->status,
            'created_at' => $this->created_at?->toIso8601String(),
        ];
    }
}
```
