# Template: Enum (All Frameworks — PHP 8.1+)
# Replace {Name} and case values with your domain.

```php
<?php declare(strict_types=1);

namespace App\Enums;

enum {Name}Status: string {
    case Draft     = 'draft';
    case Active    = 'active';
    case Completed = 'completed';
    case Failed    = 'failed';

    public function label(): string {
        return match($this) {
            self::Draft     => 'Draft',
            self::Active    => 'Active',
            self::Completed => 'Completed',
            self::Failed    => 'Failed',
        };
    }

    public function isTerminal(): bool {
        return in_array($this, [self::Completed, self::Failed], true);
    }

    public function canTransitionTo(self $next): bool {
        return match($this) {
            self::Draft     => $next === self::Active,
            self::Active    => in_array($next, [self::Completed, self::Failed], true),
            default         => false,
        };
    }

    /** @return list<string> */
    public static function values(): array {
        return array_column(self::cases(), 'value');
    }
}

// Usage:
// Laravel model cast:  'status' => {Name}Status::class
// Symfony entity:      #[Column(type: 'string', enumType: {Name}Status::class)]
// Core PHP:            cast from DB string: {Name}Status::from($row['status'])
// Validation (Laravel): Rule::enum({Name}Status::class)
// Validation (Symfony): #[Assert\Choice(callback: [{Name}Status::class, 'values'])]
```
