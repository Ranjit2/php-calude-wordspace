# Template: Value Object (All Frameworks)
# Use when a primitive has its own validation rules or domain meaning.
# Examples: Email, Money, PhoneNumber, Slug, DateRange, Percentage

```php
<?php declare(strict_types=1);

namespace App\Values;

/**
 * Example: Email value object
 * Replace with your domain concept.
 */
final readonly class Email {
    public readonly string $value;

    public function __construct(string $value) {
        $normalised = strtolower(trim($value));
        if (!filter_var($normalised, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException("Invalid email address: {$value}");
        }
        $this->value = $normalised;
    }

    public static function from(string $value): self { return new self($value); }

    public function __toString(): string { return $this->value; }
    public function equals(self $other): bool { return $this->value === $other->value; }
    public function domain(): string { return substr($this->value, strpos($this->value, '@') + 1); }
}

/**
 * Example: Money value object
 * Always store amounts in minor units (cents/pence/øre).
 */
final readonly class Money {
    public function __construct(
        public readonly int    $amount,    // always in minor units
        public readonly string $currency,  // ISO 4217 (USD, EUR, GBP)
    ) {
        if ($amount < 0) throw new \InvalidArgumentException('Amount cannot be negative');
        if (strlen($currency) !== 3) throw new \InvalidArgumentException('Currency must be ISO 4217');
    }

    public static function of(int $amount, string $currency): self { return new self($amount, $currency); }

    public function add(self $other): self {
        if ($this->currency !== $other->currency) {
            throw new \InvalidArgumentException("Cannot add {$this->currency} and {$other->currency}");
        }
        return new self($this->amount + $other->amount, $this->currency);
    }

    public function format(): string { return number_format($this->amount / 100, 2) . ' ' . $this->currency; }
    public function equals(self $other): bool { return $this->amount === $other->amount && $this->currency === $other->currency; }
    public function isZero(): bool { return $this->amount === 0; }
}
```
