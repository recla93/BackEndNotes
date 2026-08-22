---
topic: "Express — Database e ORM"
tags: [nodejs, express, database, prisma, orm, sql, migrations]
nav_prev: "[[Mappers e DTO.md]]"
nav_next: "[[Auth e Sicurezza.md]]"
---
Riferimento ufficiale: [prisma.io/docs](https://www.prisma.io/docs) | [knexjs.org](https://knexjs.org/) | [sequelize.org](https://sequelize.org/)

L'accesso al database in Node.js passa per **ORM** (Prisma, TypeORM, Sequelize) o **query builder** (Knex). Prisma è l'ORM più moderno per TypeScript: type-safe, schema-driven, con generazione automatica del client.

```bash
npm install prisma @prisma/client
npx prisma init
```

## Schema Prisma

```prisma
// prisma/schema.prisma
generator client {
    provider = "prisma-client-js"
}

datasource db {
    provider = "postgresql"   // postgresql, mysql, sqlite, sqlserver, mongodb
    url      = env("DATABASE_URL")
}

model User {
    id        String   @id @default(uuid())
    nome      String   @db.VarChar(100)
    email     String   @unique
    password  String
    ruolo     Ruolo    @default(USER)
    posts     Post[]
    createdAt DateTime @default(now())
    updatedAt DateTime @updatedAt
}

model Post {
    id        String   @id @default(uuid())
    titolo    String
    contenuto String?
    author    User     @relation(fields: [authorId], references: [id])
    authorId  String
    createdAt DateTime @default(now())
}

enum Ruolo {
    USER
    ADMIN
}
```

## Prisma Client

```typescript
import { PrismaClient } from "@prisma/client";

// Singleton — evita connessioni multiple in dev con hot reload
const prisma = new PrismaClient();

export default prisma;

// Uso nel repository
class UserRepository {
    async findById(id: string) {
        return prisma.user.findUnique({
            where: { id },
            include: { posts: true }  // eager loading
        });
    }

    async findByEmail(email: string) {
        return prisma.user.findUnique({ where: { email } });
    }

    async create(data: Prisma.UserCreateInput) {
        return prisma.user.create({ data });
    }

    async findWithPagination({ skip, limit, sort }: PaginationParams) {
        const [data, total] = await Promise.all([
            prisma.user.findMany({
                skip,
                take: limit,
                orderBy: { createdAt: sort },
                select: { id: true, nome: true, email: true, ruolo: true }
            }),
            prisma.user.count()
        ]);
        return { data, total };
    }
}
```

Prisma genera `Prisma.UserCreateInput` automaticamente dallo schema — il tipo è sempre sincronizzato col database. `select` limita i campi restituiti (utile per escludere password). `include` carica relazioni.

## Migrazioni

```bash
npx prisma migrate dev --name init         # crea migrazione e applica
npx prisma migrate deploy                   # applica migrazioni pendenti (produzione)
npx prisma db push                          # sincronizza schema senza migrazione (dev rapido)
npx prisma studio                           # UI per esplorare dati
npx prisma generate                         # rigenera client dopo modifica schema
```

## Query comuni

```typescript
// CRUD base
const user = await prisma.user.create({ data: { nome, email, password } });
const user = await prisma.user.findUnique({ where: { id } });
const user = await prisma.user.update({ where: { id }, data: { nome } });
const user = await prisma.user.delete({ where: { id } });

// Relazioni — eager loading
const posts = await prisma.post.findMany({
    include: { author: { select: { nome: true, email: true } } }
});

// Filtri complessi
const users = await prisma.user.findMany({
    where: {
        OR: [
            { nome: { contains: "Mario" } },
            { email: { startsWith: "mario" } }
        ],
        ruolo: "ADMIN",
        posts: { some: { published: true } }
    },
    orderBy: { createdAt: "desc" },
    take: 10,
    skip: 0,
});

// Transazioni
const [user, post] = await prisma.$transaction([
    prisma.user.create({ data: { nome, email, password } }),
    prisma.post.create({ data: { titolo, authorId: "..." } })
]);

// Transazione con logica custom
await prisma.$transaction(async (tx) => {
    const user = await tx.user.findUnique({ where: { id } });
    if (!user) throw new AppError(404, "User not found");
    await tx.post.deleteMany({ where: { authorId: id } });
    await tx.user.delete({ where: { id } });
});
```

## Raw queries

```typescript
// Prisma supporta raw query quando serve SQL nativo
const users = await prisma.$queryRaw<SomeType[]>`
    SELECT id, nome, email
    FROM users
    WHERE email ILIKE ${`%${search}%`}
    LIMIT 10
`;
```

Raw query è utile per: query complesse non supportate dall'ORM, report, full-text search, window functions. Per tutto il resto, usa l'ORM.

## Database connection management

```typescript
// lib/prisma.ts — gestione connessione
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

export const prisma =
    globalForPrisma.prisma ??
    new PrismaClient({ log: process.env.NODE_ENV === "development" ? ["query"] : [] });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;

// In Express, non serve disconnect esplicitamente (lo fa il processo)
```

Il pattern `globalThis` evita di creare nuove istanze Prisma a ogni hot reload in sviluppo (Next.js, nodemon).

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `PrismaClientInitializationError` | DATABASE_URL non valida o DB non raggiungibile | Controlla env e connessione DB |
| `Unique constraint failed` | Violazione vincolo UNIQUE su campo | Cattura `Prisma.PrismaClientKnownRequestError` (P2002) |
| N+1 query | Caricare relazioni in loop senza include | Usa `include` o `select` per eager loading |
| Timeout connessione | Pool esaurito (Prisma default: 10) | Aumenta `connectionLimit` in datasource |
| `Cannot destructure property 'data' of undefined` | findUnique restituisce null | Controlla se l'oggetto esiste prima di destrutturare |
| Migration pending in produzione | migrate deploy non eseguito | Esegui `prisma migrate deploy` nel deploy |

## Best practice

- **Prisma per nuovi progetti TS** — type-safe nativo, schema-driven, DX eccellente
- **Singleton del client** — un'istanza Prisma per processo (memorizzata in globalThis in dev)
- **`select` per escludere campi sensibili** — password, token mai inclusi nel select
- **Relazioni eager esplicite** — Prisma non carica relazioni automaticamente (evita N+1), devi usare `include`
- **Migrazioni in VCS** — i file in `prisma/migrations/` vanno committati
- **Transazioni per operazioni atomiche** — se modifichi più tabelle, usa `$transaction`
- **Connection pool adeguato** — Prisma default 10; per serverless o alta concorrenza, regola `connection_limit`
- **Index nel schema** — aggiungi `@@index([field])` per campi usati in filtri frequenti

## Cross-reference

- [[JS + TS/Node.js/Express/Services e Controller|Express — Services e Controller]] — repository pattern
- [[JS + TS/NestJS/TypeORM e Database|NestJS — TypeORM e Database]] — TypeORM alternativo per NestJS
- [[JS + TS/Core Concepts/Async|Async]] — await, Promise.all, transazioni
- [[JS + TS/Strumenti/Package Manager e Linting|Strumenti]] — Prisma CLI, scripts
