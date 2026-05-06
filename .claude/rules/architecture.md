# Architecture Rules
# Universal layer pattern — applies to Laravel, Symfony, and core PHP.
# Framework specifics live in rules/laravel.md, rules/symfony.md, rules/php-core.md.

---

## The universal layer pattern

```
HTTP / CLI request
       ↓
  Entry point        (Controller / Command / Handler)
       ↓               — receive input, delegate, return output
  Service layer      (PostService, BillingService)
       ↓               — owns all business logic
  Repository layer   (PostRepository, UserRepository)
       ↓               — owns all data access
  Data store         (MySQL, PostgreSQL, Redis, external API)
```

| Layer          | Responsibility                          | May call              | May NOT call           |
|----------------|-----------------------------------------|-----------------------|------------------------|
| Entry point    | Parse input, delegate, return output    | Service only          | Repository, DB, HTTP   |
| Service        | Business logic, orchestration           | Repository, Job/Worker, Event, external interface | Entry point, HTTP request object |
| Repository     | Data access and query building          | ORM / PDO / query builder | Service, HTTP        |
| Job / Worker   | Async processing                        | Service only          | Repository directly    |
| Event listener | Side effects triggered by domain events | Service only          | Entry point            |
| DTO            | Carry typed data between layers         | Nothing               | Everything             |
| Value Object   | Validated domain primitive              | Nothing               | Everything             |
| Policy         | Authorisation decisions                 | Model / Entity only   | Service, HTTP          |

---

## Module folder structure (medium to large projects)

```
src/  (or app/ in Laravel)
├── Console/              ← CLI entry points
├── Data/                 ← DTOs and Value Objects
│   ├── Post/
│   │   ├── CreatePostData.php
│   │   └── UpdatePostData.php
│   └── Shared/
│       └── PaginationData.php
├── Enums/                ← PHP 8.1+ backed enums
│   ├── PostStatus.php
│   └── UserRole.php
├── Events/               ← domain event classes
├── Exceptions/           ← domain-specific exceptions
│   ├── DomainException.php      ← base class
│   └── PostNotFoundException.php
├── Http/  (or Controller/ in core PHP)
│   ├── Controllers/
│   ├── Middleware/
│   ├── Requests/         ← Laravel FormRequest / Symfony DTO
│   └── Resources/        ← Laravel Resource / Symfony Serializer groups
├── Jobs/  (Laravel) / Messages/ (Symfony) / Workers/ (core PHP)
├── Listeners/
├── Models/  (Laravel) / Entities/ (Symfony) / Models/ (core PHP)
├── Policies/  (or Guards/)
├── Repositories/
│   └── Contracts/        ← interfaces
├── Services/
│   └── Contracts/        ← interfaces for swappable services
└── Values/               ← Value Objects (Email, Money, Slug)
```

---

## Request lifecycle — all three frameworks

```
# Laravel
HTTP → Middleware → FormRequest (validate+authorise) → Controller
     → Service → Repository → Model
     → Job dispatched (async) → Service → Event → Listener

# Symfony
HTTP → Middleware/EventListener → Controller (#[MapRequestPayload])
     → Service → Repository → Entity (Doctrine)
     → Message dispatched (Messenger) → Handler → Service → Event

# Core PHP
HTTP → Router → Middleware chain → Controller
     → Service → Repository → PDO
     → Worker job enqueued → Worker process → Service → Event
```

---

## Hard rules (all frameworks, no exceptions)

1. **Entry points contain zero business logic** — validate, delegate, respond
2. **Services are framework-agnostic** — no HTTP objects, no ORM imports inside service logic
3. **Repositories are the only place queries live** — no `findBy` in controllers or services
4. **DTOs are immutable** — `readonly` classes, never mutated after construction
5. **Interfaces on all external dependencies** — AI provider, payment gateway, email, storage
6. **Jobs/workers call services** — never contain logic themselves
7. **Events are past-tense domain signals** — `PostPublished`, `UserRegistered`
8. **Listeners are single-responsibility** — one job per listener, no multi-tasking
9. **No `new ClassName()` inside methods** — always inject via constructor
10. **Exceptions are domain-specific** — not generic `\Exception` throws

---

## Interface pattern (works in all frameworks)

```php
// 1. Define the contract — framework-agnostic
interface PaymentGatewayInterface {
    public function charge(Money $amount, string $token): PaymentResult;
    public function refund(string $transactionId, Money $amount): RefundResult;
}

// 2. Implement for the real provider
final class StripePaymentService implements PaymentGatewayInterface {
    public function charge(Money $amount, string $token): PaymentResult { ... }
    public function refund(string $transactionId, Money $amount): RefundResult { ... }
}

// 3. Implement for testing
final class FakePaymentService implements PaymentGatewayInterface {
    public array $charges = [];
    public function charge(Money $amount, string $token): PaymentResult {
        $this->charges[] = $amount;
        return PaymentResult::success('fake_txn_' . uniqid());
    }
    public function refund(string $transactionId, Money $amount): RefundResult {
        return RefundResult::success();
    }
}

// 4. Bind in the framework
// Laravel:   $this->app->bind(PaymentGatewayInterface::class, StripePaymentService::class);
// Symfony:   App\Services\Contracts\PaymentGatewayInterface: '@App\Services\StripePaymentService'
// Core PHP:  $container->bind(PaymentGatewayInterface::class, fn() => new StripePaymentService(...));
```

---

## Exception hierarchy

```php
// Base domain exception — catch-all for domain failures
class DomainException extends \RuntimeException {}

// Specific domain exceptions
class PostNotFoundException extends DomainException {}
class PostAlreadyPublishedException extends DomainException {}
class InsufficientFundsException extends DomainException {}
class ExternalServiceException extends DomainException {}  // AI, payment, email, etc.

// Map to HTTP responses at the framework boundary:
// Laravel:   bootstrap/app.php withExceptions()
// Symfony:   ExceptionListener or #[AsEventListener] on KernelEvents::EXCEPTION
// Core PHP:  set_exception_handler() in bootstrap/app.php
```
