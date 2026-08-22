---
topic: "Express — Setup e Routing"
tags: [nodejs, express, routing, setup, backend, api]
nav_next: "[[Middleware.md]]"
---
Riferimento ufficiale: [expressjs.com](https://expressjs.com/) | [github.com/expressjs/express](https://github.com/expressjs/express)

Express (2010, TJ Holowaychuk) è il framework HTTP più popolare per Node.js. È **minimale** (aggiunge uno strato sopra l'HTTP module di Node.js), **non opinionato** (nessuna struttura imposta), e basato su una **catena di middleware** (ogni richiesta passa attraverso funzioni in sequenza).

```bash
npm init -y
npm install express
npm install -D typescript @types/express @types/node
```

```typescript
// Primo server
import express, { Request, Response } from "express";

const app = express();
const port = 3000;

app.get("/", (req: Request, res: Response) => {
    res.json({ messaggio: "Benvenuto!" });
});

app.listen(port, () => {
    console.log(`Server su http://localhost:${port}`);
});
```

`express()` crea l'applicazione. `app.get("/", handler)` registra un middleware per il metodo GET sul path `/`. Quando arriva una richiesta, Express esegue i middleware che matchano il path e il metodo. `res.json()` serializza in JSON e imposta `Content-Type: application/json`.

## Path e Query Parameters

```typescript
// Path parameter — parte del URL (es. /users/123)
app.get("/users/:id", (req: Request, res: Response) => {
    const id = req.params.id;        // string — accesso a ":id"
    res.json({ id });
});

// Query parameter — dopo ? (es. /users?page=1&limit=10)
app.get("/users", (req: Request, res: Response) => {
    const page = parseInt(req.query.page as string) || 1;
    const limit = parseInt(req.query.limit as string) || 10;
    res.json({ page, limit });
});

// Path + Query insieme
app.get("/users/:id/posts", (req: Request, res: Response) => {
    const { id } = req.params;
    const { sort = "desc" } = req.query;
    // /users/123/posts?sort=asc → id=123, sort="asc"
});
```

`req.params` contiene i path parameter (dichiarati con `:`). `req.query` contiene i query parameter (sempre stringhe o array di stringhe). Attenzione: in TypeScript servono type assertion perché Express tipizza `req.query` come `ParsedQs` (non come tipo specifico).

## Route organization con Router

```typescript
// users.routes.ts — router isolato
import { Router, Request, Response } from "express";

const router = Router();

router.get("/", async (req: Request, res: Response) => {
    const users = await userService.findAll();
    res.json(users);
});

router.get("/:id", async (req: Request, res: Response) => {
    const user = await userService.findById(req.params.id);
    if (!user) return res.status(404).json({ error: "User not found" });
    res.json(user);
});

router.post("/", async (req: Request, res: Response) => {
    const user = await userService.create(req.body);
    res.status(201).json(user);
});

export default router;

// app.ts — montaggio
import userRoutes from "./routes/users.routes";
app.use("/api/users", userRoutes);  // tutti i path sono /api/users/*
```

`Router` permette di organizzare le route in file separati. Il path base ("/api/users") si monta su `app.use`, le route dentro il router sono relative ("/", "/:id"). Questo è lo standard per progetti di qualsiasi dimensione.

## Gestione body (JSON e URL-encoded)

```typescript
import express from "express";
const app = express();

// Middleware built-in per parsare il body (Express 4.16+)
app.use(express.json());                          // Content-Type: application/json
app.use(express.urlencoded({ extended: true }));   // form HTML (application/x-www-form-urlencoded)

// Ora req.body è disponibile
app.post("/users", (req: Request, res: Response) => {
    console.log(req.body);  // oggetto parsato dal JSON
    res.status(201).json(req.body);
});
```

`express.json()` parsella il body JSON e lo mette in `req.body`. Senza questo middleware, `req.body` è `undefined` per richieste POST/PUT con body. `urlencoded` serve per form HTML.

## Async handler wrapper

Express 4 non cattura errori da route async — una Promise rejected crasha il server. Serve un wrapper:

```typescript
// Async handler wrapper (da mettere in utils)
const asyncHandler = (fn: (req: Request, res: Response, next: NextFunction) => Promise<any>) =>
    (req: Request, res: Response, next: NextFunction) => {
        Promise.resolve(fn(req, res, next)).catch(next);
    };

// Uso
app.get("/users/:id", asyncHandler(async (req, res) => {
    const user = await db.users.findUnique({ where: { id: req.params.id } });
    res.json(user);
}));

// In Express 5 questo wrapper non serve più — gestisce async nativamente
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `req.body` è undefined | Middleware `express.json()` non montato | Aggiungi `app.use(express.json())` |
| Route non matcha | Path sbagliato o ordine route | Metti route specifiche PRIMA di quelle generiche |
| `Cannot set headers after they are sent` | Doppia risposta (es. res.json + res.send) | Usa return dopo res.json() o controlla con if/else |
| Promise rejection non gestita | Route async senza catch | Usa asyncHandler wrapper |
| `req.params` sempre string | Path parameter in Express è sempre string | Converti a number con `parseInt()` |
| 404 su tutte le route | Router non montato su app.use | Controlla che `app.use("/path", router)` sia attivo |

## Best practice

- **Router per dominio, non per verbo** — un file per `users.routes.ts`, uno per `posts.routes.ts`
- **Async handler wrapper sempre** — senza, le Promise rejected in route async diventano unhandled rejection
- **Path versionati** — `/api/v1/users` per permettere upgrade senza breaking change
- **Return esplicito dopo res.json()** — previene "headers already sent" (anche se return non ferma l'esecuzione, rende chiaro)
- **Validazione input** — non fidarti di `req.body`, `req.params`, `req.query` — usa un validatore (Joi, zod, class-validator)
- **Separazione delle responsabilità** — la route handler chiama un service, non contiene logica di business
- **Config via env** — porta, host, CORS origins in variabili d'ambiente, non hardcodati

## Cross-reference

- [[JS + TS/Node.js/Express/Middleware|Express — Middleware]] — catena, error middleware, CORS
- [[JS + TS/Node.js/Express/Services e Controller|Express — Services e Controller]] — pattern a strati
- [[JS + TS/Node.js/Express/Database e ORM|Express — Database e ORM]] — database integration
- [[JS + TS/Core Concepts/Async|Async]] — Promise, async/await, error handling
