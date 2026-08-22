---
topic: "Async — JavaScript"
tags: [javascript, js, async, promise, await, event-loop]
nav_prev: "[[Moduli.md]]"
nav_next: "[[Errori.md]]"
---
Riferimento ufficiale: [developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous)

JS è **single-thread** con un **event loop** che gestisce le operazioni asincrone. Le operazioni I/O (DB, HTTP, file system) non bloccano il thread principale: vengono delegate al sistema e la callback viene eseguita quando l'operazione termina. L'evoluzione: **callback → Promise → async/await**.

```javascript
// Callback pattern (stile Node.js legacy)
fs.readFile("file.txt", "utf8", (err, data) => {
    if (err) return console.error(err);
    console.log(data);
});

// Promise (ES2015)
fetch("https://api.example.com/users")
    .then(res => res.json())
    .then(data => console.log(data))
    .catch(err => console.error(err));

// async/await (ES2017) — zucchero sintattico sopra Promise
async function getUsers() {
    const res = await fetch("https://api.example.com/users");
    const data = await res.json();
    return data;
}
```

## Promise

Una Promise rappresenta un **valore futuro** (che potrebbe essere già risolto, ancora in sospeso, o fallito). Ha tre stati: `pending` (in attesa), `fulfilled` (completata con successo), `rejected` (fallita).

```javascript
// Creare una Promise
const promessa = new Promise((resolve, reject) => {
    // resolve(valore)   → passa a fulfilled
    // reject(errore)    → passa a rejected
    setTimeout(() => resolve("fatto!"), 1000);
});

// Consumare
promessa
    .then(valore => console.log(valore))     // "fatto!"
    .catch(errore => console.error(errore))  // gestione errore
    .finally(() => console.log("sempre"));   // sempre eseguito
```

Il costruttore `Promise` riceve un **executor** `(resolve, reject) => { ... }` che viene eseguito **sincronicamente** (subito). `resolve` e `reject` sono le funzioni per cambiare stato. Una volta che la Promise è settled (fulfilled o rejected), il valore non cambia più.

## Catene di Promise e metodi statici

```javascript
// Chaining — ogni .then restituisce una nuova Promise
fetch("/api/users/1")
    .then(res => res.json())
    .then(user => fetch(`/api/posts?userId=${user.id}`))
    .then(res => res.json())
    .then(posts => console.log(posts))
    .catch(err => console.error(err));  // cattura errori di TUTTA la catena

// Promise.all — attende TUTTE le Promise (fallisce se una fallisce)
const [users, posts] = await Promise.all([
    fetch("/api/users").then(r => r.json()),
    fetch("/api/posts").then(r => r.json()),
]);

// Promise.allSettled — attende TUTTE, anche se alcune falliscono
const results = await Promise.allSettled([fetch(url1), fetch(url2)]);
results.forEach(r => {
    if (r.status === "fulfilled") console.log(r.value);
    if (r.status === "rejected") console.log(r.reason);
});

// Promise.race — restituisce la prima che si risolve (o fallisce)
const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error("timeout")), 5000)
);
const data = await Promise.race([fetch(url), timeout]);

// Promise.any (ES2021) — restituisce la PRIMA che si RISOLVE, ignora reject
```

`Promise.all` fallisce veloce: se una delle Promise si rejected, l'intera Promise.all va in reject. `Promise.allSettled` aspetta tutte e restituisce lo stato di ciascuna. `Promise.race` è utile per timeout.

## async/await

`async` trasforma qualsiasi funzione in una funzione che restituisce una Promise. `await` blocca l'esecuzione **della funzione** (non del thread) fino a che la Promise non è risolta. Sotto il cofano è una Promise tradizionale con syntactic sugar del compilatore.

```javascript
async function getPost(id) {
    try {
        const res = await fetch(`/api/posts/${id}`);
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return await res.json();  // restituisce Promise<Post>
    } catch (err) {
        console.error("Errore nel fetch post:", err);
        throw err;  // ri-lancia per chi chiama
    }
}
```

L'errore va gestito con `try/catch` dentro l'`async function`. Se non lo gestisci, la Promise restituita sarà rejected (e chi chiama deve gestirla con `.catch()` o `try/catch` a sua volta).

## Event Loop

L'event loop coordina tre code: **macrotask** (setTimeout, setInterval, I/O), **microtask** (Promise.then/catch/finally, queueMicrotask), e il **task principale** (codice sincrono). Le microtask hanno priorità più alta delle macrotask.

```javascript
console.log(1);                 // sincrono — stack

setTimeout(() => console.log(2), 0);  // macrotask — coda macrotask

Promise.resolve().then(() => console.log(3));  // microtask — coda microtask

console.log(4);                 // sincrono — stack

// Output: 1, 4, 3, 2
```

Perché `3` prima di `2`? Dopo aver eseguito tutto il codice sincrono (1, 4), l'event loop svuota la coda delle **microtask** (Promise) PRIMA di prendere la prossima macrotask (setTimeout).

## Top-level await (ES2022)

Nei moduli ESM, `await` si può usare al livello più alto, senza dover racchiudere in una funzione `async`.

```javascript
// ESM modulo — top-level await
const response = await fetch("https://api.example.com/config");
const config = await response.json();
export default config;
```

Attenzione: blocca l'esecuzione del modulo e dei moduli che lo importano — usalo per inizializzazioni one-shot (config, connessione DB), non per request frequenti.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `await is only valid in async functions` | `await` usato fuori da `async` | Aggiungi `async` alla funzione o usa top-level await |
| Promise è `pending` e non si risolve mai | `resolve`/`reject` mai chiamati | Assicurati che ogni path chiami `resolve` o `reject` |
| Errore in Promise non catchato | Nessun `.catch()` o `try/catch` in async | Aggiungi sempre gestione errori |
| Fire-and-forget (Promise ignorata) | Chiami funzione async senza `await` | Usa `await` o .catch() esplicito |
| Callback hell | Callback annidate | Riscrivi con Promise chain o async/await |
| `UnhandledPromiseRejection` | Promise rejected senza catcher | `process.on('unhandledRejection', handler)` o catch globale |

## Best practice

- **async/await > then/catch** — più leggibile, stack trace migliore, debug più facile
- **try/catch sempre** nell'async function — non lasciare Promise "pendenti"
- **await in loop?** `for...of` con `await` seriale; `Promise.all` per parallelo
- **Non await inutilmente** — se due chiamate sono indipendenti, lanciale in parallelo con `Promise.all`
- **Error con throw, non return** — `throw new Error(...)` in async, non `return new Error(...)`
- **Timeout per fetch** — usa `AbortController` o `Promise.race` per timeout HTTP

## Cross-reference

- [[JS + TS/Core Concepts/Errori|Errori]] — gestione errori in async, errori non catchati
- [[JS + TS/Core Concepts/Funzioni|Funzioni]] — callback, arrow function in promise chain
- [[JS + TS/Core Concepts/Moduli|Moduli]] — top-level await, import dinamico
- [[JS + TS/Node.js/Express/Setup e Routing|Express]] — route handler async, error middleware
