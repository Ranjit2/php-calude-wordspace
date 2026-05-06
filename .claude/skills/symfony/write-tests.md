# Skill: Write Tests (Symfony — PHPUnit)

## Checklist
- [ ] 1.  `WebTestCase` for HTTP endpoint tests
- [ ] 2.  `KernelTestCase` for service unit tests with container
- [ ] 3.  `MockHttpClient` + `MockResponse` for external HTTP — never real calls
- [ ] 4.  In-memory Messenger transport for async tests
- [ ] 5.  Cover: 201, 422, 401, 403, 404 as applicable
- [ ] 6.  `php bin/phpunit --coverage-text`

## Controller test
```php
<?php declare(strict_types=1);
namespace App\Tests\Controller;
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Symfony\Component\HttpFoundation\Response;

final class {Name}ControllerTest extends WebTestCase {
    public function testCreate{Name}Returns201(): void {
        $client = static::createClient();
        // $client->loginUser($user); // for authenticated routes

        $client->request('POST', '/api/v1/{resources}', [], [], [
            'CONTENT_TYPE' => 'application/json',
        ], json_encode([/* valid payload */]));

        $this->assertResponseStatusCodeSame(Response::HTTP_CREATED);
        $body = json_decode($client->getResponse()->getContent(), true);
        $this->assertArrayHasKey('id', $body['data']);
    }

    public function testCreate{Name}Returns422WhenInvalid(): void {
        $client = static::createClient();
        $client->request('POST', '/api/v1/{resources}', [], [], [
            'CONTENT_TYPE' => 'application/json',
        ], json_encode([]));
        $this->assertResponseStatusCodeSame(Response::HTTP_UNPROCESSABLE_ENTITY);
    }
}
```

## Service unit test
```php
<?php declare(strict_types=1);
namespace App\Tests\Service;
use App\Service\{Name}Service;
use PHPUnit\Framework\TestCase;
use Symfony\Component\HttpClient\MockHttpClient;
use Symfony\Component\HttpClient\Response\MockResponse;

final class {Name}ServiceTest extends TestCase {
    public function testProcessSuccessfully(): void {
        $mockHttp = new MockHttpClient([
            new MockResponse(json_encode([/* expected response */])),
        ]);
        $service = new {Name}Service(/* inject mock */);
        $result  = $service->process(/* input */);
        $this->assertSame('expected', $result->someField);
    }

    public function testThrowsExceptionOnHttpFailure(): void {
        $mockHttp = new MockHttpClient([
            new MockResponse('', ['http_code' => 503]),
        ]);
        $this->expectException(\App\Exception\ExternalServiceException::class);
        $service = new {Name}Service(/* inject mock */);
        $service->process(/* input */);
    }
}
```
