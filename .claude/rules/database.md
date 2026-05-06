# Database Rules
# Universal schema and query rules. Framework variants shown inline.
# Supported: MySQL 8.0+ · PostgreSQL 15+ · SQLite (testing only)

---

## Column types — always choose precisely

```php
// ── IDs ──────────────────────────────────────────────────────
// Primary key: bigint unsigned auto-increment
// Public ID:   UUID (expose this in URLs, never the auto-increment)
// Foreign key: bigint unsigned, always indexed explicitly

// ── Strings ──────────────────────────────────────────────────
// varchar(255)   → names, titles, slugs, emails — anything indexed
// text           → descriptions, content — not directly indexable
// longtext       → large HTML, markdown, JSON strings

// ── Numbers ──────────────────────────────────────────────────
// decimal(10,2)  → money/prices — NEVER float for currency
// decimal(5,2)   → percentages, scores, ratings
// int unsigned   → counts, quantities — never negative
// tinyint(1)     → booleans

// ── Dates ────────────────────────────────────────────────────
// timestamp      → created_at, updated_at, event times (UTC always)
// date           → calendar dates with no time component
// datetime       → when timezone-aware storage needed

// ── JSON ─────────────────────────────────────────────────────
// json           → arrays, objects, flexible metadata
//                  MySQL 8 native json type — not text or longtext
//                  Always pair with a PHP cast/type

// ── Status fields ────────────────────────────────────────────
// varchar(20) + PHP Enum — more flexible than DB ENUM type
// Never DB ENUM — painful to ALTER in production
```

---

## Indexing strategy

```sql
-- Always index:
-- Foreign keys (MySQL does NOT auto-index them — PostgreSQL does)
-- Columns in WHERE clauses on tables over 10k rows
-- Columns in ORDER BY on paginated queries
-- Columns in GROUP BY

-- Single
CREATE INDEX idx_posts_status     ON posts(status);
CREATE INDEX idx_posts_created_at ON posts(created_at);
CREATE INDEX idx_posts_author_id  ON posts(author_id);  -- FK + explicit index

-- Composite — column order: most selective first
CREATE INDEX idx_posts_author_status ON posts(author_id, status);
CREATE INDEX idx_posts_status_date   ON posts(status, created_at);

-- Unique
CREATE UNIQUE INDEX idx_posts_slug ON posts(slug);
CREATE UNIQUE INDEX idx_posts_user_slug ON posts(user_id, slug);  -- scoped

-- Never index:
-- Boolean columns (cardinality too low)
-- JSON columns directly (use generated columns for JSON path indexing)
```

---

## Migration rules — write them as permanent contracts

```
NEVER modify an existing migration after it has run on any environment
NEVER rename a column without a multi-step deploy strategy
NEVER drop a column in the same release you stop using it
ALWAYS add a comment on non-obvious columns
ALWAYS make down() reverse up() exactly
ALWAYS test down() — run migrate:rollback in development
```

### Laravel migrations
```php
return new class extends Migration {
    public function up(): void {
        Schema::create('{table}', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->uuid('public_id')->unique()->comment('External-facing ID');

            // ── Your columns ────────────────────────────────
            // $table->string('title');
            // $table->string('slug')->unique();
            // $table->text('body')->nullable();
            // $table->decimal('price', 10, 2)->default(0.00);
            // $table->boolean('is_published')->default(false);
            // $table->json('metadata')->nullable();
            // $table->string('status', 20)->default('draft');
            // $table->timestamp('published_at')->nullable();
            // ────────────────────────────────────────────────

            $table->timestamps();
            $table->softDeletes();

            // ── Indexes ─────────────────────────────────────
            // $table->index('user_id');
            // $table->index('status');
            // $table->index('is_published');
            // $table->index(['user_id', 'status']);
            // ────────────────────────────────────────────────
        });
    }

    public function down(): void {
        Schema::dropIfExists('{table}');
    }
};
```

### Symfony / Doctrine migrations
```php
public function up(Schema $schema): void {
    $this->addSql('
        CREATE TABLE {table} (
            id         BIGINT UNSIGNED AUTO_INCREMENT NOT NULL,
            user_id    BIGINT UNSIGNED NOT NULL,
            public_id  CHAR(36) NOT NULL COMMENT "External-facing UUID",
            title      VARCHAR(255) NOT NULL,
            status     VARCHAR(20) NOT NULL DEFAULT "draft",
            metadata   JSON DEFAULT NULL,
            created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
            updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
            deleted_at TIMESTAMP DEFAULT NULL,
            PRIMARY KEY (id),
            UNIQUE INDEX uq_{table}_public_id (public_id),
            INDEX idx_{table}_user_id (user_id),
            INDEX idx_{table}_status (status),
            CONSTRAINT fk_{table}_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
    ');
}

public function down(Schema $schema): void {
    $this->addSql('DROP TABLE IF EXISTS {table}');
}
```

### Core PHP (versioned SQL files)
```
migrations/
  001_create_users_table.sql
  002_create_posts_table.sql
  003_add_published_at_to_posts.sql
```
```sql
-- migrations/002_create_posts_table.sql
CREATE TABLE posts (
    id           BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id      BIGINT UNSIGNED NOT NULL,
    public_id    CHAR(36) NOT NULL UNIQUE COMMENT 'External-facing UUID',
    title        VARCHAR(255) NOT NULL,
    status       VARCHAR(20) NOT NULL DEFAULT 'draft',
    metadata     JSON DEFAULT NULL,
    created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at   TIMESTAMP DEFAULT NULL,
    INDEX idx_posts_user_id (user_id),
    INDEX idx_posts_status (status),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

## Query performance checklist
- [ ] `EXPLAIN ANALYZE` on any query touching 10k+ rows
- [ ] N+1: eager-load all relationships before looping
- [ ] Large sets: chunk or cursor — never load everything into memory
- [ ] Select only needed columns — never `SELECT *` on wide tables
- [ ] Aggregate in the database — not in PHP loops
- [ ] Paginate collection endpoints — never return unbounded lists
- [ ] Wrap multi-table writes in a transaction
- [ ] Cache slow, read-heavy, rarely-changing queries
