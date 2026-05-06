# Skill: Refactor Code (Universal)
# These patterns apply to Laravel, Symfony, and core PHP equally.

## Checklist
- [ ] 1.  Read the code fully — understand intent before touching anything
- [ ] 2.  If no tests exist, write them first — you need a green baseline
- [ ] 3.  Make one change at a time — run tests after each change
- [ ] 4.  Run tests before starting to establish the baseline

## Universal smells and fixes

### 1. Fat entry point → service
```php
// Before (logic in controller/handler — untestable, not reusable)
public function store(Request $request): Response {
    $validated = /* validation */;
    $file      = /* file handling */;
    $result    = /* external API call */;
    /* database write */
    /* email send */
    return /* response */;
}

// After (thin entry point — testable, reusable)
public function store(StoreRequest $request): Response {
    $result = $this->service->create(Data::fromRequest($request));
    return $this->respond($result, 201);
}
```

### 2. Magic strings → Enum
```php
// Before
if ($post->status === 'published') { ... }
$post->status = 'failed';

// After
if ($post->status === PostStatus::Published) { ... }
$post->status = PostStatus::Failed;
```

### 3. Plain array → DTO
```php
// Before
return ['userId' => $id, 'email' => $email];

// After
return new UserData(userId: $id, email: $email);
```

### 4. Concrete → Interface
```php
// Before (tightly coupled, impossible to swap in tests)
public function __construct(private readonly StripeService $payment) {}

// After (swappable, mockable)
public function __construct(private readonly PaymentGatewayInterface $payment) {}
```

### 5. N+1 → eager load
```php
// Before
foreach ($posts as $post) {
    echo $post->author->name; // query per iteration
}

// After
// Laravel:   Post::with('author')->get()
// Symfony:   $qb->join('p.author', 'a') with select
// Core PHP:  JOIN in the query, or batch-fetch authors by ID
```

### 6. Missing strict_types
```php
// Before
<?php
class PostService { ... }

// After
<?php declare(strict_types=1);
class PostService { ... }
```
