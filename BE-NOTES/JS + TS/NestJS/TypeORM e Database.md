---
topic: "NestJS — TypeORM e Database"
tags: [nestjs, typeorm, database, entity, repository, migrations, orm]
nav_prev: "[[Mappers e DTO.md]]"
nav_next: "[[Auth e Sicurezza.md]]"
---
Riferimento ufficiale: [docs.nestjs.com/techniques/database](https://docs.nestjs.com/techniques/database) | [typeorm.io](https://typeorm.io/) | [docs.nestjs.com/recipes/prisma](https://docs.nestjs.com/recipes/prisma)

NestJS si integra con TypeORM (ORM classico, Data Mapper + Active Record) e Prisma (ORM moderno, type-safe). TypeORM è l'ORM storico di NestJS; Prisma è più usato nei nuovi progetti. Entrambi supportano PostgreSQL, MySQL, SQLite, SQL Server, MongoDB.

```bash
# TypeORM
npm install @nestjs/typeorm typeorm pg
# Prisma
npm install @nestjs/prisma prisma @prisma/client
```

## TypeORM — Entity

```typescript
// user.entity.ts
import {
    Entity, PrimaryGeneratedColumn, Column,
    CreateDateColumn, UpdateDateColumn,
    OneToMany, ManyToOne, JoinColumn
} from "typeorm";
import { Exclude } from "class-transformer";

@Entity("users")  // nome tabella
export class User {
    @PrimaryGeneratedColumn("uuid")
    id: string;

    @Column({ length: 100 })
    nome: string;

    @Column({ unique: true })
    email: string;

    @Column()
    @Exclude()  // nascosto nella response
    password: string;

    @Column({ type: "enum", enum: ["admin", "user"], default: "user" })
    ruolo: string;

    @OneToMany(() => Post, (post) => post.author)
    posts: Post[];

    @CreateDateColumn()
    createdAt: Date;

    @UpdateDateColumn()
    updatedAt: Date;
}
```

## TypeORM — Module setup

```typescript
// app.module.ts
import { TypeOrmModule } from "@nestjs/typeorm";

@Module({
    imports: [
        TypeOrmModule.forRoot({
            type: "postgres",
            host: process.env.DB_HOST,
            port: parseInt(process.env.DB_PORT ?? "5432"),
            username: process.env.DB_USER,
            password: process.env.DB_PASSWORD,
            database: process.env.DB_NAME,
            entities: [__dirname + "/**/*.entity{.ts,.js}"],
            synchronize: process.env.NODE_ENV !== "production",  // auto-sync schema (SOLO dev!)
            logging: process.env.NODE_ENV === "development",
        }),
        UsersModule,
    ],
})
export class AppModule {}

// users.module.ts
@Module({
    imports: [TypeOrmModule.forFeature([User])],  // registra repository User
    providers: [UsersService],
    controllers: [UsersController],
    exports: [UsersService],
})
export class UsersModule {}
```

`TypeOrmModule.forRoot` configura la connessione. `TypeOrmModule.forFeature([User])` rende disponibile `@InjectRepository(User)` nel modulo corrente. `synchronize: true` crea/aggiorna le tabelle automaticamente — comodo in dev, pericoloso in produzione (usa migrazioni).

## TypeORM — Repository Pattern

```typescript
// users.service.ts
import { Injectable } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { Repository } from "typeorm";
import { User } from "./user.entity";

@Injectable()
export class UsersService {
    constructor(
        @InjectRepository(User)
        private readonly userRepo: Repository<User>,
    ) {}

    async findAll({ page = 1, limit = 10 }: PaginationDto) {
        const [data, total] = await this.userRepo.findAndCount({
            skip: (page - 1) * limit,
            take: limit,
            order: { createdAt: "DESC" },
            relations: ["posts"],   // eager loading
        });
        return { data, total, page, limit };
    }

    async findById(id: string): Promise<User | null> {
        return this.userRepo.findOne({
            where: { id },
            relations: ["posts"],
        });
    }

    async create(dto: CreateUserDto): Promise<User> {
        const user = this.userRepo.create(dto);  // crea istanza (non salva)
        return this.userRepo.save(user);          // salva nel DB
    }

    async update(id: string, dto: UpdateUserDto): Promise<User> {
        await this.userRepo.update(id, dto);
        return this.findById(id) as Promise<User>;
    }

    async delete(id: string): Promise<void> {
        await this.userRepo.delete(id);
    }
}
```

`findAndCount` restituisce `[entities, count]` — ottimale per paginazione (una query per i dati, una per il count). `create()` prepara l'entity, `save()` la persiste. `update()` fa update diretto senza caricare l'entity.

## TypeORM — Relazioni e Query Builder

```typescript
// Relazioni
@ManyToOne(() => User, (user) => user.posts)
@JoinColumn({ name: "author_id" })  // foreign key column
author: User;

@OneToMany(() => Post, (post) => post.author)
posts: Post[];

// Query Builder — query complesse non supportate dal Repository
const users = await this.userRepo
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.posts", "post")
    .where("user.ruolo = :ruolo", { ruolo: "admin" })
    .andWhere("post.published = :pub", { pub: true })
    .orderBy("user.createdAt", "DESC")
    .skip(0)
    .take(10)
    .getMany();
```

## TypeORM — Migrations

```bash
# Config (in package.json)
"typeorm": "typeorm-ts-node-commonjs"

# Comandi
npm run typeorm migration:create ./src/migrations/Init
npm run typeorm migration:generate ./src/migrations/AddUserTable -d src/data-source.ts
npm run typeorm migration:run -d src/data-source.ts
npm run typeorm migration:revert -d src/data-source.ts
```

## Prisma con NestJS

```typescript
// prisma.service.ts
import { Injectable, OnModuleInit, OnModuleDestroy } from "@nestjs/common";
import { PrismaClient } from "@prisma/client";

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
    async onModuleInit() {
        await this.$connect();
    }

    async onModuleDestroy() {
        await this.$disconnect();
    }
}

// users.module.ts
@Module({
    providers: [PrismaService, UsersService],
    controllers: [UsersController],
})
export class UsersModule {}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `EntityMetadataNotFoundError` | Entity non registrata in forFeature() | Aggiungi a `TypeOrmModule.forFeature([Entity])` |
| `No repository for "User" was found` | Module non importa TypeOrmModule.forFeature | Aggiungi nel module della feature |
| `synchronize` droppa tabelle in produzione | sync: true in produzione | Usa migrazioni, mai sync: true in produzione |
| N+1 query con TypeORM | Relazioni caricate lazy (default) | Usa `relations: ["posts"]` nel find |
| Query runner già chiuso | Connessione chiusa prematuramente | Verifica lifecycle hooks (onModuleInit) |
| `TypeORMError: Column name is duplicated` | Due colonne con stesso nome | Usa `name` esplicito in @Column |

## Best practice

- **Mai `synchronize: true` in produzione** — usa migrazioni (perdi dati altrimenti)
- **`findAndCount` per paginazione** — evita due query separate (count + findMany)
- **Relazioni eager esplicite** — TypeORM carica relazioni solo se richieste con `relations: []`
- **Query Builder per query complesse** — join condizionali, subquery, aggregazioni
- **Soft delete** — `@DeleteDateColumn()` + `find({ withDeleted: false })` invece di delete fisico
- **Index nei campi filtro** — `@Index()` su colonne usate in WHERE frequenti
- **DTO separati dalle Entity** — mai esporre l'entity direttamente; usa mapper
- **Prisma per nuovi progetti** — type-safe nativo, schema-driven, migrazioni automatiche
- **Transazioni** — `dataSource.transaction()` per operazioni multi-tabella atomiche

## Cross-reference

- [[JS + TS/NestJS/Mappers e DTO|NestJS — Mappers e DTO]] — separazione entity/response
- [[JS + TS/NestJS/Services e Providers|NestJS — Services]] — business logic layer
- [[JS + TS/Node.js/Express/Database e ORM|Express — Database e ORM]] — Prisma in Express
- [[JS + TS/Core Concepts/Async|Async]] — async/await, transazioni, Promise.all
