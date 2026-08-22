---
topic: "Moduli — JavaScript"
tags: [javascript, js, modules, commonjs, esm, import, export]
nav_prev: "[[Classi.md]]"
nav_next: "[[Async.md]]"
---
Riferimento ufficiale: [developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)

JS ha **due sistemi di moduli** incompatibili: **CommonJS** (Node.js nativo, `require`/`module.exports`) e **ES Modules (ESM)** (standard ECMAScript, `import`/`export`). ESM è lo standard moderno, ma CommonJS è ancora dominante nell'ecosistema npm.

```javascript
// === ESM (moderno, standard) ===
// math.js
export const somma = (a, b) => a + b;
export const PI = 3.14;
export default function moltiplica(a, b) { return a * b; }

// main.js
import moltiplica, { somma, PI } from "./math.js";

// === CommonJS (Node.js legacy) ===
// math.js
const somma = (a, b) => a + b;
const PI = 3.14;
module.exports = { somma, PI };
module.exports.moltiplica = (a, b) => a * b;

// main.js
const { somma, PI } = require("./math");
```

## Differenze chiave

| Aspetto | ESM | CommonJS |
|---|---|---|
| Sintassi | `import`/`export` (statico) | `require()`/`module.exports` (dinamico) |
| Caricamento | **Statico** (analizzato prima dell'esecuzione) | **Dinamico** (eseguito al momento della chiamata) |
| Top-level await | Sì | No (solo in async function) |
| Strict mode | Sempre | Opzionale |
| `this` al top-level | `undefined` | `module.exports` |
| Static analysis | Sì — tree-shaking possibile | No |
| Cicli | Gestiti meglio (live binding) | Possono dare oggetti vuoti |
| `.mjs` / `.cjs` | `.mjs` o `"type": "module"` in package.json | `.cjs` o `"type": "commonjs"` (default) |

## ESM — Named vs Default export

```javascript
// Named export — multipli per file, importati con lo stesso nome
export const nome = "Mario";
export function saluta() { ... }
export class Utente { ... }

// Default export — uno per file, importato con qualsiasi nome
export default function() { ... }

// Importare tutto
import * as math from "./math.js";
math.somma(1, 2);

// Re-export
export { somma } from "./math.js";
export * from "./math.js";
```

Named export è preferito: il nome è obbligatorio, l'IDE trova i riferimenti, il refactoring è più sicuro. Default export permette di rinominare all'import, il che rende la ricerca dei riferimenti più difficile.

## CommonJS — module.exports vs exports

```javascript
// module.exports — l'oggetto che require restituisce
module.exports = { a: 1, b: 2 };

// exports — alias a module.exports (MAI riassegnarlo!)
exports.a = 1;    // ok — aggiunge proprietà
exports.b = 2;

// exports = { a: 1 }  // PERICOLO: rompe il riferimento a module.exports!
```

`exports` è un riferimento iniziale a `module.exports`. Se riassegni `exports`, il riferimento si rompe e `require()` restituirà un oggetto vuoto.

## Dynamic import (ES2020)

`import()` è una funzione (non una dichiarazione) che restituisce una Promise, permettendo import dinamici a runtime. Usato per: code splitting, import condizionali, lazy loading.

```javascript
// Dynamic import — carica modulo a runtime
if (condizione) {
    const modulo = await import("./feature.js");
    modulo.faiQualcosa();
}
```

## package.json e risoluzione

```json
{
    "type": "module",          // tutti i .js sono ESM (default: commonjs)
    "main": "dist/index.js",   // entry point CommonJS
    "exports": {               // entry point ESM (moderno, sovrascrive main)
        ".": "./dist/index.js",
        "./utils": "./dist/utils.js"
    }
}
```

Se `"type": "module"`, tutti i `.js` sono ESM. Per avere entrambi: file `.mjs` (ESM) e `.cjs` (CommonJS) nello stesso progetto.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `require is not defined` in ESM | Stai usando `require` in un modulo ESM | Usa `import` o cambia piano |
| `Cannot use import outside a module` | File non riconosciuto come ESM | Aggiungi `"type": "module"` o usa `.mjs` |
| `exports is not defined` in ESM | `exports` non esiste in ESM (solo CommonJS) | Usa `export` invece di `exports` |
| Ciclo di dipendenze restituisce oggetto vuoto (CJS) | CommonJS non gestisce bene i cicli | Ristruttura il codice, estrai dipendenza comune |
| `Unexpected token 'export'` | Node.js interpreta il file come CommonJS | Rinomina in `.mjs` o aggiungi `"type": "module"` |
| Import da percorso senza `./` | ESM richiede path relativi espliciti (`./file.js`, non `file.js`) | Aggiungi `./` o configura `imports` in package.json |

## Best practice

- **ESM per nuovi progetti** — è lo standard, supporta tree-shaking, static analysis, top-level await
- **Named export > default export** — meglio per refactoring e autocompletamento
- **Un solo scopo per modulo** — un file = una responsabilità (coincide con SRP)
- **Path sempre con estensione** in ESM: `import("./file.js")` non `import("./file")` — obbligatorio
- **`import()` dinamico solo se necessario** — l'import statico permette ottimizzazioni a compile-time
- **`exports` in package.json** per librerie — definisce API pubblica esplicitamente, impedisce accesso a file interni

## Cross-reference

- [[JS + TS/Core Concepts/Async|Async]] — import dinamico, top-level await
- [[JS + TS/Core Concepts/Funzioni|Funzioni]] — scope delle funzioni nei moduli
- [[JS + TS/TypeScript/TypeScript|TypeScript]] — compilazione moduli, tsconfig paths
- [[JS + TS/Strumenti/Package Manager e Linting|Package Manager]] — npm, package.json
