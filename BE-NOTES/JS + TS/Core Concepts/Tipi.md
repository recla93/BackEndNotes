---
topic: "Tipi — JavaScript"
tags: [javascript, js, base, types, coercion, truthy]
nav_prev: "[[Variabili e Scope.md]]"
nav_next: "[[Funzioni.md]]"
---
Riferimento ufficiale: [developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures)

JS ha **7 tipi primitivi** (immutabili) e **1 tipo strutturale** (object). Il tipo è dinamico: la stessa variabile può contenere tipi diversi in momenti diversi. `typeof` restituisce una stringa col nome del tipo — attenzione: `typeof null === "object"` è un bug storico mantenuto per retrocompatibilità.

```javascript
// Primitivi
let num = 42;            // number (IEEE 754 double, ±2^53 interi sicuri)
let str = "ciao";        // string (UTF-16, immutabile)
let bool = true;         // boolean
let und = undefined;     // valore predefinito di variabili non inizializzate
let vuoto = null;        // assenza deliberata di valore
let sym = Symbol("id");  // symbol (unico, immutabile — chiavi per proprietà)
let big = 9007199254740993n;  // bigint (interi arbitrariamente grandi)

// Object
let obj = { a: 1 };      // object (collezione chiave-valore, mutabile)
```

`undefined` è il valore che JS assegna automaticamente a variabili non inizializzate e proprietà inesistenti. `null` è un valore che il programmatore assegna per indicare "nessun valore". `typeof null === "object"` è un bug di implementazione del 1996 che non verrà mai corretto.

## Coerzione e operatori

JS è a **tipizzazione dinamica debole**: converte automaticamente i tipi in certi contesti (`==`, operatori aritmetici, confronti). Questo è fonte di bug famosi. `===` evita la coercizione confrontando anche il tipo.

```javascript
// Coercizione con == (da evitare)
5 == "5"       // true (converte stringa a numero)
0 == false     // true (false → 0)
"" == false    // true
null == undefined // true
[] == false    // true

// Confronto stretto con ===
5 === "5"      // false (tipi diversi)
0 === false    // false
null === undefined // false

// Aritmetica
"5" - 3        // 2 (converte "5" a number)
"5" + 3        // "53" (converte 3 a string — + è sovraccarico per concatenazione)
```

La regola: `+` con stringhe fa concatenazione; tutti gli altri operatori aritmetici (`-`, `*`, `/`, `%`) convertono a numero. Questo asimmetria è la fonte di molti bug — usa sempre `===` e conversioni esplicite.

## Truthy e Falsy

In un contesto booleano (if, while, `&&`, `||`, `!`), ogni valore viene valutato come **truthy** (vero) o **falsy** (falso). I falsy sono solo 7 valori — tutto il resto è truthy.

```javascript
// Falsy
false
0, -0, 0n
"" (stringa vuota)
null
undefined
NaN

// Truthy (tutto il resto)
"0"           // stringa non vuota → truthy!
"false"       // stringa non vuota → truthy!
[]            // array vuoto → truthy!
{}            // oggetto vuoto → truthy!
```

`if (str)` è un pattern idiomatico per controllare stringhe non vuote: passa se la stringa ha lunghezza > 0. Per numeri, `if (num >= 0)` è più esplicito perché `0` è falsy ma spesso è un valore valido.

## Type coercion tabella

| Operazione | Risultato | Spiegazione |
|---|---|---|
| `[] + []` | `""` | Array → string vuota, concatenazione |
| `[] + {}` | `"[object Object]"` | Array → `""`, Object → `"[object Object]"` |
| `{} + []` | `0` | `{}` è un blocco vuoto, `+[]` è `0` |
| `!!"false"` | `true` | `"false"` è stringa non vuota → truthy |
| `!!"0"` | `true` | `"0"` è stringa non vuota → truthy |

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `NaN` inaspettato | Operazione con `undefined` o stringa non numerica | Converti esplicitamente con `Number()` o `parseInt()` |
| `toString()` su `null` o `undefined` | Non hanno proprietà (sono primitivi) | Optional chaining: `val?.toString()` |
| `typeof null === "object"` | Bug storico di JS | Usa `val === null` per check esplicito |
| `"5" + 3` fa `"53"` e non `8` | `+` è sovraccarico concatenazione | Usa `Number()` o unario `+`: `+"5" + 3` |
| Confronto con `==` dà risultati inaspettati | Coercizione automatica | Usa sempre `===` |

## Best practice

- **Sempre `===` e `!==`** — mai `==`/`!=` se non hai un motivo esplicito (es. confrontare `null` e `undefined` insieme)
- **Conversioni esplicite**: `Number(val)`, `String(val)`, `Boolean(val)`, `BigInt(val)` — non confidare nella coercizione
- **Check null/undefined**: `val ?? "default"` (nullish coalescing) — cattura solo `null`/`undefined`, non altri falsy
- **Optional chaining**: `obj?.prop?.nested` — non lancia TypeError se intermedio è null/undefined
- **`!!val` per boolean esplicito** — converte qualsiasi valore a booleano (`!!0 === false`, `!!"a" === true`)
- **BigInt** solo per interi > 2^53 o precisione estrema — per API usa stringa (JSON non supporta BigInt nativamente)

## Cross-reference

- [[JS + TS/Core Concepts/Variabili e Scope|Variabili e Scope]] — dichiarazioni, TDZ, scope
- [[JS + TS/Core Concepts/Funzioni|Funzioni]] — arrow function, closure, this binding
- [[JS + TS/Core Concepts/Oggetti|Oggetti]] — object literal, property access, Symbol come chiave
- [[JS + TS/TypeScript/Tipi Avanzati|TypeScript — Tipi Avanzati]] — annotazioni di tipo, union, literal types
