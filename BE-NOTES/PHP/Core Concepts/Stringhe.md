---
topic: "Stringhe — PHP"
tags: [php, base, strings, interpolation, encoding]
nav_prev: "[[Variabili e Tipi.md]]"
nav_next: "[[Array.md]]"
---
Riferimento ufficiale: [php.net/manual/en/language.types.string.php](https://www.php.net/manual/en/language.types.string.php) | [php.net/manual/en/ref.strings.php](https://www.php.net/manual/en/ref.strings.php)

Le stringhe in PHP sono **binarie** (sequence di byte, non caratteri Unicode). Una stringa è un carattere null-byte non indica la fine (a differenza di C). Il supporto multibyte richiede funzioni `mb_*`.

```php
<?php

// Quattro sintassi
$a = 'stringa semplice';      // niente interpolazione, niente escaping complex
$b = "stringa con $variabile";  // interpolazione variabili
$c = <<<EOT
Heredoc — come doppi apici, interpreta variabili
EOT;
$d = <<<'EOT'
Nowdoc — come apici singoli, niente interpretazione
EOT;
```

## Apici singoli vs doppi

```php
<?php

$nome = "Mario";

// Apici singoli — niente interpolazione, escapa solo \\ e \'
echo 'Ciao $nome';       // "Ciao $nome" (letterale)
echo 'Path: C:\\';       // "Path: C:\"
echo 'I\'m';              // "I'm"

// Doppi apici — interpolazione + escape sequences
echo "Ciao $nome";       // "Ciao Mario"
echo "Tab:\tqui";         // "Tab:	qui"
echo "Newline:\n";        // va a capo
```

## Interpolazione avanzata

```php
<?php

$utente = (object)["nome" => "Mario", "eta" => 25];

// Variabile semplice
echo "Ciao $utente->nome";      // "Ciao Mario"

// Espressioni complesse con {}
echo "Ciao {$utente->nome}";    // "Ciao Mario"
echo "Hai {$utente->eta} anni";
echo "Valore: {$array['chiave']}";
echo "Metodo: {$oggetto->metodo()}";  // PHP 8.0+ — chiamata metodo in interpolazione
```

## Funzioni principali

```php
<?php

$testo = "  Hello World  ";

// Pulizia
trim($testo);                // "Hello World" (rimuove spazi ai lati)
ltrim($testo);               // "Hello World  " (solo sinistra)
rtrim($testo);               // "  Hello World" (solo destra)

// Taglio e posizione
substr($testo, 2, 5);        // "llo W" (da pos 2, 5 caratteri)
strlen($testo);              // 16 (byte, non caratteri!)
mb_strlen($testo, 'UTF-8');  // lunghezza in caratteri Unicode

// Ricerca e sostituzione
str_contains($testo, "World");    // true (PHP 8.0+)
str_starts_with($testo, "  He");  // true (PHP 8.0+)
str_ends_with($testo, "ld");      // true (PHP 8.0+)
str_replace("World", "PHP", $testo);  // "  Hello PHP  "

// Array ↔ stringa
$pezzi = explode(",", "a,b,c");                 // ["a", "b", "c"]
$unita = implode(", ", ["a", "b", "c"]);        // "a, b, c"

// Case
strtolower($testo);         // "  hello world  "
strtoupper($testo);         // "  HELLO WORLD  "
ucfirst("mario");           // "Mario"
ucwords("mario rossi");     // "Mario Rossi"
```

## mb_* — stringhe multibyte

```php
<?php

// Pericolo: strlen conta byte, non caratteri!
$emoji = "🇮🇹";  // 8 byte (4 per ogni flag letter)
strlen($emoji);         // 8
mb_strlen($emoji, 'UTF-8');  // 1 (un carattere)

// Tutte le funzioni stringa hanno equivalente mb_
substr("caffè", 3);         // byte — può tagliare a metà carattere!
mb_substr("caffè", 3, 2, 'UTF-8');  // "è"

// Always use mb_* for UTF-8 strings
mb_strtolower($testo, 'UTF-8');
mb_strtoupper($testo, 'UTF-8');
```

## Encoding e HTML

```php
<?php

// HTML escaping — fondamentale per XSS prevention
$input = "<script>alert('xss')</script>";
echo htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
// &lt;script&gt;alert(&#039;xss&#039;)&lt;/script&gt;

// URL encoding
urlencode("Mario Rossi");     // "Mario+Rossi"
urldecode("Mario+Rossi");    // "Mario Rossi"

// Base64
base64_encode($dati);         // encoding (non cifratura!)
base64_decode($stringa);      // decoding
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `strlen` restituisce lunghezza sbagliata per UTF-8 | `strlen` conta byte, non caratteri | Usa `mb_strlen($str, 'UTF-8')` |
| `substr` taglia caratteri UTF-8 a metà | `substr` lavora su byte | Usa `mb_substr()` |
| `htmlspecialchars` non chiamato prima di output | XSS vulnerability | Chiama `htmlspecialchars()` su ogni output utente |
| `trim` non rimuove spazi non-breaking | `trim` gestisce solo spazi ASCII | Usa regex `preg_replace('/^\s+|\s+$/u', '', $str)` |
| Array to string conversion | `echo $array` | Usa `implode()`, `json_encode()`, o `print_r()` |
| Hering/doc chiusura con spazi | La linea di chiusura deve essere esatta, senza indentazione | Allinea `EOT;` all'inizio della riga |

## Best practice

- **mb_* per UTF-8** — qualsiasi stringa che arriva da utente o database può contenere Unicode; usa `mb_*` sempre
- **`htmlspecialchars()` su ogni output** — previene XSS; usa `ENT_QUOTES` per gestire anche apici singoli
- **Doppi apici solo se c'è interpolazione** — altrimenti apici singoli (più veloci, più prevedibili)
- **`str_contains()` > `strpos()`** — PHP 8.0+: `str_contains($haystack, $needle)` è più leggibile di `strpos($h, $n) !== false`
- **`json_encode()` per API** — mai costruire JSON manualmente con concatenaione di stringhe
- **Encoding dichiarato** — `header('Content-Type: text/html; charset=UTF-8')` o `<meta charset="UTF-8">`
- **`sprintf()` per formati complessi** — più leggibile della concatenazione per formati con placeholder

## Cross-reference

- [[PHP/Core Concepts/Variabili e Tipi|Variabili e Tipi]] — type hinting, casting
- [[PHP/Web/Superglobali|Superglobali]] — $_GET, $_POST, input sanitization
- [[PHP/Database/PDO|PDO]] — prepared statement, SQL injection prevention
- [[PHP/Laravel/Blade e Template|Laravel — Blade]] — template engine, escaping automatico
