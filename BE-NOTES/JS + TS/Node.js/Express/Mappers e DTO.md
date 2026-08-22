---
topic: "Express — Mappers e DTO"
tags: [nodejs, express, dto, mappers, validation, serialization]
nav_prev: "[[Services e Controller.md]]"
nav_next: "[[Database e ORM.md]]"
---
Riferimento ufficiale: [github.com/typestack/class-validator](https://github.com/typestack/class-validator) | [zod.dev](https://zod.dev/) | [github.com/typestack/class-transformer](https://github.com/typestack/class-transformer)

I **DTO (Data Transfer Object)** definiscono la forma dei dati che entrano ed escono dall'API. I **Mapper** trasformano dati tra strati (es. Entity → DTO, Request DTO → Service params). Separare i DTO dalle entità del DB impedisce di esporre campi interni e protegge da attacchi di mass assignment.

```typescript
// Input DTO — cosa accetta l'API in ingresso
interface CreateUserDto {
    nome: string;
    email: string;
    password: string;
    ruolo?: "admin" | "user";
}

// Output DTO — cosa restituisce l'API
interface UserResponseDto {
    id: string;
    nome: string;
    email: string;
    ruolo: string;
    createdAt: string;
}
```

## Validazione con Zod

Zod è la libreria di validazione più moderna per TypeScript: dichiarativa, type-safe (il tipo è inferito dallo schema), senza decoratori.

```typescript
import { z } from "zod";

// Schema di validazione (genera anche il tipo)
const createUserSchema = z.object({
    nome: z.string().min(2, "Nome troppo corto").max(100),
    email: z.string().email("Email non valida"),
    password: z.string().min(8, "Password: minimo 8 caratteri")
        .regex(/[A-Z]/, "Deve contenere una maiuscola")
        .regex(/[0-9]/, "Deve contenere un numero"),
    ruolo: z.enum(["admin", "user"]).optional().default("user"),
});

// Tipo inferito — sempre sincronizzato con lo schema
type CreateUserDto = z.infer<typeof createUserSchema>;

// Middleware di validazione
const validate = (schema: z.ZodSchema) =>
    (req: Request, res: Response, next: NextFunction) => {
        const result = schema.safeParse(req.body);
        if (!result.success) {
            return res.status(400).json({
                error: "Validation failed",
                details: result.error.errors.map(e => ({
                    campo: e.path.join("."),
                    messaggio: e.message
                }))
            });
        }
        req.body = result.data;  // sostituisce body con i dati validati
        next();
    };

// Uso nella route
router.post("/users", validate(createUserSchema), userController.create);
```

`safeParse` restituisce un oggetto con `success: true` e `data` tipizzata, o `success: false` e `error` con i dettagli. `parse` diretto lancia un'eccezione se fallisce.

## Validazione con class-validator (NestJS style)

Se usi classi invece di interfacce (es. per compatibilità con NestJS o class-transformer):

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
```

## Mapper layer

Il mapper trasforma tra i vari formati dei dati attraverso gli strati:

```typescript
// user.mapper.ts
import { User as UserEntity } from "@prisma/client";

export class UserMapper {
    // Entity → Response DTO (pulisce campi sensibili)
    static toResponse(user: UserEntity): UserResponseDto {
        return {
            id: user.id,
            nome: user.nome,
            email: user.email,
            ruolo: user.ruolo,
            createdAt: user.createdAt.toISOString(),
        };
    }

    // Entity[] → Response DTO[]
    static toResponseList(users: UserEntity[]): UserResponseDto[] {
        return users.map(this.toResponse);
    }

    // Request DTO → Service params
    static toService(dto: CreateUserDto): CreateUserParams {
        return {
            nome: dto.nome,
            email: dto.email,
            password: dto.password, // hashing avviene nel service
            ruolo: dto.ruolo ?? "user",
        };
    }
}

// Uso nel controller
const user = await userService.create(data);
res.status(201).json(UserMapper.toResponse(user));
```

## DTO per query (paginazione, filtri)

```typescript
import { z } from "zod";

// Schema per parametri di query
const paginationSchema = z.object({
    page: z.coerce.number().int().positive().default(1),
    limit: z.coerce.number().int().min(1).max(100).default(10),
    sort: z.enum(["asc", "desc"]).default("desc"),
    search: z.string().optional(),
});

type PaginationDto = z.infer<typeof paginationSchema>;

// Middleware per validare query params
const validateQuery = (schema: z.ZodSchema) =>
    (req: Request, res: Response, next: NextFunction) => {
        const result = schema.safeParse(req.query);
        if (!result.success) {
            return res.status(400).json({
                error: "Invalid query parameters",
                details: result.error.errors
            });
        }
        req.query = result.data;
        next();
    };

// Uso
router.get("/users", validateQuery(paginationSchema), userController.findAll);
```

`z.coerce.number()` converte automaticamente string → number (utile perché i query params arrivano sempre come stringhe). `.default()` fornisce valori di default se il parametro è omesso.

## Mass assignment protection

Mai passare `req.body` direttamente a una query di creazione/aggiornamento — un utente malevolo potrebbe inserire campi non autorizzati (es. `{ ruolo: "admin", passwordHash: "..." }`).

```typescript
// ❌ Pericoloso — mass assignment
app.post("/users", async (req, res) => {
    const user = await prisma.user.create({ data: req.body }); // 🚫
});

// ✅ Sicuro — passi solo i campi consentiti
app.post("/users", validate(createUserSchema), async (req, res) => {
    const { nome, email, password, ruolo } = req.body; // estrazione esplicita
    const user = await userService.create({ nome, email, password, ruolo });
    res.status(201).json(UserMapper.toResponse(user));
});
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `req.body` passato direttamente a DB | Mass assignment vulnerability | Valida sempre e passa solo campi consentiti |
| DTO ed Entity identici | Duplicazione, viola DRY | Usa mapper per separare; sono due concetti diversi |
| Validazione solo in controller | Service riceve dati non validi | Valida al confine (controller) MAI fidarti degli input |
| Password in chiaro in response | Entity serializzata senza mapper | Usa mapper che esclude password, createdAt formattato |
| `z.infer` non aggiornato dopo modifica schema | Tipo manuale disallineato | Usa sempre `z.infer<typeof schema>` per il tipo |
| DTO con campi obbligatori diversi per UPDATE | PUT e PATCH hanno semantics diverse | Crea DTO separati: `UpdateUserDto` con tutti opzionali (PATCH) |

## Best practice

- **DTO separati per input e output** — mai esporre l'entità del DB direttamente
- **Un validatore per strato** — valida al confine (controller), usa tipi forti internamente
- **Mapper senza side effect** — trasforma dati, non chiama servizi o DB
- **Zod > class-validator** in Express — più leggero, type-safe nativo, functional (non servono decoratori)
- **`z.coerce` per query params** — i query params sono stringhe, convertili a number/boolean
- **DTO di UPDATE sempre parziali** — `z.object({ ... }).partial()` per PATCH
- **Non duplicare i tipi** — il tipo del DTO è l'inferenza dello schema di validazione
- **Versioning dei DTO** — se l'API cambia, crea nuovi DTO invece di modificare i vecchi (es. `CreateUserV2Dto`)

## Cross-reference

- [[JS + TS/TypeScript/Utility Types|TypeScript — Utility Types]] — Partial, Pick, Omit per DTO
- [[JS + TS/Node.js/Express/Services e Controller|Express — Services e Controller]] — layering pattern
- [[JS + TS/NestJS/Mappers e DTO|NestJS — Mappers e DTO]] — class-validator, class-transformer
- [[JS + TS/Core Concepts/Oggetti|JS — Oggetti]] — destructuring, spread per trasformazioni
