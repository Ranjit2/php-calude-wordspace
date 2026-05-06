# Template: DTO (All Frameworks — PHP 8.2+)
# Replace {Name} and {Action} with your domain names.

```php
<?php declare(strict_types=1);

namespace App\Data;

// Import from request — adapt to your framework:
// Laravel:   Illuminate\Http\Request
// Symfony:   use #[MapRequestPayload] on controller param instead
// Core PHP:  pass raw validated array from controller

final readonly class {Action}{Name}Data {
    public function __construct(
        public readonly int    $userId,
        // Add your typed fields here
        // public readonly string $title,
        // public readonly ?string $description,
        // public readonly float  $price,
        // public readonly bool   $isPublished,
        // public readonly \App\Enums\{Name}Status $status,
    ) {}

    /**
     * Laravel: create from validated FormRequest
     * Symfony: use #[MapRequestPayload] — no fromRequest needed
     * Core PHP: create from validated input array
     */
    public static function fromInput(array $input, int $userId): self {
        return new self(
            userId:      $userId,
            // title:    trim((string) ($input['title'] ?? '')),
            // price:    (float) ($input['price'] ?? 0),
            // isPublished: (bool) ($input['is_published'] ?? false),
        );
    }

    public function toArray(): array {
        return [
            'user_id' => $this->userId,
            // 'title'   => $this->title,
            // 'price'   => $this->price,
        ];
    }
}
```
