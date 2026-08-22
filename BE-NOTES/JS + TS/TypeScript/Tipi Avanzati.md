---
topic: "Tipi Avanzati — TypeScript"
tags: [typescript, ts, types, union, intersection, unknown, never]
nav_prev: "[[TypeScript.md]]"
nav_next: "[[Generics.md]]"
---
Riferimento ufficiale: [typescriptlang.org/docs/handbook/2/everyday-types.html](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)

TypeScript estende i tipi JS con **annotazioni statiche**, **type system strutturale** (duck typing a compile-time), e un ricco sistema di tipi avanzati. Il type system è **strutturale** (non nominale come Java): due tipi con la stessa struttura sono compatibili, indipendentemente dal nome.

```typescript
// Type annotation base
let nome: string = "Mario";
let eta: number = 25;
let attivo: boolean = true;
let tags: string[] = ["js", "ts"];
let generico: any = "può essere qualsiasi cosa";  // 🚫 evita
```

## any, unknown, void, never

```typescript
// any — disabilita TUTTO il type-checking (non usare)
let x: any = 5;
x = "ciao";     // ok
x.faiQualcosa(); // ok a compile-time, forse crash a runtime

// unknown — come any ma SICURO: devi fare narrowing prima di usarlo
let y: unknown = JSON.parse('{ "a": 1 }');
// y.a               // Error: Object is of type 'unknown'
if (typeof y === "object" && y !== null && "a" in y) {
    console.log((y as { a: number }).a);  // narrowing completo
}

// void — assenza di return (funzioni che non restituiscono nulla)
function log(msg: string): void {
    console.log(msg);
}

// never — non ritorna MAI (eccezione o loop infinito)
function throwError(msg: string): never {
    throw new Error(msg);
}
```

`never` è anche il tipo di un **exhaustive check** in switch/case: se tutti i casi sono coperti, il default è `never`. Se aggiungi un nuovo valore all'union e non lo gestisci, TS segnala errore.

## Union e Intersection

```typescript
// Union — può essere UNO dei tipi
type Id = string | number;
function findById(id: Id) {
    if (typeof id === "string") {
        return id.toUpperCase();    // TS sa che qui id è string
    }
    return id.toFixed(2);           // TS sa che qui id è number
}

// Discriminated union — più comune in backend (stato di una richiesta)
type Result<T> =
    | { status: "success"; data: T }
    | { status: "error"; message: string };

function handleResult(r: Result<User>) {
    if (r.status === "success") {
        console.log(r.data.nome);       // narrowing: TS sa che è success
    } else {
        console.error(r.message);       // narrowing: TS sa che è error
    }
}

// Intersection — TUTTI i tipi combinati
type WithId = { id: string };
type Timestamps = { createdAt: Date; updatedAt: Date };
type Entity = WithId & Timestamps;
// Entity = { id: string; createdAt: Date; updatedAt: Date }
```

Le **discriminated union** con un campo letterale (`status: "success" | "error"`) permettono a TS di fare narrowing automatico: dentro il ramo `if (r.status === "success")`, TS sa che `data` esiste.

## Type Aliases e typeof

```typescript
// type alias — nome per un tipo (qualsiasi tipo)
type UserID = string;
type Callback<T> = (err: Error | null, data?: T) => void;

// typeof — estrae il tipo di una variabile (utile per costanti/config)
const config = { port: 3000, host: "localhost" };
type Config = typeof config;
// Config = { port: number; host: string }

// keyof — estrae le chiavi di un tipo come union di string literal
type UserKeys = keyof User;  // "id" | "nome" | "email"
```

`typeof` in contesto di tipo (non a runtime) estrae il tipo inferito di una variabile. `keyof` trasforma le chiavi in una union di string literal.

## Literal Types e Template Literal Types

```typescript
// String literal type
type MetodoHTTP = "GET" | "POST" | "PUT" | "DELETE";

// Numeric literal type
type Port = 3000 | 3001 | 8080;

// Boolean literal type
type Flag = true | false;

// Template literal type (TS 4.1+)
type EventName = `on${Capitalize<string>}`;
// "onChange" | "onClick" | "onSubmit" | ...

// Con chiavi reali
type EventNameKey<T> = `on${Capitalize<string & keyof T>}`;
```

## Type Narrowing

TypeScript restringe automaticamente il tipo in base a:

```typescript
function elabora(valore: string | number | Date) {
    // typeof narrowing
    if (typeof valore === "string") { ... }

    // truthy/falsy narrowing
    if (valore) { ... }  // esclude null/undefined/0/""

    // equality narrowing
    if (valore === "special") { ... }

    // in narrowing (proprietà presente?)
    if ("data" in valore) { ... }

    // instanceof narrowing
    if (valore instanceof Date) { ... }

    // discriminated union narrowing
    if (valore.status === "success") { ... }

    // type predicate narrowing (custom)
    if (isUser(valore)) { ... }
}

// Type predicate — funzione che dice a TS "questo è di tipo X"
function isUser(obj: any): obj is User {
    return obj && typeof obj.nome === "string" && typeof obj.eta === "number";
}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Type 'unknown' is not assignable to type 'X'` | Usi `unknown` senza narrowing | Fai type guard (typeof, instanceof, type predicate) |
| `Property 'x' does not exist on type 'Y'` | Accesso a proprietà non esistente | Usa type assertion, controlla l'interfaccia |
| `Argument of type 'string' is not assignable to parameter of type 'X'` | Union troppo stretta | Allarga l'union o usa narrowing |
| `TS2322: Type 'string' is not assignable to type 'never'` | Tutti i casi di union coperti — ma no | L'union si è allargata, manca un nuovo caso |
| `Object is possibly 'undefined'` | strictNullChecks acceso, accesso senza guard | Optional chaining `?.` o check esplicito |

## Best practice

- **`any` è un codice smell** — ogni `any` nasconde un bug potenziale; usa `unknown` e fai narrowing
- **Discriminated union per stati** — modella stati mutualmente esclusivi con `status` come string literal
- **`as const` per costanti** — trasforma `{ port: 3000 }` in `{ readonly port: 3000 }` con literal type
- **Type predicate per validazione** — `isUser(obj: any): obj is User` — dà tipo sicuro dopo validazione runtime
- **`satisfies` (TS 4.9+)** — controlla che un valore soddisfi un tipo senza alterarlo: `const x = { a: 1 } satisfies Record<string, number>`
- **Preferisci `interface` a `type` per oggetti** — interface ha errori migliori, merging, e extends più leggibile
- **`noUncheckedIndexedAccess`** — abilita per oggetti con indice: `obj[key]` restituisce `T | undefined`

## Cross-reference

- [[JS + TS/TypeScript/Generics|Generics]] — tipi parametrici
- [[JS + TS/TypeScript/Utility Types|Utility Types]] — Partial, Pick, Omit, Record
- [[JS + TS/Core Concepts/Tipi|JS — Tipi]] — typeof, truthy/falsy, coercizione
- [[JS + TS/NestJS/Mappers e DTO|NestJS — Mappers e DTO]] — validation decorator, DTO patterns
