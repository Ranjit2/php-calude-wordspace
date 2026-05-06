# Skill: Create Migration (Core PHP)

## Checklist
- [ ] 1.  Create versioned SQL file: `migrations/NNN_create_{table}_table.sql`
- [ ] 2.  Write SQL: columns, indexes, FKs — see rules/database.md for types
- [ ] 3.  Create rollback file: `migrations/NNN_create_{table}_table.down.sql`
- [ ] 4.  Create migration runner if one does not exist
- [ ] 5.  Run migration: `php bin/migrate.php`
- [ ] 6.  Create PHP model class with typed properties
- [ ] 7.  Create repository: `src/Repositories/{Name}Repository.php`
- [ ] 8.  Write tests for repository query methods

## Migration SQL template
```sql
-- migrations/NNN_create_{table}.sql
CREATE TABLE {table} (
    id         BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id    BIGINT UNSIGNED NOT NULL,
    public_id  CHAR(36) NOT NULL UNIQUE COMMENT 'External-facing UUID',

    -- Add your columns here
    -- title      VARCHAR(255) NOT NULL,
    -- slug       VARCHAR(255) NOT NULL,
    -- body       TEXT,
    -- price      DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    -- status     VARCHAR(20) NOT NULL DEFAULT 'draft',
    -- metadata   JSON DEFAULT NULL,
    -- is_active  TINYINT(1) NOT NULL DEFAULT 1,

    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP DEFAULT NULL,

    -- Indexes
    INDEX idx_{table}_user_id (user_id),
    INDEX idx_{table}_status (status),
    -- INDEX idx_{table}_is_active (is_active),
    -- INDEX idx_{table}_user_status (user_id, status),

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Migration runner (bin/migrate.php)
```php
<?php declare(strict_types=1);

$pdo = require __DIR__ . '/../config/database.php';

$migrationPath = __DIR__ . '/../migrations/';
$files         = glob($migrationPath . '*.sql');
sort($files);

$pdo->exec('CREATE TABLE IF NOT EXISTS migrations (
    id        INT AUTO_INCREMENT PRIMARY KEY,
    filename  VARCHAR(255) NOT NULL UNIQUE,
    run_at    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
)');

foreach ($files as $file) {
    $filename = basename($file);
    if (str_ends_with($filename, '.down.sql')) continue;

    $stmt = $pdo->prepare('SELECT id FROM migrations WHERE filename = ?');
    $stmt->execute([$filename]);
    if ($stmt->fetch()) continue;

    echo "Running: {$filename}\n";
    $pdo->exec(file_get_contents($file));
    $pdo->prepare('INSERT INTO migrations (filename) VALUES (?)')->execute([$filename]);
    echo "Done: {$filename}\n";
}
```

## Repository template
```php
<?php declare(strict_types=1);
namespace App\Repositories;

use App\Repositories\Contracts\{Name}RepositoryInterface;

final class {Name}Repository implements {Name}RepositoryInterface {
    public function __construct(private readonly \PDO $pdo) {}

    public function findById(int $id): ?array {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM {table} WHERE id = :id AND deleted_at IS NULL LIMIT 1'
        );
        $stmt->execute(['id' => $id]);
        return $stmt->fetch(\PDO::FETCH_ASSOC) ?: null;
    }

    public function create(array $data): int {
        // Always use UUID for public_id
        $data['public_id'] = \Ramsey\Uuid\Uuid::uuid4()->toString();
        $data['created_at'] = date('Y-m-d H:i:s');

        $cols = implode(', ', array_keys($data));
        $phs  = ':' . implode(', :', array_keys($data));
        $stmt = $this->pdo->prepare("INSERT INTO {table} ({$cols}) VALUES ({$phs})");
        $stmt->execute($data);
        return (int) $this->pdo->lastInsertId();
    }

    public function update(int $id, array $data): bool {
        $data['updated_at'] = date('Y-m-d H:i:s');
        $sets = implode(', ', array_map(fn($k) => "{$k} = :{$k}", array_keys($data)));
        $data['id'] = $id;
        $stmt = $this->pdo->prepare("UPDATE {table} SET {$sets} WHERE id = :id");
        $stmt->execute($data);
        return $stmt->rowCount() > 0;
    }

    public function softDelete(int $id): bool {
        $stmt = $this->pdo->prepare(
            'UPDATE {table} SET deleted_at = NOW() WHERE id = :id'
        );
        $stmt->execute(['id' => $id]);
        return $stmt->rowCount() > 0;
    }
}
```
