---
topic: "Variabili e Tipi — PHP"
tags: [php, base, types, variables, type-hinting, coercion]
nav_next: "[[Stringhe.md]]"
---
Riferimento ufficiale: [php.net/manual/en/language.types.intro.php](https://www.php.net/manual/en/language.types.intro.php) | [php.net/manual/en/language.variables.php](https://www.php.net/manual/en/language.variables.php)

PHP è a **tipizzazione dinamica debole**: una variabile può cambiare tipo, e la coercizione automatica tra tipi è pervasiva (`==`). PHP 7+ ha introdotto **type hinting** (parametri e return type), che è obbligatorio in codice moderno. PHP 8+ ha aggiunto **union types**, **mixed**, e **static return type**.

```php
<?php

// Dichiarazione — $ precede sempre il nome
$nome = "Mario";       // string
$eta = 25;             // int
$prezzo = 19.99;       // float
$attivo = true;        // bool
$valori = [1, 2, 3];   // array
$vuoto = null;         // null
```

## Tipi built-in

```php
<?php

// Scalari
$int = 42;                 // int (intero, dipende da piattaforma: 64 bit su 64-bit)
$float = 3.14;             // float (double precision IEEE 754)
$string = "ciao";          // string (binaria — byte sequence)
$bool = true;              // bool (true/false)

// Composti
$array = [1, 2, 3];            // array (sia lista che mappa associativa)
$arrayAssoc = ["a" => 1];      // array associativo (equivalente a dict/oggetto)

// Speciali
$null = null;              // null (assenza di valore)
$resource = fopen("file.txt", "r");  // resource (handle esterno — file, DB, socket)

// PHP 8.0+ — union types
function foo(int|string $valore): void {}

// PHP 8.1+ — enum
enum Ruolo: string {
    case Admin = "admin";
    case User = "user";
}
```

## Type Hinting (dichiarazione dei tipi)

```php
<?php

declare(strict_types=1);  // ← ATTIVA MODE STRICT (per-file)

// Parametri e return type
function somma(int $a, int $b): int {
    return $a + $b;
}

// Union type (PHP 8.0+)
function findUser(int|string $id): User|null {
    return User::find($id);
}

// mixed — qualsiasi tipo (PHP 8.0+)
function log(mixed $valore): void {
    var_dump($valore);
}

// nullable (?) equivalenza
function find(?int $id): ?User {}       // PHP 7.1+
function find(int|null $id): User|null {}  // PHP 8.0+
```

`declare(strict_types=1)` è critico: senza, PHP cerca di convertire forzatamente i tipi (es. `"42"` diventa `42` per un `int`). Con strict types, lancia `TypeError`. Va dichiarato **per-file** (non globale).

## Coercizione e confronti

```php
<?php

// == converte automaticamente (da evitare)
"5" == 5        // true (string → int)
0 == false      // true
null == false   // true
"" == false     // true

// === confronto stretto (tipo + valore)
"5" === 5       // false
0 === false     // false
null === false  // false

// Confronti con stringhe
"abc" < "abd"   // true (comparazione alfabetica)
"10" < "2"      // false (string: "1" < "2")
"10" < 2        // true (string "10" → int 10)
```

## Variabili variabili e reference

```php
<?php

// Variable variable (raro, spesso codice smell)
$nomeVariabile = "saluto";
$$nomeVariabile = "Ciao";   // $saluto = "Ciao"

// Reference (&)
$originale = 5;
$ref = &$originale;   // $ref punta alla stessa locazione di $originale
$ref = 10;
echo $originale;      // 10 — modificato!
```

## Costanti

```php
<?php

// define() — a runtime
define("APP_NAME", "MyApp");
define("DB_CONFIG", [
    "host" => "localhost",
    "port" => 5432,
]);

// const — a compile-time (in classi o namespace)
const MAX_RETRY = 3;

// PHP 8.1+ — final class constant (non sovrascrivibile da subclassi)
class Config {
    final const VERSION = "1.0";
}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Undefined variable $x` | Variabile usata senza essere dichiarata | Inizializza prima dell'uso |
| `TypeError: Argument 1 must be int, string given` | strict_types=1 ma passi string per int | Converti esplicitamente o cambia type hint |
| `Cannot use object of type X as array` | Oggetto usato come array (non implementa ArrayAccess) | Controlla il tipo prima di usare `[]` |
| `Notice: Array to string conversion` | `echo $array` invece di `print_r()` o `json_encode()` | Usa `print_r($array, true)` per debug |
| Risultati inaspettati con `==` | Coercizione automatica del tipo | Usa sempre `===` e `!==` |
| `Uncaught TypeError: Return value must be of type int, float returned` | Return type non matcha | Converti il valore: `return (int) $valore` |

## Best practice

- **`declare(strict_types=1)` in ogni file** — senza, i type hint sono deboli e PHP converte forzatamente
- **Type hint sempre** — parametri e return type obbligatori in codice moderno; `mixed` solo se necessario
- **`===` e `!==`** — mai `==` o `!=` (coercizione imprevedibile)
- **`??` (null coalescing)** — `$valore = $array['key'] ?? 'default'` (PHP 7.0+)
- **`?->` (nullsafe, PHP 8.0+)** — `$user?->profile?->bio` (non lancia error se intermedio è null)
- **`int|float` non funziona come ci si aspetta** — `int|float` non esiste come validatore (accetta anche string); usa `int` e basta
- **Evita variable variables** — rendono il codice illeggibile; usa array associativi o `compact()`/`extract()` con cautela

## Cross-reference

- [[PHP/Core Concepts/Stringhe|Stringhe]] — escaping, interpolazione, encoding
- [[PHP/Core Concepts/Funzioni|Funzioni]] — type hinting avanzato, variadic, named arguments
- [[PHP/Core Concepts/Classi e OOP|Classi e OOP]] — typed property, readonly property
- [[PHP/Core Concepts/Gestione Errori|Gestione Errori]] — TypeError, strict types
