# Skill: Integrate an AI Provider

> **All frameworks:** same pattern — `AiProviderInterface` + concrete service + `Http` client
> **Laravel:** `Http::withToken()` facade
> **Symfony:** `HttpClientInterface` from `symfony/http-client`
> **Core PHP:** Guzzle `GuzzleHttp\Client`


## Supported providers
- **OpenRouter** — recommended; one API key accesses Claude, GPT-4, Llama, Mistral, and more
- **Anthropic** — direct Claude API
- **OpenAI** — direct GPT API

---

## Checklist

- [ ] 1.  Add API key to `.env` and `.env.example` (with a dummy value)
- [ ] 2.  Add config entry in `config/services.php`
- [ ] 3.  Create interface: `app/Services/Contracts/AiProviderInterface.php`
- [ ] 4.  Create service: `app/Services/{Provider}AiService.php`
- [ ] 5.  Bind interface in `AppServiceProvider::register()`
- [ ] 6.  Build prompt via a dedicated `buildPrompt()` method — never inline
- [ ] 7.  Store prompt templates in `resources/prompts/` — not hardcoded in PHP
- [ ] 8.  Parse response with `JSON_THROW_ON_ERROR`
- [ ] 9.  Implement retry on 5xx and timeout errors
- [ ] 10. Write unit test with mocked HTTP:
          - **Laravel:** `Http::fake([...])`
          - **Symfony:** `MockHttpClient` + `MockResponse`
          - **Core PHP:** Guzzle `MockHandler`

---

## Config entry

```php
// config/services.php
'openrouter' => [
    'key'     => env('OPENROUTER_API_KEY'),
    'base_url'=> env('OPENROUTER_BASE_URL', 'https://openrouter.ai/api/v1'),
    'model'   => env('OPENROUTER_MODEL', 'anthropic/claude-3-haiku'),  // or any model slug
    'timeout' => (int) env('OPENROUTER_TIMEOUT', 60),
],
```

---

## AiProviderInterface

```php
<?php declare(strict_types=1);

namespace App\Services\Contracts;

interface AiProviderInterface {
    /**
     * Send a prompt to the AI provider and return the parsed response.
     *
     * @param  string $systemPrompt  Instructions for the AI's role and output format
     * @param  string $userContent   The actual user/task content
     * @return array                 Parsed JSON response from the AI
     * @throws \App\Exceptions\AiProviderException
     */
    public function complete(string $systemPrompt, string $userContent): array;
}
```

---

## OpenRouter service (production-ready implementation)

```php
<?php declare(strict_types=1);

namespace App\Services;

use App\Exceptions\AiProviderException;
use App\Services\Contracts\AiProviderInterface;
use Illuminate\Http\Client\ConnectionException;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

final class OpenRouterAiService implements AiProviderInterface {
    public function complete(string $systemPrompt, string $userContent): array {
        $payload = [
            'model'           => config('services.openrouter.model'),
            'messages'        => [
                ['role' => 'system', 'content' => $systemPrompt],
                ['role' => 'user',   'content' => $userContent],
            ],
            'response_format' => ['type' => 'json_object'],
            'temperature'     => 0.1,
            'max_tokens'      => 2000,
        ];

        try {
            $response = Http::withToken(config('services.openrouter.key'))
                ->timeout(config('services.openrouter.timeout'))
                ->retry(2, 5000, fn ($e) => $e instanceof ConnectionException)
                ->post(config('services.openrouter.base_url') . '/chat/completions', $payload);
        } catch (\Throwable $e) {
            throw new AiProviderException(
                "AI provider connection failed: {$e->getMessage()}",
                previous: $e
            );
        }

        if ($response->failed()) {
            Log::warning('AI provider returned non-200', [
                'status' => $response->status(),
                'body'   => $response->body(),
            ]);
            throw new AiProviderException(
                "AI provider returned HTTP {$response->status()}"
            );
        }

        return $this->parse($response->json('choices.0.message.content'));
    }

    private function parse(mixed $content): array {
        if (!is_string($content)) {
            return [];
        }

        // Strip markdown fences defensively
        $clean = preg_replace('/^```(?:json)?\s*|\s*```$/m', '', trim($content));

        try {
            $data = json_decode($clean, true, 512, JSON_THROW_ON_ERROR);
        } catch (\JsonException $e) {
            Log::warning('AI response JSON parse failed', [
                'snippet' => substr($clean, 0, 200),
            ]);
            return [];
        }

        return is_array($data) ? $data : [];
    }
}
```

---

## Prompt template file (resources/prompts/your-prompt.txt)

```
# resources/prompts/{feature}-system.txt
#
# This file defines the system prompt for the {feature} AI task.
# Keep instructions clear and structured.
# Use XML tags to delimit user-supplied content (prevents prompt injection).
#
# ─────────────────────────────────────────────

You are an expert [describe the AI's role].

[Describe what the AI should do]

[Add any rules, scoring systems, or output requirements]

<user_content>
{USER_CONTENT}
</user_content>

Return JSON ONLY matching this schema:
{
  "field_one": "value",
  "field_two": 85,
  "field_three": ["item1", "item2"]
}
```

---

## Using the prompt template in your service

```php
private function buildPrompt(string $userContent): string {
    // Always truncate user-supplied content before inserting into prompt
    $safe = mb_substr(strip_tags($userContent), 0, 6000);
    $safe = str_replace("\0", '', $safe);  // strip null bytes

    $template = file_get_contents(resource_path('prompts/{feature}-system.txt'));

    return str_replace('{USER_CONTENT}', $safe, $template);
}
```

---

## Prompt injection prevention

```php
// NEVER interpolate raw user text directly into a system prompt
// WRONG:
$prompt = "Analyse this: {$userText}";

// RIGHT — wrap in XML delimiters so the model treats it as data, not instructions
$prompt = "<content>\n{$userText}\n</content>";
```

---

## Switching providers

To swap from OpenRouter to Anthropic direct:
1. Create `app/Services/AnthropicAiService.php` implementing `AiProviderInterface`
2. Change the binding in `AppServiceProvider::register()`:
```php
$this->app->bind(AiProviderInterface::class, AnthropicAiService::class);
```
3. All consuming services and jobs are unchanged — they depend on the interface
