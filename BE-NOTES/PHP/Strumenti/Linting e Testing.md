---
topic: "Linting e Testing — PHP"
tags: [php, testing, phpunit, pest, phpstan, phpcs, quality]
nav_prev: "[[Composer e Autoload.md]]"
---

Riferimento ufficiale: [phpunit.de](https://phpunit.de/) | [phpstan.org](https://phpstan.org/) | [github.com/PHPCSStandards/PHP_CodeSniffer](https://github.com/PHPCSStandards/PHP_CodeSniffer)

PHP ha un ecosistema maturo per qualità del codice: **static analysis** per errori a compile-time, **coding standard** per stile coerente, e **testing** per correttezza.

## PHP_CodeSniffer (syntax lint + coding standard)

```bash
composer require --dev squizlabs/php_codesniffer

# Verifica
vendor/bin/phpcs --standard=PSR12 src/

# Auto-fix
vendor/bin/phpcbf --standard=PSR12 src/

# Con report specifico
vendor/bin/phpcs --standard=PSR12 --report=full src/
vendor/bin/phpcs --standard=PSR12 --report=summary src/
```

```xml
<!-- phpcs.xml -->
<?xml version="1.0"?>
<ruleset name="MyApp">
    <description>Standard di codifica</description>

    <file>src/</file>
    <file>tests/</file>
    <exclude-pattern>*/vendor/*</exclude-pattern>

    <rule ref="PSR12"/>

    <!-- Regole custom -->
    <rule ref="Generic.Files.LineLength">
        <properties>
            <property name="lineLimit" value="120"/>
        </properties>
    </rule>
</ruleset>
```

## PHPStan (static analysis)

```bash
composer require --dev phpstan/phpstan

# Analisi base
vendor/bin/phpstan analyse src/

# Con livello massimo
vendor/bin/phpstan analyse --level max src/
```

```neon
# phpstan.neon
parameters:
    level: 6                          # 0-9, max = highest
    paths:
        - src/
        - tests/
    checkMissingIterableValueType: true
    checkGenericClassInNonGenericObjectType: true
    treatPhpDocTypesAsCertain: false
    ignoreErrors:
        - '#Method App\\.*::.* has no return type specified#'
    excludes_analyse:
        - src/Migrations/
```

### Livelli PHPStan

| Livello | Cosa controlla |
|---|---|
| 0 | Classi/funzioni sconosciute, parametri errati |
| 1 | Tipo `mixed` su parametri/return mancanti |
| 2 | Metodi sconosciuti su `$this` |
| 3 | Return type mancanti su metodi |
| 4 | Valori `mixed` passati dove ci si aspetta un tipo |
| 5 | Parametri di funzione/metodo non tipizzati |
| 6 | Callable non verificabili |
| 7 | Union type non usati correttamente |
| 8 | Valori `mixed` in return type |
| 9 | Strict types non dichiarato |

## PHPUnit (testing)

```bash
composer require --dev phpunit/phpunit

# Esecuzione
vendor/bin/phpunit
vendor/bin/phpunit --filter=UserTest
vendor/bin/phpunit tests/Feature/UserTest.php
vendor/bin/phpunit --coverage-html coverage/
```

```xml
<!-- phpunit.xml -->
<?xml version="1.0"?>
<phpunit
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
    bootstrap="vendor/autoload.php"
    colors="true"
>
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
    </testsuites>

    <source>
        <include>
            <directory>src</directory>
        </include>
    </source>

    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="DB_CONNECTION" value="sqlite"/>
        <env name="DB_DATABASE" value=":memory:"/>
    </php>
</phpunit>
```

### Test di base

```php
<?php
// tests/Unit/UserServiceTest.php

namespace Tests\Unit;

use App\Services\UserService;
use App\Repositories\UserRepository;
use PHPUnit\Framework\TestCase;

class UserServiceTest extends TestCase
{
    private UserService $service;
    private UserRepository $repository;

    protected function setUp(): void
    {
        $this->repository = $this->createMock(UserRepository::class);
        $this->service    = new UserService($this->repository);
    }

    public function testCreateUserSuccessfully(): void
    {
        $data = [
            "name"     => "Mario",
            "email"    => "mario@example.com",
            "password" => "securePass123",
        ];

        $this->repository
            ->expects($this->once())
            ->method("findByEmail")
            ->with("mario@example.com")
            ->willReturn(null);

        $this->repository
            ->expects($this->once())
            ->method("create")
            ->willReturn(["id" => 1, ...$data]);

        $result = $this->service->create($data);

        $this->assertArrayHasKey("id", $result);
        $this->assertEquals("Mario", $result["name"]);
    }

    public function testCreateUserDuplicateEmailThrowsException(): void
    {
        $this->expectException(\RuntimeException::class);
        $this->expectExceptionMessage("Email already exists");

        $this->repository
            ->method("findByEmail")
            ->willReturn(["id" => 1, "email" => "mario@example.com"]);

        $this->service->create([
            "name"     => "Mario",
            "email"    => "mario@example.com",
            "password" => "securePass123",
        ]);
    }
}
```

### Data Provider

```php
<?php

class MathTest extends TestCase
{
    /** @dataProvider additionProvider */
    public function testAdd(int $a, int $b, int $expected): void
    {
        $this->assertSame($expected, $a + $b);
    }

    public static function additionProvider(): array
    {
        return [
            "zero"     => [0, 0, 0],
            "positive" => [1, 2, 3],
            "negative" => [-1, -1, -2],
        ];
    }
}
```

### Test di integrazione con PDO

```php
<?php

class UserRepositoryTest extends TestCase
{
    private static \PDO $pdo;
    private UserRepository $repository;

    public static function setUpBeforeClass(): void
    {
        self::$pdo = new \PDO("sqlite::memory:");
        self::$pdo->exec("CREATE TABLE users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT NOT NULL UNIQUE,
            password TEXT NOT NULL
        )");
    }

    protected function setUp(): void
    {
        self::$pdo->exec("DELETE FROM users");
        $this->repository = new UserRepository(self::$pdo);
    }

    public function testCreateAndFindUser(): void
    {
        $user = $this->repository->create([
            "name"     => "Mario",
            "email"    => "mario@example.com",
            "password" => "hash123",
        ]);

        $found = $this->repository->findById($user["id"]);
        $this->assertNotNull($found);
        $this->assertEquals("mario@example.com", $found["email"]);
    }
}
```

## Pest (alternativa moderna a PHPUnit)

```php
<?php
// tests/Unit/UserServiceTest.php

use function Pest\Faker\fake;

it("creates a user successfully", function () {
    $repository = mock(UserRepository::class);
    $repository->shouldReceive("findByEmail")->andReturn(null);
    $repository->shouldReceive("create")->andReturn(["id" => 1, "name" => "Mario"]);

    $service = new UserService($repository);
    $result  = $service->create(["name" => "Mario", "email" => "mario@test.com", "password" => "12345678"]);

    expect($result)->toHaveKey("id");
    expect($result["name"])->toBe("Mario");
});
```

```bash
composer require --dev pestphp/pest
vendor/bin/pest
```

## CI integration

```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: "8.3"
          tools: composer

      - name: Install dependencies
        run: composer install --prefer-dist --no-progress

      - name: PHPCS
        run: vendor/bin/phpcs --standard=PSR12 src/

      - name: PHPStan
        run: vendor/bin/phpstan analyse --level 6 src/

      - name: PHPUnit
        run: vendor/bin/phpunit
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `PHPUnit: Class ... not found` | Autoloader non aggiornato | `composer dump-autoload` |
| `PHPStan: Function app not found` | Helper functions non caricate | Aggiungi `files: []` in autoload composer |
| `PHP_CodeSniffer: ERROR: the "PSR12" coding standard is not recognized` | Standard ruleset non trovato | Usa `vendor/bin/phpcs --standard=PSR12` con vendor presente |
| `Memory exhausted running PHPStan` | Progetto grande e livello alto | Usa `php -d memory_limit=512M vendor/bin/phpstan` |
| `Test double non verifica chiamata` | `expects($this->once())` ma metodo non chiamato | Verifica logica del test o rimuovi expects |
| `Risultati falsi positivi PHPStan` | `@phpdoc` incoerente con tipo reale | Aggiorna type hint o usa `assert()` per narrowing |
| `phpunit.xml not found` | Esegui da directory sbagliata | Lancia da root progetto; specifica `--configuration` |

## Best practice

- **PHPStan livello 6 come minimo** — cattura la maggior parte dei bug senza essere eccessivo
- **PHPCS con PSR-12** — standard PHP-FIG; auto-fix con phpcbf
- **Test per ogni service/repository** — unit test con mock per service, integration test per repository
- **Data provider per casi multipli** — evita duplicazione di test simili
- **In-memory SQLite per test DB** — veloce, isolato, nessuna dipendenza esterna
- **`@covers` annotation** — documenta quale classe è testata (utile per coverage)
- **CI con tutti e 3 gli strumenti** — lint (PHPCS) → static analysis (PHPStan) → test (PHPUnit)
- **Pest per progetti nuovi** — sintassi più espressiva di PHPUnit
- **Test nomi descrittivi** — `testCreateUserWithInvalidEmailThrowsException()` non `testUser1()`
- **Coverage minimo 80%** — misura con `--coverage-html` e blocca sotto soglia

## Cross-reference

- [[PHP/Strumenti/Composer e Autoload|Composer e Autoload]] — installazione strumenti come dev-dependency
- [[PHP/Laravel/Artisan e Testing|Laravel — Testing]] — PHPUnit in contesto Laravel
- [[PHP/Core Concepts/Gestione Errori|Gestione Errori]] — exception handling nei test
- [[PHP/Database/PDO|PDO]] — test di integrazione con SQLite in-memory
