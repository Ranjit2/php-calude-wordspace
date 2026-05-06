# Skill: Create Migration + Model

> **Laravel:** `php artisan make:migration` + Eloquent model + `php artisan migrate`
> **Symfony:** Doctrine entity + `php bin/console doctrine:migrations:diff && migrate`
> **Core PHP:** write raw SQL migration file + PDO execution script


## When to use
Any time a new table is needed or an existing table needs new columns.

---

## Checklist

- [ ] 1.  Create the migration:
          - **Laravel:** `php artisan make:migration create_{table}_table`
          - **Symfony:** `php bin/console make:entity` then `doctrine:migrations:diff`
          - **Core PHP:** create a versioned SQL file and a migration runner script
- [ ] 2.  Write migration: column types, nullable, defaults, indexes, FKs, softDeletes
- [ ] 3.  Write `down()` to exactly reverse `up()`
- [ ] 4.  Create the model/entity:
          - **Laravel:** `php artisan make:model {Name} --factory --policy`
          - **Symfony:** entity already created with `make:entity` in step 1
          - **Core PHP:** create a plain PHP class with typed properties
- [ ] 5.  Define `$fillable` (explicit list — never `guarded = []`)
- [ ] 6.  Define `$hidden` (passwords, tokens, internal paths)
- [ ] 7.  Define `$casts` (all JSON, bool, float, enum, date columns)
- [ ] 8.  Add relationships (`belongsTo`, `hasMany`, `belongsToMany`, etc.)
- [ ] 9.  Add Eloquent scopes for common query patterns
- [ ] 10. Add computed `Attribute` accessors (PHP 8.x syntax)
- [ ] 11. Update factory with realistic fake data
- [ ] 12. Register Observer in `AppServiceProvider` if lifecycle hooks needed
- [ ] 13. Run migrations:
          - **Laravel:** `php artisan migrate`
          - **Symfony:** `php bin/console doctrine:migrations:migrate`
          - **Core PHP:** run your migration script manually
- [ ] 14. Run tests to confirm no regressions

---

## Migration template

```php
<?php declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('{table}', function (Blueprint $table) {
            $table->id();

            // Foreign keys — always with explicit index
            $table->foreignId('user_id')
                  ->constrained()
                  ->cascadeOnDelete();

            // ─────────────────────────────────────────────
            // ADD YOUR COLUMNS HERE
            // Use the correct type for each column.
            // See rules/database.md for the full type reference.
            //
            // Common examples:
            // $table->string('title');
            // $table->string('slug')->unique();
            // $table->text('description')->nullable();
            // $table->decimal('price', 10, 2)->default(0.00);
            // $table->boolean('is_published')->default(false);
            // $table->json('metadata')->nullable();
            // $table->string('status', 20)->default('draft');
            // $table->timestamp('published_at')->nullable();
            // ─────────────────────────────────────────────

            $table->timestamps();
            $table->softDeletes();

            // ─────────────────────────────────────────────
            // INDEXES — add for every column used in WHERE / ORDER BY
            // MySQL does NOT auto-index foreign keys — always add explicitly
            //
            // $table->index('user_id');
            // $table->index('status');
            // $table->index('created_at');
            // $table->index(['user_id', 'created_at']);  // composite
            // ─────────────────────────────────────────────
        });
    }

    public function down(): void {
        Schema::dropIfExists('{table}');
    }
};
```

---

## Model template

```php
<?php declare(strict_types=1);

namespace App\Models;

use App\Enums\{Name}Status;   // create this enum — see templates/enum.php.md
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\SoftDeletes;

final class {Name} extends Model {
    use HasFactory, SoftDeletes;

    protected $fillable = [
        // List every column that can be mass-assigned
        // e.g. 'user_id', 'title', 'slug', 'description', 'status'
    ];

    protected $hidden = [
        // Columns never exposed in JSON serialisation
        // e.g. 'internal_notes', 'raw_payload'
    ];

    protected $casts = [
        // ─────────────────────────────────────────────
        // Cast every JSON, enum, bool, decimal, and date column
        //
        // JSON arrays:
        // 'metadata'     => 'array',
        // 'settings'     => 'array',
        //
        // Enum:
        // 'status'       => {Name}Status::class,
        //
        // Boolean:
        // 'is_published' => 'boolean',
        //
        // Decimal:
        // 'price'        => 'decimal:2',
        //
        // Dates:
        // 'published_at' => 'datetime',
        // 'deleted_at'   => 'datetime',
        // ─────────────────────────────────────────────
    ];

    // ─── Relationships ────────────────────────────────────────
    public function user(): BelongsTo {
        return $this->belongsTo(User::class);
    }
    // Add more relationships as needed

    // ─── Scopes ───────────────────────────────────────────────
    // Add scopes for common query patterns in your domain
    // e.g.
    // public function scopeActive(Builder $q): Builder {
    //     return $q->where('is_active', true);
    // }
    // public function scopeForUser(Builder $q, int $userId): Builder {
    //     return $q->where('user_id', $userId);
    // }

    // ─── Computed attributes ──────────────────────────────────
    // Add computed properties using PHP 8.x Attribute syntax
    // e.g.
    // protected function displayName(): Attribute {
    //     return Attribute::make(
    //         get: fn () => ucfirst($this->name)
    //     );
    // }
}
```
