---
topic: "Laravel — Services e Repository"
tags: [laravel, services, repository, pattern, di, business-logic, architecture]
nav_prev: "[[Blade e Template.md]]"
nav_next: "[[Artisan e Testing.md]]"
---

Riferimento ufficiale: [laravel.com/docs/container](https://laravel.com/docs/container) | PHP The Right Way — Service Layer

I **Service** contengono la logica di business. I **Repository** isolano l'accesso dati. I controller gestiscono solo HTTP (request/response). Questa separazione produce codice testabile, manutenibile, e framework-agnostico.

## Service Layer

```php
<?php
// app/Services/UserService.php

namespace App\Services;

use App\Models\User;
use App\Repositories\UserRepository;
use App\Notifications\WelcomeEmail;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;

class UserService
{
    public function __construct(
        private readonly UserRepository $repository,
        private readonly MailService $mailService,
        private readonly AuditService $auditService,
    ) {}

    public function create(array $data): User
    {
        $this->validateEmailNotTaken($data["email"]);

        return DB::transaction(function () use ($data) {
            $data["password"] = bcrypt($data["password"]);
            $data["uuid"]     = (string) Str::uuid();

            // Repository fa INSERT
            $user = $this->repository->create($data);

            // Side-effect: notifica
            $this->mailService->sendWelcomeEmail($user);

            // Side-effect: audit log
            $this->auditService->log("user.created", ["user_id" => $user->id]);

            // Dispatch evento
            UserCreated::dispatch($user);

            return $user;
        });
    }

    public function update(int $id, array $data): User
    {
        $user = $this->repository->findOrFail($id);

        if (isset($data["email"]) && $data["email"] !== $user->email) {
            $this->validateEmailNotTaken($data["email"]);
        }

        return DB::transaction(function () use ($user, $data) {
            $this->repository->update($user, $data);
            $this->auditService->log("user.updated", ["user_id" => $user->id]);
            return $user->fresh();
        });
    }

    public function delete(int $id): bool
    {
        $user = $this->repository->findOrFail($id);

        return DB::transaction(function () use ($user) {
            $this->repository->delete($user);
            $this->auditService->log("user.deleted", ["user_id" => $user->id]);
            return true;
        });
    }

    public function paginate(int $perPage = 15): LengthAwarePaginator
    {
        return $this->repository->paginate($perPage);
    }

    private function validateEmailNotTaken(string $email): void
    {
        if ($this->repository->findByEmail($email)) {
            throw new \RuntimeException("Email già registrata");
        }
    }
}
```

### Controller che consuma il Service

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Services\UserService;
use App\Http\Requests\StoreUserRequest;
use App\Http\Resources\UserResource;
use Illuminate\Http\JsonResponse;

class UserController extends Controller
{
    public function __construct(
        private readonly UserService $userService,
    ) {}

    public function index(): JsonResponse
    {
        $users = $this->userService->paginate(
            request("per_page", 15)
        );

        return response()->json([
            "data" => UserResource::collection($users),
            "meta" => [
                "page"  => $users->currentPage(),
                "total" => $users->total(),
            ],
        ]);
    }

    public function store(StoreUserRequest $request): JsonResponse
    {
        $user = $this->userService->create($request->validated());

        return response()->json([
            "data" => new UserResource($user),
        ], 201);
    }

    public function show(int $id): JsonResponse
    {
        $user = $this->userService->findOrFail($id);

        return response()->json([
            "data" => new UserResource($user->load("posts", "role")),
        ]);
    }

    public function update(StoreUserRequest $request, int $id): JsonResponse
    {
        $user = $this->userService->update($id, $request->validated());

        return response()->json([
            "data" => new UserResource($user),
        ]);
    }

    public function destroy(int $id): JsonResponse
    {
        $this->userService->delete($id);

        return response()->noContent();
    }
}
```

## Repository Pattern

```php
<?php
// app/Repositories/UserRepository.php

namespace App\Repositories;

use App\Models\User;
use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Database\Eloquent\Collection;

class UserRepository
{
    public function __construct(
        private readonly User $model,
    ) {}

    public function findById(int $id): ?User
    {
        return $this->model->find($id);
    }

    public function findOrFail(int $id): User
    {
        return $this->model->findOrFail($id);
    }

    public function findByEmail(string $email): ?User
    {
        return $this->model->where("email", $email)->first();
    }

    public function findAllActive(): Collection
    {
        return $this->model->active()->verified()->get();
    }

    public function paginate(int $perPage = 15): LengthAwarePaginator
    {
        return $this->model->orderBy("created_at", "desc")->paginate($perPage);
    }

    public function create(array $data): User
    {
        return $this->model->create($data);
    }

    public function update(User $user, array $data): User
    {
        $user->update($data);
        return $user;
    }

    public function delete(User $user): bool
    {
        return $user->delete();
    }

    public function countByRole(int $roleId): int
    {
        return $this->model->where("role_id", $roleId)->count();
    }
}
```

## Action Classes (alternativa a Service)

```php
<?php
// app/Actions/RegisterUserAction.php

namespace App\Actions;

use App\Models\User;
use App\Notifications\WelcomeEmail;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;

class RegisterUserAction
{
    public function __construct(
        private readonly MailService $mailService,
    ) {}

    public function execute(array $data): User
    {
        return DB::transaction(function () use ($data) {
            $user = User::create([
                "uuid"     => (string) Str::uuid(),
                "name"     => $data["name"],
                "email"    => $data["email"],
                "password" => bcrypt($data["password"]),
            ]);

            $user->assignRole("user");
            $this->mailService->sendWelcomeEmail($user);
            UserRegistered::dispatch($user);

            return $user;
        });
    }
}
```

```php
<?php
// Uso in controller
use App\Actions\RegisterUserAction;

class AuthController extends Controller
{
    public function register(RegisterRequest $request): JsonResponse
    {
        $user = (new RegisterUserAction($this->mailService))->execute(
            $request->validated()
        );

        return response()->json(["data" => new UserResource($user)], 201);
    }
}
```

### Service vs Action

| Criterio | Service | Action |
|---|---|---|
| **Scopo** | CRUD per entità | Operazione singola specifica |
| **Metodi** | Multipli (create, update, delete, paginate) | Singolo (`execute()`) |
| **Dimensione** | Medio (5-10 metodi) | Piccolo (1 metodo) |
| **Riutilizzo** | Condiviso tra controller, command, job | Specifico per use case |
| **Quando usare** | CRUD standard, gestione entità | Use case complesso (registrazione, checkout, export) |

## Dependency Injection

```php
<?php
// app/Providers/AppServiceProvider.php

use App\Services\UserService;
use App\Repositories\UserRepository;

public function register(): void
{
    // Service — singleton
    $this->app->singleton(UserService::class);

    // Repository — transient (nuova istanza per ogni richiesta)
    $this->app->bind(UserRepository::class);

    // Interface → Implementation
    $this->app->bind(UserRepositoryInterface::class, UserRepository::class);
}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Call to undefined method ...::paginate()` | Repository non restituisce LengthAwarePaginator | Assicura che metodo repository usi `->paginate()` |
| `Transaction deadlock` | Transazioni annidate o timeout | Verifica ordine operazioni; aumenta timeout |
| `Model not found in repository` | `findOrFail` ma record cancellato | Usa `SoftDeletes` + `withTrashed()` se serve |
| `Service troppo grande` | Service cresce con troppe responsabilità | Dividi in action class o servizi specializzati |
| `Repository che non usa Eloquent` | Query builder raw persiste nel repository | Usa model Eloquent nel repository, non DB facade |
| `Controller che chiama repository direttamente` | Controller bypassa service layer | Controller → Service → Repository (catena) |

## Best practice

- **Controller sottili, Service spessi** — controller gestisce solo HTTP (status code, response format); service la logica
- **Repository isola ORM** — se cambi da Eloquent a Doctrine, cambi solo repository
- **Transazioni nel service** — `DB::transaction()` nel service, mai nel controller o repository
- **Side-effect solo dopo commit** — notifiche, eventi, log dentro la transazione (o dispatchAfterResponse)
- **Action Class per use case complessi** — più di 3-4 step in un metodo → action class separata
- **Repository non per CRUD banale** — se il repository è solo `find/create/update/delete` delegati a Eloquent, è overhead inutile
- **Validator fuori dal service** — usa Form Request per validazione input; service assume dati già validi
- **`fresh()` dopo update** — `$user->fresh()` ricarica da DB (relazioni, computed)
- **Testabilità** — service che inietta repository è testabile con mock

## Cross-reference

- [[PHP/Laravel/Routing e Controller|Laravel — Routing e Controller]] — controller che consumano service
- [[PHP/Laravel/Eloquent e Database|Laravel — Eloquent e Database]] — model che repository usa
- [[PHP/Web/REST API|REST API]] — pattern service/repository senza framework
- [[PHP/Core Concepts/Classi e OOP|Classi e OOP]] — DI, interfacce, tipo generico
