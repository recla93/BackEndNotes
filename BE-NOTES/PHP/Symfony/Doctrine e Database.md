---
topic: "Symfony — Doctrine e Database"
tags: [symfony, doctrine, orm, database, entity, repository, dql, migrations]
nav_prev: "[[Routing e Controller.md]]"
nav_next: "[[Twig e Security.md]]"
---

Riferimento ufficiale: [symfony.com/doc/current/doctrine.html](https://symfony.com/doc/current/doctrine.html) | [doctrine-project.org](https://www.doctrine-project.org/)

Doctrine è l'ORM di Symfony. Implementa **Data Mapper** (a differenza di Eloquent che è Active Record): le entity sono Plain PHP Objects, e un `EntityManager` gestisce la persistenza. Supporta DQL (Doctrine Query Language) e un repository pattern nativo.

```bash
php bin/console make:entity User          # genera entity + repository
php bin/console make:migration            # genera migration da entity
php bin/console doctrine:migrations:migrate  # esegue migration
php bin/console doctrine:migrations:list  # elenca migration
php bin/console doctrine:migrations:diff  # genera migration da differenze entity/DB
php bin/console doctrine:schema:validate  # verifica sync entity↔DB
php bin/console dbal:run-sql "SELECT 1"   # esegue SQL raw
```

## Entity

```php
<?php
// src/Entity/User.php

namespace App\Entity;

use App\Repository\UserRepository;
use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;
use Doctrine\DBAL\Types\Types;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface;
use Symfony\Component\Security\Core\User\UserInterface;

#[ORM\Entity(repositoryClass: UserRepository::class)]
#[ORM\Table(name: "`users`")]
class User implements UserInterface, PasswordAuthenticatedUserInterface
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: Types::INTEGER)]
    private int $id;

    #[ORM\Column(type: Types::STRING, length: 100)]
    private string $name;

    #[ORM\Column(type: Types::STRING, length: 255, unique: true)]
    private string $email;

    #[ORM\Column(type: Types::BOOLEAN, options: ["default" => true])]
    private bool $active = true;

    #[ORM\Column(type: Types::DATETIME_MUTABLE, nullable: true)]
    private ?\DateTimeInterface $emailVerifiedAt = null;

    #[ORM\Column(type: Types::STRING)]
    private string $password;

    #[ORM\Column(type: Types::JSON)]
    private array $roles = [];

    #[ORM\OneToMany(targetEntity: Post::class, mappedBy: "user", cascade: ["persist", "remove"])]
    private Collection $posts;

    #[ORM\Column(type: Types::DATETIME_MUTABLE)]
    private \DateTimeInterface $createdAt;

    #[ORM\Column(type: Types::DATETIME_MUTABLE, nullable: true)]
    private ?\DateTimeInterface $updatedAt = null;

    // Lifecycle callback
    public function __construct()
    {
        $this->posts    = new ArrayCollection();
        $this->createdAt = new \DateTime();
    }

    // — Getters/Setters —

    public function getId(): int { return $this->id; }

    public function getName(): string { return $this->name; }
    public function setName(string $name): self { $this->name = $name; return $this; }

    public function getEmail(): string { return $this->email; }
    public function setEmail(string $email): self { $this->email = $email; return $this; }

    public function getPassword(): string { return $this->password; }
    public function setPassword(string $password): self { $this->password = $password; return $this; }

    public function getRoles(): array { return array_unique([...$this->roles, "ROLE_USER"]); }

    public function getPosts(): Collection { return $this->posts; }
    public function addPost(Post $post): self { $this->posts->add($post); $post->setUser($this); return $this; }
    public function removePost(Post $post): self { $this->posts->removeElement($post); return $this; }

    // UserInterface
    public function getUserIdentifier(): string { return $this->email; }
    public function eraseCredentials(): void {}
}
```

## Repository

```php
<?php
// src/Repository/UserRepository.php

namespace App\Repository;

use App\Entity\User;
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Doctrine\Persistence\ManagerRegistry;

/**
 * @extends ServiceEntityRepository<User>
 */
class UserRepository extends ServiceEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, User::class);
    }

    public function findActive(): array
    {
        return $this->createQueryBuilder("u")
            ->where("u.active = :active")
            ->setParameter("active", true)
            ->orderBy("u.name", "ASC")
            ->getQuery()
            ->getResult();
    }

    public function findByEmail(string $email): ?User
    {
        return $this->findOneBy(["email" => $email]);
    }

    public function findWithPosts(int $id): ?User
    {
        return $this->createQueryBuilder("u")
            ->leftJoin("u.posts", "p")
            ->addSelect("p")
            ->where("u.id = :id")
            ->setParameter("id", $id)
            ->getQuery()
            ->getOneOrNullResult();
    }

    /**
     * @return User[]
     */
    public function search(string $query, int $limit = 10): array
    {
        return $this->createQueryBuilder("u")
            ->where("u.name LIKE :query OR u.email LIKE :query")
            ->setParameter("query", "%$query%")
            ->setMaxResults($limit)
            ->getQuery()
            ->getResult();
    }

    // Con DQL raw
    public function countByRole(string $role): int
    {
        return $this->createQueryBuilder("u")
            ->select("COUNT(u.id)")
            ->where("u.roles LIKE :role")
            ->setParameter("role", "%\"$role\"%")
            ->getQuery()
            ->getSingleScalarResult();
    }
}
```

## CRUD con EntityManager

```php
<?php
// src/Service/UserService.php

namespace App\Service;

use App\Entity\User;
use App\Repository\UserRepository;
use Doctrine\ORM\EntityManagerInterface;

class UserService
{
    public function __construct(
        private readonly EntityManagerInterface $em,
        private readonly UserRepository $repository,
    ) {}

    public function create(array $data): User
    {
        $user = new User();
        $user->setName($data["name"]);
        $user->setEmail($data["email"]);
        $user->setPassword(password_hash($data["password"], PASSWORD_BCRYPT));

        $this->em->persist($user);   // tracked dal UnitOfWork
        $this->em->flush();           // write to DB

        return $user;
    }

    public function update(int $id, array $data): ?User
    {
        $user = $this->repository->find($id);
        if (!$user) return null;

        if (isset($data["name"])) $user->setName($data["name"]);
        if (isset($data["email"])) $user->setEmail($data["email"]);

        $this->em->flush();  // flush tracked changes — non serve persist()

        return $user;
    }

    public function delete(int $id): bool
    {
        $user = $this->repository->find($id);
        if (!$user) return false;

        $this->em->remove($user);
        $this->em->flush();

        return true;
    }

    public function findAll(): array
    {
        return $this->repository->findActive();
    }
}
```

## Migration

```php
<?php
// migrations/Version20260619120000.php

namespace DoctrineMigrations;

use Doctrine\DBAL\Schema\Schema;
use Doctrine\Migrations\AbstractMigration;

final class Version20260619120000 extends AbstractMigration
{
    public function getDescription(): string
    {
        return "Crea tabella users e posts";
    }

    public function up(Schema $schema): void
    {
        $this->addSql("
            CREATE TABLE users (
                id INT AUTO_INCREMENT NOT NULL,
                name VARCHAR(100) NOT NULL,
                email VARCHAR(255) NOT NULL UNIQUE,
                password VARCHAR(255) NOT NULL,
                roles JSON NOT NULL,
                active TINYINT(1) DEFAULT 1 NOT NULL,
                email_verified_at DATETIME DEFAULT NULL,
                created_at DATETIME NOT NULL,
                updated_at DATETIME DEFAULT NULL,
                PRIMARY KEY(id)
            ) DEFAULT CHARACTER SET utf8mb4
        ");
    }

    public function down(Schema $schema): void
    {
        $this->addSql("DROP TABLE IF EXISTS users");
    }
}
```

## DQL vs QueryBuilder vs Raw SQL

```php
<?php

// 1. QueryBuilder (preferito)
$qb = $this->createQueryBuilder("u");
$query = $qb
    ->where("u.active = :active")
    ->andWhere("u.createdAt >= :date")
    ->setParameter("active", true)
    ->setParameter("date", new \DateTime("-30 days"))
    ->orderBy("u.name", "ASC")
    ->setMaxResults(10)
    ->getQuery();

// 2. DQL raw
$dql = "SELECT u FROM App\Entity\User u WHERE u.active = :active ORDER BY u.name ASC";
$query = $this->em->createQuery($dql);
$query->setParameter("active", true);

// 3. Native SQL
$sql = "SELECT * FROM users WHERE active = :active";
$stmt = $this->em->getConnection()->prepare($sql);
$stmt->bindValue("active", true);
$result = $stmt->executeQuery()->fetchAllAssociative();
```

## Relazioni

```php
<?php
// src/Entity/Post.php

// ManyToOne (inverso di OneToMany)
#[ORM\ManyToOne(targetEntity: User::class, inversedBy: "posts")]
#[ORM\JoinColumn(name: "user_id", referencedColumnName: "id", nullable: false)]
private User $user;

// ManyToMany
#[ORM\ManyToMany(targetEntity: Category::class, inversedBy: "posts")]
#[ORM\JoinTable(name: "posts_categories")]
private Collection $categories;

// OneToOne
#[ORM\OneToOne(targetEntity: Profile::class, cascade: ["persist", "remove"])]
#[ORM\JoinColumn(name: "profile_id", referencedColumnName: "id")]
private ?Profile $profile = null;
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Entity has no field "email"` | Proprietà entity non mappata come colonna | Aggiungi `#[ORM\Column(type: Types::STRING)]` |
| `MappingException: Invalid field override` | Nome colonna riservato (es. `group`) | Usa backtick nel name: `#[ORM\Column(name: "\`group\`")]` |
| `A new entity was found through the relationship` | Entity non persistita ma referenziata | `$em->persist($entity)` prima di flush |
| `The class 'App\Entity\User' was not found in the chain` | Entity in bundle non registrato | Verifica `bundles.php` |
| `Migration not executed` | Migration generata ma non eseguita | `php bin/console doctrine:migrations:migrate` |
| `N+1 con Doctrine` | Relazioni lazy caricate in loop | Eager loading con `->leftJoin()->addSelect()` nel query |
| `"ObjectManager" not found` | EntityManager non type-hintato correttamente | Usa `EntityManagerInterface` (non `EntityManager`) |

## Best practice

- **Entity: Plain PHP Object** — nessuna logica di business nelle entity; solo getter/setter e logica di dominio semplice
- **Repository per query custom** — DQL in repository, non nel controller
- **`EntityManagerInterface`** — type-hint sull'interfaccia, non sulla classe concreta
- **Flush una sola volta** — raccogli operazioni, flush alla fine (batch)
- **Eager loading per relazioni in API** — `->leftJoin()->addSelect()` per evitare N+1
- **Migration versionate** — una migration per modifica dello schema; mai modificare migration già eseguita
- **DTO per input/output API** — entity non esposta direttamente; usa serialization groups o DTO
- **Cascade con cautela** — `cascade: ["persist"]` utile ma può causare operazioni involontarie
- **`getOneOrNullResult()` vs `getSingleResult()`** — preferisci `getOneOrNullResult()` (non lancia `NoResultException`)
- **`make:entity` + `make:migration`** — flusso entity → diff → migrate

## Cross-reference

- [[PHP/Symfony/Routing e Controller|Symfony — Routing e Controller]] — DTO, validazione entity
- [[PHP/Symfony/Twig e Security|Symfony — Twig e Security]] — User entity con UserInterface
- [[PHP/Database/PDO|PDO]] — Doctrine DBAL usa PDO sotto
- [[PHP/Laravel/Eloquent e Database|Laravel — Eloquent e Database]] — confronto Active Record vs Data Mapper
