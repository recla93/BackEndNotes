---
topic: "Array — JavaScript"
tags: [javascript, js, base, arrays, map, filter, reduce]
nav_prev: "[[Oggetti.md]]"
nav_next: "[[Classi.md]]"
---
Riferimento ufficiale: [developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)

Gli array in JS sono **dinamici** (crescono/shrinkano automaticamente) e **eterogenei** (possono contenere tipi misti). Sono un tipo speciale di oggetto con chiavi numeriche e una proprietà `length` aggiornata automaticamente.

```javascript
const numeri = [1, 2, 3, 4, 5];
const misto = [1, "ciao", true, { a: 1 }, [2, 3]];
numeri.push(6);   // aggiunge in coda → [1, 2, 3, 4, 5, 6]
numeri.pop();     // rimuove ultimo    → 6
numeri[0] = 10;   // modifica per indice
console.log(numeri.length);  // 5
```

## Metodi funzionali

I metodi `map`, `filter`, `reduce` e `forEach` sono il cuore della programmazione funzionale su array. Non mutano l'array originale (tranne `forEach` e alcuni altri). Il pattern è: **trasformazione dati** senza side effect.

```javascript
const nums = [1, 2, 3, 4, 5];

// map — trasforma ogni elemento
const doppi = nums.map(n => n * 2);            // [2, 4, 6, 8, 10]

// filter — mantiene elementi che superano il test
const pari = nums.filter(n => n % 2 === 0);    // [2, 4]

// reduce — riduce a un singolo valore (acc cumulatore → sempre specificare valore iniziale)
const somma = nums.reduce((acc, n) => acc + n, 0);  // 15

// forEach — side effect, non restituisce nulla
nums.forEach(n => console.log(n));

// find — primo elemento che supera il test (o undefined)
const primoPari = nums.find(n => n % 2 === 0);      // 2

// some/every — test booleani
const haPari = nums.some(n => n % 2 === 0);          // true
const tuttiPari = nums.every(n => n % 2 === 0);      // false
```

`map` crea un nuovo array della stessa lunghezza dell'originale. `filter` può ridurre la lunghezza. `reduce` è il più generico: qualsiasi operazione su array può essere espressa con `reduce` (ma usa `map`/`filter` quando più specifici — più leggibili). `forEach` è l'ultima scelta: usa `for...of` se devi iterare senza trasformare.

## Ricerca e ordinamento

```javascript
// includes — valore primitivo presente?
[1, 2, 3].includes(2);              // true

// indexOf — indice del valore (primitivo)
[1, 2, 3].indexOf(2);               // 1

// findIndex — indice per condizione
const idx = nums.findIndex(n => n > 3);  // 3

// sort — trasforma l'array IN LOCO (mutazione!), restituisce l'array mutato
[3, 1, 4, 1, 5].sort((a, b) => a - b);   // [1, 1, 3, 4, 5]

// reverse — muta l'array in loco
[1, 2, 3].reverse();                // [3, 2, 1]
```

`sort` senza comparatore converte gli elementi a stringa e ordina lessicograficamente (es. `[1, 10, 2]`). Il comparatore deve restituire: negativo se `a < b`, positivo se `a > b`, 0 se uguale. `sort` e `reverse` **mutano** l'array originale — fai `[...arr].sort()` per evitare side effect.

## Slicing, splicing e flat

```javascript
const arr = [1, 2, 3, 4, 5];

// slice — copia senza mutare (da indice, a indice)
arr.slice(1, 3);          // [2, 3] — originale invariato
arr.slice(-2);            // [4, 5] — ultimi 2

// splice — muta l'array (da indice, quanti rimuovere, ...da aggiungere)
arr.splice(1, 2);         // rimuove [2, 3], arr → [1, 4, 5]
arr.splice(1, 0, "a", "b"); // inserisce senza rimuovere

// flat — appiattisce n livelli
[1, [2, [3]]].flat(1);    // [1, 2, [3]]
[1, [2, [3]]].flat(2);    // [1, 2, 3]
[1, [2, [3]]].flat(Infinity); // appiattisce tutto

// flatMap — map + flat(1) in un solo passaggio
["ciao mondo", "hello world"].flatMap(s => s.split(" "));
// ["ciao", "mondo", "hello", "world"]
```

`slice` è sicuro (non muta). `splice` muta — usalo con cautela. `flatMap` è più efficiente di `map(...).flat()` perché evita di creare l'array intermedio.

## Spread su array

```javascript
const a = [1, 2, 3];
const b = [4, 5, 6];

const merge = [...a, ...b];          // [1, 2, 3, 4, 5, 6]
const clone = [...a];                // shallow copy
const [primo, secondo, ...resto] = a; // destructuring con rest
console.log(resto);                  // [3]
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `sort()` restituisce `[1, 10, 2]` | sort senza comparatore ordina per stringa | `arr.sort((a, b) => a - b)` |
| `map` usato per side effect | `map` crea array inutile | Usa `for...of` o `forEach` |
| Array mutato inaspettatamente | `sort`/`splice`/`reverse` mutano l'originale | Fai `[...arr].sort()` |
| `includes` non funziona con oggetti | `includes` usa `===` (reference per oggetti) | Usa `some(o => o.id === targetId)` |
| `forEach` non supporta `break` | `forEach` esegue per ogni elemento | Usa `for...of` o `some` per early exit |

## Best practice

- **Catene di metodi**: `arr.filter(...).map(...).reduce(...)` — leggibile, dichiarativo, immutabile
- **`reduce` con valore iniziale sempre** — senza iniziale fallisce su array vuoto
- **Shallow copy prima di mutare**: `[...arr].sort()` o `arr.toSorted()` (ES2023, non muta)
- **`for...of` > `forEach`** se hai bisogno di `break`, `continue`, o `await`
- **`flatMap` > `map + flat`** — un passaggio, array intermedio in meno
- **Usa `Set` per unicità**: `[...new Set(arr)]` rimuove duplicati più velocemente di filter+indexOf

## Cross-reference

- [[JS + TS/Core Concepts/Funzioni|Funzioni]] — callback, higher-order function
- [[JS + TS/Core Concepts/Oggetti|Oggetti]] — destructuring, spread/rest, oggetti vs array
- [[JS + TS/Core Concepts/Async|Async]] — array asincroni, Promise.all su array di Promise
- [[JS + TS/Core Concepts/Errori|Errori]] — try/catch in catene di metodi su array
