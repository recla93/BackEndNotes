---
topic: "Express — Testing"
tags: [nodejs, express, testing, jest, supertest, integration, unit]
nav_prev: "[[Auth e Sicurezza.md]]"
---
Riferimento ufficiale: [jestjs.io](https://jestjs.io/) | [github.com/ladjs/supertest](https://github.com/ladjs/supertest) | [testing-library.com/docs/jest-dom](https://testing-library.com/docs/jest-dom)

Il testing di un'API Express si divide in: **unit test** (servizi e repository isolati), **integration test** (route con DB reale o in memoria), e **e2e test** (tutta l'applicazione, DB reale). Il tool standard è Jest + Supertest per HTTP.

```bash
npm install -D jest @types/jest ts-jest supertest @types/supertest
npm install -D mongodb-memory-server  # per DB in memoria (opzionale)
```

```json
// jest.config.json
{
    "preset": "ts-jest",
    "testEnvironment": "node",
    "roots": ["<rootDir>/src"],
    "testMatch": ["**/__tests__/**/*.test.ts"],
    "clearMocks": true
}
```

## Unit test — Service Layer

Il service layer è il più facile da testare: non dipende da HTTP e le dipendenze (repository) si mockano.

```typescript
// users.service.test.ts
import { UserService } from "./users.service";
import { AppError } from "../common/app-error";

// Mock del repository
const mockUserRepo = {
    findById: jest.fn(),
    findByEmail: jest.fn(),
    create: jest.fn(),
};

const service = new UserService(mockUserRepo as any);

describe("UserService", () => {
    beforeEach(() => {
        jest.clearAllMocks();
    });

    describe("findById", () => {
        it("restituisce l'utente se trovato", async () => {
            mockUserRepo.findById.mockResolvedValue({ id: "1", nome: "Mario" });

            const user = await service.findById("1");

            expect(user).toEqual({ id: "1", nome: "Mario" });
            expect(mockUserRepo.findById).toHaveBeenCalledWith("1");
            expect(mockUserRepo.findById).toHaveBeenCalledTimes(1);
        });

        it("restituisce null se non trovato", async () => {
            mockUserRepo.findById.mockResolvedValue(null);

            const user = await service.findById("999");

            expect(user).toBeNull();
        });
    });

    describe("create", () => {
        it("lancia AppError se email già in uso", async () => {
            mockUserRepo.findByEmail.mockResolvedValue({ id: "1" });

            await expect(service.create({
                nome: "Mario",
                email: "mario@test.it",
                password: "Password1"
            })).rejects.toThrow(AppError);
        });

        it("crea utente con password hashata", async () => {
            mockUserRepo.findByEmail.mockResolvedValue(null);
            mockUserRepo.create.mockImplementation((data) => Promise.resolve(data));

            const result = await service.create({
                nome: "Mario",
                email: "mario@test.it",
                password: "Password1"
            });

            expect(result.password).not.toBe("Password1");  // hashata
            expect(result.nome).toBe("Mario");
        });
    });
});
```

## Integration test — API endpoint

Supertest permette di inviare richieste HTTP reali all'app Express senza avviare un server (usa l'app direttamente).

```typescript
// users.integration.test.ts
import request from "supertest";
import app from "../app";  // app Express esportata (senza listen!)
import { prisma } from "../lib/prisma";
import { hashPassword } from "../utils/auth";

describe("Users API", () => {
    // Setup: popola DB con dati di test
    beforeAll(async () => {
        await prisma.user.create({
            data: {
                nome: "Test User",
                email: "test@test.it",
                password: await hashPassword("Password1"),
                ruolo: "USER"
            }
        });
    });

    // Cleanup: pulisci dopo i test
    afterAll(async () => {
        await prisma.user.deleteMany();
        await prisma.$disconnect();
    });

    describe("GET /api/users/:id", () => {
        it("restituisce 200 e l'utente", async () => {
            const res = await request(app)
                .get("/api/users/test-id")
                .expect(200);

            expect(res.body).toHaveProperty("id");
            expect(res.body.nome).toBe("Test User");
        });

        it("restituisce 404 se non trovato", async () => {
            const res = await request(app)
                .get("/api/users/999")
                .expect(404);

            expect(res.body).toHaveProperty("error");
        });
    });

    describe("POST /api/users", () => {
        it("restituisce 201 per creazione valida", async () => {
            const res = await request(app)
                .post("/api/users")
                .send({
                    nome: "Nuovo",
                    email: "nuovo@test.it",
                    password: "Password1"
                })
                .expect(201);

            expect(res.body).toHaveProperty("id");
            expect(res.body.email).toBe("nuovo@test.it");
            expect(res.body).not.toHaveProperty("password");
        });

        it("restituisce 400 per body non valido", async () => {
            const res = await request(app)
                .post("/api/users")
                .send({ nome: "A" })  // nome troppo corto
                .expect(400);

            expect(res.body).toHaveProperty("error");
        });
    });
});
```

## Test DB isolation

Per test che toccano il DB, usa un database separato o un DB in memoria:

```typescript
// setup-test.ts — configurazione ambiente di test
import { PrismaClient } from "@prisma/client";
import { execSync } from "child_process";

// Usa DATABASE_URL di test (es. SQLite in memoria o PostgreSQL di test)
process.env.DATABASE_URL = "postgresql://localhost:5432/myapp_test";

const prisma = new PrismaClient();

beforeAll(async () => {
    // Esegui migrazioni per il DB di test
    execSync("npx prisma migrate deploy", { env: process.env });
});

afterAll(async () => {
    await prisma.$disconnect();
});
```

## Test di autenticazione

```typescript
describe("Protected routes", () => {
    it("restituisce 401 senza token", async () => {
        await request(app)
            .get("/api/users/me")
            .expect(401);
    });

    it("restituisce 200 con token valido", async () => {
        const token = generateToken({ userId: "test", role: "USER" });

        await request(app)
            .get("/api/users/me")
            .set("Authorization", `Bearer ${token}`)
            .expect(200);
    });

    it("restituisce 403 per ruolo non autorizzato", async () => {
        const token = generateToken({ userId: "test", role: "USER" });

        await request(app)
            .delete("/api/users/admin-action")
            .set("Authorization", `Bearer ${token}`)
            .expect(403);
    });
});
```

## Coverage e configurazione

```json
// jest.config.json con coverage
{
    "collectCoverageFrom": [
        "src/**/*.service.ts",
        "src/**/*.controller.ts",
        "!src/**/*.test.ts",
        "!src/**/index.ts"
    ],
    "coverageThreshold": {
        "global": {
            "branches": 80,
            "functions": 80,
            "lines": 80,
            "statements": 80
        }
    }
}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Jest did not exit one second after the test` | Connessione DB o timer aperti | `afterAll` con `prisma.$disconnect()` o `done()` |
| `listen EADDRINUSE` | app.listen() chiamato in test | Esporta app senza listen(), usa supertest |
| Mock non resettato tra test | `jest.clearAllMocks()` non chiamato in beforeEach | Aggiungi `clearMocks: true` in config |
| Test che passano in isolation ma falliscono in suite | Mock condivisi tra test | Resetta mock in beforeEach, non in beforeAll |
| Supertest non invia header | `.set()` non chiamato | `request(app).get("/path").set("Authorization", "Bearer ...")` |

## Best practice

- **Test per ogni strato** — unit per service (veloci), integration per API (lenti ma completi)
- **DB di test separato** — mai testare sul DB di produzione o sviluppo
- **Supertest per integration, non per unit** — per unit test usa direttamente il service con mock
- **Descrivere scenario, non implementazione** — il nome del test dice COSA fa, non COME (`restituisce 404 se utente non trovato`)
- **Una assertion per test (concettuale)** — un test verifica un comportamento; se serve più roba, scrivi più test
- **Factory per dati di test** — usa factory function invece di hardcodare dati: `buildUser({ ruolo: "ADMIN" })`
- **`clearMocks: true` in jest.config** — evita contaminazione tra test
- **Test CI** — esegui test in ogni PR, blocca merge se coverage scende sotto soglia

## Cross-reference

- [[JS + TS/Node.js/Express/Services e Controller|Express — Services e Controller]] — repository pattern testabile
- [[JS + TS/Node.js/Express/Mappers e DTO|Express — Mappers e DTO]] — validazione testabile
- [[JS + TS/NestJS/Testing|NestJS — Testing]] — Test Bed, e2e in NestJS
- [[JS + TS/Strumenti/Package Manager e Linting|Strumenti]] — Jest config, coverage
