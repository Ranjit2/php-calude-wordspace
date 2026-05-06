# Coding Standards
# Applies to all frameworks. Pure PHP — no Laravel, Symfony, or framework imports.

## Non-negotiable: first line of every PHP file
```php
<?php declare(strict_types=1);
```
No exceptions. A file without this is incomplete.

---

## Type system — fully typed, always

```php
// Every parameter typed
// Every return type declared
// No mixed, no untyped arrays where a DTO exists

// Wrong
public function process($data) {
    return $this->doSomething($data['id']);
}

// Right
public function process(ProcessData $data): ProcessResult {
    return $this->doSomething($data->id);
}
```

---

## PHP 8.x features — use actively in all frameworks

```php
// ── Enums (PHP 8.1) ──────────────────────────────────────────
// Always for status, type, and flag fields. Never magic strings.
enum Status: string {
    case Draft     = 'draft';
    case Active    = 'active';
    case Archived  = 'archived';

    public function label(): string {
        return match($this) {
            self::Draft    => 'Draft',
            self::Active   => 'Active',
            self::Archived => 'Archived',
        };
    }

    public function isPublishable(): bool {
        return $this === self::Draft;
    }

    /** @return list<string> */
    public static function values(): array {
        return array_column(self::cases(), 'value');
    }
}

// ── Readonly DTO (PHP 8.2) ────────────────────────────────────
// Always for data crossing layer boundaries. Never plain arrays.
final readonly class CreatePostData {
    public function __construct(
        public int    $authorId,
        public string $title,
        public string $body,
        public Status $status = Status::Draft,
    ) {}
}

// ── Constructor promotion ─────────────────────────────────────
// Always — removes boilerplate, keeps DI clean
final class PostService {
    public function __construct(
        private readonly PostRepositoryInterface    $repository,
        private readonly NotificationInterface     $notifier,
    ) {}
}

// ── Match expression ──────────────────────────────────────────
// Over if/else chains — works identically in all three frameworks
$response = match($status) {
    Status::Active   => 'Published and visible',
    Status::Draft    => 'Saved but not visible',
    Status::Archived => 'Hidden from all views',
};

// ── Nullsafe operator ─────────────────────────────────────────
$city = $user->address?->city ?? 'Unknown';

// ── Named arguments ───────────────────────────────────────────
// Use when more than 2 params or when meaning is not obvious
$data = new CreatePostData(
    authorId: $user->id,
    title:    $request->title,
    body:     $request->body,
);

// ── First-class callables (PHP 8.1) ───────────────────────────
$titles = array_map($this->formatTitle(...), $rawTitles);

// ── Fibers (PHP 8.1) ──────────────────────────────────────────
// For concurrent I/O in CLI scripts only — not in web requests

// ── Union types ───────────────────────────────────────────────
public function find(int|string $id): ?UserInterface {}

// ── Intersection types (PHP 8.1) ──────────────────────────────
public function process(Countable&Traversable $items): void {}
```

---

## Naming conventions

| Thing            | Convention     | Examples                                  |
|------------------|----------------|-------------------------------------------|
| Class            | PascalCase     | `PostService`, `UserRepository`           |
| Interface        | PascalCase+Interface | `PostRepositoryInterface`           |
| Abstract class   | Abstract+Name  | `AbstractRepository`                      |
| Enum             | PascalCase     | `PostStatus`, `UserRole`                  |
| Trait            | PascalCase     | `HasTimestamps`, `SoftDeletable`          |
| Method           | camelCase      | `createPost()`, `sendWelcomeEmail()`      |
| Property         | camelCase      | `$authorId`, `$publishedAt`               |
| Variable         | camelCase      | `$postData`, `$httpClient`                |
| Constant         | UPPER_SNAKE    | `MAX_FILE_SIZE`, `DEFAULT_TIMEOUT`        |
| Service          | {Domain}Service | `PostService`, `BillingService`          |
| Repository       | {Domain}Repository | `PostRepository`, `UserRepository`   |
| Interface        | {Domain}Interface | `PaymentGatewayInterface`             |
| DTO              | {Action}{Domain}Data | `CreatePostData`, `UpdateUserData` |
| Value Object     | {Concept}      | `Email`, `Money`, `Slug`                  |
| Enum             | {Domain}Status/Role/Type | `PostStatus`, `UserRole`     |
| Test             | {Subject}Test  | `PostServiceTest`, `CreatePostTest`       |

---

## Method and class length limits

| Location           | Max lines | Action if exceeded             |
|--------------------|-----------|--------------------------------|
| Entry point action | 15        | Extract to service             |
| Service method     | 20        | Extract private helper method  |
| Async handler      | 25        | Delegate entirely to service   |
| Repository method  | 15        | Extract query scope/builder    |
| Class total        | 200       | Split into focused sub-classes |

---

## Formatting (enforce with PHP CS Fixer or Pint)
- 4 spaces — no tabs
- Opening brace on same line as class/method
- One blank line between methods
- No trailing whitespace
- LF line endings

```bash
# Laravel
./vendor/bin/pint

# Symfony / Core PHP
./vendor/bin/php-cs-fixer fix
```

---

## Docblocks — minimal and meaningful

```php
// Only on public service methods where the signature does not fully explain intent
// Never repeat what the types already say

/**
 * Publish a post and notify all subscribers.
 *
 * @throws PostAlreadyPublishedException when post is not in Draft status
 * @throws NotificationException when the notification provider fails
 */
public function publish(Post $post): void {}

// Do NOT write this — it adds zero value:
/**
 * @param int $id
 * @return Post
 */
public function find(int $id): Post {}
```

---

## Config and environment variables

```php
// All frameworks: never call environment functions directly in business code
// Values must come through a configuration layer

// Laravel
config('services.stripe.key')          // correct
env('STRIPE_KEY')                      // wrong inside a service class

// Symfony
$this->stripeKey                       // injected via services.yaml %env(STRIPE_KEY)%

// Core PHP
$this->config['stripe']['key']         // injected from config/services.php
// config/services.php reads: $_ENV['STRIPE_KEY'] ?? throw new RuntimeException(...)
```

---

## Value Objects — wrap primitives with meaning

```php
// When a string or int has its own validation rules, use a Value Object
// Works identically in all three frameworks

final readonly class Email {
    public readonly string $value;

    public function __construct(string $value) {
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException("Invalid email: {$value}");
        }
        $this->value = strtolower(trim($value));
    }

    public function __toString(): string { return $this->value; }
    public function equals(self $other): bool { return $this->value === $other->value; }
    public function domain(): string { return substr($this->value, strpos($this->value, '@') + 1); }
}

final readonly class Money {
    public function __construct(
        public readonly int    $amount,   // always in minor units (cents/pence)
        public readonly string $currency, // ISO 4217
    ) {
        if ($amount < 0) throw new \InvalidArgumentException('Amount cannot be negative');
    }

    public function add(self $other): self {
        if ($this->currency !== $other->currency) {
            throw new \InvalidArgumentException('Cannot add different currencies');
        }
        return new self($this->amount + $other->amount, $this->currency);
    }

    public function format(): string {
        return number_format($this->amount / 100, 2) . ' ' . $this->currency;
    }
}
```
