---
topic: "Symfony — Setup e Architettura"
tags: [symfony, setup, architecture, bundle, service-container, kernel]
nav_prev: "[[Auth e Sicurezza.md]]"
nav_next: "[[Routing e Controller.md]]"
---

Riferimento ufficiale: [symfony.com/doc/current](https://symfony.com/doc/current/) | [symfony.com/doc/current/bundles.html](https://symfony.com/doc/current/bundles.html)

Symfony (2005, Fabien Potencier) è il framework PHP enterprise. Basato su **bundle** (moduli riutilizzabili), **Service Container** (DI), e componenti indipendenti (HttpKernel, HttpFoundation, Routing, Form, Security). Segue architettura flessibile: puoi usare componenti Symfony anche senza il framework completo.

```bash
# Installazione con Symfony CLI
symfony new my-app --version=lts       # LTS corrente
symfony new my-app --webapp             # con strumenti per web (profiler, DB, mail)
symfony new my-app --no-interaction     # non interattivo

# Server di sviluppo
symfony serve                           # avvia server locale
symfony serve --no-tls                  # HTTP invece di HTTPS

# Docker
symfony docker:compose                  # genera docker-compose.yml
```

## Struttura progetto

```
my-app/
├── assets/                  # Frontend (Webpack Encore / Vite)
├── bin/
│   └── console              # CLI entry point (php bin/console)
├── config/
│   ├── packages/            # Config per environment
│   │   ├── framework.yaml   # Config framework core
│   │   ├── doctrine.yaml    # Doctrine ORM
│   │   ├── security.yaml    # Security / firewall
│   │   └── twig.yaml        # Twig template
│   ├── routes/              # Route definition
│   │   ├── attributes.yaml  # Route con attributi PHP 8
│   │   └── api.yaml         # Route API
│   ├── services.yaml        # DI container config
│   └── bundles.php          # Registrazione bundle
├── migrations/              # Doctrine migration
├── public/
│   └── index.php            # Entry point (front controller)
├── src/
│   ├── Controller/          # Controller HTTP
│   ├── Entity/              # Doctrine entity
│   ├── Repository/          # Doctrine repository
│   ├── Service/             # Business logic
│   ├── Form/                # Form type
│   ├── EventSubscriber/     # Event listener/subscriber
│   ├── Security/            # Voter, User provider, Authenticator
│   ├── DTO/                 # Data Transfer Object
│   └── Kernel.php           # App kernel
├── templates/               # Twig template
├── tests/
├── translations/            # File di traduzione
├── var/                     # Cache, log (non in VCS)
└── .env / .env.local        # Config per ambiente
```

## Service Container (autowiring)

```php
<?php
// config/services.yaml

services:
    _defaults:
        autowire: true      # iniezione automatica
        autoconfigure: true # registra automaticamente tag (event, form, security)

    App\:
        resource: '../src/'
        exclude:
            - '../src/DependencyInjection/'
            - '../src/Entity/'
            - '../src/Kernel.php'
```

```php
<?php
// src/Service/UserService.php
// Autowiring — nessuna registrazione manuale

namespace App\Service;

use App\Repository\UserRepository;
use Psr\Log\LoggerInterface;

class UserService
{
    public function __construct(
        private readonly UserRepository $repository,
        private readonly LoggerInterface $logger,  // inject automatico per type-hint su interfaccia
        private readonly string $uploadDir,         // parametro scalare (richiede binding)
    ) {}
}
```

### Binding parametri scalari

```yaml
services:
    App\Service\UserService:
        arguments:
            $uploadDir: '%kernel.project_dir%/public/uploads'
```

### Service tags

```php
<?php
// config/services.yaml

App\EventSubscriber\ExceptionSubscriber:
    tags:
        - { name: kernel.event_listener, event: kernel.exception }

// Con autoconfigure, i tag molti sono automatici:
// - implenta EventSubscriberInterface → tag kernel.event_subscriber
// - implenta FormTypeInterface → tag form.type
// - implenta VoterInterface → tag security.voter
```

## Kernel e Bundle

```php
<?php
// config/bundles.php

return [
    Symfony\Bundle\FrameworkBundle\FrameworkBundle::class => ["all" => true],
    Symfony\Bundle\SecurityBundle\SecurityBundle::class   => ["all" => true],
    Symfony\Bundle\TwigBundle\TwigBundle::class           => ["all" => true],
    Doctrine\Bundle\DoctrineBundle\DoctrineBundle::class  => ["all" => true],
    Doctrine\Bundle\MigrationsBundle\DoctrineMigrationsBundle::class => ["all" => true],
    Symfony\Bundle\MakerBundle\MakerBundle::class         => ["dev" => true],
    Symfony\Bundle\DebugBundle\DebugBundle::class         => ["dev" => true],
    Symfony\Bundle\WebProfilerBundle\WebProfilerBundle::class => ["dev" => true],
];
```

## Comandi CLI

```bash
php bin/console                          # lista comandi
php bin/console debug:router             # mostra tutte le route
php bin/console debug:container          # mostra servizi registrati
php bin/console debug:autowiring         # mostra type-hint disponibili
php bin/console make:controller          # genera controller
php bin/console make:entity              # genera entity + repository
php bin/console make:command             # genera comando custom
php bin/console make:voter               # genera security voter
php bin/console make:form                # genera form type
php bin/console doctrine:migrations:migrate
php bin/console cache:clear
```

## Ciclo di vita di una richiesta

```
Request → index.php → Kernel::handle() → RouterListener → ControllerResolver → Controller → Service → Response
```

Il **Kernel** orchestri: carica bundle, configura container, gestisce il ciclo di vita (boot → handle → terminate). Gli **EventSubscriber** si agganciano agli eventi del kernel (`kernel.request`, `kernel.controller`, `kernel.response`, `kernel.exception`).

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `The autoloader expected class "App\Kernel"` | Namespace sbagliato in Kernel | Verifica namespace in `src/Kernel.php` |
| `Cannot autowire service "App\Service\UserService": argument "$uploadDir"` | Parametro scalare non configurato | Aggiungi `arguments: { $uploadDir: '...' }` in services.yaml |
| `Class "App\Entity\User" not found` | Entity senza attributi Doctrine | Verifica `#[ORM\Entity]` e `#[ORM\Column]` |
| `The "app" environment is not defined` | `.env` mancante | Copia `.env` o crea `.env.local` |
| `You have requested a non-existent service "doctrine"` | Doctrine bundle non registrato | Aggiungi in `config/bundles.php` |
| `The controller must return a Response` | Controller non restituisce Response | `return $this->json($data)` o `return $this->render(...)` |
| `Cannot load resource "../src/Controller/"` | Path sbagliato in services.yaml | Verifica `resource` path in `config/services.yaml` |

## Best practice

- **Autowiring di default** — nessuna registrazione manuale per servizi standard
- **Bundle per feature cross-cutting** — logging, mailing, caching come bundle dedicati
- **Entity mai in Controller** — usa Service/Repository per accesso dati
- **DTO per input/output** — mai esporre Entity direttamente in API
- **Maker bundle per scaffolding** — `php bin/console make:entity`, `make:controller`
- **`.env.local` per credenziali** — mai committare; `.env` per default sicuri
- **Cache clear in deployment** — `php bin/console cache:clear --env=prod`
- **Profiler solo in dev** — `Symfony\Bundle\WebProfilerBundle::class => ["dev" => true]`
- **Base su LTS** — usa versione LTS per progetti stabili

## Cross-reference

- [[PHP/Symfony/Routing e Controller|Symfony — Routing e Controller]] — attributi route, controller, DI
- [[PHP/Symfony/Doctrine e Database|Symfony — Doctrine e Database]] — ORM, DQL, repository
- [[PHP/Symfony/Twig e Security|Symfony — Twig e Security]] — template e autenticazione
- [[PHP/Core Concepts/Namespace e Autoload|Namespace e Autoload]] — PSR-4, autoload
