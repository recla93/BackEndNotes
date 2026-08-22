---
topic: "Funzioni — PHP"
tags: [php, base, functions, closures, variadic, type-hinting]
nav_prev: "[[Array.md]]"
nav_next: "[[Controllo e Loop.md]]"
---
Riferimento ufficiale: [php.net/manual/en/language.functions.php](https://www.php.net/manual/en/language.functions.php) | [php.net/manual/en/functions.arrow.php](https://www.php.net/manual/en/functions.arrow.php)

PHP supporta funzioni di prima classe dalle versioni iniziali. Con PHP 7+ (type hinting), 7.4+ (arrow function, typed property), e 8.0+ (named argument, union types), le funzioni PHP sono diventate più espressive e sicure.

```php
<?php

declare(strict_types=1);

// Dichiarazione standard
function somma(int $a, int $b): int {
    return $a + $b;
}

// Nessun return = null implicito (void esplicito meglio)
function logMsg(string $msg): void {
    echo "[LOG] $msg";
}
```

## Parametri avanzati

```php
<?php

// Parametri opzionali (devono essere DOPO quelli obbligatori)
function saluta(string $nome, string $saluto = "Ciao"): string {
    return "$saluto, $nome!";
}
saluta("Mario");           // "Ciao, Mario!"
saluta("Mario", "Buongiorno");  // "Buongiorno, Mario!"

// Variadic — numero variabile di argomenti (PHP 5.6+)
function sommaTutti(int ...$numeri): int {
    return array_sum($numeri);
}
sommaTutti(1, 2, 3, 4, 5);  // 15

// Named arguments (PHP 8.0+)
function creaUtente(string $nome, int $eta, string $città = "Roma"): string {
    return "$nome, $eta anni, $città";
}
creaUtente(nome: "Mario", città: "Milano", eta: 25);
// Ordine non conta con named arguments!
```

## Arrow function (PHP 7.4+)

```php
<?php

// Arrow function — più concisa di closure, cattura variabili da outer scope automaticamente
$numeri = [1, 2, 3, 4, 5];
$fattore = 2;
$doppi = array_map(fn($n) => $n * $fattore, $numeri);
// ↑ cattura $fattore automaticamente (non serve 'use')

// Equivalente con closure (più verboso)
$doppi = array_map(function($n) use ($fattore) {
    return $n * $fattore;
}, $numeri);
```

## Closure e callable

```php
<?php

// Closure anonima
$quadrato = function(int $x): int {
    return $x * $x;
};
echo $quadrato(5);  // 25

// Closure con 'use' — cattura variabile da outer scope
$messaggio = "Ciao";
$saluta = function(string $nome) use ($messaggio): string {
    return "$messaggio, $nome!";
};
echo $saluta("Mario");  // "Ciao, Mario!"

// Callable come parametro
function applica(array $items, callable $callback): array {
    $risultato = [];
    foreach ($items as $item) {
        $risultato[] = $callback($item);
    }
    return $risultato;
}
$doppi = applica([1, 2, 3], fn($n) => $n * 2);
```

## Type hinting avanzato

```php
<?php

declare(strict_types=1);

// Union types (PHP 8.0+)
function findUser(int|string $id): User|array|null {
    return is_numeric($id) ? User::find($id) : User::whereEmail($id)->first();
}

// mixed (PHP 8.0+)
function logValue(mixed $valore): void {
    var_dump($valore);
}

// self, parent, static (per OOP)
class Utente {
    public static function create(array $data): static {
        return new static($data);
    }
}

// never (PHP 8.1+) — funzione che non torna mai (die/exit/eccezione)
function abort(int $code): never {
    http_response_code($code);
    exit;
}
```

## First-class callable (PHP 8.1+)

```php
<?php

// Prima: closure
$fn = function(int $x) { return strlen($x); };

// PHP 8.1+: first-class callable syntax
$fn = strlen(...);  // reference alla funzione senza chiamarla
$chars = array_map($fn, $strings);
```

## Funzioni built-in utili

```php
<?php

// Debug
var_dump($variabile);     // tipo + valore, ricorsivo
print_r($variabile, true); // leggibile, return string con true

// Tipi
is_string($val);
is_int($val);
is_array($val);
is_null($val);
gettype($val);            // stringa col tipo

// Casting esplicito
$int = (int) "42";
$float = (float) "3.14";
$string = (string) 42;
$array = (array) $oggetto;
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Uncaught TypeError: Argument 1 must be int, string given` | strict_types=1 ma passi string | Converti esplicitamente: `(int) $input` |
| `Cannot use object of type Closure as array` | Closure chiamata con `[]` | Usa `$closure()` non `$closure[]` |
| Variable `$x` not defined in closure | `use` non dichiarato per variabile esterna | `function() use ($x) { ... }` |
| `call_user_func_array()` deprecato (PHP 8.1+) | Usa spread `...` | `$fn(...$args)` |
| Named argument dopo positional (PHP 8.0+) | `fn(1, nome: "Mario")` non permesso | Metti named args alla fine |
| `Return value must be of type int, float returned` | Return type int ma restituisci float | Cast: `return (int) $valore` |

## Best practice

- **`declare(strict_types=1)` in ogni file funzioni** — senza, type hint è decorativo
- **Tipo restituito sempre dichiarato** — anche `void`, mai lasciare implicito
- **Arrow function per callback semplici** — più leggibile, cattura automatica variabili
- **Named arguments per parametri opzionali** — migliora leggibilità quando saltiamo parametri
- **Pochi parametri** — più di 3 → usa un oggetto DTO o array nominato
- **Return early** — guard clause per casi limite, return principale alla fine
- **`mixed` solo se strettamente necessario** — meglio union type esplicita
- **`never` per funzioni che terminano** — `exit`, `die`, eccezione sempre lanciata

## Cross-reference

- [[PHP/Core Concepts/Variabili e Tipi|Variabili e Tipi]] — strict_types, type system
- [[PHP/Core Concepts/Array|Array]] — array_map/filter/reduce (callback pattern)
- [[PHP/Core Concepts/Classi e OOP|Classi e OOP]] — metodi, visibility, static
- [[PHP/Core Concepts/Nome e Autoload|Namespace]] — namespace nelle funzioni
