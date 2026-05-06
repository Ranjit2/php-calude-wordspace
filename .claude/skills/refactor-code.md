# Skill: Refactor Code

> **All frameworks:** same patterns apply — fat controller, magic strings, missing types, N+1.


## When to use
Cleaning up existing code — reducing complexity, fixing architecture
violations, improving testability, or preparing for a new feature.

---

## Checklist

- [ ] 1.  Read the code fully before touching anything — understand intent first
- [ ] 2.  If no tests exist, write them first before refactoring
- [ ] 3.  Run tests to establish a green baseline: `./vendor/bin/pest`
- [ ] 4.  Identify the smell — what specific problem are we solving?
- [ ] 5.  Make one change at a time — run tests after each change
- [ ] 6.  Extract to service if controller has more than 15 lines of logic
- [ ] 7.  Replace magic strings with Enums
- [ ] 8.  Add missing return types and `strict_types=1`
- [ ] 9.  Replace plain arrays with DTOs where data crosses layer boundaries
- [ ] 10. Run: `./vendor/bin/pint` then `./vendor/bin/pest`

---

## Common smells and fixes

### Fat controller → service

```php
// Before (all logic in controller — hard to test, impossible to reuse)
public function store(Request $request): JsonResponse {
    $validated = $request->validate([...]);
    $file      = $request->file('attachment');
    $path      = $file->storeAs('files', Str::uuid().'.pdf', 'private');
    $result    = Http::post('https://external-api.com/process', ['path' => $path]);
    Model::create([...$validated, 'result' => $result->json()]);
    Mail::to($request->user())->send(new ConfirmationMail());
    return response()->json(['ok' => true]);
}

// After (thin controller — testable, reusable)
public function store(StoreResourceRequest $request): JsonResponse {
    $resource = $this->service->create(ResourceData::fromRequest($request));
    return (new ResourceResource($resource))->response()->setStatusCode(201);
}
```

### Magic strings → Enum

```php
// Before
if ($order->status === 'completed') { ... }
$order->update(['status' => 'failed']);

// After
if ($order->status === OrderStatus::Completed) { ... }
$order->update(['status' => OrderStatus::Failed]);
```

### Array return → DTO

```php
// Before
return ['userId' => $id, 'email' => $email, 'role' => 'admin'];

// After
return new UserData(userId: $id, email: $email, role: UserRole::Admin);
```

### Missing type safety

```php
// Before
public function process($data) {
    return $this->doSomething($data['id']);
}

// After
public function process(ProcessData $data): ProcessResult {
    return $this->doSomething($data->id);
}
```

### Missing interface on external dependency

```php
// Before — tightly coupled to one provider
public function __construct(
    private readonly StripePaymentService $payment,
) {}

// After — swappable, testable
public function __construct(
    private readonly PaymentGatewayInterface $payment,
) {}
// Bind in AppServiceProvider: PaymentGatewayInterface → StripePaymentService
```

### N+1 query in service

```php
// Before
$orders = Order::all();
foreach ($orders as $order) {
    echo $order->customer->name; // query per iteration
}

// After
$orders = Order::with('customer')->get();
foreach ($orders as $order) {
    echo $order->customer->name; // no extra queries
}
```
