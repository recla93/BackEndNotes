---
topic: "Errori — JavaScript"
tags: [javascript, js, base, errors, exceptions, try-catch]
nav_prev: "[[Async.md]]"
nav_next: "[[Prototype.md]]"
---
Riferimento ufficiale: [developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling)

JS gestisce gli errori con `try/catch/finally` e il tipo `Error`. A differenza di Java, non c'è obbligo di dichiarare le eccezioni lanciate (checked exceptions), e qualsiasi cosa si può lanciare (anche non-Error), ma è buona pratica lanciare solo oggetti `Error`.

```javascript
try {
    const data = JSON.parse(malformedJson);
    console.log(data.nome);
} catch (err) {
    console.error(err.message);      // stringa descrittiva
    console.error(err.stack);        // stack trace completo
} finally {
    console.log("eseguito sempre");  // cleanup: chiudere file, DB, etc.
}
```

`catch` cattura qualsiasi eccezione lanciata nel `try` (sync) o Promise rejected (se dentro async). Il blocco `finally` viene eseguito **sempre**, anche se c'è un `return` o un `throw` dentro `try`/`catch`.

## Lanciare errori

```javascript
function dividi(a, b) {
    if (b === 0) {
        throw new Error("Divisione per zero");
    }
    if (typeof a !== "number" || typeof b !== "number") {
        throw new TypeError("Gli argomenti devono essere numeri");
    }
    return a / b;
}
```

`throw` interrompe l'esecuzione corrente e propaga l'errore fino al primo `catch` nella call stack. Se non c'è nessun `catch`, il programma termina (o la Promise viene rejected).

## Tipi di Error built-in

```javascript
new Error("messaggio");            // generico
new SyntaxError("JSON non valido"); // errore di parsing
new TypeError("non è una funzione"); // tipo sbagliato
new ReferenceError("x non definita"); // variabile inesistente
new RangeError("valore fuori range"); // argomento non nel range consentito
new URIError("malformato");        // URI malformato
```

Ogni tipo specializzato è sottoclasse di `Error`. Usare il tipo giusto aiuta il debugging e il pattern matching sull'errore.

## Errori asincroni

```javascript
// Promise — usa reject
async function fetchData() {
    const res = await fetch("/api/data");
    if (!res.ok) {
        throw new Error(`HTTP ${res.status}`);  // rejected Promise
    }
    return res.json();
}

// EventEmitter (Node.js)
stream.on("error", err => {
    console.error("Stream error:", err);
});
```

`throw` dentro una funzione `async` equivale a `Promise.reject(error)`. L'errore si cattura con `.catch()` o `try/catch` in un'altra `async`. Negli stream e EventEmitter di Node.js, gli errori vanno gestiti con l'evento `error` — altrimenti il processo crasha.

## Custom Error

```javascript
class ValidationError extends Error {
    constructor(message, campo) {
        super(message);
        this.name = "ValidationError";    // default: "Error"
        this.campo = campo;
        // Mantiene stack trace corretto
        Error.captureStackTrace?.(this, this.constructor);
    }
}

try {
    throw new ValidationError("Campo richiesto", "email");
} catch (err) {
    if (err instanceof ValidationError) {
        console.log(`Campo ${err.campo}: ${err.message}`);
    }
}
```

`instanceof` permette di differenziare il tipo di errore. In TypeScript, il type narrowing funziona automaticamente dopo `instanceof`.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Cannot read property of undefined` | Accedi a proprietà su oggetto inesistente | Optional chaining: `obj?.prop` |
| Errore non catchato in Promise | Promise reject senza `.catch()` o `try/catch` in async | Aggiungi sempre catch o await dentro try |
| `JSON.parse` lancia senza catch | Parsing di stringa non JSON valida | Wrapper try/catch intorno a ogni JSON.parse |
| `Error: ENOENT` (Node.js) | File non trovato | Controlla path assoluto o esistenza file prima |
| `throw` di oggetto non-Error | `throw "errore"` o `throw { message: "..." }` | Usa sempre `throw new Error(...)` — mantiene stack trace |
| Stack trace perso in callback | Errore lanciato fuori dal contesto try/catch | Passa errore come primo argomento della callback (error-first) |

## Best practice

- **Sempre `throw new Error(...)`** — mai stringhe o oggetti plain (perdi stack trace)
- **Errori specifici** — usa classi custom o i built-in TypeError/SyntaxError per distinguere semanticamente
- **Mai ingoiare errori** — `catch(e) { /* vuoto */ }` nasconde bug; almeno logga
- **try/catch stretto** — avvolgi solo il codice che PUÒ lanciare, non tutto il file
- **finally per cleanup** — chiudi file, DB connection, stream in finally, non in try o catch
- **`instanceof` per distinguere** errori custom, non `err.name === "..."` (instanceof funziona con ereditarietà)
- **Async: sempre try/catch o .catch()** — una Promise rejected senza catcher crasha il processo in Node.js (unhandledRejection)

## Cross-reference

- [[JS + TS/Core Concepts/Async|Async]] — gestione errori asincroni, Promise.reject, unhandledRejection
- [[JS + TS/Core Concepts/Funzioni|Funzioni]] — error-first callback
- [[JS + TS/Core Concepts/Classi|Classi]] — estendere Error, custom error class
- [[JS + TS/Node.js/Express/Middleware|Express — Middleware]] — error middleware pattern
