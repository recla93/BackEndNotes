---
topic: "Oggetti — JavaScript"
tags: [javascript, js, base, objects, this, destructuring, spread]
nav_prev: "[[Funzioni.md]]"
nav_next: "[[Array.md]]"
---
Riferimento ufficiale: [developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects)

Gli oggetti JS sono **mappe dinamiche chiave-valore**. Le chiavi sono stringhe o Symbol, i valori possono essere qualsiasi tipo (inclusi altri oggetti e funzioni). A differenza di Java, non c'è una classe da definire — un oggetto si crea direttamente con `{}`.

```javascript
const utente = {
    nome: "Mario",
    eta: 25,
    saluta() {                    // shorthand method (ES6)
        return `Ciao, sono ${this.nome}`;
    },
    get anni() { return this.eta; }  // getter
};
console.log(utente.nome);         // "Mario" (dot notation)
console.log(utente["nome"]);      // "Mario" (bracket — per chiavi dinamiche)
```

Dot notation è più leggibile ma statica. Bracket notation permette chiavi dinamiche e chiavi con caratteri speciali (`"nome-cognome"`, emoji, spazi).

## Proprietà dinamiche e shorthand

```javascript
const key = "email";
const valore = "mario@test.it";

// Computed property name (ES6)
const utente = {
    nome: "Mario",
    [key]: valore,       // chiave calcolata a runtime
};

// Shorthand property (ES6) — se variabile ha stesso nome della chiave
const nome = "Luigi";
const persona = { nome };  // { nome: "Luigi" }
```

## `this`

`this` in JS è **dinamico**, non lessicale: dipende da **come** la funzione viene chiamata, non **dove** è definita. Eccezione: arrow function (this lessicale).

```javascript
const obj = {
    nome: "Mario",
    normale() {
        console.log(this.nome);  // this = obj (chiamata come metodo)
    },
    arrow: () => {
        console.log(this.nome);  // this = outer scope (NON obj!)
    }
};
obj.normale();  // "Mario"
obj.arrow();    // undefined

const stacca = obj.normale;
stacca();       // undefined (this = global/window, o undefined in strict)
```

Regola: se la funzione è chiamata come **metodo** (es. `obj.metodo()`), `this` è l'oggetto. Se è chiamata **da sola** (`fn()`), `this` è global (o `undefined` in strict mode / ESM). Arrow function ereditano `this` dal contesto di definizione.

## Metodi statici utili

```javascript
const target = { a: 1, b: 2 };
const source = { b: 3, c: 4 };

// Object.assign — copia proprietà (shallow)
Object.assign(target, source);   // target = { a: 1, b: 3, c: 4 }

// Object.keys/values/entries (ES2017)
Object.keys(utente);     // ["nome", "eta"]
Object.values(utente);   // ["Mario", 25]
Object.entries(utente);  // [["nome", "Mario"], ["eta", 25]]

// Object.freeze — rende l'oggetto immutabile (shallow)
Object.freeze(obj);
obj.nuovo = 1;    // ignorato in strict mode, oppure TypeError
```

`Object.assign` fa copia **shallow**: se una proprietà è un oggetto, copia il riferimento, non l'oggetto. Per copia profonda: `structuredClone(obj)` (nativo, ES2021) o librerie (lodash `cloneDeep`).

## Destructuring e Spread/Rest

Destructuring estrae proprietà in variabili. Spread `...` espande le proprietà di un oggetto in un altro. Rest `...` in destructuring raccoglie le proprietà rimanenti.

```javascript
// Destructuring
const persona = { nome: "Mario", eta: 25, città: "Roma" };
const { nome, eta, ...resto } = persona;
console.log(nome);   // "Mario"
console.log(resto);  // { città: "Roma" }

// Con rinominazione e default
const { nome: fullName, città = "Milano" } = persona;

// Spread — clonazione shallow e merge
const clone = { ...persona };
const esteso = { ...persona, ruolo: "admin" };
const merge = { ...objA, ...objB };  // objB sovrascrive objA se chiavi duplicate
```

## Property descriptors

Ogni proprietà ha un **descrittore**: `value`, `writable`, `enumerable`, `configurable`. Si può controllare il comportamento con `Object.defineProperty`.

```javascript
const obj = {};
Object.defineProperty(obj, "nascosto", {
    value: 42,
    writable: false,     // non riassegnabile
    enumerable: false,   // invisibile a Object.keys/for-in
    configurable: false  // non eliminabile, non riconfigurabile
});
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `this` undefined in callback di metodo | `this` perso quando il metodo è passato come callback | Arrow function o `.bind(this)` |
| Spread non clona profondamente | `...` copia solo il primo livello | `structuredClone()` per copia profonda |
| `Object.keys` non trova Symbol | `.keys()` esclude Symbol | `Object.getOwnPropertySymbols()` |
| Proprietà con nomi calcolati sovrascritte | Due computed property con stessa chiave | Usa Map per chiavi non-stringa |
| `Object.freeze` non congela annidati | Freeze è shallow | Congela ricorsivamente o usa libreria |

## Best practice

- **Preferisci `{}`** a `new Object()` — più leggibile, nessuna sorpresa
- **Costante e nome breve** per oggetti: `const utente = ...` — mai riassegnare l'oggetto
- **Destructuring nei parametri** — funzione `function f({ a, b, c })` è auto-documentante
- **Spread > assign** — `{ ...a, ...b }` è più leggibile di `Object.assign({}, a, b)`
- **`Object.hasOwn(obj, prop)`** — metodo moderno (ES2022) più sicuro di `obj.hasOwnProperty()` (non protetto da oggetti che sovrascrivono il metodo)
- **Shallow copy intenzionale** — sai che stai copiando riferimenti, non valori

## Cross-reference

- [[JS + TS/Core Concepts/Classi|Classi]] — sugar syntax su oggetti e prototype
- [[JS + TS/Core Concepts/Prototype|Prototype]] — ereditarietà, catena prototipale
- [[JS + TS/Core Concepts/Funzioni|Funzioni]] — `this` dinamico, call/apply/bind
- [[JS + TS/TypeScript/Interfacce e Type|TypeScript — Interfacce]] — tipizzazione degli oggetti
