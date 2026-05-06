# Skill: Create Migration + Entity (Symfony / Doctrine)

## Checklist
- [ ] 1.  `php bin/console make:entity {Name}` — generates entity + repository scaffold
- [ ] 2.  Add typed properties with Doctrine attributes
- [ ] 3.  `php bin/console doctrine:migrations:diff` — generates migration from entity diff
- [ ] 4.  Review generated migration SQL — add missing indexes
- [ ] 5.  `php bin/console doctrine:migrations:migrate`
- [ ] 6.  Write unit test for repository query methods
- [ ] 7.  `php bin/phpunit`

## Entity template
```php
<?php declare(strict_types=1);
namespace App\Entity;
use App\Repository\{Name}Repository;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: {Name}Repository::class)]
#[ORM\Table(name: '{table}')]
#[ORM\HasLifecycleCallbacks]
class {Name} {
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private int $id;

    #[ORM\Column(type: 'string', length: 255)]
    private string $title;

    #[ORM\Column(type: 'string', length: 20)]
    private string $status = 'draft';

    #[ORM\Column(type: 'json', nullable: true)]
    private ?array $metadata = null;

    #[ORM\Column(type: 'datetime_immutable')]
    private \DateTimeImmutable $createdAt;

    // Add ManyToOne, OneToMany, etc. as needed
    // #[ORM\ManyToOne(targetEntity: User::class)]
    // #[ORM\JoinColumn(nullable: false, onDelete: 'CASCADE')]
    // private User $author;

    #[ORM\PrePersist]
    public function onPrePersist(): void { $this->createdAt = new \DateTimeImmutable(); }

    // Getters and setters...
    public function getId(): int { return $this->id; }
    public function getTitle(): string { return $this->title; }
    public function setTitle(string $title): self { $this->title = $title; return $this; }
}
```

## Repository template
```php
<?php declare(strict_types=1);
namespace App\Repository;
use App\Entity\{Name};
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Doctrine\Persistence\ManagerRegistry;

class {Name}Repository extends ServiceEntityRepository {
    public function __construct(ManagerRegistry $registry) {
        parent::__construct($registry, {Name}::class);
    }

    public function findActiveByUser(int $userId): array {
        return $this->createQueryBuilder('n')
            ->where('n.author = :userId AND n.status = :status')
            ->setParameter('userId', $userId)
            ->setParameter('status', 'active')
            ->orderBy('n.createdAt', 'DESC')
            ->getQuery()
            ->getResult();
    }
}
```
