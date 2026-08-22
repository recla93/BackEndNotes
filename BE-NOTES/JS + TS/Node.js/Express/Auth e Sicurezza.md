---
topic: "Express — Auth e Sicurezza"
tags: [nodejs, express, auth, security, jwt, bcrypt, helmet]
nav_prev: "[[Database e ORM.md]]"
nav_next: "[[Testing.md]]"
---
Riferimento ufficiale: [jwt.io](https://jwt.io/) | [npmjs.com/package/bcrypt](https://www.npmjs.com/package/bcrypt) | [helmetjs.github.io](https://helmetjs.github.io/)

L'autenticazione in Node.js si basa su due pattern principali: **JWT (JSON Web Token)** — stateless, scalabile, standard per API REST — e **Session** (stateful, con store esterno). JWT è la scelta più comune per API backend.

```bash
npm install bcrypt jsonwebtoken
npm install -D @types/bcrypt @types/jsonwebtoken
```

## Password hashing con bcrypt

```typescript
import bcrypt from "bcrypt";

const SALT_ROUNDS = 12;  // 12 è il default raccomandato (10-12)

export async function hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, SALT_ROUNDS);
}

export async function comparePassword(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
}
```

`bcrypt.hash(password, saltRounds)` genera un sale casuale e applica l'algoritmo Blowfish. Più saltRounds = più sicuro ma più lento (12 ~ 250ms per hash). `bcrypt.compare` estrae il sale dall'hash e verifica — la stessa password produce hash diversi ogni volta (grazie al sale casuale).

## JWT — sign e verify

```typescript
import jwt from "jsonwebtoken";

const JWT_SECRET = process.env.JWT_SECRET!;       // almeno 256 bit
const JWT_EXPIRES_IN = "7d";

export function generateToken(payload: { userId: string; role: string }): string {
    return jwt.sign(payload, JWT_SECRET, { expiresIn: JWT_EXPIRES_IN });
}

export function verifyToken(token: string): { userId: string; role: string } {
    return jwt.verify(token, JWT_SECRET) as { userId: string; role: string };
}
```

`jwt.sign` crea un token con header (algoritmo), payload (dati), e signature (firmata con il segreto). Il token è stateless: il server può verificarlo senza DB. Il payload NON è cifrato (solo base64url) — non metterci dati sensibili.

## Auth middleware

```typescript
// auth.middleware.ts
import { Request, Response, NextFunction } from "express";
import { verifyToken } from "./jwt";

export const authenticate = (req: Request, res: Response, next: NextFunction) => {
    const authHeader = req.headers.authorization;

    if (!authHeader?.startsWith("Bearer ")) {
        return res.status(401).json({ error: "Token mancante o formato errato" });
    }

    try {
        const token = authHeader.split(" ")[1];
        const decoded = verifyToken(token);
        (req as any).user = decoded;
        next();
    } catch {
        res.status(401).json({ error: "Token non valido o scaduto" });
    }
};

// Authorization middleware (RBAC)
export const authorize = (...roles: string[]) =>
    (req: Request, res: Response, next: NextFunction) => {
        const user = (req as any).user;
        if (!user || !roles.includes(user.role)) {
            return res.status(403).json({ error: "Accesso negato" });
        }
        next();
    };
```

`authenticate` estrae il token, lo verifica, e attacca `req.user`. `authorize` controlla il ruolo — si usa in combinazione: `router.get("/admin", authenticate, authorize("ADMIN"), handler)`.

## Refresh Token pattern

```typescript
// Auth service — login con refresh token
class AuthService {
    async login(email: string, password: string) {
        const user = await userRepository.findByEmail(email);
        if (!user) throw new AppError(401, "Credenziali non valide");

        const valid = await comparePassword(password, user.password);
        if (!valid) throw new AppError(401, "Credenziali non valide");

        // Access token — breve durata
        const accessToken = generateToken(
            { userId: user.id, role: user.ruolo },
            "15m"   // 15 minuti
        );

        // Refresh token — lunga durata, memorizzato in DB
        const refreshToken = crypto.randomUUID();
        await refreshTokenRepository.create({
            token: refreshToken,
            userId: user.id,
            expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7gg
        });

        return { accessToken, refreshToken, user: UserMapper.toResponse(user) };
    }

    async refresh(refreshToken: string) {
        const stored = await refreshTokenRepository.findByToken(refreshToken);
        if (!stored || stored.expiresAt < new Date()) {
            throw new AppError(401, "Refresh token non valido o scaduto");
        }

        const newAccessToken = generateToken(
            { userId: stored.userId, role: stored.user.ruolo },
            "15m"
        );

        return { accessToken: newAccessToken };
    }
}
```

## Header di sicurezza (Helmet)

```typescript
import helmet from "helmet";

app.use(helmet());  // imposta header HTTP di sicurezza

// Equivalenti manuali dei principali header impostati da helmet:
// Content-Security-Policy: default-src 'self'
// X-Content-Type-Options: nosniff
// X-Frame-Options: DENY
// Strict-Transport-Security: max-age=15552000; includeSubDomains
// X-XSS-Protection: 0 (deprecato, più dannoso che utile)
```

## Rate limiting

```typescript
import rateLimit from "express-rate-limit";

const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,     // 15 minuti
    max: 5,                        // 5 tentativi per finestra
    message: { error: "Troppi tentativi. Riprova tra 15 minuti." },
    standardHeaders: true,
    legacyHeaders: false,
});

// Applicato solo agli endpoint di login
app.use("/api/auth/login", authLimiter);
```

## CORS per sicurezza

```typescript
app.use(cors({
    origin: process.env.CORS_ORIGIN?.split(",") || "http://localhost:4200",
    methods: ["GET", "POST", "PUT", "DELETE", "PATCH"],
    allowedHeaders: ["Content-Type", "Authorization"],
    credentials: true,
    maxAge: 86400,  // preflight cached per 24h
}));
```

## Data sanitization

```typescript
// Prevenzione NoSQL injection (per MongoDB) e XSS
import mongoSanitize from "express-mongo-sanitize";
import xss from "xss-clean";

app.use(mongoSanitize());   // rimuove $ e . da req.body/params/query
app.use(xss());             // sanifica input HTML-like

// Per SQL injection — Prisma/TypeORM usano parametri con binding, già sicuri
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| Token decodificato ma non verificato | `jwt.decode()` (solo base64) invece di `jwt.verify()` | Usa sempre `jwt.verify()` |
| `secret` troppo corto | JWT_SECRET < 256 bit | Almeno 32 caratteri casuali |
| Password in chiaro nel DB | Password non hashata prima del salvataggio | Hasha con bcrypt PRIMA di salvare |
| Token nel localStorage | Vulnerabile a XSS | Usa HttpOnly cookie per refresh token |
| Rate limiting non applicato | `app.use(rateLimit())` globale blocca tutto | Applica selettivamente sugli endpoint sensibili |
| CORS troppo permissivo | `origin: "*"` con `credentials: true` | Specifica origins esatte o disabilita credential |

## Best practice

- **Hash sempre con bcrypt** — non usare SHA o MD5 per password (troppo veloci, vulnerabili a brute force)
- **JWT_SECRET in env (non nel codice)** — mai hardcodato, almeno 32 caratteri casuali
- **Mai dati sensibili nel payload JWT** — il payload è solo base64, non cifrato
- **Access token breve (15m), refresh token lungo (7gg)** — minimizza la finestra di esposizione
- **Helmet in produzione** — header di sicurezza gratuiti
- **Rate limiting su /login e /register** — previene brute force e credential stuffing
- **HttpOnly cookie per refresh token** — non accessibile da JS (XSS protection)
- **Logging dei tentativi di accesso** — registra IP, timestamp, successo/fallimento
- **HTTPS sempre** — niente auth su HTTP (token intercettabili)
- **`npm audit` regolare** — controlla vulnerabilità nelle dipendenze

## Cross-reference

- [[JS + TS/Node.js/Express/Middleware|Express — Middleware]] — auth middleware, error middleware
- [[JS + TS/Node.js/Express/Services e Controller|Express — Services e Controller]] — auth in service layer
- [[JS + TS/Node.js/Express/Database e ORM|Express — Database e ORM]] — repository per refresh token
- [[JS + TS/Core Concepts/Async|Async]] — async/await in auth flow
