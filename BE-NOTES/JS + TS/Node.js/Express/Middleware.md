---
topic: "Express — Middleware"
tags: [nodejs, express, middleware, chain, cors, logging]
nav_prev: "[[Setup e Routing.md]]"
nav_next: "[[Services e Controller.md]]"
---
Riferimento ufficiale: [expressjs.com/en/guide/using-middleware.html](https://expressjs.com/en/guide/using-middleware.html)

Il **middleware** è il cuore di Express: ogni richiesta passa attraverso una catena di funzioni, in ordine, prima di arrivare all'handler finale. Ogni middleware può: modificare `req`/`res`, terminare la richiesta, o passare al prossimo con `next()`.

```typescript
// Struttura di un middleware
function myMiddleware(req: Request, res: Response, next: NextFunction) {
    // 1. Elabora (logga, modifica req, controlla auth)
    console.log(`${req.method} ${req.path}`);

    // 2. O termina la risposta
    // res.status(403).json({ error: "Access denied" });

    // 3. O passa al prossimo middleware
    next();
}

app.use(myMiddleware);  // globale — per TUTTE le route
app.get("/protetto", authMiddleware, handler);  // specifico per route
```

## Tipi di middleware

```typescript
// 1. Application-level — app.use() o app.VERBO()
app.use(cors());
app.use(express.json());

// 2. Router-level — router.use()
const router = Router();
router.use(authMiddleware);

// 3. Error-handling — 4 parametri (Express lo riconosce dalla firma)
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
    console.error(err.stack);
    res.status(500).json({ error: "Internal server error" });
});

// 4. Built-in — express.json(), express.static(), express.urlencoded()
app.use(express.static("public"));

// 5. Third-party — cors, helmet, morgan, compression
import cors from "cors";
import helmet from "helmet";
import morgan from "morgan";

app.use(cors());
app.use(helmet());
app.use(morgan("dev"));
```

## Ordine dei middleware

L'ordine in cui si montano i middleware è **critico**: Express li esegue nell'ordine di registrazione.

```typescript
// ✅ Ordine corretto
app.use(helmet());              // 1. Sicurezza (prima di tutto)
app.use(cors());                // 2. CORS
app.use(morgan("dev"));         // 3. Logging
app.use(express.json());        // 4. Parsing body
app.use("/api/users", userRoutes);  // 5. Route
app.use(errorHandler);          // 6. Error handler (ULTIMO)

// ❌ Ordine sbagliato — error handler prima delle route
// app.use(errorHandler);   // CATCHA TUTTO, le route non vengono mai eseguite
// app.use("/api/users", userRoutes);
```

L'error handler (4 parametri) deve essere **sempre l'ultimo** nella catena. Se è prima, catturerà le richieste prima che arrivino alle route.

## Middleware CORS

```typescript
import cors from "cors";

// Default — permette TUTTE le origini (NON per produzione)
app.use(cors());

// Configurato
app.use(cors({
    origin: [
        "http://localhost:4200",          // Angular dev
        "https://miosito.com"             // produzione
    ],
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"],
    credentials: true,                     // cookie/cross-origin
    maxAge: 86400                          // cache preflight per 24h
}));
```

In produzione, `origin` deve essere stretto: non usare `"*"` se usi `credentials: true` (il browser li rifiuta). Per sviluppo, specifichi l'URL del frontend.

## Error middleware pattern

```typescript
// Custom error class
class AppError extends Error {
    constructor(
        public statusCode: number,
        public message: string,
        public isOperational = true
    ) {
        super(message);
        Error.captureStackTrace?.(this, this.constructor);
    }
}

// Error middleware
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
    // Errore operazionale (atteso) vs errore di programmazione
    if (err instanceof AppError) {
        return res.status(err.statusCode).json({
            error: err.message,
            ...(process.env.NODE_ENV === "development" && { stack: err.stack })
        });
    }

    // Errore sconosciuto — logga e restituisci 500 generico
    console.error("UNEXPECTED ERROR:", err);
    res.status(500).json({
        error: "Internal server error"
    });
});
```

Separa errori **operazionali** (input sbagliato, 404, 400) da **errori di programmazione** (bug, null pointer). I primi si restituiscono al client con status code appropriato. I secondi vanno loggati e restituiti come 500 generico (mai stack trace in produzione).

## Middleware per autenticazione

```typescript
// Auth middleware — estrae e verifica token
const authMiddleware = async (req: Request, res: Response, next: NextFunction) => {
    const token = req.headers.authorization?.replace("Bearer ", "");

    if (!token) {
        return res.status(401).json({ error: "Token mancante" });
    }

    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET!);
        (req as any).user = decoded;  // aggiunge user a req
        next();
    } catch {
        res.status(401).json({ error: "Token non valido" });
    }
};
```

Aggiungere proprietà a `req` richiede un **module augmentation** in TypeScript per non perdere il type-checking:

```typescript
// types/express.d.ts
declare global {
    namespace Express {
        interface Request {
            user?: { id: string; role: string };
        }
    }
}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `res.json` chiamato due volte | next() chiamato dopo res.json() | Metti `return` prima di `res.json()` |
| Error middleware ignorato | Non chiamato con `next(err)` | `next(err)` passa al primo error middleware |
| CORS blocca richieste frontend | origin non corrisponde | Specifica origin esatto o usa `cors({ origin: true })` per sviluppo |
| `next()` senza argomento passa al prossimo | Vero — ma se passi `next(err)`, salta tutti i middleware normali | Controlla logica: `next(err)` è per errori |
| Middleware non eseguito | Montato dopo le route che dovrebbe proteggere | Metti middleware auth PRIMA delle route protette |
| `TypeError: app.use() requires middleware functions` | Passato oggetto invece di funzione | Verifica che il modulo esporti correttamente |

## Best practice

- **Ordine di montaggio: sicurezza → parsing → logging → route → error handler** — protegge prima, logga dopo, errori ultimi
- **Error middleware ULTIMO** — sempre l'ultimo `app.use()` o non funziona
- **Non mischiare logica di business nel middleware** — middleware gestisce cross-cutting (auth, logging, parsing), non business logic
- **`next(err)` per errori** — mai throware da middleware sync (Express non cattura throw in middleware async)
- **Middleware piccoli e riutilizzabili** — un middleware = una responsabilità (auth, rate-limit, logging)
- **Helmet** in produzione — imposta header HTTP di sicurezza (CSP, X-Frame-Options, HSTS)
- **rate-limiting** — `express-rate-limit` per prevenire brute force
- **Compression** in produzione — `compression` per gzip/brotli su risposte

## Cross-reference

- [[JS + TS/Node.js/Express/Setup e Routing|Express — Setup e Routing]] — base server, route handlers
- [[JS + TS/Node.js/Express/Services e Controller|Express — Services e Controller]] — pattern a strati
- [[JS + TS/Node.js/Express/Auth e Sicurezza|Express — Auth e Sicurezza]] — auth middleware, JWT, session
- [[JS + TS/Core Concepts/Errori|Errori]] — error class, stack trace, custom error
