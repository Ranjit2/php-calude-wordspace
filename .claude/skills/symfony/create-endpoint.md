# Skill: Create Endpoint (Symfony)

## Checklist
- [ ] 1.  Add route via `#[Route]` attribute on controller method
- [ ] 2.  Create DTO with `#[Assert\*]` constraints for validation
- [ ] 3.  Use `#[MapRequestPayload]` to auto-validate and hydrate DTO
- [ ] 4.  Create or extend service in `src/Service/{Name}Service.php`
- [ ] 5.  Bind interface in `services.yaml` if interface created
- [ ] 6.  Write PHPUnit `WebTestCase` feature test
- [ ] 7.  `./vendor/bin/php-cs-fixer fix && php bin/phpunit`

## Controller template
```php
<?php declare(strict_types=1);
namespace App\Controller\Api\V1;
use App\Dto\{Name}Data;
use App\Service\{Name}Service;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Attribute\MapRequestPayload;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\Security\Http\Attribute\IsGranted;

#[Route('/api/v1/{resources}', name: 'api_{resources}_')]
final class {Name}Controller extends AbstractController {
    public function __construct(private readonly {Name}Service $service) {}

    #[Route('', name: 'create', methods: ['POST'])]
    #[IsGranted('ROLE_USER')]
    public function create(#[MapRequestPayload] {Name}Data $data): JsonResponse {
        $result = $this->service->create($data);
        return $this->json(['data' => $result], Response::HTTP_CREATED);
    }

    #[Route('/{id}', name: 'show', methods: ['GET'])]
    public function show(int $id): JsonResponse {
        $entity = $this->service->findOrFail($id);
        $this->denyAccessUnlessGranted({Name}Voter::VIEW, $entity);
        return $this->json(['data' => $entity]);
    }
}
```

## DTO with validation
```php
<?php declare(strict_types=1);
namespace App\Dto;
use Symfony\Component\Validator\Constraints as Assert;

final readonly class {Name}Data {
    public function __construct(
        #[Assert\NotBlank]
        #[Assert\Length(min: 3, max: 255)]
        public readonly string $title,
        // add your fields with Assert constraints
    ) {}
}
```
