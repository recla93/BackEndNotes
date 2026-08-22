---
topic: "Utility Types — TypeScript"
tags: [typescript, ts, utility-types, partial, pick, omit, record]
nav_prev: "[[Generics.md]]"
nav_next: "[[Decoratori.md]]"
---
Riferimento ufficiale: [typescriptlang.org/docs/handbook/utility-types.html](https://www.typescriptlang.org/docs/handbook/utility-types.html)

TypeScript include tipi **utility** built-in che trasformano altri tipi. Sono implementati come **mapped types** e **conditional types**. I più usati in backend: `Partial`, `Required`, `Pick`, `Omit`, `Record`, `Exclude`, `Extract`, `NonNullable`, `Readonly`, `Awaited`.

```typescript
interface User {
    id: string;
    nome: string;
    email: string;
    eta: number;
    createdAt: Date;
}
```

## Partial, Required, Readonly

```typescript
// Partial<T> — tutte le proprietà opzionali
type PartialUser = Partial<User>;
// { id?: string; nome?: string; email?: string; eta?: number; createdAt?: Date }

// Required<T> — tutte le proprietà obbligatorie
type RequiredUser = Required<PartialUser>;  // torna come User

// Readonly<T> — tutte le proprietà readonly
type ReadonlyUser = Readonly<User>;
// { readonly id: string; readonly nome: string; ... }
```

`Partial` è utilissimo nei `PATCH` (aggiornamento parziale). `Readonly` per oggetti di configurazione. `Required` per validare che un oggetto parziale abbia tutti i campi.

## Pick e Omit

```typescript
// Pick<T, K> — seleziona solo le chiavi K
type UserPublic = Pick<User, "id" | "nome">;
// { id: string; nome: string }

// Omit<T, K> — rimuove le chiavi K
type UserWithoutSensitive = Omit<User, "email">;
// { id: string; nome: string; eta: number; createdAt: Date }
```

`Pick` e `Omit` sono i mattoncini per creare DTO e proiezioni. `Omit` è più dichiarativo: "User senza email" invece di elencare le 4 proprietà che rimangono.

## Record

```typescript
// Record<K, V> — oggetto con chiavi K e valori V
type UserMap = Record<string, User>;
// { [key: string]: User }

// Con chiavi letterali
type RouteMap = Record<"/users" | "/posts", string>;
// { "/users": string; "/posts": string }

// Record vs index signature
// ❌ { [key: string]: User } — non dice niente sulle chiavi
// ✅ Record<string, User> — più leggibile
// ⚠️ Se le chiavi sono note a compile-time, preferisci tipo dedicato
```

`Record` è perfetto per mappe e dizionari. Attenzione: se le chiavi sono note a compile-time (es. enum), un tipo dedicato dà più sicurezza.

## Exclude, Extract, NonNullable

```typescript
// Exclude<T, U> — rimuove da T i tipi assegnabili a U
type T1 = Exclude<string | number | Date, string | number>;  // Date

// Extract<T, U> — estrae da T i tipi assegnabili a U
type T2 = Extract<string | number | Date, string | number>;  // string | number

// NonNullable<T> — rimuove null e undefined
type T3 = NonNullable<string | null | undefined>;  // string
```

Utili per manipolare union type. `NonNullable` è il modo rapido per pulire un tipo che può essere null.

## Awaited e ReturnType

```typescript
// Awaited<T> — estrae il tipo risolto da Promise/Thenable (TS 4.5+)
type AsyncResult = Promise<User>;
type UserType = Awaited<AsyncResult>;  // User

// ReturnType<T> — estrae il tipo restituito da una funzione
function createUser(): User { ... }
type CreateUserResult = ReturnType<typeof createUser>;  // User

// Parameters<T> — estrae i parametri di una funzione come tuple
type CreateUserParams = Parameters<typeof createUser>;  // []

// ConstructorParameters<T> — parametri del costruttore
class UserService { constructor(config: Config) {} }
type ServiceConfig = ConstructorParameters<typeof UserService>;  // [Config]
```

`Awaited` è utile per tipi derivati da funzioni asincrone. `ReturnType` e `Parameters` permettono di estrarre firme da funzioni esistenti — evita duplicazione di tipo.

## Combinations (casi reali backend)

```typescript
// Update DTO — tutte le proprietà opzionali tranne id
type UpdateUserDTO = Partial<Omit<User, "id" | "createdAt">> & { id: string };

// Response con stato
type ApiResponse<T> = {
    data: T;
    timestamp: string;
} & (
    | { success: true; error: null }
    | { success: false; error: string }
);

// Query parameters tipizzati
type PaginatedQuery = {
    page?: number;
    limit?: number;
    sort?: string;
    order?: "asc" | "desc";
};

// Pick con chiavi dinamiche
type Projected<T, K extends keyof T> = Pick<T, K>;
type UserSummary = Projected<User, "id" | "nome">;
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Type 'Pick<T, K>' is not assignable to type 'T'` | Pick ha solo alcune chiavi, T le vuole tutte | Usa Partial o cambia logica |
| `'K' is not a key of 'T'` | Chiave inesistente passata a Pick/Omit | Controlla i nomi delle proprietà |
| `Type instantiation is excessively deep` | Utility type troppo annidato | Semplifica, usa tipi intermedi |
| `Record<string, any>` troppo permissivo | Record perde informazione sulle chiavi | Usa tipo dedicato se le chiavi sono note |
| Non puoi Omit su type che non esporti | Omit richiede il type completo | Esporta l'interfaccia o usa mapped type custom |

## Best practice

- **Crea alias con nome semantico** — `type UpdateUserDTO = Partial<Omit<User, "id">>` è meglio che inlining Partial/Omit ovunque
- **`Omit` per API pubbliche** — nasconde campi interni/sensibili dai DTO di risposta
- **`Partial` per aggiornamenti** — i DTO di UPDATE devono essere parziali (PATCH, non PUT)
- **`Record<string, T>` con cautela** — se le chiavi sono note, preferisci tipo esatto con chiavi obbligatorie
- **Utility composition** — `Required<Pick<T, K>>` per forzare solo alcune proprietà come obbligatorie
- **`satisfies` (TS 4.9+)** invece di type annotation per costanti: assicura che il valore soddisfi il tipo senza cambiarlo
- **Mantieni un file `types/utilities.ts`** con utility type custom del progetto (es. `DeepPartial`, `Serializable`, `Brand`)

## Cross-reference

- [[JS + TS/TypeScript/Generics|Generics]] — mapped types, conditional types
- [[JS + TS/TypeScript/Tipi Avanzati|Tipi Avanzati]] — union, intersection, keyof
- [[JS + TS/NestJS/Mappers e DTO|NestJS — Mappers e DTO]] — uso di Partial/Omit/Pick nei DTO
- [[JS + TS/Core Concepts/Oggetti|JS — Oggetti]] — object spread, assign
