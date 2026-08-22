---
topic: "Express — Services e Controller"
tags: [nodejs, express, architecture, services, controllers, layering]
nav_prev: "[[Middleware.md]]"
nav_next: "[[Mappers e DTO.md]]"
---
Riferimento ufficiale: [expressjs.com/en/guide/routing.html](https://expressjs.com/en/guide/routing.html) | [softwareengineering.stackexchange.com/questions/324804/](https://softwareengineering.stackexchange.com/questions/324804/)

Express non impone una struttura. Per progetti che superano le 5-10 route, si adotta il **pattern a strati** (layered architecture): ogni strato ha una responsabilità specifica e comunica solo con lo strato adiacente.

```
Request → Controller → Service → Repository → Database
              ↓            ↓          ↓
           valida      business     ORM/query
           risponde    logic
```

## Controller (o Route Handler)

Il controller **gestisce la richiesta HTTP**: estrae parametri, chiama il service, restituisce la risposta. Non contiene logica di business. Non accede direttamente al database.

```typescript
// users.controller.ts
import { Request, Response, NextFunction } from "express";
import { userService } from "./users.service";

export class UserController {
    // GET /api/users
    async findAll(req: Request, res: Response, next: NextFunction) {
        try {
            const page = parseInt(req.query.page as string) || 1;
            const limit = parseInt(req.query.limit as string) || 10;
            const result = await userService.findAll({ page, limit });
            res.json(result);
        } catch (err) {
            next(err);  // passa all'error middleware
        }
    }

    // GET /api/users/:id
    async findById(req: Request, res: Response, next: NextFunction) {
        try {
            const user = await userService.findById(req.params.id);
            if (!user) return res.status(404).json({ error: "User not found" });
            res.json(user);
        } catch (err) {
            next(err);
        }
    }

    // POST /api/users
    async create(req: Request, res: Response, next: NextFunction) {
        try {
            const user = await userService.create(req.body);
            res.status(201).json(user);
        } catch (err) {
            next(err);
        }
    }
}
```

Ogni metodo del controller è un **middleware** con `(req, res, next)`. Usa `try/catch` + `next(err)` per delegare la gestione errori all'error middleware. Non lancia HTTPException direttamente ma usa errori dal service.

## Service Layer

Il service **contiene la logica di business**: validazione semantica, orchestrazione, regole di dominio. Non sa nulla di HTTP (nessun `req`, `res`, status code). Usa repository/ORM per accedere ai dati.

```typescript
// users.service.ts
import { userRepository } from "./users.repository";
import { hashPassword } from "../utils/auth";

export class UserService {
    async findAll({ page, limit }: { page: number; limit: number }) {
        const skip = (page - 1) * limit;
        const [users, total] = await userRepository.findMany({ skip, limit });

        return {
            data: users,
            meta: { page, limit, total, totalPages: Math.ceil(total / limit) }
        };
    }

    async findById(id: string) {
        const user = await userRepository.findById(id);
        if (!user) return null;
        return user;
    }

    async create(data: CreateUserDto) {
        // Business logic: email unica
        const existing = await userRepository.findByEmail(data.email);
        if (existing) {
            throw new AppError(409, "Email già in uso");
        }

        // Business logic: hash password
        const hashedPassword = await hashPassword(data.password);
        return userRepository.create({ ...data, password: hashedPassword });
    }
}
```

Il service lancia `AppError` con status code appropriato (404, 409, 400). Il controller cattura e passa all'error middleware. Il service non sa che esiste HTTP — è puramente logica applicativa.

## Repository Layer

Il repository **astrae l'accesso ai dati**. Il service lavora con interfacce, non con l'ORM direttamente. Questo permette di cambiare database senza modificare il service.

```typescript
// users.repository.ts
import { prisma } from "../lib/prisma";

export class UserRepository {
    findMany({ skip, limit }: { skip: number; limit: number }) {
        return Promise.all([
            prisma.user.findMany({ skip, take: limit }),
            prisma.user.count()
        ]);
    }

    findById(id: string) {
        return prisma.user.findUnique({ where: { id } });
    }

    findByEmail(email: string) {
        return prisma.user.findUnique({ where: { email } });
    }

    create(data: Prisma.UserCreateInput) {
        return prisma.user.create({ data });
    }
}
```

Repository è lo strato più vicino al DB. Se usi un ORM type-safe come Prisma, il repository è quasi trasparente (delega all'ORM). Se cambi ORM, cambi solo questo strato.

## Wiring tutto insieme

```typescript
// users.module.ts — assemblea le dipendenze
import { Router } from "express";
import { UserController } from "./users.controller";
import { UserService } from "./users.service";
import { UserRepository } from "./users.repository";
import { authMiddleware } from "../common/auth.middleware";

// Instantiate manually (DI semplice)
const userRepository = new UserRepository();
const userService = new UserService(userRepository);
const userController = new UserController(userService);
const router = Router();

// Route → controller method
router.get("/", userController.findAll.bind(userController));
router.get("/:id", userController.findById.bind(userController));
router.post("/", authMiddleware, userController.create.bind(userController));

export default router;
```

`.bind(userController)` preserva `this` nel controller. In alternativa, usa arrow function nei metodi del controller o un wrapper: `(req, res, next) => controller.findAll(req, res, next)`.

## Pattern alternativi

```typescript
// 1. Functional controllers (più semplice, senza classi)
// users.controller.functional.ts
export const findAll = asyncHandler(async (req: Request, res: Response) => {
    const users = await userService.findAll(req.query);
    res.json(users);
});

// 2. Dependency Injection con tsyringe
import { injectable, inject } from "tsyringe";

@injectable()
class UserService {
    constructor(@inject("UserRepository") private repo: UserRepository) {}
}
```

Functional controllers sono più semplici per progetti piccoli. Class-based con DI manuale è lo standard per progetti medi. Per progetti grandi, NestJS (con DI built-in) è la scelta migliore.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| Controller con logica di business | Controller che fa query o validazioni complesse | Sposta tutto nel service layer |
| Service che accede a req/res | Service importa Request/Response di Express | Service deve essere puro — niente dipendenze HTTP |
| Dipendenze accoppiate | Service istanzia il repository dentro | Usa Dependency Injection (costruttore o parametri) |
| `this` undefined nel controller | Metodo passato senza bind | .bind() o arrow function |
| Troppa logica nel controller | Controller con 50+ righe | Estrai in service, il controller deve essere magro |

## Best practice

- **Controller magri** — massimo 10-15 righe per metodo; estrai logica in service
- **Service puri** — nessuna dipendenza da HTTP (req, res, session, cookie)
- **Repository astratti** — interfacce per repository permettono di mockare nei test
- **Errori semantici dal service** — service lancia `AppError(409, "Email duplicata")`, controller passa a error middleware
- **Niente `console.log` nei controller o service** — usa un logger strutturato (pino, winston)
- **Single-responsibility** — un file = una classe = una responsabilità
- **Dipendenze iniettate** — mai creare dipendenze dentro il service (es. `new Repository()`); passale dall'esterno

## Cross-reference

- [[JS + TS/Node.js/Express/Middleware|Express — Middleware]] — auth, logging, error middleware
- [[JS + TS/Node.js/Express/Mappers e DTO|Express — Mappers e DTO]] — validazione, trasformazione dati
- [[JS + TS/Node.js/Express/Database e ORM|Express — Database e ORM]] — repository, prisma, query
- [[JS + TS/NestJS/Services e Providers|NestJS — Services e Providers]] — DI built-in, provider scope
