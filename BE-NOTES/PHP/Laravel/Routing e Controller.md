---
topic: "Laravel — Routing e Controller"
tags: [laravel, routing, controller, middleware, request, validation]
nav_prev: "[[Setup e Architettura.md]]"
nav_next: "[[Eloquent e Database.md]]"
---

Riferimento ufficiale: [laravel.com/docs/routing](https://laravel.com/docs/routing) | [laravel.com/docs/controllers](https://laravel.com/docs/controllers)

Laravel separa la definizione delle route (file dedicati) dalla logica di gestione (controller). Il routing supporta parametri, middleware, gruppi, e binding automatico del model.

## Definizione route

```php
<?php
// routes/web.php — session, CSRF, cookie
// routes/api.php — stateless, token, prefix /api

use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;

// GET
Route::get("/users", [UserController::class, "index"]);
Route::get("/users/{id}", [UserController::class, "show"]);

// POST
Route::post("/users", [UserController::class, "store"]);

// PUT / PATCH
Route::put("/users/{id}", [UserController::class, "update"]);
Route::patch("/users/{id}", [UserController::class, "partial"]);

// DELETE
Route::delete("/users/{id}", [UserController::class, "destroy"]);

// Multi-method
Route::match(["GET", "POST"], "/users/search", [UserController::class, "search"]);
Route::any("/users/status", [UserController::class, "status"]);
```

### Parametri e vincoli

```php
<?php

// Parametro obbligatorio
Route::get("/users/{id}", ...);

// Parametro opzionale
Route::get("/users/{id?}", ...)->whereNumber("id");

// Vincoli regex
Route::get("/users/{id}", ...)->where("id", "[0-9]+");
Route::get("/posts/{slug}", ...)->where("slug", "[a-z0-9-]+");

// Vincoli globali (in App\Providers\AppServiceProvider)
public function boot(): void
{
    Route::pattern("id", "[0-9]+");
}
```

### Route naming

```php
<?php

// Nome — genera URL con route()
Route::get("/users/{id}", ...)->name("users.show");

// Uso: redirect, link
$url = route("users.show", ["id" => 42]);
// /users/42

// Resource controller
Route::resource("users", UserController::class);
// GET       /users              → index    → users.index
// GET       /users/create       → create   → users.create
// POST      /users              → store    → users.store
// GET       /users/{user}       → show     → users.show
// GET       /users/{user}/edit  → edit     → users.edit
// PUT/PATCH /users/{user}       → update   → users.update
// DELETE    /users/{user}       → destroy  → users.destroy
```

### Route group

```php
<?php

Route::prefix("admin")->group(function () {
    Route::get("/dashboard", [DashboardController::class, "index"]);
    Route::resource("/users", AdminUserController::class);
});

Route::middleware(["auth", "verified"])->group(function () {
    Route::get("/profile", [ProfileController::class, "show"]);
    Route::post("/profile", [ProfileController::class, "update"]);
});

Route::prefix("api/v1")->middleware("api")->group(base_path("routes/api.php"));
```

### Route model binding

```php
<?php

// Binding implicito — {user} → istanza User
Route::get("/users/{user}", function (User $user) {
    return $user;  // già caricato dal DB
});

// Binding esplicito (in RouteServiceProvider)
public function boot(): void
{
    Route::bind("user", function (string $value): User {
        return User::where("slug", $value)->firstOrFail();
    });
}
```

## Controller

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Models\User;
use App\Services\UserService;
use App\Http\Requests\StoreUserRequest;
use App\Http\Resources\UserResource;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function __construct(
        private readonly UserService $userService,
    ) {}

    // Index — lista paginata
    public function index(Request $request): JsonResponse
    {
        $users = $this->userService->paginate(
            $request->input("per_page", 15)
        );

        return response()->json([
            "data" => UserResource::collection($users),
            "meta" => [
                "page" => $users->currentPage(),
                "total" => $users->total(),
            ],
        ]);
    }

    // Show — singolo con resource
    public function show(User $user): JsonResponse
    {
        return response()->json([
            "data" => new UserResource($user->load("posts")),
        ]);
    }

    // Store — validazione via Form Request
    public function store(StoreUserRequest $request): JsonResponse
    {
        $user = $this->userService->create($request->validated());

        return response()->json([
            "data" => new UserResource($user),
        ], 201);
    }

    // Update
    public function update(StoreUserRequest $request, User $user): JsonResponse
    {
        $user = $this->userService->update($user, $request->validated());

        return response()->json([
            "data" => new UserResource($user),
        ]);
    }

    // Destroy
    public function destroy(User $user): JsonResponse
    {
        $this->userService->delete($user);

        return response()->noContent();
    }
}
```

## Form Request (validazione)

```php
<?php
// app/Http/Requests/StoreUserRequest.php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        // Solo admin può creare utenti
        return $this->user()?->isAdmin() ?? false;
    }

    public function rules(): array
    {
        return [
            "name"     => "required|string|max:255",
            "email"    => "required|email|unique:users,email",
            "password" => "required|string|min:8|confirmed",
            "roles"    => "array",
            "roles.*"  => "exists:roles,id",
        ];
    }

    public function messages(): array
    {
        return [
            "name.required"     => "Il nome è obbligatorio",
            "email.unique"      => "Email già registrata",
            "password.min"      => "Password minima 8 caratteri",
            "password.confirmed" => "Le password non corrispondono",
        ];
    }
}
```

## Middleware

```php
<?php
// app/Http/Middleware/CheckUserRole.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CheckUserRole
{
    public function handle(Request $request, Closure $next, string ...$roles): mixed
    {
        if (!$request->user() || !in_array($request->user()->role, $roles)) {
            abort(403, "Unauthorized");
        }

        return $next($request);
    }
}
```

```php
<?php
// Registrazione in Kernel o route
Route::middleware("role:admin,editor")->group(function () {
    Route::resource("/admin/users", AdminController::class);
});
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Route [login] not defined` | auth middleware → redirect a login | `php artisan make:auth` o definisci `route("login")` |
| `405 Method Not Allowed` | POST su route definita come GET | Matcha il metodo HTTP nella definizione route |
| `NotFoundHttpException` | Route non matcha | Verifica metodo HTTP, URI, e `Route::resource` naming |
| `MassAssignmentException` | Campi non in `$fillable` nel model | Aggiungi `$fillable` o `$guarded` nel Model |
| `ValidationException` | Form request fallisce | Leggi `$errors` in Blade o response JSON |
| `Relation "posts" not found` | `load()` chiama relazione inesistente | Verifica metodo relazione nel Model |
| `Route [password.reset] not defined` | Route di reset password non registrata | `php artisan make:auth` o definisci manualmente |

## Best practice

- **Resource controller per CRUD standard** — evita duplicazione; solo metodi `index/create/store/show/edit/update/destroy`
- **Form Request per validazione** — separa logica di validazione dal controller
- **Route model binding** — evita `User::findOrFail($id)` manuali nei controller
- **Route name per URL** — mai hardcodare `/users/42`; usa `route("users.show", $id)`
- **Middleware sottili** — una responsabilità per middleware (auth, throttle, log, role)
- **API resource per response JSON** — `php artisan make:resource UserResource` per trasformare model in JSON
- **Group per prefisso/middleware comune** — evita ripetizione su ogni route
- **`php artisan route:list`** — mostra tutte le route registrate con middleware

## Cross-reference

- [[PHP/Laravel/Services e Repository|Laravel — Services e Repository]] — logica di business in service
- [[PHP/Laravel/Eloquent e Database|Laravel — Eloquent e Database]] — model binding, query
- [[PHP/Laravel/Auth e Sicurezza|Laravel — Auth e Sicurezza]] — middleware auth, gate
- [[PHP/Web/REST API|REST API]] — API routing pattern (senza framework)
