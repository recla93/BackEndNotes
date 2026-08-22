---
topic: "PHP — panoramica e indice"
tags: [php, backend, web, laravel, symfony, scripting]
---

PHP (1994, Rasmus Lerdorf) è un linguaggio **interpretato**, **tipizzazione dinamica debole**, nato per il web. Originariamente "Personal Home Page", oggi è "PHP: Hypertext Preprocessor". Esegue lato server e genera HTML, JSON, o qualsiasi output testuale.

Rispetto ad altri linguaggi backend:
- **Embedded nel template** — codice PHP si mischia con HTML (originariamente, ma i template engine moderni separano)
- **Request-response life cycle** — ogni richiesta parte "pulita" (nessuna condivisione di stato tra richieste, a differenza di JVM/Node.js)
- **Tipizzazione dinamica debole** — confronti con `==` fanno coercizione (come JS); `===` per confronto stretto
- **Zend Engine** — interprete compilato (PHP 7+ ha JIT, PHP 8.0+ con JIT compilation)
- **Ecosistema web nativo** — superglobali (`$_GET`, `$_POST`, `$_SESSION`), integrazione HTTP built-in

## Aree

- [[PHP/Core Concepts/Variabili e Tipi|Variabili e Tipi]] — dichiarazione, tipi, coercizione, type hinting (PHP 7+)
- [[PHP/Core Concepts/Stringhe|Stringhe]] — escaping, interpolazione, heredoc, nowdoc, encoding
- [[PHP/Core Concepts/Array|Array]] — array associativi, funzioni built-in, array_map/filter/reduce
- [[PHP/Core Concepts/Funzioni|Funzioni]] — dichiarazione, parametri tipizzati, closure, arrow function (PHP 7.4+)
- [[PHP/Core Concepts/Controllo e Loop|Controllo e Loop]] — if/else, match (PHP 8+), for/foreach/while
- [[PHP/Core Concepts/Classi e OOP|Classi e OOP]] — classi, ereditarietà, interfacce, trait, property promotion (PHP 8+)
- [[PHP/Core Concepts/Namespace e Autoload|Namespace e Autoload]] — namespace, use, PSR-4, Composer autoload
- [[PHP/Core Concepts/Gestione Errori|Gestione Errori]] — try/catch, Error vs Exception, error handler custom

## Web (HTTP nativo)

- [[PHP/Web/Superglobali|Superglobali]] — $_GET, $_POST, $_SERVER, $_SESSION, $_FILES, $_COOKIE
- [[PHP/Web/Sessioni e Cookie|Sessioni e Cookie]] — session handling, cookie security, flash messages
- [[PHP/Web/REST API|REST API]] — routing senza framework, Content-Type negotiation, CORS

## Database

- [[PHP/Database/PDO|PDO]] — connessione, prepared statement, CRUD, transazioni, fetch modes

## Framework

- [[PHP/Laravel/Setup e Architettura|Laravel — Setup e Architettura]] — struttura progetto, Service Container, facades
- [[PHP/Laravel/Routing e Controller|Laravel — Routing e Controller]] — Route, Controller, dependency injection, middleware
- [[PHP/Laravel/Eloquent e Database|Laravel — Eloquent e Database]] — ORM, relazioni, eager loading, mutator/accessor, scope
- [[PHP/Laravel/Blade e Template|Laravel — Blade e Template]] — template engine, component, slot, section
- [[PHP/Laravel/Services e Repository|Laravel — Services e Repository]] — service layer, repository pattern, action classes
- [[PHP/Laravel/Artisan e Testing|Laravel — Artisan e Testing]] — CLI, custom command, PHPUnit, TDD
- [[PHP/Laravel/Auth e Sicurezza|Laravel — Auth e Sicurezza]] — authentication, authorization (gates/policies), sanctum, rate limiting

- [[PHP/Symfony/Setup e Architettura|Symfony — Setup e Architettura]] — struttura, bundle, service container, kernel
- [[PHP/Symfony/Routing e Controller|Symfony — Routing e Controller]] — annotazioni/attribute routing, controller, DI
- [[PHP/Symfony/Doctrine e Database|Symfony — Doctrine e Database]] — ORM, entity, repository, DQL, migrazioni
- [[PHP/Symfony/Twig e Security|Symfony — Twig e Security]] — template engine, autenticazione, voter, firewall

## Strumenti

- [[PHP/Strumenti/Composer e Autoload|Composer e Autoload]] — dipendenze, autoloading PSR-4, versioning, packagist
- [[PHP/Strumenti/Linting e Testing|Linting e Testing]] — PHP_CodeSniffer, PHPStan, PHPUnit, Pest

## Convenzioni generali

### Versioni PHP consigliate
- **PHP 8.3+** per nuovi progetti — performance, named argument, readonly class, fiber
- **PHP 8.1+** minimo per progetti moderni — enum, readonly property, first-class callable
- **PHP 7.4** legacy — typed property, arrow function, spread operator

### Naming conventions (PSR-12 / PHP-FIG)
- **classi, interfacce, trait**: `PascalCase` (`UserController`, `Authenticatable`)
- **metodi, funzioni, variabili**: `camelCase` (`getUser()`, `$userName`)
- **costanti globali**: `UPPER_SNAKE_CASE` (`MAX_RETRY_COUNT`)
- **namespace**: `PascalCase`, strutturato come `Vendor\Package\SubNamespace` (PSR-4)

### PSR standards principali
| Standard | Cosa definisce |
|---|---|
| **PSR-1** | Basic coding standard (tag PHP, side effect, naming) |
| **PSR-4** | Autoloading (namespace → directory mapping) |
| **PSR-7** | HTTP message interfaces (Request, Response) |
| **PSR-12** | Extended coding style (derive da PSR-2) |
| **PSR-14** | Event dispatcher |
| **PSR-15** | HTTP handlers / middleware |

## Cross-reference

- [[PHP/Core Concepts/Variabili e Tipi|Variabili e Tipi]] — fondamenti del linguaggio
- [[PHP/Laravel/Setup e Architettura|Laravel]] — framework backend dominante
- [[PHP/Symfony/Setup e Architettura|Symfony]] — framework enterprise
- [[PHP/Strumenti/Composer e Autoload|Composer]] — gestione dipendenze
