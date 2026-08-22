---
topic: "Generics — TypeScript"
tags: [typescript, ts, generics, type-parameter, constraints]
nav_prev: "[[Tipi Avanzati.md]]"
nav_next: "[[Utility Types.md]]"
---
Riferimento ufficiale: [typescriptlang.org/docs/handbook/2/generics.html](https://www.typescriptlang.org/docs/handbook/2/generics.html)

I **generics** (tipi parametrici) permettono di scrivere funzioni, classi e interfacce che funzionano con **qualsiasi tipo**, mantenendo il type-checking. Invece di usare `any` (che perde il tipo), si usa un **type parameter** che viene inferito o passato esplicitamente.

```typescript
// Senza generics — dobbiamo usare any, perdendo il tipo
function identicoSenzaGenerics(arg: any): any {
    return arg;
}

// Con generics — il tipo è preservato
function identico<T>(arg: T): T {
    return arg;
}

const risultato = identico("ciao");   // tipo: "ciao" (string literal)
const numero = identico(42);          // tipo: 42 (number literal)
```

Il type parameter `T` è come un "placeholder": TS lo inferisce dall'argomento passato. Puoi anche passarlo esplicitamente: `identico<string>("ciao")`.

## Generics con constraint

```typescript
// Constraint — T deve avere almeno queste proprietà
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

const user = { nome: "Mario", eta: 25 };
getProperty(user, "nome");   // tipo: string
getProperty(user, "eta");    // tipo: number
// getProperty(user, "email"); // Error: Argument of type '"email"' is not assignable to parameter of type '"nome" | "eta"'
```

`extends` in generics non è ereditarietà — è un **vincolo**: T deve essere assegnabile al tipo vincolato. `K extends keyof T` significa "K è una delle chiavi di T". `T[K]` è un **indexed access type**.

## Generics in interfacce e type

```typescript
// Generic interface
interface Repository<T> {
    findById(id: string): Promise<T | null>;
    findAll(): Promise<T[]>;
    create(data: Omit<T, "id">): Promise<T>;
    update(id: string, data: Partial<T>): Promise<T>;
    delete(id: string): Promise<void>;
}

// Generic type
type ApiResponse<T> = {
    data: T;
    status: number;
    message?: string;
};

// Implementazione
class UserRepository implements Repository<User> {
    async findById(id: string): Promise<User | null> {
        return db.users.findUnique({ where: { id } });
    }
    // ...
}
```

## Generics in classi

```typescript
class PaginatedResult<T> {
    constructor(
        public data: T[],
        public total: number,
        public page: number,
        public limit: number
    ) {}

    get totalPages(): number {
        return Math.ceil(this.total / this.limit);
    }

    hasNextPage(): boolean {
        return this.page < this.totalPages;
    }
}

const result = new PaginatedResult<User>(users, 100, 1, 20);
```

## Multiple type parameters

```typescript
// Pair di due tipi
function pair<A, B>(a: A, b: B): [A, B] {
    return [a, b];
}
const p = pair("id", 123);  // tipo: [string, number]

// Mappa generica con chiavi e valori tipizzati
function mapValues<T, R>(obj: { [K in keyof T]: T[K] }, fn: (value: T[keyof T]) => R): { [K in keyof T]: R } {
    const result = {} as { [K in keyof T]: R };
    for (const key in obj) {
        result[key] = fn(obj[key]);
    }
    return result;
}
```

## Conditional Types (TS 2.8+)

```typescript
// Se T è string, restituisci U, altrimenti T
type IsString<T, U> = T extends string ? U : T;

type A = IsString<string, "si">;      // "si"
type B = IsString<number, "si">;      // number

// Conditional type con infer — estrae il tipo da Promise
type Unwrap<T> = T extends Promise<infer U> ? U : T;
type C = Unwrap<Promise<string>>;     // string
type D = Unwrap<number>;              // number

// Mapped type condizionale
type Nullable<T> = {
    [K in keyof T]: T[K] | null;
};
```

## Mapped Types (TS 2.1+)

```typescript
// Rende tutte le proprietà readonly
type Readonly<T> = {
    readonly [K in keyof T]: T[K];
};

// Rende tutte le proprietà opzionali
type Optional<T> = {
    [K in keyof T]?: T[K];
};

// Trasforma i tipi
type Serialized<T> = {
    [K in keyof T]: T[K] extends Date ? string : T[K];
};

// Filtra chiavi per tipo di valore
type KeysOfType<T, V> = {
    [K in keyof T]: T[K] extends V ? K : never;
}[keyof T];

type UserStringKeys = KeysOfType<User, string>;  // "nome" | "email"
```

## Template Literal Types con Generics

```typescript
// Combinazione di template literal + generics
type ApiEndpoint<T extends string> = `/api/${T}`;
type UserEndpoint = ApiEndpoint<"users">;  // "/api/users"

// Parsing di route pattern
type ExtractParam<Path extends string> =
    Path extends `${string}:${infer Param}/${infer Rest}`
        ? Param | ExtractParam<Rest>
        : Path extends `${string}:${infer Param}`
            ? Param
            : never;

type RouteParams = ExtractParam<"/api/users/:id/posts/:postId">;
// "id" | "postId"
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Type 'T' cannot be used to index type 'X'` | Nessun constraint che garantisca l'accesso | Aggiungi `T extends { [key: string]: any }` |
| `Cannot find name 'T'` | T non dichiarato nella firma | Aggiungi `<T>` prima dei parametri |
| `Type parameter 'T' has a circular constraint` | T vincolato a se stesso indirettamente | Rivedi la gerarchia dei vincoli |
| `Excessive stack depth comparing types` | Conditional type troppo complesso/ricorsivo | Semplifica il type, riduci ricorsione |
| `Argument of type 'X' is not assignable to parameter of type 'T'` | TS non riesce a inferire T dal contesto | Passa type parameter esplicitamente |

## Best practice

- **Inferenza > esplicito** — lascia che TS inferisca T dall'argomento, non passarlo mai se non necessario
- **Constraint minimi** — il vincolo più largo possibile: `T extends { id: string }` non `T extends User`
- **`infer` per estrarre tipi** — `Unwrap<T>` con `T extends Promise<infer U> ? U : T` è più elegante di utility manuale
- **Avoid complex nested conditionals** — se un conditional type è lungo > 5 righe, estrai in type helper
- **Branded types per sicurezza** — `type UserId = string & { __brand: "UserId" }` — impedisce scambi tra tipi con stessa struttura
- **Generics nei nomi**: singola lettera (T, K, V) per casi semplici, nomi descrittivi (`TData`, `TResponse`) per complessi

## Cross-reference

- [[JS + TS/TypeScript/Utility Types|Utility Types]] — built-in basati su generics (Pick, Omit, Record)
- [[JS + TS/TypeScript/Tipi Avanzati|Tipi Avanzati]] — conditional types, mapped types
- [[JS + TS/Core Concepts/Funzioni|JS — Funzioni]] — callback generiche
- [[JS + TS/NestJS/Services e Providers|NestJS — Services]] — generics nei servizi
