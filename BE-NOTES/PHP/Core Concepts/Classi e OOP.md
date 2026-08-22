---
topic: "Classi e OOP — PHP"
tags: [php, oop, classes, inheritance, interfaces, traits, php8]
nav_prev: "[[Controllo e Loop.md]]"
nav_next: "[[Namespace e Autoload.md]]"
---
Riferimento ufficiale: [php.net/manual/en/language.oop5.php](https://www.php.net/manual/en/language.oop5.php) | [php.net/manual/en/language.oop5.traits.php](https://www.php.net/manual/en/language.oop5.traits.php)

PHP ha OOP classica (classi, ereditarietà, interfacce) più **trait** (riutilizzo orizzontale) e **enum** (PHP 8.1+). Da PHP 8.0+ ha **property promotion** (costruttore + proprietà in una riga), **readonly property** (PHP 8.1+), e **new in initializer** (PHP 8.1+).

```php
<?php

declare(strict_types=1);

class Utente {
    // Proprietà tipizzate (PHP 7.4+)
    private int $id;
    private string $nome;
    protected string $email;
    public string $ruolo = "user";  // con default

    public function __construct(int $id, string $nome, string $email) {
        $this->id = $id;
        $this->nome = $nome;
        $this->email = $email;
    }

    public function saluta(): string {
        return "Ciao, sono {$this->nome}";
    }

    public function getId(): int {
        return $this->id;
    }
}
```

## Constructor Property Promotion (PHP 8.0+)

```php
<?php

// Prima (PHP 7) — 10 righe per un costruttore
class Utente {
    private string $nome;
    private string $email;
    public function __construct(string $nome, string $email) {
        $this->nome = $nome;
        $this->email = $email;
    }
}

// PHP 8.0+ — property promotion (dichiara e assegna in una riga)
class Utente {
    public function __construct(
        private string $nome = "",
        private string $email = "",
        private int $eta = 0,
    ) {}
}
```

## Readonly property (PHP 8.1+)

```php
<?php

class Config {
    public function __construct(
        readonly public string $host = "localhost",
        readonly public int $port = 5432,
    ) {}
}

$config = new Config(host: "db.example.com");
// $config->port = 8080;  // Error: Cannot modify readonly property
```

## Ereditarietà e interfacce

```php
<?php

// Interfaccia
interface Authenticatable {
    public function getPassword(): string;
    public function verifyPassword(string $password): bool;
}

// Classe astratta
abstract class BaseModel {
    protected int $id;
    abstract public function validate(): bool;

    protected function getId(): int {
        return $this->id;
    }
}

// Implementazione
class Admin extends BaseModel implements Authenticatable {
    public function __construct(
        private string $nome,
        private string $password,
    ) {}

    public function getPassword(): string {
        return $this->password;
    }

    public function verifyPassword(string $password): bool {
        return password_verify($password, $this->password);
    }

    public function validate(): bool {
        return strlen($this->nome) >= 2;
    }
}
```

## Trait (riutilizzo orizzontale)

PHP non ha ereditarietà multipla. I trait permettono di condividere metodi tra classi non correlate gerarchicamente.

```php
<?php

trait Timestampable {
    private DateTime $createdAt;
    private DateTime $updatedAt;

    public function touch(): void {
        $this->updatedAt = new DateTime();
    }

    public function getCreatedAt(): DateTime {
        return $this->createdAt;
    }
}

class Article {
    use Timestampable;

    public function __construct(
        private string $title,
    ) {
        $this->createdAt = new DateTime();
        $this->updatedAt = new DateTime();
    }
}
```

## Enum (PHP 8.1+)

```php
<?php

// Pure enum
enum Ruolo {
    case Admin;
    case User;
    case Editor;
}

// Backed enum (con valore scalare)
enum StatoOrdine: string {
    case InAttesa = "pending";
    case Confermato = "confirmed";
    case Spedito = "shipped";
    case Consegnato = "delivered";
}

$ordine = new Ordine(StatoOrdine::InAttesa);
echo $ordine->stato->value;     // "pending"
echo $ordine->stato->name;      // "InAttesa"

// Enum con metodi
enum Ruolo: string {
    case Admin = "admin";
    case User = "user";

    public function permessi(): array {
        return match ($this) {
            self::Admin => ['*'],
            self::User => ['read', 'write'],
        };
    }
}
```

## Visibility e polimorfismo

```php
<?php

class Base {
    public string $pubblico = "ovunque";
    protected string $protetto = "classe + subclassi";
    private string $privato = "solo questa classe";

    final public function nonSovrascrivibile(): void {}  // PHP 8.0+: final sui metodi
}

final class Finale extends Base {}  // classe final — non si può estendere
// class Prova extends Finale {}    // Error: Cannot extend final class
```

## Magic methods

```php
<?php

class Utente {
    private array $dati = [];

    // __get, __set — accesso a proprietà dinamiche
    public function __get(string $name): mixed {
        return $this->dati[$name] ?? null;
    }

    public function __set(string $name, mixed $value): void {
        $this->dati[$name] = $value;
    }

    // __call — per metodi dinamici
    public function __call(string $name, array $args): mixed {
        if (str_starts_with($name, 'get')) {
            $prop = lcfirst(substr($name, 3));
            return $this->dati[$prop] ?? null;
        }
        throw new BadMethodCallException("Metodo $name non esiste");
    }

    // __toString
    public function __toString(): string {
        return $this->dati['nome'] ?? 'Anonimo';
    }
}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Cannot override final method` | Metodo final sovrascritto in subclass | Rimuovi `final` dal metodo genitore |
| `Typed property must not be accessed before initialization` | Proprietà tipizzata letta prima di essere assegnata | Inizializza nel costruttore o con default |
| `Cannot use 'parent' in class without parent` | `parent::` usato in classe senza extends | Controlla che la classe estenda un'altra classe |
| `Access level to ... must be ...` | Visibilità più restrittiva in subclass | La subclass può allargare la visibilità, non restringerla |
| `Trait method ... has not been applied` | Conflitto tra due trait con stesso metodo | Usa `insteadof` o `as` per alias |
| `Enum backing type must be int or string` | Backed enum con tipo non supportato | Usa solo `string` o `int` per backed enum |

## Best practice

- **Property promotion sempre** (PHP 8.0+) — riduce boilerplate del 50%
- **`readonly` per DTO e Value Object** — immutabilità, meno bug
- **Composizione > ereditarietà** — usa trait e interfacce, non catene deep di extends
- **Enum > class con costanti** per stati fissi — type-safe, niente stringhe magiche
- **Final per default** — `final class` se non pensata per essere estesa (rende il codice più predicibile)
- **Getter/Setter solo se necessari** — PHP non ha Java-style property; se serve incapsulamento, usa `__get`/`__set` (con cautela) o metodi espliciti
- **`declare(strict_types=1)`** in ogni file OOP — errori di tipo a runtime invece di coercizione silenziosa
- **Evita magic methods per logica reale** — `__call`, `__get`, `__set` rendono il codice difficile da seguire

## Cross-reference

- [[PHP/Core Concepts/Funzioni|Funzioni]] — type hinting nei metodi
- [[PHP/Core Concepts/Namespace e Autoload|Namespace]] — namespace delle classi, PSR-4
- [[PHP/Core Concepts/Gestione Errori|Gestione Errori]] — eccezioni OOP, try/catch
- [[PHP/Laravel/Eloquent e Database|Laravel — Eloquent]] — ORM, Active Record, Model
