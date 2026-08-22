---
topic: "Classi — JavaScript"
tags: [javascript, js, base, classes, oop, extends, es6]
nav_prev: "[[Array.md]]"
nav_next: "[[Moduli.md]]"
---
Riferimento ufficiale: [developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

Le classi in JS (ES6, 2015) sono **zucchero sintattico** sopra il sistema prototipale. Non introducono un nuovo modello di OOP: sotto il cofano, le classi sono funzioni costruttore con prototype. L'ereditarietà con `extends` implementa la catena prototipale in modo più familiare per chi viene da Java/C++.

```javascript
class Persona {
    constructor(nome, eta) {
        this.nome = nome;
        this.eta = eta;
    }

    saluta() {
        return `Ciao, sono ${this.nome}`;
    }

    get èMaggiorenne() {
        return this.eta >= 18;
    }
}

const mario = new Persona("Mario", 25);
console.log(mario.saluta());         // "Ciao, sono Mario"
console.log(mario.èMaggiorenne);     // true
```

`constructor` viene eseguito automaticamente all'atto della creazione con `new`. I metodi definiti nella classe finiscono sul **prototype** (condivisi tra tutte le istanze), non su ogni singolo oggetto — risparmio di memoria rispetto a definirli nel costruttore.

## Campi privati e pubblici

```javascript
class ContoBancario {
    // Campo privato (ES2022) — # garantisce vera privacy (non accessibile via reflection)
    #saldo = 0;

    // Campo pubblico
    titolare;

    constructor(titolare, saldoIniziale = 0) {
        this.titolare = titolare;
        this.#saldo = saldoIniziale;
    }

    deposita(importo) {
        this.#saldo += importo;
    }

    get saldo() {
        return this.#saldo;
    }

    // Metodo privato
    #logOperazione(tipo) {
        console.log(`[${tipo}] ${this.titolare}`);
    }
}

const conto = new ContoBancario("Mario", 1000);
conto.deposita(500);
console.log(conto.saldo);       // 1500
// console.log(conto.#saldo);   // SyntaxError: Private field
```

I campi privati `#` sono una feature recente (ES2022) con vera privacy — non sono accessibili nemmeno via `Object.getOwnPropertyNames()`. Prima si usava la convenzione `_privato` (solo visiva) o closure.

## Getter e Setter

```javascript
class Utente {
    #nome;

    constructor(nome) { this.#nome = nome; }

    get nome() { return this.#nome; }

    set nome(valore) {
        if (!valore) throw new Error("Il nome non può essere vuoto");
        this.#nome = valore;
    }
}
```

## Ereditarietà con `extends`

```javascript
class Studente extends Persona {
    #matricola;

    constructor(nome, eta, matricola) {
        super(nome, eta);        // chiama il costruttore della superclasse (obbligatorio)
        this.#matricola = matricola;
    }

    saluta() {
        return `${super.saluta()} (matricola: ${this.#matricola})`;
    }
}
```

`extends` collega i prototype: `Studente.prototype.__proto__ === Persona.prototype`. `super()` deve essere chiamato **prima** di usare `this` nel costruttore — altrimenti ReferenceError. `super.metodo()` chiama il metodo della superclasse.

## Instanceof

`instanceof` controlla se l'oggetto ha nel suo prototype chain il `.prototype` del costruttore specificato.

```javascript
mario instanceof Persona;    // true
mario instanceof Studente;   // false (se mario è Persona, non Studente)
mario instanceof Object;     // true (tutto deriva da Object)
```

## Metodi statici

```javascript
class MathUtils {
    static somma(a, b) {
        return a + b;
    }
    static #privatoStatico() { /* ... */ }
}
MathUtils.somma(1, 2);  // 3
// MathUtils.somma chiamabile SULLA CLASSE, non sull'istanza
```

I metodi statici sono utili per factory e utility: non richiedono un'istanza.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Must call super constructor before using this` | `this` usato prima di `super()` in costruttore erede | Chiama `super()` come prima istruzione |
| `Class constructor cannot be invoked without 'new'` | Classe chiamata come funzione normale | Usa `new NomeClasse(...)` |
| `Private field '#x' must be declared in an enclosing class` | Accesso a `#` fuori dalla classe | `#` è privato per classe, non per istanza da fuori |
| Metodo non trovato su istanza | Metodo definito come arrow nel costruttore vs prototype | Definisci metodi normali (sul prototype) |
| `super()` non chiamato in erede | Ogni costruttore di sottoclasse deve chiamare `super()` | Aggiungi `super( parametri del padre )` |

## Best practice

- **Classi per oggetti con stato e comportamento** — per semplici contenitori dati, usa oggetti `{}` o interfacce TS
- **Campi privati `#`** per incapsulamento reale — `_privato` è solo convenzione
- **Getter/setter** solo se c'è logica di validazione o calcolo — non per semplice accesso
- **Composizione > ereditarietà** — preferisci composizione di oggetti a catene deep di `extends`
- **Metodi sul prototype** — evita definire metodi come arrow nel costruttore (crea una copia per istanza)
- **Classi piccole** — una classe = una responsabilità (SRP)

## Cross-reference

- [[JS + TS/Core Concepts/Prototype|Prototype e Ereditarietà]] — la macchina sotto le classi
- [[JS + TS/Core Concepts/Oggetti|Oggetti]] — object literal, this, property descriptor
- [[JS + TS/Core Concepts/Funzioni|Funzioni]] — costruttori pre-ES6
- [[JS + TS/TypeScript/Decoratori|TypeScript — Decoratori]] — decoratori di classe
