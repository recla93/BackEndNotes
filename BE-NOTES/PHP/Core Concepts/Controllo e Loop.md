---
topic: "Controllo e Loop — PHP"
tags: [php, base, control-flow, loops, match, foreach]
nav_prev: "[[Funzioni.md]]"
nav_next: "[[Classi e OOP.md]]"
---
Riferimento ufficiale: [php.net/manual/en/language.control-structures.php](https://www.php.net/manual/en/language.control-structures.php) | [php.net/manual/en/control-structures.match.php](https://www.php.net/manual/en/control-structures.match.php)

PHP ha le strutture di controllo standard (if/else, for, while) più `foreach` (iterazione su array) e `match` (PHP 8.0+, valuta espressioni). Le sintassi alternative con `:` e `endif` sono legacy da template PHP (oggi evitate).

```php
<?php

$eta = 25;

// if/elseif/else
if ($eta >= 18) {
    echo "Maggiorenne";
} elseif ($eta >= 13) {
    echo "Adolescente";
} else {
    echo "Minore";
}
```

## match (PHP 8.0+)

`match` è un'espressione (restituisce valore), fa **strict comparison** (`===`), e non serve `break` (solo un ramo viene eseguito).

```php
<?php

$ruolo = "admin";

// switch (legacy) — loose comparison, serve break
switch ($ruolo) {
    case "admin":
        $permessi = "tutti";
        break;
    case "user":
        $permessi = "lettura";
        break;
    default:
        $permessi = "nessuno";
}

// match (PHP 8.0+) — strict, espressione, senza break
$permessi = match ($ruolo) {
    "admin" => "tutti",
    "user" => "lettura",
    "editor" => "scrittura",
    default => "nessuno",  // UnhandledMatchError se manca default e nessun match
};
```

## foreach

```php
<?php

$utenti = [
    ["nome" => "Mario", "eta" => 25],
    ["nome" => "Luigi", "eta" => 30],
];

// foreach con valore
foreach ($utenti as $utente) {
    echo $utente["nome"];
}

// foreach con chiave e valore
foreach ($utenti as $indice => $utente) {
    echo "$indice: {$utente['nome']}";
}

// Modifica durante foreach — usa reference
foreach ($utenti as &$utente) {
    $utente["eta"] += 1;
}
unset($utente);  // ← rompi il riferimento!

// Break e continue
foreach ($utenti as $utente) {
    if ($utente["nome"] === "Luigi") continue;  // salta
    if ($utente["eta"] > 50) break;              // esci
    echo $utente["nome"];
}
```

## For, While, Do-While

```php
<?php

// for classico
for ($i = 0; $i < 10; $i++) {
    echo $i;
}

// while
$i = 0;
while ($i < 10) {
    echo $i++;
}

// do-while (almeno una esecuzione)
$i = 0;
do {
    echo $i++;
} while ($i < 10);
```

## Sintassi alternativa (template legacy)

```php
<?php /* template.phtml — stile legacy, evita in codice moderno */ ?>
<ul>
<?php foreach ($items as $item): ?>
    <li><?= htmlspecialchars($item) ?></li>
<?php endforeach; ?>
</ul>
```

`<?= ... ?>` è shorthand per `<?php echo ... ?>` (disponibile da PHP 5.4+). Usalo nei template, ma sempre con `htmlspecialchars()` per sicurezza.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `switch` esegue più rami | `break` dimenticato in switch | Usa `match` invece di switch (PHP 8.0+) |
| `foreach` con reference modifica array originale | `&$valore` modifica l'array | `unset($ref)` dopo il loop |
| `UnhandledMatchError` (PHP 8.0+) | `match` non copre tutti i casi | Aggiungi `default =>` |
| Loop infinito | Condizione while mai falsa | Controlla che la condizione cambi nel loop |
| `continue` in `switch` (PHP 8.0) | `continue` con `switch` fa `continue 2` | Usa `break` per switch, `continue` per loop |
| `foreach` su variabile non iterabile | `foreach($nonArray as $x)` — stringa, null, oggetto non Traversable | Controlla `is_iterable($var)` prima |

## Best practice

- **`match` > `switch`** in PHP 8.0+ — strict comparison, espressione, nessun fallthrough
- **`foreach` > `for`** per iterare array — più leggibile, meno errori off-by-one
- **`unset($ref)` dopo foreach con reference** — rompe il riferimento, evita side effect
- **`for` solo per contatori** — iterazione con indice numerico; per tutto il resto usa `foreach`
- **Early return** — gestisci edge case all'inizio, main path dopo : `if (!$valido) return; ... logica ...`
- **`break $n`** per uscire da loop annidati — `break 2` esce da due livelli (ma meglio non annidare)

## Cross-reference

- [[PHP/Core Concepts/Array|Array]] — foreach su array, array_map/filter
- [[PHP/Core Concepts/Funzioni|Funzioni]] — return nelle funzioni, early return
- [[PHP/Core Concepts/Classi e OOP|Classi e OOP]] — iterable, Iterator pattern
- [[PHP/Laravel/Blade e Template|Laravel — Blade]] — @foreach, @if nel template engine
