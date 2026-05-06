# Template: DTO (Data Transfer Object)

Replace `{Name}` with your domain name (e.g. `CreateOrder`, `UpdateProfile`, `ProcessPayment`).
Readonly classes require PHP 8.2+. For PHP 8.1, use `readonly` on individual properties.

---

```php
<?php declare(strict_types=1);

namespace App\Data;

use Illuminate\Http\Request;

final readonly class {Name}Data {
    public function __construct(
        // Define your typed properties here
        // Use the most specific type available
        // e.g.
        // public int    $userId,
        // public string $title,
        // public ?string $description,
        // public float  $amount,
        // public bool   $isPublished,
        // public array  $tags,
    ) {}

    /**
     * Create from a validated HTTP request.
     * Only call this after FormRequest validation has passed.
     */
    public static function fromRequest(Request $request): self {
        return new self(
            // Map request inputs to constructor parameters
            // Use typed helpers: integer(), string(), float(), boolean()
            // e.g.
            // userId:      $request->user()->id,
            // title:       $request->string('title')->toString(),
            // description: $request->string('description')->toString() ?: null,
            // amount:      $request->float('amount'),
            // isPublished: $request->boolean('is_published'),
            // tags:        $request->array('tags'),
        );
    }

    /**
     * Convert to array for model creation / update.
     * Only include columns that map to database columns.
     */
    public function toArray(): array {
        return [
            // Map DTO properties to database column names
            // e.g.
            // 'user_id'      => $this->userId,
            // 'title'        => $this->title,
            // 'description'  => $this->description,
            // 'amount'       => $this->amount,
            // 'is_published' => $this->isPublished,
        ];
    }
}
```

---

## Value Object variation (for validated domain primitives)

Use when a value has its own validation rules or business meaning.
For example: `Email`, `Money`, `PhoneNumber`, `Slug`, `DateRange`.

```php
<?php declare(strict_types=1);

namespace App\Values;

final readonly class {Name} {
    public function __construct(
        public readonly string $value,
    ) {
        // Validate in the constructor — throw on invalid input
        if (/* validation fails */) {
            throw new \InvalidArgumentException("Invalid {name}: {$value}");
        }
    }

    public static function from(string $value): self {
        return new self($value);
    }

    public function __toString(): string {
        return $this->value;
    }

    public function equals(self $other): bool {
        return $this->value === $other->value;
    }
}
```
