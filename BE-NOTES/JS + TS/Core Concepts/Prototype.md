---
topic: "Prototype e Ereditarietà — JavaScript"
tags: [javascript, js, prototype, inheritance, oop, proto]
nav_prev: "[[Errori.md]]"
---
Riferimento ufficiale: [developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

JS ha **ereditarietà prototipale**, non classica: ogni oggetto ha un link interno (`[[Prototype]]`) a un altro oggetto chiamato **prototype**. Quando accedi a una proprietà, JS cerca prima sull'oggetto, poi risale la catena dei prototype fino a trovarla o arrivare a `null`.

```javascript
const genitore = { nome: "Mario", saluta() { return `Ciao ${this.nome}`; } };
const figlio = { nome: "Luigi" };

// Colleghiamo figlio → genitore come prototype
Object.setPrototypeOf(figlio, genitore);

console.log(figlio.saluta());    // "Ciao Luigi" — trovato su genitore!
console.log(figlio.nome);        // "Luigi" — trovato sul figlio
```

`figlio.saluta()` non è definito su `figlio`. JS cerca su `figlio.__proto__` (che è `genitore`), trova `saluta`, e lo esegue con `this = figlio` (quindi `this.nome` restituisce `"Luigi"`, non `"Mario"`). Se non fosse trovato nemmeno su `genitore`, JS salirebbe a `Object.prototype`, poi `null`.

## Prototype chain

```javascript
const arr = [1, 2, 3];
// arr → Array.prototype → Object.prototype → null

const str = "ciao";
// str → String.prototype → Object.prototype → null

function f() {}
// f → Function.prototype → Object.prototype → null
```

Quando chiami `arr.map(...)`, JS cerca `map` su `arr` (non c'è), poi su `Array.prototype` (lo trova). Ogni costruttore built-in (`Array`, `String`, `Function`, `Object`) ha un `.prototype` con i metodi condivisi.

## `.prototype` vs `[[Prototype]]` vs `__proto__`

```javascript
function Persona(nome) {
    this.nome = nome;
}
// Persona.prototype — oggetto che sarà il prototype delle istanze create con new Persona()
Persona.prototype.saluta = function() {
    return `Ciao ${this.nome}`;
};

const mario = new Persona("Mario");
// mario.__proto__ === Persona.prototype     (oggetto prototype del costruttore)
// Persona.prototype.__proto__ === Object.prototype
// mario.__proto__.__proto__ === Object.prototype
// mario.__proto__.__proto__.__proto__ === null
```

`Persona.prototype` è l'oggetto che diventa il `[[Prototype]]` (prototype nascosto) delle istanze. `__proto__` è il getter/setter deprecato (ma ancora largamente usato) per accedere a `[[Prototype]]`. Il metodo moderno è `Object.getPrototypeOf(obj)`.

## Object.create

`Object.create(proto)` crea un nuovo oggetto con `proto` come `[[Prototype]]`. Modo più diretto per usare l'ereditarietà prototipale senza funzioni costruttore.

```javascript
const animale = {
    respira() { return "respiro"; }
};

const cane = Object.create(animale);
cane.parla = function() { return "bau"; };

console.log(cane.parla());   // "bau" — su cane
console.log(cane.respira()); // "respiro" — su animale (prototype)
```

## `new` — cosa fa

```javascript
function Persona(nome) {
    // 1. Crea oggetto vuoto con this.__proto__ = Persona.prototype
    // 2. this = nuovo oggetto
    this.nome = nome;
    // 3. Return implicito di this (se non restituisci un oggetto)
}

// Equivalente manuale:
function newOperator(Costruttore, ...args) {
    const obj = Object.create(Costruttore.prototype);
    const result = Costruttore.apply(obj, args);
    return result instanceof Object ? result : obj;
}
```

## `instanceof`

`instanceof` controlla se `Costruttore.prototype` è nella catena di prototype di `obj`.

```javascript
[] instanceof Array;              // true
[] instanceof Object;             // true (Array.prototype → Object.prototype)
[] instanceof String;             // false

// instanceof non funziona tra realm diversi (iframe, vm)
```

## Ereditarietà con prototype (pre-ES6)

```javascript
function Studente(nome, matricola) {
    Persona.call(this, nome);     // chiama il costruttore genitore
    this.matricola = matricola;
}

// Collega i prototype
Studente.prototype = Object.create(Persona.prototype);
Studente.prototype.constructor = Studente;  // ripristina constructor

Studente.prototype.studia = function() {
    return `${this.nome} studia`;
};
```

Questa era la sintassi classica pre-ES6 per l'ereditarietà. Le `class` moderne (ES6) sono zucchero sintattico sopra questo meccanismo.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| Metodo non trovato su istanza | Metodo aggiunto dopo la creazione dell'istanza | Aggiungi al prototype PRIMA di creare istanze |
| `instanceof` dà false tra moduli | Due diverse versioni della stessa classe in moduli diversi | Usa duck typing o Symbol.hasInstance |
| `constructor` non punta alla classe giusta | Dopo `Object.create(parent)` senza ripristino | `Child.prototype.constructor = Child` |
| Arrow function nel prototype | Arrow non ha proprio `this` — non funziona su prototype (this dinamico) | Usa function regolare per metodi prototype |
| `__proto__` non standard | `__proto__` è deprecato (ma supportato ovunque) | Usa `Object.getPrototypeOf` / `Object.setPrototypeOf` |

## Best practice

- **Non modificare `Object.prototype`** — crea conflitti globali (tutti gli oggetti ne sono affetti)
- **`Object.create(null)`** per oggetti senza prototype (dict puri) — nessuna proprietà ereditata, mappe sicure
- **Preferisci class ES6** per modellare OOP — sintassi più chiara, meno sorprese con `this`
- **`hasOwnProperty` o `Object.hasOwn(obj, prop)`** per distinguere proprietà proprie da ereditate
- **Composizione > ereditarietà** — combina oggetti via mixin o composizione invece di catene prototype profonde (> 2 livelli)
- **`Object.freeze(Object.prototype)`** in ambienti critici — impedisce monkey patching malevolo

## Cross-reference

- [[JS + TS/Core Concepts/Classi|Classi]] — zucchero sintattico sopra prototype
- [[JS + TS/Core Concepts/Oggetti|Oggetti]] — proprietà, property descriptor, hasOwnProperty
- [[JS + TS/Core Concepts/Funzioni|Funzioni]] — funzioni costruttore, new, call/apply/bind
- [[JS + TS/TypeScript/Decoratori|TypeScript — Decoratori]] — decoratori su classi e proprietà
