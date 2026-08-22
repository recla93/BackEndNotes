---
topic: "NestJS — Controller e Routes"
tags: [nestjs, controller, routes, http, params, validation]
nav_prev: "[[Setup e Architettura.md]]"
nav_next: "[[Services e Providers.md]]"
---
Riferimento ufficiale: [docs.nestjs.com/controllers](https://docs.nestjs.com/controllers) | [docs.nestjs.com/custom-decorators](https://docs.nestjs.com/custom-decorators)

I controller gestiscono le richieste HTTP: ogni metodo decorato diventa un endpoint. NestJS usa **decoratori** per dichiarare metodo, path, parametri, status code, e validazione — niente configurazione esterna di routing.

```typescript
@Controller("users")           // path base: /api/users (se globalPrefix = "api/v1")
export class UsersController {
    constructor(private readonly usersService: UsersService) {}

    @Get()                     // GET /users
    async findAll(@Query() query: PaginationDto) {
        return this.usersService.findAll(query);
    }

    @Get(":id")                // GET /users/:id
    async findOne(@Param("id") id: string) {
        return this.usersService.findOne(id);
    }

    @Post()                    // POST /users
    @HttpCode(201)
    async create(@Body() dto: CreateUserDto) {
        return this.usersService.create(dto);
    }

    @Patch(":id")              // PATCH /users/:id
    async update(@Param("id") id: string, @Body() dto: UpdateUserDto) {
        return this.usersService.update(id, dto);
    }

    @Delete(":id")             // DELETE /users/:id
    @HttpCode(204)
    async delete(@Param("id") id: string) {
        await this.usersService.delete(id);
    }
}
```

## Decoratori di parametri

```typescript
@Get(":id")
async findOne(
    @Param("id") id: string,                 // path parameter
    @Query("include") include?: string,      // query parameter opzionale
    @Body() body?: SomeDto,                  // request body (POST/PATCH)
    @Headers("authorization") auth: string,  // header specifico
    @Ip() ip: string,                        // IP del client
    @Req() req: Request,                     // oggetto Request completo (evita!)
) {}
```

`@Req()` e `@Res()` esistono ma vanno evitati: legano il controller a Express e impediscono di usare interceptor/pipe. Preferisci i decoratori specifici (`@Param`, `@Body`, `@Query`, `@Headers`).

## DTO e validazione

```typescript
import { IsString, IsEmail, MinLength, IsOptional, IsEnum } from "class-validator";

export class CreateUserDto {
    @IsString()
    @MinLength(2)
    nome: string;

    @IsEmail()
    email: string;

    @IsString()
    @MinLength(8)
    password: string;

    @IsOptional()
    @IsEnum(["admin", "user"])
    ruolo?: string;
}

// Uso — validazione automatica grazie a ValidationPipe globale
@Post()
async create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
}
```

Il `ValidationPipe` globale esegue `class-validator` su ogni parametro decorato con `@Body()`, `@Query()`, `@Param()`. Se la validazione fallisce, NestJS risponde con 400 e i dettagli dell'errore automaticamente.

## Custom decorators

```typescript
// Param decorator personalizzato
import { createParamDecorator, ExecutionContext } from "@nestjs/common";

export const CurrentUser = createParamDecorator(
    (data: keyof User | undefined, ctx: ExecutionContext) => {
        const request = ctx.switchToHttp().getRequest();
        const user = request.user;
        return data ? user?.[data] : user;
    }
);

// Uso — estrae user dal request senza iniettare req
@Get("me")
async getProfile(@CurrentUser("id") userId: string) {
    return this.usersService.findById(userId);
}
```

## Controller asincroni

```typescript
@Post()
async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    return this.usersService.create(dto);
    // NestJS si aspetta Promise — gestisce automaticamente
}

// Non serve asyncHandler wrapper — NestJS gestisce async nativamente (a differenza di Express 4)
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| 404 su tutte le route | Controller non registrato nel module | Aggiungi a `controllers: []` nel module |
| `@Param("id")` è `undefined` | Nome param nel decoratore non matcha il path | `@Get(":userId")` → `@Param("userId")` |
| `ValidationPipe` non applicato | Pipe non registrato in main.ts | `app.useGlobalPipes(new ValidationPipe())` |
| Route non matcha per ordine | `@Get(":id")` prima di `@Get("me")` | Metti route fisse PRIMA di quelle con param |
| `Body` non parsato | `Content-Type` non è application/json | Imposta header correttamente dal client |

## Best practice

- **Controller magri** — massimo chiama un service; se il metodo supera 10 righe, estrai logica
- **Niente `@Req()`/`@Res()`** — accoppia a Express, impedisce interceptor/pipe
- **DTO per ogni operazione** — input separati per POST, PATCH, PUT (non riusare CreateUserDto per update)
- **Path coerenti** — risorse al plurale (`/users`, `/posts`), non `/getUser` o `/createUser`
- **Versioning via prefix** — `app.setGlobalPrefix("api/v1")` in main.ts per versionare
- **Custom decorator per user** — `@CurrentUser()` evita di accedere a `req.user` manualmente
- **Async sempre** — i controller devono restituire Promise (async); NestJS gestisce la risposta

## Cross-reference

- [[JS + TS/NestJS/Services e Providers|NestJS — Services e Providers]] — business logic
- [[JS + TS/NestJS/Mappers e DTO|NestJS — Mappers e DTO]] — validazione, transform, whitelist
- [[JS + TS/NestJS/Dependency Injection|NestJS — DI]] — provider injection nel controller
- [[JS + TS/Core Concepts/Classi|JS — Classi]] — decoratori, metodi, costruttore
