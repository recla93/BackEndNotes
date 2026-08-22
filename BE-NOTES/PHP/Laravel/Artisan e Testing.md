---
topic: "Laravel — Artisan e Testing"
tags: [laravel, artisan, testing, phpunit, tdd, cli, command]
nav_prev: "[[Services e Repository.md]]"
nav_next: "[[Auth e Sicurezza.md]]"
---

Riferimento ufficiale: [laravel.com/docs/artisan](https://laravel.com/docs/artisan) | [laravel.com/docs/testing](https://laravel.com/docs/testing)

Artisan è la CLI di Laravel. Genera codice, gestisce migration, esegue comandi custom. I test usano PHPUnit con estensioni Laravel (HTTP test, DB test, mock).

## Artisan — comandi built-in

```bash
# Sviluppo
php artisan serve                     # server built-in su :8000
php artisan tinker                    # REPL interattivo (psysh)
php artisan make:model User -m        # genera model + migration
php artisan make:controller UserController --resource
php artisan make:request StoreUserRequest
php artisan make:resource UserResource
php artisan make:mail WelcomeEmail
php artisan make:notification WelcomeNotification
php artisan make:event UserCreated
php artisan make:listener SendWelcomeEmail --event=UserCreated
php artisan make:job ProcessPodcast
php artisan make:middleware CheckRole
php artisan make:observer UserObserver --model=User
php artisan make:scope ActiveUsers
php artisan make:factory UserFactory --model=User
php artisan make:seeder UserSeeder
php artisan make:test UserServiceTest
php artisan make:test UserApiTest --unit

# Database
php artisan migrate
php artisan migrate:fresh --seed
php artisan db:seed
php artisan db:seed --class=UserSeeder

# Cache
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan cache:clear                # rimuove cache

# Ottimizzazione produzione
php artisan optimize
php artisan storage:link               # collega storage pubblico
```

## Comandi custom

```php
<?php
// app/Console/Commands/ImportUsers.php

namespace App\Console\Commands;

use App\Services\ImportService;
use Illuminate\Console\Command;

class ImportUsers extends Command
{
    protected $signature = "import:users
        {file : Percorso del file CSV}
        {--dry-run : Simula senza salvare}
        {--batch=100 : Numero righe per batch}
        {--encoding=utf-8 : Encoding del file}";

    protected $description = "Importa utenti da file CSV";

    public function __construct(
        private readonly ImportService $importService,
    ) {
        parent::__construct();
    }

    public function handle(): int
    {
        $file    = $this->argument("file");
        $dryRun  = $this->option("dry-run");
        $batch   = (int) $this->option("batch");
        $encoding = $this->option("encoding");

        if (!file_exists($file)) {
            $this->error("File non trovato: $file");
            return Command::FAILURE;
        }

        $this->info("Avvio import da: $file");
        $this->warn($dryRun ? "MODALITÀ DRY-RUN — nessun dato salvato" : "");

        $bar = $this->output->createProgressBar(100);

        try {
            $result = $this->importService->import($file, $dryRun, $batch, $encoding);

            $bar->finish();
            $this->newLine();

            $this->table(
                ["Importati", "Saltati", "Errori"],
                [[$result["imported"], $result["skipped"], $result["errors"]]]
            );

            if ($result["errors"] > 0) {
                $this->line("Log errori: storage/logs/import-errors.log");
            }

            return Command::SUCCESS;
        } catch (\Throwable $e) {
            $this->error("Errore: " . $e->getMessage());
            return Command::FAILURE;
        }
    }
}
```

```bash
# Esecuzione
php artisan import:users storage/app/users.csv
php artisan import:users storage/app/users.csv --dry-run
php artisan import:users storage/app/users.csv --batch=500 --encoding=windows-1252
```

## Testing con PHPUnit

```xml
<!-- phpunit.xml — config già presente in Laravel -->
<phpunit>
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
    </testsuites>
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="DB_CONNECTION" value="sqlite"/>
        <env name="DB_DATABASE" value=":memory:"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="QUEUE_CONNECTION" value="sync"/>
        <env name="MAIL_MAILER" value="array"/>
    </php>
</phpunit>
```

```bash
php artisan test                       # esegue tutti i test
php artisan test --filter=UserTest     # filtra per classe
php artisan test --parallel            # esecuzione parallela (Laravel 10+)
php artisan test --coverage            # coverage HTML
```

### Unit test (servizi, repository)

```php
<?php
// tests/Unit/Services/UserServiceTest.php

namespace Tests\Unit\Services;

use App\Models\User;
use App\Services\UserService;
use App\Repositories\UserRepository;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class UserServiceTest extends TestCase
{
    use RefreshDatabase;

    private UserService $service;

    protected function setUp(): void
    {
        parent::setUp();
        $this->service = app(UserService::class);
    }

    public function test_create_user_successfully(): void
    {
        $data = [
            "name"     => "Mario",
            "email"    => "mario@example.com",
            "password" => "securePass123",
        ];

        $user = $this->service->create($data);

        $this->assertInstanceOf(User::class, $user);
        $this->assertEquals("Mario", $user->name);
        $this->assertDatabaseHas("users", ["email" => "mario@example.com"]);
    }

    public function test_create_user_duplicate_email_throws_exception(): void
    {
        $this->expectException(\RuntimeException::class);

        User::factory()->create(["email" => "mario@example.com"]);
        $this->service->create([
            "name"     => "Mario",
            "email"    => "mario@example.com",
            "password" => "securePass123",
        ]);
    }
}
```

### Feature test (HTTP)

```php
<?php
// tests/Feature/Http/Controllers/UserControllerTest.php

namespace Tests\Feature\Http\Controllers;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class UserControllerTest extends TestCase
{
    use RefreshDatabase;

    private string $baseUrl = "/api/v1/users";

    public function test_can_list_users(): void
    {
        User::factory()->count(3)->create();

        $response = $this->getJson($this->baseUrl);

        $response->assertStatus(200)
                 ->assertJsonCount(3, "data")
                 ->assertJsonStructure([
                     "data"  => [["id", "name", "email"]],
                     "meta"  => ["page", "total"],
                 ]);
    }

    public function test_can_create_user(): void
    {
        $data = [
            "name"     => "Mario",
            "email"    => "mario@example.com",
            "password" => "securePass123",
            "password_confirmation" => "securePass123",
        ];

        $response = $this->postJson($this->baseUrl, $data);

        $response->assertStatus(201)
                 ->assertJsonPath("data.name", "Mario");

        $this->assertDatabaseHas("users", ["email" => "mario@example.com"]);
    }

    public function test_create_user_validation_fails(): void
    {
        $response = $this->postJson($this->baseUrl, [
            "name" => "",  // name required
        ]);

        $response->assertStatus(422)
                 ->assertJsonValidationErrors(["name", "email", "password"]);
    }

    public function test_can_show_user(): void
    {
        $user = User::factory()->create();

        $response = $this->getJson("{$this->baseUrl}/{$user->id}");

        $response->assertStatus(200)
                 ->assertJsonPath("data.id", $user->id);
    }

    public function test_show_nonexistent_user_returns_404(): void
    {
        $response = $this->getJson("{$this->baseUrl}/99999");

        $response->assertStatus(404);
    }

    public function test_can_delete_user(): void
    {
        $user = User::factory()->create();

        $response = $this->deleteJson("{$this->baseUrl}/{$user->id}");

        $response->assertStatus(204);
        $this->assertDatabaseMissing("users", ["id" => $user->id]);
    }
}
```

### Model factory

```php
<?php
// database/factories/UserFactory.php

namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class UserFactory extends Factory
{
    protected $model = User::class;

    public function definition(): array
    {
        return [
            "name"              => fake()->name(),
            "email"             => fake()->unique()->safeEmail(),
            "email_verified_at" => now(),
            "password"          => bcrypt("password"),
            "remember_token"    => Str::random(10),
        ];
    }

    public function unverified(): static
    {
        return $this->state(fn(array $attrs) => [
            "email_verified_at" => null,
        ]);
    }

    public function admin(): static
    {
        return $this->state(fn(array $attrs) => [
            "role_id" => 1,
        ]);
    }
}
```

```php
<?php
// Uso nei test
User::factory()->count(10)->create();
User::factory()->unverified()->create();
User::factory()->admin()->create();
```

## Test con mock

```php
<?php

use App\Services\MailService;
use App\Services\UserService;

public function test_create_user_sends_welcome_email(): void
{
    $mailMock = $this->createMock(MailService::class);
    $mailMock->expects($this->once())
             ->method("sendWelcomeEmail");

    $service = new UserService(
        new UserRepository(new User()),
        $mailMock,
        $this->createMock(AuditService::class),
    );

    $service->create([
        "name"     => "Test",
        "email"    => "test@example.com",
        "password" => "password123",
    ]);
}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `PDOException: SQLSTATE[HY000] [2002] Connection refused` | Test su DB remoto senza env testing | Usa SQLite `:memory:` in phpunit.xml |
| `Test does not refresh database` | RefreshDatabase trait mancante | Aggiungi `use RefreshDatabase;` |
| `Route [login] not defined in test` | Auth middleware senza route login | Definisci route login o usa `$this->actingAs($user)` |
| `Too few arguments to function` | Parametri mancanti in comando artisan | Verifica `$this->argument("name")` matcha `{name}` nella signature |
| `Factory callazione fuori dai test` | Factory chiamata in seeder per errore | Usa solo nei test o nei DatabaseSeeder |
| `Test lento per mancanza di mock` | Service reale che fa IO | Usa mock per mail, API, queue nei test unit |
| `View [mail] not found` | Test mail senza view | `Mail::fake()` evita rendering view |

## Best practice

- **Test con ``RefreshDatabase``** — ogni test parte con DB pulito
- **SQLite `:memory:` per test** — velocissimo, nessuna dipendenza
- **Factory per dati di test** — `User::factory()->create()` più leggibile di array manuali
- **Feature test per HTTP** — testa API endpoint con `getJson()`, `postJson()`
- **Unit test per servizi** — mail, queue, cache mockate; logica di business testata isolatamente
- **`Mail::fake()`, `Queue::fake()`, `Event::fake()`** — evita side-effect reali nei test
- **Comando custom testabile** — inietta servizi nel costruttore (invece di `app()` dentro handle)
- **`php artisan test --parallel`** per CI — dimezza tempo d'esecuzione
- **Nomina test come** `test_can_create_user` o `test_create_user_duplicate_email_fails`
- **Coverage minimo 80%** — `--coverage --min=80` in CI

## Cross-reference

- [[PHP/Laravel/Services e Repository|Laravel — Services e Repository]] — servizi testabili per injection
- [[PHP/Strumenti/Linting e Testing|Linting e Testing]] — PHPUnit, PHPStan, PHPCS
- [[PHP/Laravel/Routing e Controller|Laravel — Routing e Controller]] — endpoint testati
- [[PHP/Core Concepts/Namespace e Autoload|Namespace e Autoload]] — PSR-4 mapping per test
