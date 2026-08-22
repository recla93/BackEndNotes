---
topic: "JavaScript — panoramica e indice"
tags: [javascript, js, backend, nodejs, express, web]
---

JavaScript (1995, Eich) è un linguaggio **interpretato**, **tipizzazione dinamica debole**, **multi-paradigma** (OOP prototipale, funzionale, event-driven). È il linguaggio del web (unico runtime nativo nei browser), ma dal 2009 con Node.js è diventato un pilastro anche del backend.

Rispetto a Java:
- **Tipizzazione dinamica** — il tipo è determinato a runtime, non dichiarato
- **Ereditarietà prototipale** — non classica (anche se ES6 ha introdotto `class`, è zucchero sintattico)
- **First-class functions** — le funzioni sono valori come qualsiasi altro
- **Event loop** — modello concorrente single-thread con I/O non bloccante
- **Truthy/falsy** — valori che vengono valutati come booleani in contesti di condizione

TypeScript (2012, Anders Hejlsberg) è un **superset** di JS che aggiunge **tipizzazione statica opzionale**. Il codice TS viene compilato in JS: non esiste un runtime TS nativo. È lo standard de facto per progetti backend seri con Node.js.

## Aree

- [[JS + TS/Core Concepts/Variabili e Scope|Variabili e Scope]] — var/let/const, hoisting, scope chain, closure
- [[JS + TS/Core Concepts/Tipi|Tipi]] — primitivi, typeof, coerzione, truthy/falsy, Symbol, BigInt
- [[JS + TS/Core Concepts/Funzioni|Funzioni]] — dichiarazioni, arrow, closure, IIFE, callback, parametri
- [[JS + TS/Core Concepts/Oggetti|Oggetti]] — object literal, this, Object methods, destructuring, spread/rest
- [[JS + TS/Core Concepts/Array|Array]] — metodi funzionali: map, filter, reduce, forEach, find, some, every
- [[JS + TS/Core Concepts/Classi|Classi]] — class syntax, extends, super, campi privati, getter/setter
- [[JS + TS/Core Concepts/Moduli|Moduli]] — CommonJS (require/module.exports) vs ESM (import/export)
- [[JS + TS/Core Concepts/Async|Async]] — callback, Promise, async/await, event loop, microtask queue
- [[JS + TS/Core Concepts/Errori|Errori]] — try/catch, throw, Error types, stack trace, custom errors
- [[JS + TS/Core Concepts/Prototype|Prototype e Ereditarietà]] — catena prototipale, \_\_proto\_\_, Object.create, instanceof

## TypeScript

- [[JS + TS/TypeScript/TypeScript|TypeScript]] — setup, tsconfig, relazione con JS, benefici per backend
- [[JS + TS/TypeScript/Tipi Avanzati|Tipi Avanzati]] — any, unknown, never, void, union, intersection, literal types
- [[JS + TS/TypeScript/Generics|Generics]] — type parameter, constraints, conditional types, mapped types
- [[JS + TS/TypeScript/Utility Types|Utility Types]] — Partial, Required, Pick, Omit, Record, Exclude, Extract, NonNullable, Awaited
- [[JS + TS/TypeScript/Decoratori|Decoratori]] — class, method, property, parameter decorators (legacy e TC39 stage 3)

## Runtime e Framework

- [[JS + TS/Node.js/Express/Setup e Routing|Express — Setup e Routing]] — primo server, path/query params, route organization
- [[JS + TS/Node.js/Express/Middleware|Express — Middleware]] — catena di elaborazione, error middleware, cors, logging
- [[JS + TS/Node.js/Express/Services e Controller|Express — Services e Controller]] — pattern a strati, separazione delle responsabilità
- [[JS + TS/Node.js/Express/Mappers e DTO|Express — Mappers e DTO]] — trasformazione dati, validazione, serializzazione
- [[JS + TS/Node.js/Express/Database e ORM|Express — Database e ORM]] — Prisma, SQL/NoSQL, migrazioni, query
- [[JS + TS/Node.js/Express/Auth e Sicurezza|Express — Auth e Sicurezza]] — JWT, bcrypt, session, helmet, rate limiting
- [[JS + TS/Node.js/Express/Testing|Express — Testing]] — Jest, Supertest, mocks, test di integrazione

- [[JS + TS/NestJS/Setup e Architettura|NestJS — Setup e Architettura]] — CLI, modules, decoratori, dependency injection
- [[JS + TS/NestJS/Controller e Routes|NestJS — Controller e Routes]] — @Controller, @Get/@Post, parametri, validazione
- [[JS + TS/NestJS/Services e Providers|NestJS — Services e Providers]] — @Injectable, business logic, provider scope
- [[JS + TS/NestJS/Dependency Injection|NestJS — Dependency Injection]] — custom provider, forwardRef, injection token
- [[JS + TS/NestJS/Mappers e DTO|NestJS — Mappers e DTO]] — class-validator, class-transformer, mapping layer
- [[JS + TS/NestJS/TypeORM e Database|NestJS — TypeORM e Database]] — entity, repository, relazioni, migrazioni, query builder
- [[JS + TS/NestJS/Auth e Sicurezza|NestJS — Auth e Sicurezza]] — Guards, Passport, JWT, RBAC, rate limiting
- [[JS + TS/NestJS/Testing|NestJS — Testing]] — Jest, Test Bed, e2e testing, mocks

## Strumenti

- [[JS + TS/Strumenti/Package Manager e Linting|Package Manager, Linting e Testing]] — npm/yarn/pnpm, ESLint, Prettier, Jest, Vitest

## Convenzioni generali

### ECMAScript e versioni
JavaScript è governato dallo standard **ECMAScript (ECMA-262)**. Le versioni chiave:
- **ES5 (2009)** — JSON, strict mode, Array methods (forEach, map, filter)
- **ES6/ES2015** — let/const, arrow functions, class, Promise, modules (import/export), destructuring, template literals
- **ES2017** — async/await, Object.values/entries
- **ES2020** — optional chaining (?.), nullish coalescing (??), Promise.allSettled, globalThis
- **ES2021+** — replaceAll, logical assignment (&&=, ||=, ??=), Array.at, top-level await

### Naming conventions (non obbligatorie, ma seguite dalla comunità)
- **variabili/funzioni**: `camelCase` (`getUser`, `userName`)
- **classi/nomi di tipo TS**: `PascalCase` (`UserService`, `CreateUserDto`)
- **costanti globali**: `UPPER_SNAKE_CASE` (`MAX_RETRY_COUNT`)
- **file**: `kebab-case` o `camelCase` (dipende dal progetto), es. `user-service.ts` o `userService.ts`
- **underscore privato**: `_privato` (convenzione visiva, non enforcement)
- **Enum TS**: `PascalCase` per il nome, `UPPER_SNAKE_CASE` o `PascalCase` per i membri

### TypeScript over JS puro
Per qualsiasi progetto backend con Node.js, usa **TypeScript**:
- Catch errori a compile-time (typo, null, shape sbagliata)
- Autocompletion e refactoring molto migliori
- documentazione vivente (il tipo è la documentazione)
- Transpilazione a JS con `tsc` o bundler (es. `tsup`, `esbuild`)

Il JS puro rimane utile per: scripting veloce, progetti monouso, config file, prototipazione rapida.

### Altre convenzioni
- **Error-first callback** — pattern storico di Node.js: `function(err, result)` — oggi sostituito da Promise/async-await
- **Naming asincrono** — suffisso `Async` per funzioni che restituiscono Promise: `getUserAsync()`
