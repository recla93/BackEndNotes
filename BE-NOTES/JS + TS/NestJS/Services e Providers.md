---
topic: "NestJS — Services e Providers"
tags: [nestjs, services, providers, injectable, scope, business-logic]
nav_prev: "[[Controller e Routes.md]]"
nav_next: "[[Dependency Injection.md]]"
---
Riferimento ufficiale: [docs.nestjs.com/providers](https://docs.nestjs.com/providers) | [docs.nestjs.com/fundamentals/injection-scopes](https://docs.nestjs.com/fundamentals/injection-scopes)

I **provider** (decorati con `@Injectable()`) sono classi che NestJS istanzia e gestisce tramite **Dependency Injection**. Il service è il provider più comune: contiene la logica di business, è iniettabile in controller e altri service, e segue il principio di singola responsabilità.

```typescript
@Injectable()
export class UsersService {
    constructor(
        private readonly usersRepository: UsersRepository,
        private readonly mailService: MailService,
    ) {}

    async findAll(query: PaginationDto): Promise<PaginatedResult<UserResponseDto>> {
        const { data, total } = await this.usersRepository.findMany(query);
        return {
            data: data.map(UserMapper.toResponse),
            meta: { total, page: query.page, limit: query.limit }
        };
    }

    async create(dto: CreateUserDto): Promise<UserResponseDto> {
        const existing = await this.usersRepository.findByEmail(dto.email);
        if (existing) throw new ConflictException("Email già in uso");

        const user = await this.usersRepository.create({
            ...dto,
            password: await hashPassword(dto.password),
        });

        await this.mailService.sendWelcomeEmail(user.email);
        return UserMapper.toResponse(user);
    }
}
```

## Provider Scope

```typescript
// DEFAULT — singleton (una istanza per tutta l'app)
@Injectable()  // scope: Scope.DEFAULT
export class UsersService {}

// REQUEST — nuova istanza per ogni richiesta HTTP
@Injectable({ scope: Scope.REQUEST })
export class RequestService {
    // Accesso a Request (iniettata da NestJS)
    constructor(@Inject(REQUEST) private request: Request) {}
}

// TRANSIENT — nuova istanza per ogni iniezione
@Injectable({ scope: Scope.TRANSIENT })
export class TransientService {}
```

- **DEFAULT (singleton)**: unica istanza, condivisa tra tutte le richieste. Performance migliore, ma attenzione allo stato condiviso (non metterci dati di richiesta).
- **REQUEST**: nuova istanza per ogni richiesta. Può iniettare `REQUEST` per accedere ai dati HTTP. Più lento (creazione oggetto).
- **TRANSIENT**: nuova istanza ogni volta che viene iniettato in un altro provider. Raro — usato quando ogni consumer deve avere una copia isolata.

## Factory provider

```typescript
// Factory provider — creazione programmatica
@Module({
    providers: [
        {
            provide: "CACHE_OPTIONS",
            useFactory: (config: ConfigService) => ({
                ttl: config.get("CACHE_TTL"),
                max: config.get("CACHE_MAX_ITEMS"),
            }),
            inject: [ConfigService],
        },
    ],
})
export class CacheModule {}
```

## Custom provider (useValue, useClass)

```typescript
// useValue — valore costante (mock, config)
@Module({
    providers: [
        { provide: "DATABASE_NAME", useValue: "myapp_db" },
        {
            provide: UsersRepository,
            useValue: mockUsersRepository,  // per testing
        },
    ],
})

// useClass — implementazione alternativa
@Module({
    providers: [
        {
            provide: MailService,
            useClass: process.env.NODE_ENV === "production"
                ? SmtpMailService
                : ConsoleMailService,  // mock in sviluppo
        },
    ],
})
```

## Optional provider

```typescript
@Injectable()
export class LoggerService {
    constructor(
        @Optional() private readonly configService?: ConfigService
    ) {
        // Se ConfigService non è registrato, usa default
        this.logLevel = configService?.get("LOG_LEVEL") ?? "info";
    }
}
```

## Provider in altri moduli

```typescript
// users.module.ts
@Module({
    controllers: [UsersController],
    providers: [UsersService],
    exports: [UsersService],  //← essenziale: rende UsersService disponibile
})
export class UsersModule {}

// orders.module.ts
@Module({
    imports: [UsersModule],  // importa UsersModule per accedere a UsersService
    providers: [OrdersService],  // OrdersService può iniettare UsersService
})
export class OrdersModule {}
```

Se un modulo non esporta un provider, quel provider è **privato** e non accessibile da altri moduli, anche se importano il modulo.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Nest can't resolve dependencies of UsersService` | UsersRepository non registrato nel modulo | Aggiungi UsersRepository a `providers: []` |
| Provider non esportato non accessibile | Provider non in `exports: []` | Aggiungi a `exports: []` o sposta nel modulo corrente |
| `Circular dependency` | Service A inietta B, B inietta A (diretta o indiretta) | Usa `@Inject(forwardRef(() => ServiceB))` |
| `Scope.REQUEST` perde contesto | Provider request-scoped iniettato in singleton | Usa `@Injectable({ scope: Scope.DEFAULT })` |
| Configurazione non disponibile in factory | Parametri mancanti in `inject: []` | Aggiungi tutti i provider richiesti all'array `inject` |

## Best practice

- **Singleton di default** — performance migliore; usa REQUEST solo se serve stato per richiesta
- **Provider piccoli** — un service = un'entità/dominio (UsersService, OrdersService, PaymentsService)
- **Mai stato condiviso in singleton** — variabili d'istanza modificate da richieste diverse creano race condition
- **Dipendenze nel costruttore** — non creare dipendenze dentro il service (es. `new Repository()`)
- **exports esplicito** — solo i provider che altri moduli devono usare; mantieni privati quelli interni
- **Provider factory per logica condizionale** — `useFactory` + `inject` per configurazione dinamica
- **Testabilità** — un service che inietta tutto via costruttore è immediatamente testabile con mock

## Cross-reference

- [[JS + TS/NestJS/Dependency Injection|NestJS — DI]] — custom provider, forwardRef, injection token
- [[JS + TS/NestJS/Controller e Routes|NestJS — Controller]] — service injection nei controller
- [[JS + TS/Node.js/Express/Services e Controller|Express — Services e Controller]] — pattern simile senza DI
- [[JS + TS/TypeScript/Decoratori|TypeScript — Decoratori]] — @Injectable, @Inject
