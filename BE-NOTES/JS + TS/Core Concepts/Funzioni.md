---
topic: "Funzioni — JavaScript"
tags: [javascript, js, base, functions, closure, callback, arrow]
nav_prev: "[[Tipi.md]]"
nav_next: "[[Oggetti.md]]"
---
Riferimento ufficiale: [developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions)

In JS le funzioni sono **first-class citizens**: possono essere assegnate a variabili, passate come argomenti, restituite da altre funzioni. Questo è il fondamento del **programmazione funzionale** e del **callback pattern** in JS.

```javascript
// Dichiarazione (hoisted — può essere chiamata prima della definizione)
function somma(a, b) {
    return a + b;
}

// Espressione (non hoisted — deve essere definita prima dell'uso)
const somma = function(a, b) {
    return a + b;
};

// Arrow function (ES6) — più concisa, senza proprio this/arguments
const somma = (a, b) => a + b;
```

La differenza chiave: le **dichiarazioni** sono hoisted (l'intera funzione viene issata), le **espressioni** e le arrow no. Le **arrow function** non hanno `this` proprio, non hanno `arguments`, e non possono essere usate come costruttori. Questo le rende ideali come callback dove vuoi ereditare il `this` del contesto esterno.

## Parametri e default

```javascript
// Default parameters (ES6)
function saluta(nome = "Mondo") {
    return `Ciao ${nome}`;
}

// Rest parameters (ES6) — raggruppa argomenti extra in array
function sommaTutti(...numeri) {
    return numeri.reduce((acc, n) => acc + n, 0);
}

// Destructuring nei parametri
function stampaUtente({ nome, eta, città = "Roma" }) {
    console.log(`${nome}, ${eta} anni, ${città}`);
}
```

`arguments` (oggetto array-like disponibile solo nelle funzioni tradizionali) è deprecato a favore di rest parameters `...args`. Le arrow function non hanno `arguments`.

## Closure

Una **closure** è creata quando una funzione interna mantiene accesso a variabili dello scope esterno dopo che la funzione esterna è terminata. È il meccanismo dietro callback, event handler, factory pattern, e incapsulamento.

```javascript
function creaLogger(prefix) {
    return function(message) {
        console.log(`[${prefix}] ${message}`);
    };
}
const errorLogger = creaLogger("ERROR");
errorLogger("Connessione persa");  // [ERROR] Connessione persa
```

Ogni chiamata a `creaLogger` crea un nuovo scope con la sua `prefix` — le closure multiple non condividono lo stesso binding.

## IIFE (Immediately Invoked Function Expression)

Usata prima di ES6 per creare scope isolati ed evitare pollution del global scope. Oggi i modelli (ESM) risolvono lo stesso problema in modo più pulito.

```javascript
// IIFE — funzione definita e invocata subito
(function() {
    const privato = "non accessibile dall'esterno";
    console.log(privato);
})();
```

## Callback e Higher-order function

Una **higher-order function** è una funzione che prende un'altra funzione come argomento o la restituisce. Le callback sono il pattern asincrono originale di JS (prima delle Promise). Un esempio classico è `setTimeout` o `Array.prototype.forEach`.

```javascript
function eseguiDopo(fn, ms) {
    setTimeout(fn, ms);
}
eseguiDopo(() => console.log("passati 1 secondo"), 1000);

// Higher-order: prende e restituisce funzioni
function moltiplica(fattore) {
    return (numero) => numero * fattore;
}
const raddoppia = moltiplica(2);  // closure su fattore=2
console.log(raddoppia(5));        // 10
```

## Arrow function e `this`

`this` in JS è **dinamico**: dipende da come la funzione viene chiamata (non da dove è definita). Le arrow function ereditano `this` dal contesto lessicale — non hanno un proprio `this`. Questo le rende ideali per callback in classi o gestori di eventi.

```javascript
const utente = {
    nome: "Mario",
    salutaTardivo: function() {
        // function: this dipende dalla chiamata
        setTimeout(function() {
            console.log(this.nome);  // undefined (this = global/window)
        }, 100);
    },
    salutaArrow: function() {
        // arrow: this ereditato dal contesto (salutaArrow → utente)
        setTimeout(() => {
            console.log(this.nome);  // "Mario"
        }, 100);
    }
};
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `this` è `undefined` o `window` in callback | Arrow function non usata o chiamata senza contesto | Usa arrow function o `.bind(this)` |
| `arguments` non definito in arrow | Arrow function non ha `arguments` | Usa rest parameter `...args` |
| Closure cattura vecchio valore in loop | `var` nel loop crea un binding condiviso | Usa `let` (binding per iterazione) |
| `Cannot access before initialization` | IIFE richiamata prima della definizione | Sposta IIFE dopo la dichiarazione |
| Maximum call stack size exceeded | Ricorsione senza base case | Aggiungi condizione di terminazione |

## Best practice

- **Arrow per callback brevi** — concisa, `this` lessicale, return implicito per espressioni semplici
- **Named function per logica complessa** — migliore stack trace in debugging
- **Default parameters, non `||`** — `function f(x = 1)` è più esplicito di `x = x || 1` (che fallisce per falsy validi come `0`)
- **Pochi parametri** — più di 3 parametri → usa un oggetto destrutturato: `function f({ a, b, c, d })`
- **Never mutate parametri** — trattali come `const` (soprattutto oggetti e array passati per reference)
- **Preferisci `reduce`/`map`** su loop espliciti per trasformazioni di array (più dichiarativo, meno side effect)

## Cross-reference

- [[JS + TS/Core Concepts/Array|Array]] — map, filter, reduce (callback in azione)
- [[JS + TS/Core Concepts/Async|Async]] — callback → Promise → async/await
- [[JS + TS/Core Concepts/Classi|Classi]] — metodi, constructor, super
- [[JS + TS/Core Concepts/Moduli|Moduli]] — export/import di funzioni tra file
