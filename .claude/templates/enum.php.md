# Template: PHP 8.1+ Backed Enum

Use for ALL status, type, and flag fields. Never magic strings.

```php
<?php declare(strict_types=1);

namespace App\Enums;

enum {Name}Status: string {
    case Pending   = 'pending';
    case Running   = 'running';
    case Completed = 'completed';
    case Failed    = 'failed';

    public function label(): string {
        return match($this) {
            self::Pending   => 'Pending',
            self::Running   => 'Processing',
            self::Completed => 'Completed',
            self::Failed    => 'Failed',
        };
    }

    public function colour(): string {
        return match($this) {
            self::Pending   => 'yellow',
            self::Running   => 'blue',
            self::Completed => 'green',
            self::Failed    => 'red',
        };
    }

    public function isTerminal(): bool {
        return in_array($this, [self::Completed, self::Failed]);
    }

    /** @return list<string> all raw values for validation rules */
    public static function values(): array {
        return array_column(self::cases(), 'value');
    }
}

// Model cast: 'status' => {Name}Status::class
// Validation:  'status' => ['required', Rule::enum({Name}Status::class)]
// Comparison:  $model->status === {Name}Status::Completed
// Blade:       {{ $model->status->label() }}
```
