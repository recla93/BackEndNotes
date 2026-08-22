---
topic: "Symfony — Routing e Controller"
tags: [symfony, routing, controller, di, attributes, request, response]
nav_prev: "[[Setup e Architettura.md]]"
nav_next: "[[Doctrine e Database.md]]"
---

Riferimento ufficiale: [symfony.com/doc/current/routing.html](https://symfony.com/doc/current/routing.html) | [symfony.com/doc/current/controller.html](https://symfony.com/doc/current/controller.html)

Symfony usa **attributi PHP 8** per definire route direttamente nei controller. Il routing supporta parametri, metodi HTTP, prefissi, e name generation. Il controller è un servizio (autowired) con type-hint per request, response, validazione.

## Route con attributi

```php
<?php
// src/Controller/UserController.php

namespace App\Controller;

use App\Entity\User;
use App\Repository\UserRepository;
use App\Service\UserService;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Attribute\Route;

#[Route("/api/v1/users")]  // prefisso su tutto il controller
class UserController extends AbstractController
{
    public function __construct(
        private readonly UserService $userService,
    ) {}

    // GET /api/v1/users
    #[Route("", name: "users_index", methods: ["GET"])]
    public function index(): JsonResponse
    {
        $users = $this->userService->findAll();

        return $this->json([
            "data" => $users,
            "meta" => ["count" => count($users)],
        ]);
    }

    // GET /api/v1/users/{id}
    #[Route("/{id}", name: "users_show", methods: ["GET"], requirements: ["id" => "\d+"])]
    public function show(int $id): JsonResponse
    {
        $user = $this->userService->findById($id);

        if (!$user) {
            return $this->json(["error" => "User not found"], 404);
        }

        return $this->json(["data" => $user]);
    }

    // POST /api/v1/users
    #[Route("", name: "users_create", methods: ["POST"])]
    public function create(Request $request): JsonResponse
    {
        $data = json_decode($request->getContent(), true);

        // Validazione
        if (empty($data["email"])) {
            return $this->json(["error" => "Email is required"], 422);
        }

        try {
            $user = $this->userService->create($data);
            return $this->json(["data" => $user], 201);
        } catch (\RuntimeException $e) {
            return $this->json(["error" => $e->getMessage()], 409);
        }
    }

    // PUT /api/v1/users/{id}
    #[Route("/{id}", name: "users_update", methods: ["PUT"])]
    public function update(int $id, Request $request): JsonResponse
    {
        $data = json_decode($request->getContent(), true);
        $user = $this->userService->update($id, $data ?? []);

        if (!$user) {
            return $this->json(["error" => "User not found"], 404);
        }

        return $this->json(["data" => $user]);
    }

    // DELETE /api/v1/users/{id}
    #[Route("/{id}", name: "users_delete", methods: ["DELETE"])]
    public function delete(int $id): JsonResponse
    {
        $deleted = $this->userService->delete($id);

        if (!$deleted) {
            return $this->json(["error" => "User not found"], 404);
        }

        return $this->json(null, 204);
    }
}
```

### Route con ParamConverter

```php
<?php

use App\Entity\User;
use Symfony\Component\HttpKernel\Attribute\MapEntity;

// ParamConverter automatico — Symfony converte {id} in Entity
#[Route("/users/{id}", name: "users_show", methods: ["GET"])]
public function show(User $user): JsonResponse
{
    return $this->json(["data" => $user]);
}

// Con mappatura esplicita (via #[MapEntity])
#[Route("/posts/{postId}/comments/{commentId}")]
public function showComment(
    #[MapEntity(id: "postId")] Post $post,
    #[MapEntity(id: "commentId")] Comment $comment,
): JsonResponse {
    return $this->json(["post" => $post, "comment" => $comment]);
}
```

### Route YAML (alternativa)

```yaml
# config/routes.yaml
users_index:
    path: /api/v1/users
    controller: App\Controller\UserController::index
    methods: GET

users_show:
    path: /api/v1/users/{id}
    controller: App\Controller\UserController::show
    methods: GET
    requirements:
        id: '\d+'
```

## Controller best practices

```php
<?php

// AbstractController fornisce helper:
// - $this->json($data, $status, $headers, $context)
// - $this->redirectToRoute("route_name", $params)
// - $this->render("template.html.twig", $vars)
// - $this->isGranted("ROLE_ADMIN")
// - $this->denyAccessUnlessGranted("ROLE_ADMIN")
// - $this->createNotFoundException()
// - $this->getUser()

// Service injection nel costruttore
class UserController extends AbstractController
{
    public function __construct(
        private readonly UserService $userService,
        private readonly LoggerInterface $logger,     // PSR-3
    ) {}

    // Validazione con Symfony Validator
    #[Route("/users", methods: ["POST"])]
    public function create(Request $request, ValidatorInterface $validator): JsonResponse
    {
        $dto = new CreateUserDTO(json_decode($request->getContent(), true));

        $errors = $validator->validate($dto);
        if (count($errors) > 0) {
            return $this->json(["errors" => (string) $errors], 422);
        }

        $user = $this->userService->create($dto);
        return $this->json(["data" => $user], 201);
    }
}
```

## DTO per input

```php
<?php
// src/DTO/CreateUserDTO.php

namespace App\DTO;

use Symfony\Component\Validator\Constraints as Assert;

class CreateUserDTO
{
    #[Assert\NotBlank]
    #[Assert\Length(max: 255)]
    public readonly string $name;

    #[Assert\NotBlank]
    #[Assert\Email]
    public readonly string $email;

    #[Assert\NotBlank]
    #[Assert\Length(min: 8)]
    public readonly string $password;

    public function __construct(array $data)
    {
        $this->name     = $data["name"] ?? "";
        $this->email    = $data["email"] ?? "";
        $this->password = $data["password"] ?? "";
    }
}
```

## Exception handling

```php
<?php
// src/EventSubscriber/ExceptionSubscriber.php

namespace App\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpKernel\Event\ExceptionEvent;
use Symfony\Component\HttpKernel\Exception\HttpExceptionInterface;
use Symfony\Component\HttpKernel\KernelEvents;

class ExceptionSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [KernelEvents::EXCEPTION => "onKernelException"];
    }

    public function onKernelException(ExceptionEvent $event): void
    {
        $exception = $event->getThrowable();

        $statusCode = $exception instanceof HttpExceptionInterface
            ? $exception->getStatusCode()
            : 500;

        $event->setResponse(new JsonResponse([
            "error"   => $exception->getMessage(),
            "code"    => $statusCode,
            "trace"   => $_ENV["APP_ENV"] === "dev"
                ? $exception->getTraceAsString()
                : null,
        ], $statusCode));
    }
}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `No route found for "GET /users"` | Route non definita o metodo HTTP sbagliato | `php bin/console debug:router` per elenco route |
| `Controller must return a Response` | Controller non restituisce `Response` o `JsonResponse` | Usa `$this->json()` o `$this->render()` |
| `Cannot autowire argument $request` | Request non type-hintata come `Request` | `use Symfony\Component\HttpFoundation\Request;` |
| `The controller for URI "/users/abc" is not callable` | Parametro non matcha requisiti | Aggiungi `requirements: ["id" => "\d+"]` |
| `Argument #1 must be of type User, string given` | ParamConverter fallisce per ID inesistente | Usa `#[MapEntity]` o `findOrFail` |
| `Attempted to load class "User" from namespace` | Entity import mancante | `use App\Entity\User;` |
| `Unable to generate a URL for the named route` | `$this->generateUrl()` con nome route sbagliato | Verifica `name:` nella route o `debug:router` |

## Best practice

- **Attributi PHP 8 per route** — più leggibili e co-localizzati col controller
- **AbstractController** — fornisce helper JSON, render, redirect, deny access
- **DTO per input** — mai usare array associativi; DTO con validazione integrata
- **Service injection** — controller sottile, service spesso
- **ParamConverter per entity** — evita `$repo->find($id)` manuali
- **Exception subscriber** — error handling centralizzato, non try/catch in ogni controller
- **`$this->json()`** — setta Content-Type automaticamente, accetta serialization groups
- **`debug:router`** — comando essenziale per debugging routing
- **Route YAML per gruppi esterni** — se preferisci separare route dal codice
- **Versione API in prefisso** — `/api/v1/` per versionamento

## Cross-reference

- [[PHP/Symfony/Doctrine e Database|Symfony — Doctrine e Database]] — entity, repository, DQL
- [[PHP/Symfony/Twig e Security|Symfony — Twig e Security]] — template e autenticazione
- [[PHP/Symfony/Setup e Architettura|Symfony — Setup]] — service container, autowiring
- [[PHP/Laravel/Routing e Controller|Laravel — Routing e Controller]] — confronto approccio
