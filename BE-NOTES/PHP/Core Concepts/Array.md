---
topic: "Array — PHP"
tags: [php, base, arrays, associative, iterators, functions]
nav_prev: "[[Stringhe.md]]"
nav_next: "[[Funzioni.md]]"
---
Riferimento ufficiale: [php.net/manual/en/language.types.array.php](https://www.php.net/manual/en/language.types.array.php) | [php.net/manual/en/ref.array.php](https://www.php.net/manual/en/ref.array.php)

Gli array in PHP sono **mappe ordinate** (chiave → valore). Fungono sia da lista (indice numerico) che da mappa associativa (string key) e da insieme, spesso nello stesso array. Mantengono l'ordine di inserimento.

```php
<?php

// Lista (indice numerico automatico)
$numeri = [1, 2, 3, 4, 5];

// Mappa associativa
$utente = [
    "nome" => "Mario",
    "eta" => 25,
    "città" => "Roma",
];

// Misto (PHP permette, ma da evitare)
$misto = [1, "nome" => "Mario", 2, "altro" => true];
```

## Operazioni base

```php
<?php

$arr = [1, 2, 3];

// Aggiungere elementi
$arr[] = 4;              // [1, 2, 3, 4] — push in coda (solo indici numerici)
array_push($arr, 5);     // [1, 2, 3, 4, 5]
array_unshift($arr, 0);  // [0, 1, 2, 3, 4, 5] — aggiunge in testa

// Rimuovere
array_pop($arr);         // rimuove ultimo, lo restituisce
array_shift($arr);       // rimuove primo, lo restituisce (ri-indicizza!)
unset($arr[1]);          // rimuove per chiave (mantiene indice!)

// Verifica
isset($utente["nome"]);  // true (esiste e non è null)
array_key_exists("nome", $utente);  // true (esiste anche se null)
in_array("Mario", $utente);        // true (valore presente)
array_search("Mario", $utente);    // "nome" (chiave del valore)
```

## Array functions built-in

```php
<?php

$numeri = [1, 2, 3, 4, 5];

// map/filter/reduce
$doppi = array_map(fn($n) => $n * 2, $numeri);            // [2, 4, 6, 8, 10]
$pari = array_filter($numeri, fn($n) => $n % 2 === 0);    // [2, 4]
$somma = array_reduce($numeri, fn($acc, $n) => $acc + $n, 0);  // 15

// Ordinamento (tutti MUTANO l'array originale)
sort($numeri);         // per valore (re-indicizza)
asort($utente);        // per valore (mantiene chiavi)
ksort($utente);        // per chiave
usort($numeri, fn($a, $b) => $b <=> $a);  // comparatore custom

// Estrazione
$chiavi = array_keys($utente);      // ["nome", "eta", "città"]
$valori = array_values($utente);    // ["Mario", 25, "Roma"]
$pezzi = array_chunk($numeri, 2);   // [[1,2], [3,4], [5]]

// Unione e differenza
$merged = array_merge($a, $b);      // concatena (sovrascrive chiavi uguali)
$diff = array_diff($a, $b);         // valori in a non in b
$intersect = array_intersect($a, $b); // valori in comune
```

## Operatore `...` (spread) PHP 7.4+

```php
<?php

$arr1 = [1, 2, 3];
$arr2 = [4, 5, 6];
$merged = [...$arr1, ...$arr2];    // [1, 2, 3, 4, 5, 6]

// Con chiavi associative (sovrascrive duplicati)
$a = ["a" => 1, "b" => 2];
$b = ["b" => 3, "c" => 4];
$c = [...$a, ...$b];  // ["a" => 1, "b" => 3, "c" => 4]
```

## Destrutturazione

```php
<?php

// List assignment
[$a, $b, $c] = [1, 2, 3];       // $a=1, $b=2, $c=3
[0 => $a, 2 => $c] = [1, 2, 3]; // $a=1, $c=3

// Con chiavi associative (PHP 7.1+)
["nome" => $nome, "eta" => $eta] = $utente;
echo $nome;  // "Mario"
```

## Array e foreach

```php
<?php

$utente = ["nome" => "Mario", "eta" => 25];

foreach ($utente as $chiave => $valore) {
    echo "$chiave: $valore";
}

// Modificare elementi durante foreach (richiede reference &)
foreach ($utente as &$valore) {
    $valore = strtoupper((string)$valore);
}
unset($valore);  // rompe reference — IMPORTANTE!
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Undefined array key "x"` (PHP 8.0+ warning) | Chiave inesistente | Usa `$arr['key'] ?? 'default'` |
| `array_shift` re-indicizza tutto | `array_shift` ri-numera gli indici numerici | Usa `array_slice($arr, 1)` invece (non muta) |
| `sort` perde chiavi associative | `sort()` re-indicizza sempre | Usa `asort()` (mantiene le chiavi) |
| `foreach` modifica elemento non funziona | `foreach` opera su copia | Usa reference `&$valore` |
| `array_merge` con array numerico re-indicizza | `array_merge` ri-numera array numerici | Usa `$a + $b` per unione senza re-indicizzare |
| `empty($arr['key'])` dà false se key esiste ma è null/false/0 | `empty()` è truthy-check | Usa `!isset($arr['key'])` o `!array_key_exists(...)` |

## Best practice

- **`??` (null coalescing)** per accesso sicuro — `$val = $arr['key'] ?? 'default'` (PHP 7.0+)
- **`match` (PHP 8.0+) > `switch`** — `match` è espressione (restituisce valore), strict comparison
- **`array_map`/`array_filter` > `foreach`** per trasformazioni — più dichiarativo, meno side effect
- **Chiavi stringa sempre con apici** — `$arr['key']` non `$arr[key]` (PHP cerca costante `key`, poi stringa "key")
- **`unset($ref)` dopo foreach con reference** — rompe il riferimento per evitare side effect
- **`[...$a, ...$b]` > `array_merge`** — più leggibile, supporta chiavi numeriche senza re-indicizzare
- **Usa `array_is_list($arr)` (PHP 8.1+)** per verificare se un array è una lista (indice 0..n-1)

## Cross-reference

- [[PHP/Core Concepts/Funzioni|Funzioni]] — array_map/filter/reduce, closure, arrow function
- [[PHP/Core Concepts/Controllo e Loop|Controllo e Loop]] — foreach, for, while
- [[PHP/Database/PDO|PDO]] — fetch modes: assoc, obj, num
- [[PHP/Core Concepts/Gestione Errori|Gestione Errori]] — warning per undefined array key
