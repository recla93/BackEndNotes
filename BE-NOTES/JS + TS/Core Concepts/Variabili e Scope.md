---
topic: "Variabili e Scope — JavaScript"
tags: [javascript, js, base, variables, scope, hoisting]
nav_next: "[[Tipi.md]]"
---
Riferimento ufficiale: [developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types)

In JS le variabili si dichiarano con tre keyword: `var` (legacy, function-scope), `let` (block-scope, mutabile), `const` (block-scope, immutabile). La scelta tra `let` e `const` comunica l'intenzione: `const` significa "non riassegnerò questa variabile".

```javascript
let nome = "Mario";   // mutabile, block-scope
const PI = 3.14;       // immutabile (riassegnazione), block-scope
var vecchio = "no";    // function-scope, hoisting problematico
```

`let` e `const` sono block-scoped: esistono solo dentro il blocco `{}` in cui sono dichiarate. `var` ignora i blocchi (if, for, while) e scappa fino alla funzione contenitore — causa di bug storici. `const` vieta la **riassegnazione** della variabile, non la mutazione dell'oggetto: `const arr = [1]; arr.push(2)` è permesso.

```javascript
if (true) {
    var a = 1;    // scappa fuori dal blocco
    let b = 2;    // muore qui
}
console.log(a);  // 1
console.log(b);  // ReferenceError: b is not defined

const x = [1, 2];
x.push(3);       // ok — muto l'oggetto, non riassegno x
// x = [4];      // TypeError: Assignment to constant variable
```

## Hoisting

`var` viene **issata** (hoisting): la dichiarazione viene spostata in cima allo scope, ma l'assegnazione resta dov'è. Questo codice non dà errore ma stampa `undefined`:

```javascript
console.log(x);  // undefined (non ReferenceError)
var x = 5;
// equivale a: var x; console.log(x); x = 5;
```

`let` e `const` sono **issate ma in TDZ (Temporal Dead Zone)**: esistono nello scope ma non sono accessibili fino alla riga di dichiarazione. Accedervi prima dà ReferenceError — comportamento più sicuro.

```javascript
console.log(y);  // ReferenceError: Cannot access 'y' before initialization
let y = 10;
```

## Scope chain e Closure

JS cerca una variabile risalendo la catena degli scope (dall'interno verso l'esterno). Una **closure** è una funzione che "cattura" le variabili dello scope esterno, anche dopo che la funzione esterna è terminata — meccanismo fondamentale per callback, event handler, factory function.

```javascript
function creaContatore() {
    let count = 0;               // variabile "catturata"
    return function() {
        count++;                 // closure: accede a count
        return count;
    };
}
const contatore = creaContatore();
console.log(contatore());  // 1
console.log(contatore());  // 2
// count non è accessibile dall'esterno — incapsulamento via closure
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `ReferenceError: x is not defined` | Variabile non dichiarata | Dichiarala con `let`/`const` prima di usarla |
| `Cannot access before initialization` | TDZ — usi `let`/`const` prima della riga di dichiarazione | Sposta l'accesso dopo la dichiarazione |
| `Assignment to constant variable` | Riassegni una variabile `const` | Usa `let` se devi riassegnare |
| `undefined` inaspettato con `var` | Hoisting: dichiarazione spostata, assegnazione no | Usa `let`/`const` invece di `var` |
| Closure loop classico | Usare `var i` in un loop crea una sola variabile condivisa | Usa `let i` (crea un binding per iterazione) |

## Best practice

- **`const` di default, `let` solo se devi riassegnare** — comunica intenzione, previene errori
- **Mai `var` in codice moderno** — non offre vantaggi, crea bug di scope
- **Dichiara sempre** prima di usare — evita pollution del global scope (strict mode aiuta)
- **TDZ è tua amica** — ReferenceError su `let`/`const` è meglio di `undefined` silenzioso su `var`
- **Closure per incapsulamento** — pattern factory + closure è più leggero di una classe per oggetti semplici

## Cross-reference

- [[JS + TS/Core Concepts/Funzioni|Funzioni]] — closure approfondite, IIFE, factory function
- [[JS + TS/Core Concepts/Moduli|Moduli]] — scope dei moduli, export/import, global scope
- [[JS + TS/Core Concepts/Prototype|Prototype]] — scope di `this`, ereditarietà, prototype chain
