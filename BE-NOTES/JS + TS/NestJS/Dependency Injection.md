---
topic: "NestJS — Dependency Injection"
tags: [nestjs, di, dependency-injection, providers, tokens, forward-ref]
nav_prev: "[[Services e Providers.md]]"
nav_next: "[[Mappers e DTO.md]]"
---
Riferimento ufficiale: [docs.nestjs.com/fundamentals/custom-providers](https://docs.nestjs.com/fundamentals/custom-providers) | [docs.nestjs.com/fundamentals/dynamic-modules](https://docs.nestjs.com/fundamentals/dynamic-modules)

NestJS ha un **Dependency Injection container** built-in (basato su TypeDi / tsyringe). Ogni provider registrato in un modulo viene istanziato e gestito dal container. La DI permette di disaccoppiare interfacce da implementazioni, facilitando test e manutenibilità.

Il container di NestJS è un dizionario di **token → istanza**. Quando un controller dichiara `constructor(private readonly service: UsersService)`, NestJS cerca un provider registrato con token `UsersService` (il nome della classe). Lo istanzia (se non esiste già) e lo inietta.

## Injection Token

```typescript
// Token di default (il nome della classe)
constructor(private readonly usersService: UsersService) {}

// Token stringa (per provider non-class-based)
@Inject("CACHE_OPTIONS") private readonly cacheOptions: CacheOptions;

// Token Symbol (per evitare collisioni)
export const DATABASE_CONNECTION = Symbol("DATABASE_CONNECTION");
@Inject(DATABASE_CONNECTION) private readonly db: Connection;

// Provider registrato con token stringa
@Module({
    providers: [
        {
            provide: "CACHE_OPTIONS",
            useValue: { ttl: 60, max: 100 },
        },
    ],
})
```

## forwardRef — dipendenze circolari

```typescript
// users.module.ts
@Module({
    imports: [forwardRef(() => AuthModule)],  // ← forwardRef
    providers: [UsersService],
    exports: [UsersService],
})
export class UsersModule {}

// auth.module.ts
@Module({
    imports: [forwardRef(() => UsersModule)],  // ← forwardRef
    providers: [AuthService],
})
export class AuthModule {}

// Service con forwardRef
@Injectable()
export class AuthService {
    constructor(
        @Inject(forwardRef(() => UsersService))
        private readonly usersService: UsersService,
    ) {}
}
```

`forwardRef` risolve il riferimento in modo lazy: NestJS non cerca UsersService al momento della registrazione, ma solo quando viene effettivamente istanziato AuthService. Viene creato un proxy che viene risolto dopo.

## Dynamic Modules

I moduli dinamici permettono di configurare un modulo al momento dell'import:

```typescript
// database.module.ts
@Module({})
export class DatabaseModule {
    static forRoot(options: DatabaseOptions): DynamicModule {
        return {
            module: DatabaseModule,
            providers: [
                {
                    provide: "DATABASE_OPTIONS",
                    useValue: options,
                },
                DatabaseService,
            ],
            exports: [DatabaseService],
            global: true,
        };
    }
}

// app.module.ts — uso
@Module({
    imports: [
        DatabaseModule.forRoot({
            host: process.env.DB_HOST,
            port: parseInt(process.env.DB_PORT ?? "5432"),
        }),
    ],
})
export class AppModule {}
```

## Provider injection patterns

```typescript
// 1. Classe (default)
@Module({ providers: [UsersService] })  // shorthand per { provide: UsersService, useClass: UsersService }

// 2. Valore
@Module({
    providers: [{ provide: "CONFIG", useValue: config }]
})

// 3. Factory (con dipendenze)
@Module({
    providers: [{
        provide: "DB_CONNECTION",
        useFactory: (config: ConfigService) => {
            return createConnection(config.get("DATABASE_URL"));
        },
        inject: [ConfigService],
    }]
})

// 4. Aliasing
@Module({
    providers: [{
        provide: "ALIASED_SERVICE",
        useExisting: UsersService,
    }]
})

// 5. Async factory
@Module({
    providers: [{
        provide: "ASYNC_CONNECTION",
        useFactory: async (config: ConfigService) => {
            const connection = await createConnection(config.get("DATABASE_URL"));
            return connection;
        },
        inject: [ConfigService],
    }]
})
```

## Global modules

```typescript
@Global()  // ← rende il modulo globale: provider visibili ovunque senza import
@Module({
    providers: [LoggerService],
    exports: [LoggerService],
})
export class CommonModule {}
```

I moduli globali vanno usati con parsimonia: creano dipendenze implicite (un service può iniettare un provider globale senza che il suo modulo lo importi), rendendo più difficile capire le dipendenze leggendo i moduli. Buoni candidati: logging, database connection, cache.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Nest cannot resolve dependencies` | Token non registrato | Aggiungi il provider nel module |
| `Circular dependency detected` | Modulo A importa B, B importa A | forwardRef su almeno uno dei due lati |
| `Unknown dependencies` in factory | Provider in `inject: []` non registrati | Aggiungi i provider nel module corrente |
| Provider esiste ma non viene iniettato | Scope diverso (es. provider request-scoped in singleton) | Allinea gli scope |
| `Cannot read properties of undefined` in factory | Ordine di inizializzazione errato | Verifica che i provider in `inject: []` siano disponibili |
| Provider globale non trovato | Module non decorato con `@Global()` | Aggiungi `@Global()` o importa il modulo |

## Best practice

- **Token classe per default** — usa string token solo per configurazione o servizi esterni
- **Evita dipendenze circolari** — se due moduli si importano a vicenda, probabilmente c'è un problema di design
- **Global module con parsimonia** — meglio import esplicito: rende le dipendenze visibili
- **Dynamic modules per librerie** — `forRoot()`/`forFeature()` per moduli configurabili (es. DatabaseModule, CacheModule)
- **Async factory per connessioni** — connessione DB, client HTTP, Redis — usa `useFactory: async () =>`
- **Provider testabili** — ogni provider dovrebbe essere sostituibile con un mock in test (via `overrideProvider`)
- **`@Optional()` per dipendenze non obbligatorie** — se un provider può mancare, rendilo opzionale
- **Scope coerente** — non iniettare provider request-scoped in singleton; usa `@Inject(REQUEST)` solo in provider request-scoped

## Cross-reference

- [[JS + TS/NestJS/Services e Providers|NestJS — Services e Providers]] — @Injectable, provider scope
- [[JS + TS/NestJS/Setup e Architettura|NestJS — Setup]] — moduli, architettura
- [[JS + TS/TypeScript/Decoratori|TypeScript — Decoratori]] — @Injectable, @Inject, factory
- [[JS + TS/Core Concepts/Funzioni|JS — Funzioni]] — closure, factory function pattern
