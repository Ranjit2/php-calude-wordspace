# Skill: Create Migration + Model (Laravel)

## Checklist
- [ ] 1.  `php artisan make:migration create_{table}_table`
- [ ] 2.  Write migration: columns, indexes, FKs, softDeletes — see rules/database.md
- [ ] 3.  `down()` must exactly reverse `up()`
- [ ] 4.  `php artisan make:model {Name} --factory --policy`
- [ ] 5.  Define `$fillable`, `$hidden`, `$casts` explicitly
- [ ] 6.  Add relationships, scopes, computed attributes
- [ ] 7.  `php artisan migrate`
- [ ] 8.  `php artisan test` — no regressions

## Model template
```php
<?php declare(strict_types=1);
namespace App\Models;
use App\Enums\{Name}Status;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

final class {Name} extends Model {
    use HasFactory, SoftDeletes;

    protected $fillable = [/* explicit list */];
    protected $hidden   = ['password', 'remember_token'];
    protected $casts    = [
        // 'status'       => {Name}Status::class,
        // 'metadata'     => 'array',
        // 'is_active'    => 'boolean',
        // 'price'        => 'decimal:2',
        // 'published_at' => 'datetime',
    ];

    // Relationships
    // public function user(): BelongsTo { return $this->belongsTo(User::class); }

    // Scopes
    // public function scopeActive(Builder $q): Builder { return $q->where('is_active', true); }
    // public function scopeForUser(Builder $q, int $userId): Builder { return $q->where('user_id', $userId); }

    // Computed attributes (PHP 8.x syntax)
    // protected function displayTitle(): Attribute {
    //     return Attribute::make(get: fn () => ucfirst($this->title));
    // }
}
```
