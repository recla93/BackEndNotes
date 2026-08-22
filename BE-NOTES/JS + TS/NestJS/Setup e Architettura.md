---
topic: "NestJS — Setup e Architettura"
tags: [nestjs, setup, architecture, modules, decorators, backend]
nav_next: "[[Controller e Routes.md]]"
---
Riferimento ufficiale: [docs.nestjs.com](https://docs.nestjs.com/) | [github.com/nestjs/nest](https://github.com/nestjs/nest)

NestJS (2017, Kamil Mysliwiec) è un framework backend per Node.js con **architettura modulare**. Ispirato ad Angular per struttura (moduli, decoratori, DI), usa Express (default) o Fastify come HTTP engine. È lo standard de facto per backend TypeScript enterprise.

```bash
npm i -g @nestjs/cli
nest new my-project          # crea progetto
nest generate module users   # genera modulo
nest generate service users  # genera service
nest generate controller users # genera controller
```

## Architettura a moduli

```
src/
├── main.ts                  # entry point
├── app.module.ts            # root module
└── users/
    ├── users.module.ts      # modulo users
    ├── users.controller.ts  # route handler
    ├── users.service.ts     # business logic
    └── users.repository.ts  # data access
```

Ogni feature è un **modulo** che raggruppa: controller (HTTP), service (logica), provider (DI), e sub-moduli. I moduli possono importare altri moduli per riutilizzare servizi.

```typescript
// users.module.ts
import { Module } from "@nestjs/common";
import { UsersController } from "./users.controller";
import { UsersService } from "./users.service";

@Module({
    controllers: [UsersController],
    providers: [UsersService],
    exports: [UsersService],  // disponibile per moduli che importano UsersModule
})
export class UsersModule {}
```

## Decoratori chiave

```typescript
// Controller
@Controller("users")          // path base: /api/users
export class UsersController {
    constructor(private readonly usersService: UsersService) {}

    @Get(":id")               // GET /api/users/:id
    @HttpCode(200)
    findOne(@Param("id") id: string) {
        return this.usersService.findOne(id);
    }
}

// Service — registrato come provider
@Injectable()
export class UsersService {
    constructor(
        @InjectRepository(User) private repo: Repository<User>,  // TypeORM
        @Inject(CACHE_MANAGER) private cache: Cache,              // custom provider
    ) {}
}

// Guard — auth
@Injectable()
export class AuthGuard implements CanActivate {
    canActivate(context: ExecutionContext): boolean {
        const request = context.switchToHttp().getRequest();
        return validateToken(request.headers.authorization);
    }
}
```

## main.ts — entry point

```typescript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { ValidationPipe } from "@nestjs/common";

async function bootstrap() {
    const app = await NestFactory.create(AppModule);

    // Pipes globali
    app.useGlobalPipes(
        new ValidationPipe({
            whitelist: true,           // rimuove campi non dichiarati nel DTO
            forbidNonWhitelisted: true, // errore se ci sono campi extra
            transform: true,           // converte tipi (es. string → number)
        })
    );

    app.enableCors({ origin: process.env.CORS_ORIGIN });
    app.setGlobalPrefix("api/v1");

    await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

`ValidationPipe` con `whitelist: true` è la difesa principale contro mass assignment: qualsiasi proprietà extra nel body viene rimossa automaticamente. `transform` converte i tipi (es. `"123"` → `123` per `@Param("id") id: number`).

## Lifecycle di una richiesta

```
Request → Guard (auth) → Interceptor (logging/transform) → Pipe (validation) → Controller → Service → Interceptor (response mapping) → Response
```

L'ordine è: **Guard** (prima di tutto, autorizzazione) → **Interceptor** (pre-processing, logging) → **Pipe** (validazione input) → **Handler** (controller) → **Interceptor** (post-processing, response mapping) → **Exception Filter** (se errore).

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Nest can't resolve dependencies` | Provider non registrato nel modulo | Aggiungi a `providers: []` o `exports: []` |
| `Circular dependency detected` | Modulo A importa B, B importa A | Usa `forwardRef(() => Module)` |
| `Cannot find module` | CLI genera file ma non aggiorna module | Registra manualmente controller/service nel module |
| Provider sovrascritto inaspettatamente | Provider con stesso token in moduli diversi | Usa `@Injectable()` con scopo esplicito |
| `ValidationPipe` non applicato | Pipe non registrato globalmente | `app.useGlobalPipes(new ValidationPipe())` in main.ts |

## Best practice

- **Moduli per dominio, non per ruolo tecnico** — `users/`, `orders/`, `payments/` non `controllers/`, `services/`
- **Feature module per ogni entità** — un modulo = una responsabilità di dominio
- **Shared module per feature cross-cutting** — auth, logging, database config in un modulo condiviso
- **CLI per scaffolding** — `nest generate` crea file con la struttura corretta e aggiorna il module
- **Moduli piccoli** — max 5-7 provider per modulo; se cresce, dividi in sub-moduli
- **`global: true` con cautela** — rende un modulo globale (accessibile ovunque), ma crea dipendenze implicite
- **Version tracking** — specifica versione NestJS in package.json; breaking changes tra major sono frequenti

## Cross-reference

- [[JS + TS/NestJS/Controller e Routes|NestJS — Controller e Routes]] — decoratori HTTP, parametri
- [[JS + TS/NestJS/Services e Providers|NestJS — Services e Providers]] — DI, provider scope
- [[JS + TS/NestJS/Dependency Injection|NestJS — Dependency Injection]] — custom provider, forwardRef
- [[JS + TS/TypeScript/Decoratori|TypeScript — Decoratori]] — experimentalDecorators, metadata
