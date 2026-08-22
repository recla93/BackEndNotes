---
topic: "NestJS — Mappers e DTO"
tags: [nestjs, dto, mappers, class-validator, class-transformer, validation]
nav_prev: "[[Dependency Injection.md]]"
nav_next: "[[TypeORM e Database.md]]"
---
Riferimento ufficiale: [docs.nestjs.com/techniques/validation](https://docs.nestjs.com/techniques/validation) | [github.com/typestack/class-validator](https://github.com/typestack/class-validator) | [github.com/typestack/class-transformer](https://github.com/typestack/class-transformer)

NestJS usa **class-validator** (decoratori di validazione) e **class-transformer** (trasformazione tra classi) per i DTO. A differenza di Zod (funzionale), qui i DTO sono **classi** con decoratori — integrazione nativa con il `ValidationPipe` di NestJS.

```typescript
// create-user.dto.ts
import { IsString, IsEmail, MinLength, IsOptional, IsEnum } from "class-validator";
import { ApiProperty } from "@nestjs/swagger";

export class CreateUserDto {
    @ApiProperty({ example: "Mario Rossi" })
    @IsString()
    @MinLength(2)
    nome: string;

    @ApiProperty({ example: "mario@test.it" })
    @IsEmail()
    email: string;

    @ApiProperty({ example: "Password1" })
    @IsString()
    @MinLength(8)
    password: string;

    @ApiProperty({ enum: ["admin", "user"], default: "user" })
    @IsOptional()
    @IsEnum(["admin", "user"])
    ruolo?: string;
}
```

## ValidationPipe configurazione

```typescript
// main.ts — pipe globale
app.useGlobalPipes(
    new ValidationPipe({
        whitelist: true,               // rimuove proprietà non nel DTO
        forbidNonWhitelisted: true,    // errore se ci sono proprietà extra
        transform: true,               // converte tipi (string → number, etc.)
        transformOptions: {
            enableImplicitConversion: true,  // converte automaticamente (es. "true" → true)
        },
        disableErrorMessages: process.env.NODE_ENV === "production",
    })
);
```

`whitelist: true` è la difesa principale da mass assignment: qualsiasi proprietà non dichiarata nel DTO viene rimossa dal body. `forbidNonWhitelisted` fa sì che invece di silenziosamente rimuovere, NestJS risponda con 400 e l'elenco delle proprietà non consentite.

## Class-transformer per output

```typescript
// user-response.dto.ts
import { Exclude, Expose, Transform } from "class-transformer";

export class UserResponseDto {
    @Expose()
    id: string;

    @Expose()
    nome: string;

    @Expose()
    email: string;

    @Exclude()                          // esclude dalla serializzazione
    password: string;

    @Expose()
    @Transform(({ value }) => value.toISOString())  // trasforma Date → string
    createdAt: Date;

    @Expose({ name: "created_at" })     // rinominazione per API
    createdAtAlias: Date;
}

// Uso nel service — trasforma l'entity in DTO
import { plainToInstance } from "class-transformer";

@Injectable()
export class UsersService {
    async findOne(id: string): Promise<UserResponseDto> {
        const user = await this.usersRepository.findById(id);
        return plainToInstance(UserResponseDto, user, {
            excludeExtraneousValues: true,  // solo proprietà @Expose
        });
    }
}
```

`plainToInstance` trasforma un oggetto plain (l'entity del DB) in un'istanza del DTO. `excludeExtraneousValues: true` mantiene solo le proprietà decorate con `@Expose()`.

## Mapper manuale vs automatico

```typescript
// 1. Manuale (esplicito, più sicuro)
export class UserMapper {
    static toResponse(user: User): UserResponseDto {
        return {
            id: user.id,
            nome: user.nome,
            email: user.email,
            createdAt: user.createdAt.toISOString(),
        };
    }
}

// 2. Automatico con class-transformer (meno boilerplate)
// plainToInstance(UserResponseDto, user, { excludeExtraneousValues: true })
```

Il mapper manuale è più verboso ma: più facile da debuggare, nessuna sorpresa di serializzazione, e non dipende da `reflect-metadata` per funzionare. Quello automatico è più rapido da scrivere ma può dare comportamenti inaspettati con tipi complessi.

## DTO per PATCH (update parziale)

```typescript
import { PartialType } from "@nestjs/swagger";   // o @nestjs/mapped-types
// Oppure manuale
export class UpdateUserDto {
    @IsOptional()
    @IsString()
    @MinLength(2)
    nome?: string;

    @IsOptional()
    @IsEmail()
    email?: string;

    @IsOptional()
    @IsString()
    @MinLength(8)
    password?: string;
}

// Con mapped-types di NestJS (se usi Swagger)
export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

`PartialType` genera un DTO identico ma con tutte le proprietà opzionali. Attenzione: eredita anche i decoratori di validazione ma li rende opzionali — se il campo non è presente, passa la validazione.

## DTO per Query Parameters

```typescript
import { Type } from "class-transformer";
import { IsOptional, IsInt, Min, Max, IsString } from "class-validator";

export class PaginationDto {
    @IsOptional()
    @IsInt()
    @Min(1)
    @Type(() => Number)      // trasforma stringa → number
    page?: number = 1;

    @IsOptional()
    @IsInt()
    @Min(1)
    @Max(100)
    @Type(() => Number)
    limit?: number = 10;

    @IsOptional()
    @IsString()
    sort?: "asc" | "desc";

    @IsOptional()
    @IsString()
    search?: string;
}

// Uso
@Get()
async findAll(@Query() query: PaginationDto) {
    return this.usersService.findAll(query);
}
```

`@Type(() => Number)` è necessario perché i query params arrivano come stringhe e `class-transformer` ha bisogno di sapere a quale tipo convertirli.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| Validazione non eseguita | ValidationPipe non registrato | `app.useGlobalPipes(new ValidationPipe())` |
| DTO con campi extra passano comunque | `whitelist: false` | Imposta `whitelist: true` |
| `@Exclude` non funziona | class-transformer non chiamato | Usa `plainToInstance` con `excludeExtraneousValues: true` |
| `@Type(() => Number)` non applicato | Missing `transform: true` in ValidationPipe | Abilita `transform: true` |
| DTO per UPDATE troppo stretti | Tutti i campi obbligatori per PATCH | Usa `@IsOptional()` su ogni campo o `PartialType` |
| Decoratori duplicati in DTO estesi | Ereditarietà non gestita da class-validator | I decoratori si ereditano — controlla che siano corretti per il sottotipo |

## Best practice

- **DTO separati per ogni operazione** — `CreateUserDto`, `UpdateUserDto`, `UserResponseDto` (non riusare lo stesso)
- **`whitelist: true` sempre** — impedisce mass assignment di campi non autorizzati
- **DTO sono classi, non interfacce** — i decoratori richiedono classi per `class-validator`
- **Mapper layer dedicato** — separa la trasformazione entity ↔ DTO in un file mapper
- **`@ApiProperty` per Swagger** — documenta il DTO per OpenAPI (integra con decoratori di validazione)
- **Query DTO con `@Type()`** — per conversioni automatiche string → number/boolean nei query params
- **Factory per DTO complessi** — se un DTO ha logica di costruzione, usa un metodo factory statico
- **Non esporre ID interni** — usa DTO separati per risposte pubbliche vs amministrative

## Cross-reference

- [[JS + TS/NestJS/Controller e Routes|NestJS — Controller]] — body, query, param decorators
- [[JS + TS/NestJS/TypeORM e Database|NestJS — TypeORM e Database]] — entity separata dal DTO
- [[JS + TS/Node.js/Express/Mappers e DTO|Express — Mappers e DTO]] — pattern simile (Zod vs class-validator)
- [[JS + TS/TypeScript/Decoratori|TypeScript — Decoratori]] — decoratori di classe e proprietà
