---
topic: "Laravel — Setup e Architettura"
tags: [laravel, setup, architecture, service-container, facades, artisan]
nav_prev: "[[PDO.md]]"
nav_next: "[[Routing e Controller.md]]"
---

Riferimento ufficiale: [laravel.com/docs](https://laravel.com/docs/) | [github.com/laravel/laravel](https://github.com/laravel/laravel)

Laravel (2011, Taylor Otwell) è il framework PHP più diffuso. Architettura **modulare** con Service Container, Facades, Eloquent ORM, e un ecosistema integrato (Forge, Vapor, Jetstream, Octane). Segue il pattern **Model-View-Controller** ma lo estende con Service Layer, Repository, Action Classes.

```bash
# Installazione
composer create-project laravel/laravel my-app
cd my-app

# Avvio sviluppo
php artisan serve                    # server built-in su :8000
php artisan serve --port=8080

# Struttura
php artisan make:controller UserController
php artisan make:model User -m       # model + migration
php artisan make:service UserService  # (richiede package o app/ custom)
```

## Struttura progetto

```
my-app/
├── app/
│   ├── Http/
│   │   ├── Controllers/     # Controller (HTTP handlers)
│   │   ├── Middleware/       # Filtri richiesta (auth, throttle, CORS)
│   │   ├── Requests/        # Form request (validazione)
│   │   └── Kernel.php       # Config middleware stack
│   ├── Models/              # Eloquent model
│   ├── Services/            # Business logic (non generato da artisan)
│   ├── Repositories/        # Data access (non generato da artisan)
│   ├── Providers/           # Service provider (registrazione bind)
│   └── Console/
│       └── Commands/        # Comandi Artisan custom
├── bootstrap/
├── config/                  # Config per ambiente (env)
├── database/
│   ├── migrations/          # Schema DB versionato
│   ├── factories/           # Factory per test/seeder
│   └── seeders/             # Dati iniziali
├── resources/
│   └── views/               # Blade template (in .blade.php)
├── routes/
│   ├── web.php              # Route web (con sessione, CSRF)
│   └── api.php              # Route API (stateless, token)
├── storage/
├── tests/                   # PHPUnit test
└── artisan                  # CLI entry point
```

## Service Container

```php
<?php

// Binding (in ServiceProvider)
$this->app->bind(UserServiceInterface::class, UserService::class);
$this->app->singleton(LoggerService::class);

// Resolve automatico — type hint nel costruttore
class UserController
{
    public function __construct(
        private UserServiceInterface $userService,
        private LoggerService $logger,
    ) {}
}

// Resolve esplicito
$service = app(UserServiceInterface::class);
$service = resolve(UserServiceInterface::class);
```

### Facades

```php
<?php

// Facade = proxy statico a un bind del container
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;

Cache::put("key", "value", 3600);    // Cache facade
DB::table("users")->get();           // DB facade
Log::info("Messaggio");              // Log facade

// Equivalente senza facade:
app("cache")->put("key", "value", 3600);
app("db")->table("users")->get();
```

Le Facades forniscono accesso statico a servizi registrati nel container. Non sono metodi statici reali — ogni Facade chiama `getFacadeAccessor()` per risolvere l'istanza dal container.

## Service Provider

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\UserService;
use App\Repositories\UserRepository;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(UserRepository::class, function ($app) {
            return new UserRepository($app->make("db")->connection());
        });

        $this->app->singleton(UserService::class);
    }

    public function boot(): void
    {
        // Eseguito dopo che tutti i provider sono registrati
        // Qui: observer, route model binding, event listener
        \App\Models\User::observe(\App\Observers\UserObserver::class);
    }
}
```

## Config per ambiente

```php
<?php

// config/app.php — valori con fallback env
"name" => env("APP_NAME", "Laravel"),
"debug" => env("APP_DEBUG", false),
"url" => env("APP_URL", "http://localhost"),

// config/database.php
"mysql" => [
    "host" => env("DB_HOST", "127.0.0.1"),
    "port" => env("DB_PORT", "3306"),
    "database" => env("DB_DATABASE", "forge"),
    "username" => env("DB_USERNAME", "forge"),
    "password" => env("DB_PASSWORD", ""),
],
```

```bash
# .env
APP_NAME=MyApp
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myapp
```

## Ciclo di vita di una richiesta

```
Request → public/index.php → bootstrap/app.php → Kernel → Middleware (global) → Route → Middleware (route) → Controller → Service → Response
```

1. **index.php** carica autoloader e bootstrap
2. **Kernel** (HTTP) applica middleware globali
3. **Router** matcha la route e applica middleware di gruppo/route
4. **Controller** riceve request, delega a service, restituisce response
5. **Response** torna attraverso middleware (in reverse) → client

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Target [ClassName] is not instantiable` | Interface binding mancante | Registra bind in ServiceProvider |
| `Class "App\Models\User" not found` | Model non creato o namespace sbagliato | `php artisan make:model User` o correggi use |
| `SQLSTATE[HY000] [1045] Access denied for user` | Credenziali DB sbagliate in .env | Verifica .env e .env.example |
| `The only supported ciphers are AES-128-CBC and AES-256-CBC` | APP_KEY non generata | `php artisan key:generate` |
| `Route [login] not defined` | Middleware auth reindirizza a route login non definita | Definisci `Route::get("login", ...)->name("login")` |
| `Call to undefined function base_path()` | Helper functions non caricate | Verifica `composer.json` autoload files include `app/helpers.php` |
| `View [dashboard] not found` | Blade file mancante o path sbagliato | Crea `resources/views/dashboard.blade.php` |

## Best practice

- **`.env` mai in VCS** — solo `.env.example` con placeholder
- **`APP_KEY` generata** — `php artisan key:generate` obbligatorio dopo clone
- **Service provider per binding** — non mettere logica di bind in controller o service
- **Facades comode ma attenzione alle dipendenze implicite** — preferisci dependency injection nei controller
- **Config caching in produzione** — `php artisan config:cache` (non usare `env()` in config dopo cache)
- **Route caching in produzione** — `php artisan route:cache` (non usare closure route)
- **Struttura per dominio** — raggruppa controller/service/repository per feature, non per ruolo tecnico
- **Artisan per scaffolding** — genera model, controller, migration da CLI

## Cross-reference

- [[PHP/Laravel/Routing e Controller|Laravel — Routing e Controller]] — route definition, controller injection
- [[PHP/Laravel/Services e Repository|Laravel — Services e Repository]] — service layer pattern
- [[PHP/Laravel/Artisan e Testing|Laravel — Artisan e Testing]] — CLI comandi e test
- [[PHP/Core Concepts/Namespace e Autoload|Namespace e Autoload]] — PSR-4 mapping
