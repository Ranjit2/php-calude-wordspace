# Skill: Add API Authentication

> **Laravel:** Sanctum (recommended) or JWT via `tymon/jwt-auth`
> **Symfony:** LexikJWTAuthenticationBundle or API Platform token
> **Core PHP:** manual JWT (e.g. `firebase/php-jwt`) or API key middleware


## Options
- Laravel Sanctum — SPA + mobile token auth (recommended for most apps)
- JWT (tymon/jwt-auth or lexik/jwt-auth for Symfony)
- API Key — simple header-based for server-to-server

## Sanctum checklist (Laravel)
- [ ] `php artisan install:api`
- [ ] Add `HasApiTokens` trait to User model
- [ ] Add `auth:sanctum` middleware to API route group
- [ ] Create AuthController with login/logout/refresh actions
- [ ] Set token expiry: `createToken('api', ['*'], now()->addDays(30))`
- [ ] Write Feature tests for login success, wrong password, token revocation

## AuthController template (Sanctum)
```php
<?php declare(strict_types=1);

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Http\Requests\Auth\LoginRequest;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\ValidationException;

final class AuthController extends Controller {
    public function login(LoginRequest $request): JsonResponse {
        if (!Auth::attempt($request->only('email', 'password'))) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        $token = $request->user()->createToken(
            name:       'api-token',
            abilities:  ['*'],
            expiresAt:  now()->addDays(30),
        );

        return response()->json([
            'data' => [
                'token'      => $token->plainTextToken,
                'expires_at' => $token->accessToken->expires_at?->toIso8601String(),
                'user'       => [
                    'id'    => $request->user()->id,
                    'name'  => $request->user()->name,
                    'email' => $request->user()->email,
                ],
            ],
        ]);
    }

    public function logout(Request $request): JsonResponse {
        $request->user()->currentAccessToken()->delete();
        return response()->json(null, 204);
    }

    public function me(Request $request): JsonResponse {
        return response()->json(['data' => $request->user()]);
    }
}
```

## Rate limiter (AppServiceProvider::boot)
```php
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});

RateLimiter::for('login', function (Request $request) {
    return [
        Limit::perMinute(5)->by($request->input('email')),
        Limit::perMinute(20)->by($request->ip()),
    ];
});
```
