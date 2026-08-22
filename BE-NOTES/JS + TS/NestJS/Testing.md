---
topic: "NestJS — Testing"
tags: [nestjs, testing, jest, e2e, test-bed, unit-test]
nav_prev: "[[Auth e Sicurezza.md]]"
---
Riferimento ufficiale: [docs.nestjs.com/fundamentals/testing](https://docs.nestjs.com/fundamentals/testing) | [docs.nestjs.com/testing](https://docs.nestjs.com/testing)

NestJS offre **Test Bed** (integrato con Jest) per isolare e testare ogni strato: unit test per service/guard, integration test per controller con Test Bed, e2e test per l'applicazione completa con supertest.

```bash
npm install -D @nestjs/testing jest supertest @types/supertest
```

```json
// package.json
{
    "scripts": {
        "test": "jest",
        "test:watch": "jest --watch",
        "test:cov": "jest --coverage",
        "test:e2e": "jest --config ./test/jest-e2e.json"
    }
}
```

## Unit test — Service

```typescript
// users.service.spec.ts
import { Test, TestingModule } from "@nestjs/testing";
import { UsersService } from "./users.service";
import { getRepositoryToken } from "@nestjs/typeorm";
import { User } from "./user.entity";
import { ConflictException } from "@nestjs/common";

describe("UsersService", () => {
    let service: UsersService;
    const mockRepo = {
        findOne: jest.fn(),
        findAndCount: jest.fn(),
        create: jest.fn(),
        save: jest.fn(),
    };

    beforeEach(async () => {
        const module: TestingModule = await Test.createTestingModule({
            providers: [
                UsersService,
                { provide: getRepositoryToken(User), useValue: mockRepo },
            ],
        }).compile();

        service = module.get<UsersService>(UsersService);
        jest.clearAllMocks();
    });

    it("restituisce utente se trovato", async () => {
        const user = { id: "1", nome: "Mario" };
        mockRepo.findOne.mockResolvedValue(user);

        const result = await service.findById("1");
        expect(result).toEqual(user);
        expect(mockRepo.findOne).toHaveBeenCalledWith({
            where: { id: "1" },
            relations: ["posts"],
        });
    });

    it("lancia ConflictException se email duplicata", async () => {
        mockRepo.findOne.mockResolvedValue({ id: "1" });

        await expect(service.create({
            nome: "Mario",
            email: "mario@test.it",
            password: "Password1"
        })).rejects.toThrow(ConflictException);
    });
});
```

`Test.createTestingModule` crea un modulo NestJS isolato con solo i provider che servono. I repository/DB reali vengono sostituiti con mock. `module.get<UsersService>(UsersService)` recupera l'istanza di UsersService dal container DI con tutte le dipendenze mockate.

## Unit test — Guard

```typescript
// roles.guard.spec.ts
import { RolesGuard } from "./roles.guard";
import { Reflector } from "@nestjs/core";
import { ExecutionContext } from "@nestjs/common";

describe("RolesGuard", () => {
    let guard: RolesGuard;
    let reflector: Reflector;
    let mockContext: Partial<ExecutionContext>;

    beforeEach(() => {
        reflector = new Reflector();
        guard = new RolesGuard(reflector);
    });

    it("permette accesso se ruolo corrisponde", () => {
        mockContext = {
            getHandler: () => ({}),
            getClass: () => ({}),
            switchToHttp: () => ({
                getRequest: () => ({ user: { ruolo: "admin" } }),
            }),
        } as ExecutionContext;

        jest.spyOn(reflector, "getAllAndOverride").mockReturnValue(["admin"]);
        expect(guard.canActivate(mockContext)).toBe(true);
    });

    it("nega accesso se ruolo non corrisponde", () => {
        mockContext = {
            getHandler: () => ({}),
            getClass: () => ({}),
            switchToHttp: () => ({
                getRequest: () => ({ user: { ruolo: "user" } }),
            }),
        } as ExecutionContext;

        jest.spyOn(reflector, "getAllAndOverride").mockReturnValue(["admin"]);
        expect(guard.canActivate(mockContext)).toBe(false);
    });
});
```

## Integration test — Controller

```typescript
// users.controller.spec.ts
import { Test, TestingModule } from "@nestjs/testing";
import { UsersController } from "./users.controller";
import { UsersService } from "./users.service";

describe("UsersController", () => {
    let controller: UsersController;
    const mockService = {
        findAll: jest.fn(),
        findById: jest.fn(),
        create: jest.fn(),
    };

    beforeEach(async () => {
        const module: TestingModule = await Test.createTestingModule({
            controllers: [UsersController],
            providers: [{ provide: UsersService, useValue: mockService }],
        }).compile();

        controller = module.get<UsersController>(UsersController);
    });

    describe("findAll", () => {
        it("restituisce lista paginata", async () => {
            const expected = { data: [{ id: "1", nome: "Mario" }], meta: {} };
            mockService.findAll.mockResolvedValue(expected);

            const result = await controller.findAll({ page: 1, limit: 10 });
            expect(result).toEqual(expected);
        });
    });

    describe("findOne", () => {
        it("restituisce utente per id", async () => {
            const user = { id: "1", nome: "Mario" };
            mockService.findById.mockResolvedValue(user);

            const result = await controller.findOne("1");
            expect(result).toEqual(user);
        });
    });
});
```

## E2E test — Applicazione completa

```typescript
// test/app.e2e-spec.ts
import { Test, TestingModule } from "@nestjs/testing";
import { INestApplication, ValidationPipe } from "@nestjs/common";
import * as request from "supertest";
import { AppModule } from "./../src/app.module";
import { PrismaService } from "./../src/prisma.service";

describe("Users (e2e)", () => {
    let app: INestApplication;
    let prisma: PrismaService;

    beforeAll(async () => {
        const moduleFixture: TestingModule = await Test.createTestingModule({
            imports: [AppModule],
        }).compile();

        app = moduleFixture.createNestApplication();
        app.useGlobalPipes(new ValidationPipe({ whitelist: true, transform: true }));
        await app.init();

        prisma = app.get(PrismaService);
        await prisma.$executeRawUnsafe(`DELETE FROM "users"`);  // pulizia
    });

    afterAll(async () => {
        await prisma.$disconnect();
        await app.close();
    });

    describe("POST /api/v1/users", () => {
        it("crea utente e restituisce 201", async () => {
            const res = await request(app.getHttpServer())
                .post("/api/v1/users")
                .send({ nome: "Mario", email: "mario@test.it", password: "Password1" })
                .expect(201);

            expect(res.body).toHaveProperty("id");
            expect(res.body.email).toBe("mario@test.it");
        });

        it("restituisce 400 per email non valida", async () => {
            const res = await request(app.getHttpServer())
                .post("/api/v1/users")
                .send({ nome: "Mario", email: "non-valida", password: "Password1" })
                .expect(400);
        });
    });
});
```

## Test DB isolation

```typescript
// Testcontainers per DB isolation (alternativa a in-memory)
import { PostgreSqlContainer, StartedPostgreSqlContainer } from "@testcontainers/postgresql";

describe("Users (e2e)", () => {
    let container: StartedPostgreSqlContainer;

    beforeAll(async () => {
        container = await new PostgreSqlContainer("postgres:16-alpine")
            .withDatabase("testdb")
            .start();

        process.env.DATABASE_URL = container.getConnectionUri();
        // Ora l'app si connette al container PostgreSQL
    }, 30000);  // timeout 30s per avviare container

    afterAll(async () => {
        await container.stop();
    });
});
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Nest can't resolve dependencies` in test | Provider non mockato | Aggiungi mock per ogni provider reale |
| `ValidationPipe` non applicato in e2e | Pipe registrato in main.ts ma non nel test | `app.useGlobalPipes(new ValidationPipe(...))` in beforeAll |
| DB test contaminato tra test | Dati non puliti dopo ogni test | `afterEach` con truncate/tabella di test |
| `app.close()` non chiamato | Test rimane in ascolto | `afterAll` con `await app.close()` |
| Supertest 401 su route protette | Token non inviato | `set("Authorization", "Bearer " + token)` |

## Best practice

- **Unit test per service** — veloci (<10ms), mockano repository, coprono tutta la logica di business
- **E2E test per flussi critici** — lenti ma testano tutto (auth, DB, response); coprono i path principali
- **Test DB isolato** — container PostgreSQL con Testcontainers o SQLite in memoria
- **`beforeEach` con `jest.clearAllMocks()`** — mai mock condivisi tra test
- **Override dei provider** — `overrideProvider(UsersService).useValue(mockService)` per casi specifici
- **Non testare il framework** — non testare che `@Get()` funzioni; testa la tua logica
- **Coverage minimo 80%** — per service layer, punta a 90%+; per controller, testa i flussi principali
- **Factory per dati di test** — `buildUser()` per generare entità valide (evita hardcoding)
- **Test CI** — `npm run test:cov` e `npm run test:e2e` in pipeline; blocca se sotto soglia

## Cross-reference

- [[JS + TS/NestJS/Services e Providers|NestJS — Services]] — provider testabili via DI
- [[JS + TS/NestJS/Auth e Sicurezza|NestJS — Auth]] — mock auth in test
- [[JS + TS/Node.js/Express/Testing|Express — Testing]] — Jest, supertest pattern simili
- [[JS + TS/Strumenti/Package Manager e Linting|Strumenti]] — Jest config, coverage, npm scripts
