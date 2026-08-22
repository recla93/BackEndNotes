---
topic: "Composer e Autoload — PHP"
tags: [php, composer, autoload, dependencies, psr-4, packagist]
nav_prev: "[[Twig e Security.md]]"
nav_next: "[[Linting e Testing.md]]"
---

Riferimento ufficiale: [getcomposer.org](https://getcomposer.org/) | [packagist.org](https://packagist.org/) | [PSR-4](https://www.php-fig.org/psr/psr-4/)

Composer (2012, Nils Adermann / Jordi Boggiano) è il package manager de facto per PHP. Gestisce dipendenze, autoloading PSR-4, e script. L'equivalente di npm per Node.js o Maven per Java.

```bash
# Installazione globale
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
mv composer.phar /usr/local/bin/composer

# Su Windows (con PHP in PATH)
# Scarica composer-setup.exe da getcomposer.org
```

## Comandi essenziali

```bash
composer init                          # crea composer.json interattivo
composer require vendor/package        # aggiunge dipendenza + installa
composer require vendor/package:^2.0   # con versione specifica
composer require --dev vendor/package  # dipendenza di sviluppo (PHPUnit, PHPStan)
composer install                       # installa da composer.lock
composer update                        # aggiorna dipendenze (rigenera lock)
composer update vendor/package         # aggiorna solo un package
composer remove vendor/package         # rimuove dipendenza
composer dump-autoload                 # rigenera autoloader (dopo nuove classi)
```

## composer.json — struttura

```json
{
    "name": "myapp/my-project",
    "description": "Applicazione di esempio",
    "type": "project",
    "require": {
        "php": ">=8.1",
        "laravel/framework": "^10.0",
        "vlucas/phpdotenv": "^5.5"
    },
    "require-dev": {
        "phpunit/phpunit": "^10.0",
        "phpstan/phpstan": "^1.10"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    },
    "scripts": {
        "test": "phpunit",
        "stan": "phpstan analyse",
        "check": ["@test", "@stan"]
    },
    "config": {
        "optimize-autoloader": true,
        "sort-packages": true
    }
}
```

### Autoloading PSR-4

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

Mappa namespace `App\` al directory `src/`. Quindi:
- `App\Controllers\UserController` → `src/Controllers/UserController.php`
- `App\Services\UserService` → `src/Services/UserService.php`
- `App\Exceptions\ValidationException` → `src/Exceptions/ValidationException.php`

## Versionamento (SemVer)

```
^1.2.3  → >=1.2.3, <2.0.0  (compatibile major)
~1.2.3  → >=1.2.3, <1.3.0  (compatibile minor)
1.2.*   → >=1.2.0, <1.3.0  (wildcard minor)
dev-main → branch specifico (non raccomandato per produzione)
```

## Composer.lock

```bash
# Crea lock file con versioni esatte
composer install

# Aggiorna versioni consentite da composer.json
composer update
```

`composer.lock` va **committato** nel VCS. Garantisce che tutti gli ambienti usino le stesse versioni esatte. `composer install` legge il lock; `composer update` lo rigenera.

## Autoloader generato

```php
<?php

// Unico require necessario in tutta l'app
require_once __DIR__ . "/../vendor/autoload.php";

use App\Controllers\UserController;
use App\Services\UserService;

$controller = new UserController(new UserService());
```

## Script Composer

```json
{
    "scripts": {
        "pre-update-cmd": "echo 'Aggiornamento in corso...'",
        "post-install-cmd": [
            "php artisan cache:clear",
            "php artisan config:cache"
        ],
        "lint": "phpcs --standard=PSR12 src/",
        "fix": "phpcbf --standard=PSR12 src/"
    }
}
```

```bash
composer run-script lint
composer run-script fix
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Class "App\Controller\UserController" not found` | Autoloader non rigenerato | `composer dump-autoload` |
| `Could not find package` | Nome package errato o non su Packagist | Verifica su packagist.org |
| `Your requirements could not be resolved` | Conflitto versioni | `composer why vendor/package` per capire chi lo richiede |
| `Allowed memory exhausted` | PHP memory limit troppo basso per composer | `php -d memory_limit=-1 composer.phar install` |
| `The openssl extension is required` | PHP senza OpenSSL | Abilita extension=openssl in php.ini |
| `Proc-open is not available` | PHP senza proc_open (spesso su hosting) | Verifica disabled_functions in php.ini |
| `composer install non trova il lock` | Lock non committato o cancellato | `composer update` per generarlo (ma occhio alle versioni) |

## Best practice

- **Commatta composer.lock** — riproducibilità tra ambienti
- **`require` per produzione, `require-dev` per sviluppo** — PHPUnit, PHPStan non servono in prod
- **`composer install --no-dev` in produzione** — esclude dev-dependency
- **`composer dump-autoload -o`** — genera autoloader ottimizzato con class-map
- **`sort-packages: true`** — require alphabetically sorted (evita merge conflict)
- **Versione PHP in require** — `"php": ">=8.1"` previene installazione su versioni incompatibili
- **Package obsoleto: usa `composer why`** — scopri chi richiede un package prima di rimuoverlo
- **Evita `dev-master` e `@dev`** — instabili, possono rompere senza preavviso
- **`composer audit` in CI** — controlla vulnerabilità note nei package (PHP 8.1+)
- **Non modificare mai vendor/** — le modifiche vengono perse a ogni install; fork del package se serve

## Cross-reference

- [[PHP/Strumenti/Linting e Testing|Linting e Testing]] — PHPUnit, PHPStan come dev-dependency
- [[PHP/Core Concepts/Namespace e Autoload|Namespace e Autoload]] — PSR-4, namespace
- [[PHP/Laravel/Setup e Architettura|Laravel — Setup]] — Laravel installer via Composer
- [[PHP/Symfony/Setup e Architettura|Symfony — Setup]] — Symfony CLI via Composer
