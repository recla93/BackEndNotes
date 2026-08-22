---
topic: "Laravel — Auth e Sicurezza"
tags: [laravel, auth, security, sanctum, gate, policy, middleware]
nav_prev: "[[Artisan e Testing.md]]"
---

Riferimento ufficiale: [laravel.com/docs/authentication](https://laravel.com/docs/authentication) | [laravel.com/docs/authorization](https://laravel.com/docs/authorization) | [laravel.com/docs/sanctum](https://laravel.com/docs/sanctum)

Laravel fornisce authentication (chi è) e authorization (cosa può fare) built-in. Supporta session-based (web) e token-based (API) con Sanctum. Policies e Gates per autorizzazione granulare.

## Authentication — Web (session)

```php
<?php
// config/auth.php

return [
    "defaults" => [
        "guard" => "web",
        "passwords" => "users",
    ],

    "guards" => [
        "web" => [
            "driver" => "session",
            "provider" => "users",
        ],
        "api" => [
            "driver" => "sanctum",
            "provider" => "users",
        ],
    ],

    "providers" => [
        "users" => [
            "driver" => "eloquent",
            "model" => App\Models\User::class,
        ],
    ],
];
```

### Login manuale

```php
<?php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function login(LoginRequest $request): JsonResponse
    {
        $credentials = $request->only("email", "password");

        if (!Auth::attempt($credentials, $request->boolean("remember"))) {
            throw new AuthenticationException("Credenziali non valide");
        }

        $request->session()->regenerate();

        return response()->json([
            "data" => new UserResource(Auth::user()),
        ]);
    }

    public function logout(Request $request): JsonResponse
    {
        Auth::logout();
        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return response()->json(["message" => "Logout effettuato"]);
    }
}
```

## Authentication — API con Sanctum

Sanctum è il token API nativo di Laravel. Usa token semplici (per API stateless) o SPA authentication (con session cookie).

```bash
php artisan install:api               # setup Sanctum (Laravel 11+)
php artisan make:controller AuthController  # controller auth
```

```php
<?php
// app/Http/Controllers/AuthController.php

use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    public function register(RegisterRequest $request): JsonResponse
    {
        $user = User::create([
            "name"     => $request->name,
            "email"    => $request->email,
            "password" => bcrypt($request->password),
        ]);

        $token = $user->createToken("auth-token")->plainTextToken;

        return response()->json([
            "data"  => new UserResource($user),
            "token" => $token,
        ], 201);
    }

    public function login(LoginRequest $request): JsonResponse
    {
        $user = User::where("email", $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                "email" => ["Credenziali non valide"],
            ]);
        }

        $token = $user->createToken("auth-token")->plainTextToken;

        return response()->json([
            "data"  => new UserResource($user),
            "token" => $token,
        ]);
    }

    public function logout(Request $request): JsonResponse
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json(["message" => "Logout effettuato"]);
    }
}
```

### Middleware Sanctum

```php
<?php
// routes/api.php

use Illuminate\Support\Facades\Route;

Route::post("/register", [AuthController::class, "register"]);
Route::post("/login", [AuthController::class, "login"]);

Route::middleware("auth:sanctum")->group(function () {
    Route::get("/user", fn(Request $r) => new UserResource($r->user()));
    Route::post("/logout", [AuthController::class, "logout"]);
    Route::apiResource("/users", UserController::class);
});
```

### Token abilities

```php
<?php

// Creazione con abilità specifiche
$token = $user->createToken("admin-token", ["user:create", "user:delete"]);
$token = $user->createToken("readonly-token", ["user:read"]);

// Verifica
if ($request->user()->tokenCan("user:create")) {
    // può creare utenti
}

// Verifica in middleware
Route::middleware("abilities:user:create,user:delete")->group(...);
Route::middleware("ability:admin")->group(...);    // almeno una
```

## Authorization — Gates

```php
<?php
// App\Providers\AppServiceProvider::boot()

use App\Models\User;
use App\Models\Post;
use Illuminate\Support\Facades\Gate;

// Definizione gate
Gate::define("update-post", function (User $user, Post $post): bool {
    return $user->id === $post->user_id || $user->isAdmin();
});

Gate::define("delete-post", function (User $user, Post $post): bool {
    return $user->id === $post->user_id;
});
```

```php
<?php
// Uso in controller

public function update(UpdatePostRequest $request, Post $post): JsonResponse
{
    Gate::authorize("update-post", $post);
    // ... aggiorna
}

public function destroy(Post $post): JsonResponse
{
    if (Gate::denies("delete-post", $post)) {
        abort(403);
    }
    // ... elimina
}
```

## Authorization — Policies

```php
<?php
// app/Policies/PostPolicy.php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\HandlesAuthorization;

class PostPolicy
{
    use HandlesAuthorization;

    public function viewAny(User $user): bool
    {
        return true;  // tutti vedono la lista
    }

    public function view(User $user, Post $post): bool
    {
        return $post->is_published || $user->id === $post->user_id;
    }

    public function create(User $user): bool
    {
        return $user->hasVerifiedEmail();
    }

    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->isAdmin();
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->isAdmin();
    }

    // Before — override globale (es. admin bypass)
    public function before(User $user, string $ability): ?bool
    {
        if ($user->isSuperAdmin()) {
            return true;  // super-admin fa tutto
        }
        return null;      // delega alla policy specifica
    }
}
```

```php
<?php
// Registrazione in AppServiceProvider
use App\Models\Post;
use App\Policies\PostPolicy;

public function boot(): void
{
    Gate::policy(Post::class, PostPolicy::class);
}
```

```php
<?php
// Uso in controller (con autorizzazione automatica)
use App\Models\Post;

class PostController extends Controller
{
    public function __construct()
    {
        $this->authorizeResource(Post::class, "post");
    }
}

// Uso esplicito
public function update(Post $post): JsonResponse
{
    $this->authorize("update", $post);
    // ...
}

// In Blade
@can("update", $post)
    <a href="/posts/{{ $post->id }}/edit">Modifica</a>
@endcan

@cannot("delete", $post)
    <p>Non puoi eliminare questo post</p>
@endcannot
```

## Rate Limiting

```php
<?php
// App\Http\Kernel (o routes/api.php)

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

RateLimiter::for("api", fn(Request $r) => Limit::perMinute(60)->by($r->user()?->id ?: $r->ip()));
RateLimiter::for("login", fn(Request $r) => Limit::perMinute(5)->by($r->input("email") . "|" . $r->ip()));
```

```php
<?php
// Applicazione su route
Route::middleware("throttle:login")->post("/login", [AuthController::class, "login"]);
Route::middleware("throttle:api")->group(base_path("routes/api.php"));
```

## Sicurezza — middleware built-in

```php
<?php
// App\Http\Kernel

protected $middleware = [
    // \App\Http\Middleware\TrustHosts::class,
    \App\Http\Middleware\TrustProxies::class,
    \Illuminate\Http\Middleware\HandleCors::class,
    \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
    \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
    \App\Http\Middleware\TrimStrings::class,
    \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
];

protected $routeMiddleware = [
    "auth"             => \App\Http\Middleware\Authenticate::class,
    "auth.basic"       => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    "auth.session"     => \Illuminate\Session\Middleware\AuthenticateSession::class,
    "cache.headers"    => \Illuminate\Http\Middleware\SetCacheHeaders::class,
    "can"              => \Illuminate\Auth\Middleware\Authorize::class,
    "guest"            => \App\Http\Middleware\RedirectIfAuthenticated::class,
    "password.confirm" => \Illuminate\Auth\Middleware\RequirePassword::class,
    "signed"           => \App\Http\Middleware\ValidateSignature::class,
    "throttle"         => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    "verified"         => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
];
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Route [login] not defined` | Middleware auth redirect a `/login` non definito | Definisci route login o customizza `unauthenticated()` |
| `Unauthenticated` su route API | Token non inviato in header | `Authorization: Bearer {token}` |
| `Token non trovato` | Tabella `personal_access_tokens` non creata | `php artisan migrate` (Sanctum migration) |
| `403 This action is unauthorized` | Gate/Policy nega accesso | Verifica `$user->id === $post->user_id` |
| `CSRF token mismatch` | POST senza `@csrf` in Blade o `X-XSRF-TOKEN` in SPA | Aggiungi token o disabilita per API |
| `Rate limit exceeded` | Troppe richieste in finestra | Aumenta limit o attendi |
| `Email verification required` | `verified` middleware su utente non verificato | Verifica email o rimuovi middleware |
| `Method [emailVerified] does not exist` | Config email verification senza trait | `User implements MustVerifyEmail` |

## Best practice

- **Sanctum per API** — token semplice, built-in, nessuna dependency extra (Passport/JWT)
- **`auth:sanctum` su route API** — protegge endpoint autenticati
- **`createToken()` con abilities** — limita permessi al minimo necessario
- **Gate/Policy per autorizzazione** — logica centralizzata, riusabile in controller, Blade, test
- **`authorizeResource` per CRUD** — autorizza automaticamente su resource controller
- **Rate limiting su login/register** — previene brute force
- **HTTPS in produzione** — `APP_URL=https://` + `force scheme` in middleware TrustProxies
- **CORS configurato** — `config/cors.php` con `allowed_origins` espliciti
- **`before()` in policy per admin** — super-admin bypassa controlli
- **`Hash::check()` per password** — mai confrontare password in chiaro

## Cross-reference

- [[PHP/Laravel/Routing e Controller|Laravel — Routing e Controller]] — middleware auth su route
- [[PHP/Laravel/Services e Repository|Laravel — Services e Repository]] — servizi che usano auth
- [[PHP/Web/Sessioni e Cookie|Sessioni e Cookie]] — session-based auth (web)
- [[PHP/Core Concepts/Classi e OOP|Classi e OOP]] — trait, interfacce (MustVerifyEmail)
