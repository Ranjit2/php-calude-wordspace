# Security Rules
# Applies to all frameworks. Framework-specific implementations noted inline.

## The hard stops — check these before writing any code

- [ ] Does this handle file uploads? → Read the File Uploads section first
- [ ] Does this accept user input for a database query? → Read SQL Injection section
- [ ] Does this put user content into an AI prompt? → Read Prompt Injection section
- [ ] Does this access an API key or secret? → Read Secrets section
- [ ] Does this output user content to HTML? → Read Output Escaping section

---

## File uploads

```php
// ── Validate server-side (never trust client MIME or extension) ───────────

// Laravel
'file' => ['required', 'file', 'mimes:pdf,jpg,png', 'max:10240']

// Symfony
#[Assert\File(maxSize: '10M', mimeTypes: ['application/pdf', 'image/jpeg', 'image/png'])]

// Core PHP
$allowed = ['application/pdf', 'image/jpeg', 'image/png'];
$finfo   = new \finfo(FILEINFO_MIME_TYPE);
$mime    = $finfo->file($_FILES['file']['tmp_name']);
if (!in_array($mime, $allowed, true)) {
    throw new \InvalidArgumentException('Invalid file type');
}
if ($_FILES['file']['size'] > 10 * 1024 * 1024) {
    throw new \InvalidArgumentException('File too large');
}

// ── Always store with UUID filename ──────────────────────────────────────
// Never use the original filename — it can contain path traversal attacks

// Laravel
$path = $file->storeAs('uploads', \Str::uuid().'.'.$file->extension(), 'private');

// Symfony (Flysystem)
$filename = \Symfony\Component\Uid\Uuid::v4()->toRfc4122().'.'.$file->guessExtension();
$filesystem->write('uploads/'.$filename, file_get_contents($file->getPathname()));

// Core PHP
$ext      = pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION);
$filename = bin2hex(random_bytes(16)).'.'.$ext;
move_uploaded_file($_FILES['file']['tmp_name'], '/storage/private/uploads/'.$filename);

// ── Always store on private disk (never public web root) ─────────────────
// ── Generate time-limited signed URLs for access ─────────────────────────

// Laravel
Storage::disk('private')->temporaryUrl($path, now()->addMinutes(15));

// Symfony / Core PHP
// Generate a signed token, serve file through a controller that validates token + expiry
// Never expose the raw filesystem path in any URL or response
```

---

## SQL injection — parameterised bindings always

```php
// NEVER string interpolation or concatenation in queries

// Wrong — every framework
$results = $db->query("SELECT * FROM users WHERE email = '$email'");

// Laravel (Eloquent / Query Builder — safe by default)
User::where('email', $email)->first();
DB::table('users')->where('email', $email)->first();
DB::select('SELECT * FROM users WHERE email = ?', [$email]);

// Symfony (Doctrine)
$em->getRepository(User::class)->findOneBy(['email' => $email]);
$conn->fetchOne('SELECT id FROM users WHERE email = ?', [$email]);

// Core PHP (PDO)
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);
$user = $stmt->fetch(\PDO::FETCH_ASSOC);
```

---

## Prompt injection prevention

```php
// User-supplied content going into an AI prompt MUST be
// wrapped in XML delimiters so the model treats it as data, not instructions

// Wrong
$prompt = "Summarise this: {$userContent}";

// Right — works the same in all frameworks
$safe   = mb_substr(strip_tags($userContent), 0, 6000);
$safe   = str_replace("\0", '', $safe);  // strip null bytes

$prompt = <<<PROMPT
<user_content>
{$safe}
</user_content>
PROMPT;

// The system prompt (role + instructions) must come from a file,
// never from user input:
// resources/prompts/summarise-system.txt  (Laravel)
// config/prompts/summarise.txt            (Symfony / Core PHP)
```

---

## Secrets and credentials

```php
// All frameworks: never hardcode credentials, never call env functions
// in business code. Values must flow through a config layer.

// Laravel
config('services.stripe.key')      // correct
env('STRIPE_KEY')                  // wrong — bypasses config caching

// Symfony
// Inject via constructor from %env(STRIPE_KEY)% in services.yaml

// Core PHP
$config['stripe']['key']           // injected from config/services.php
// config/services.php:
return [
    'stripe' => [
        'key' => $_ENV['STRIPE_KEY']
                 ?? throw new \RuntimeException('STRIPE_KEY not set'),
    ],
];

// .env.example — document every required key with a dummy value
// STRIPE_KEY=sk_test_your_key_here

// Never log credentials — check Log/logger calls near HTTP clients
// Rotate immediately if accidentally committed
```

---

## Authentication and authorisation

```php
// Always authenticate before authorising
// Always authorise before accessing owned data

// Laravel — never trust client-supplied IDs for ownership
// Wrong:
$post = Post::find($request->id);
// Right:
$post = $request->user()->posts()->findOrFail($request->id);

// Symfony — use Voter pattern, not inline checks in controllers
$this->denyAccessUnlessGranted(PostVoter::EDIT, $post);

// Core PHP — middleware chain checks auth token, then ownership in service
$this->authMiddleware->requireOwnership($user, $resource);

// Rate limiting — on every mutating endpoint
// Laravel:  RateLimiter::for('action-name', fn() => Limit::perMinute(10)->by($user->id))
// Symfony:  rate_limiter.yaml config with sliding_window policy
// Core PHP: token bucket in Redis or database
```

---

## Mass assignment protection

```php
// Always define exactly what can be set — no open models/entities

// Laravel — explicit $fillable, never $guarded = []
protected $fillable = ['title', 'body', 'status'];
protected $hidden   = ['password', 'remember_token'];

// Symfony — use DTOs + setters with validation, never hydrate entity from raw request array

// Core PHP — validate and whitelist every field before insert/update
$allowed = ['title', 'body', 'status'];
$data    = array_intersect_key($input, array_flip($allowed));
```

---

## Output escaping

```php
// Always escape user data before rendering

// Laravel Blade
{{ $post->title }}             // auto-escaped — always use this
{!! $post->renderedHtml !!}    // raw — only for pre-sanitised HTML

// Symfony Twig
{{ post.title }}               // auto-escaped — always use this
{{ post.renderedHtml|raw }}    // raw — only for pre-sanitised HTML

// Core PHP
echo htmlspecialchars($post['title'], ENT_QUOTES, 'UTF-8');

// Pre-sanitise with:
$clean = strip_tags($html, '<p><br><strong><em><ul><li>');
```

---

## Security headers (set at web server or middleware level)

```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Content-Security-Policy: default-src 'self'
```
