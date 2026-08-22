---
topic: "Namespace e Autoload — PHP"
tags: [php, namespace, autoload, psr-4, composer, use]
nav_prev: "[[Classi e OOP.md]]"
nav_next: "[[Gestione Errori.md]]"
---
Riferimento ufficiale: [php.net/manual/en/language.namespaces.php](https://www.php.net/manual/en/language.namespaces.php) | [php.net/manual/en/language.oop5.autoload.php](https://www.php.net/manual/en/language.oop5.autoload.php) | [www.php-fig.org/psr/psr-4/](https://www.php-fig.org/psr/psr-4/)

I namespace (PHP 5.3+) organizzano il codice in gerarchie logiche, evitando collisioni tra classi con lo stesso nome (es. `App\Models\User` vs `MyPackage\Models\User`). L'autoloading (PSR-4) carica automaticamente le classi senza `require` espliciti.

```php
<?php

// src/Models/User.php
namespace App\Models;

class User {
    public function __construct(
        private string $nome,
    ) {}
}
```

## namespace e use

```php
<?php

// src/Controllers/UserController.php
namespace App\Controllers;

use App\Models\User;           // importa classe dal namespace
use App\Services\MailService as MailSvc;  // alias
use App\Exceptions\NotFoundException;
use function App\Helpers\formatDate;      // importa funzione (PHP 5.6+)
use const App\Config\MAX_RETRY;           // importa costante

class UserController {
    public function show(int $id): User {
        $user = new User($id);
        $mail = new MailSvc();
        return $user;
    }
}
```

## PSR-4 Autoloading con Composer

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "Database\\Seeders\\": "database/seeders/"
        }
    },
    "autoload-dev": {
        "App\\Tests\\": "tests/"
    }
}
```

La mappa PSR-4 dice: il namespace `App\Models\User` si trova in `src/Models/User.php`. Il namespace radice `App\\` corrisponde alla cartella `src/`. Questo mapping è case-sensitive su Linux.

```bash
composer dump-autoload    # rigenera autoloader dopo modifiche
composer dump-autoload -o # genera classmap ottimizzata (più veloce in produzione)
```

## Autoloader generato

```php
<?php

// public/index.php — entry point
require __DIR__ . '/../vendor/autoload.php';

use App\Controllers\UserController;

$controller = new UserController();
```

Una sola riga carica tutte le classi del progetto e delle dipendenze. `vendor/autoload.php` include il caricatore generato da Composer.

## PSR-4 struttura cartelle

```
src/
├── Controllers/
│   └── UserController.php        # namespace App\Controllers\UserController
├── Models/
│   ├── User.php                  # namespace App\Models\User
│   └── Order.php                 # namespace App\Models\Order
├── Services/
│   └── AuthService.php           # namespace App\Services\AuthService
├── Exceptions/
│   └── NotFoundException.php     # namespace App\Exceptions\NotFoundException
└── Helpers/
    └── functions.php             # namespace App\Helpers (funzioni)
```

## Namespace multipli e globale

```php
<?php

// File con più namespace (sconsigliato, meglio un file per classe)
namespace App\Models {
    class User {}
}

namespace App\Services {
    class Logger {}
}

// Namespace globale (backslash)
$user = new \DateTime();       // classe built-in, namespace globale
$user = new DateTime();         // se nessun use, PHP cerca nel namespace corrente
$user = new \App\Models\User(); // full qualified name — sempre sicuro
```

## SplClassLoader manuale (senza Composer)

```php
<?php

// Autoloader minimale PSR-4 (per progetti senza Composer)
spl_autoload_register(function (string $class): void {
    $prefix = 'App\\';
    $baseDir = __DIR__ . '/src/';

    if (strncmp($prefix, $class, strlen($prefix)) === 0) {
        $relativeClass = substr($class, strlen($prefix));
        $file = $baseDir . str_replace('\\', '/', $relativeClass) . '.php';

        if (file_exists($file)) {
            require $file;
        }
    }
});
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Class "App\Models\User" not found` | Autoload non rigenerato o path sbagliato | `composer dump-autoload` (o -o per produzione) |
| `Cannot use App\Models\User as User because the name is already in use` | Due classi importate con stesso alias | Usa alias diverso: `use ... as ...` |
| `Namespace declaration must be the first statement` | Whitespace o HTML prima di `namespace` | Nessun output prima di `namespace` (neanche spazi) |
| `Call to undefined function App\Helpers\formatDate()` | Funzione non importata con `use function` | `use function App\Helpers\formatDate;` |
| `The use statement with non-compound name 'User' has no effect` | `use User` in namespace globale per classe globale | Non serve `use` per classi globali (`DateTime`, `Exception`) |

## Best practice

- **PSR-4 strict** — namespace = percorso cartella (case-sensitive su Linux, ma mantieni consistenza)
- **Un namespace per file** — mai più classi o namespace in un file
- **Composer autoload > autoload manuale** — `require __DIR__.'/vendor/autoload.php'` basta
- **`composer dump-autoload -o` in produzione** — genera classmap ottimizzato, più veloce
- **Mai `require` o `include` manuali** — se devi fare `require`, probabilmente manca un autoloading
- **Namespace radice descrittivo** — `App\`, `Vendor\Package\`, `Database\` (non generico)
- **`use` ordinati** — classi prima, poi function, poi const; raggruppa per vendor

## Cross-reference

- [[PHP/Core Concepts/Classi e OOP|Classi e OOP]] — classi, interfacce, namespace
- [[PHP/Strumenti/Composer e Autoload|Composer]] — autoloading PSR-4, installazione pacchetti
- [[PHP/Laravel/Setup e Architettura|Laravel — Setup]] — namespace Laravel, autoload delle app
- [[PHP/Symfony/Setup e Architettura|Symfony — Setup]] — namespace Symfony, autoload bundles
