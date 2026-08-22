---
topic: "NestJS — Auth e Sicurezza"
tags: [nestjs, auth, security, guards, jwt, passport, rbac]
nav_prev: "[[TypeORM e Database.md]]"
nav_next: "[[Testing.md]]"
---
Riferimento ufficiale: [docs.nestjs.com/security/authentication](https://docs.nestjs.com/security/authentication) | [docs.nestjs.com/security/authorization](https://docs.nestjs.com/security/authorization) | [www.passportjs.org](https://www.passportjs.org/)

NestJS ha un sistema di autenticazione modulare basato su **Guards** (autorizzazione) e **Passport** (autenticazione). Guards decidono se una richiesta può procedere; Passport gestisce le strategie di login (JWT, OAuth2, local).

```bash
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install -D @types/passport-jwt
npm install bcrypt
npm install -D @types/bcrypt
```

## Auth Module

```typescript
// auth.module.ts
import { Module } from "@nestjs/common";
import { JwtModule } from "@nestjs/jwt";
import { PassportModule } from "@nestjs/passport";
import { AuthService } from "./auth.service";
import { AuthController } from "./auth.controller";
import { JwtStrategy } from "./jwt.strategy";

@Module({
    imports: [
        PassportModule.register({ defaultStrategy: "jwt" }),
        JwtModule.register({
            secret: process.env.JWT_SECRET,
            signOptions: { expiresIn: "15m" },
        }),
        UsersModule,  // per validare credenziali
    ],
    providers: [AuthService, JwtStrategy],
    controllers: [AuthController],
    exports: [JwtStrategy, PassportModule],
})
export class AuthModule {}
```

## Auth Service

```typescript
// auth.service.ts
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { JwtService } from "@nestjs/jwt";
import * as bcrypt from "bcrypt";

@Injectable()
export class AuthService {
    constructor(
        private readonly usersService: UsersService,
        private readonly jwtService: JwtService,
    ) {}

    async login(email: string, password: string) {
        const user = await this.usersService.findByEmail(email);
        if (!user) throw new UnauthorizedException("Credenziali non valide");

        const valid = await bcrypt.compare(password, user.password);
        if (!valid) throw new UnauthorizedException("Credenziali non valide");

        const payload = { sub: user.id, ruolo: user.ruolo };
        return {
            accessToken: this.jwtService.sign(payload),
            user: { id: user.id, nome: user.nome, email: user.email },
        };
    }
}
```

## JWT Strategy (Passport)

```typescript
// jwt.strategy.ts
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { ExtractJwt, Strategy } from "passport-jwt";
import { UsersService } from "../users/users.service";

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
    constructor(private readonly usersService: UsersService) {
        super({
            jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
            ignoreExpiration: false,
            secretOrKey: process.env.JWT_SECRET,
        });
    }

    // Il payload decodificato del JWT
    async validate(payload: { sub: string; ruolo: string }) {
        const user = await this.usersService.findById(payload.sub);
        if (!user) throw new UnauthorizedException("Utente non trovato");
        return { id: user.id, nome: user.nome, ruolo: user.ruolo };
        // → diventa req.user
    }
}
```

`validate` è chiamata automaticamente da Passport dopo aver verificato il JWT. Il valore restituito viene assegnato a `req.user`. Se lancia `UnauthorizedException`, NestJS restituisce 401.

## Guards (autorizzazione)

```typescript
// roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from "@nestjs/common";
import { Reflector } from "@nestjs/core";
import { ROLES_KEY } from "./roles.decorator";

@Injectable()
export class RolesGuard implements CanActivate {
    constructor(private reflector: Reflector) {}

    canActivate(context: ExecutionContext): boolean {
        const requiredRoles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
            context.getHandler(),
            context.getClass(),
        ]);
        if (!requiredRoles) return true;  // nessun ruolo richiesto

        const { user } = context.switchToHttp().getRequest();
        return requiredRoles.includes(user.ruolo);
    }
}

// roles.decorator.ts
import { SetMetadata } from "@nestjs/common";
export const ROLES_KEY = "roles";
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);
```

## Uso in controller

```typescript
// users.controller.ts
import { Controller, Get, Post, UseGuards, Body, Param } from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";
import { RolesGuard } from "./roles.guard";
import { Roles } from "./roles.decorator";
import { CurrentUser } from "./current-user.decorator";

@Controller("users")
@UseGuards(AuthGuard("jwt"))  // protegge TUTTI i metodi del controller
export class UsersController {
    @Get("me")
    getProfile(@CurrentUser() user: User) {
        return user;
    }

    @Post()
    @Roles("admin")                         // solo admin
    @UseGuards(RolesGuard)                  // AuthGuard già applicato sopra
    async create(@Body() dto: CreateUserDto) {
        return this.usersService.create(dto);
    }

    @Delete(":id")
    @Roles("admin")
    @UseGuards(RolesGuard)
    async delete(@Param("id") id: string) {
        await this.usersService.delete(id);
    }
}
```

## Public endpoints

```typescript
// public.decorator.ts
import { SetMetadata } from "@nestjs/common";
export const IS_PUBLIC_KEY = "isPublic";
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// JwtAuthGuard modificato
@Injectable()
export class JwtAuthGuard extends AuthGuard("jwt") {
    constructor(private reflector: Reflector) {
        super();
    }

    canActivate(context: ExecutionContext) {
        const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
            context.getHandler(),
            context.getClass(),
        ]);
        if (isPublic) return true;  // salta autenticazione
        return super.canActivate(context);
    }
}

// Uso
@Public()
@Post("login")
async login(@Body() dto: LoginDto) { ... }
```

## Rate limiting

```typescript
npm install @nestjs/throttler

// app.module.ts
import { ThrottlerModule } from "@nestjs/throttler";

@Module({
    imports: [
        ThrottlerModule.forRoot([
            { ttl: 60000, limit: 10 },  // 10 richieste al minuto (globale)
        ]),
    ],
})

// Per-endpoint
@SkipThrottle()       // esclude dal rate limiting globale
@Throttle({ default: { limit: 3, ttl: 60000 } })  // 3 richieste/minuto specifiche
```

## Helmet e CORS

```typescript
// main.ts
import helmet from "helmet";

async function bootstrap() {
    const app = await NestFactory.create(AppModule);

    app.use(helmet());  // header di sicurezza

    app.enableCors({
        origin: process.env.CORS_ORIGIN?.split(",") ?? "http://localhost:4200",
        methods: ["GET", "POST", "PUT", "DELETE", "PATCH"],
        credentials: true,
    });

    await app.listen(3000);
}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `UnauthorizedException` anche con token valido | JWT secret diverso tra sign e verify | Usa stessa `JwtModule.register({ secret })` per entrambi |
| `req.user` è undefined | JwtStrategy.validate non restituisce oggetto | validate() deve restituire un oggetto (non null/undefined) |
| Guard non eseguito | Guard non applicato con `@UseGuards()` | Aggiungi decoratore sul controller o metodo |
| Token JWT decodificato ma non verificato | `jwt.decode()` usato invece di `jwt.verify()` | Usa Passport strategy (chiama verify automaticamente) |
| Rate limiting blocca tutto | Limite globale troppo basso | Aumenta limite o usa `@SkipThrottle()` su endpoint pubblici |

## Best practice

- **JWT Strategy + Guards per auth** — Passport gestisce la verifica, Guards l'autorizzazione
- **Refresh token pattern** — access token breve (15m), refresh token lungo (7gg) con rotazione
- **Ruoli via enum** — `"admin" | "user"` invece di stringhe libere (type-safe)
- **Guards globali + @Public()** — Proteggi TUTTI gli endpoint di default, marca come pubblici solo quelli che servono
- **Rate limiting su /auth/login** — previene brute force (3 tentativi/minuto)
- **Hash password sempre** — bcrypt con saltRounds >= 12, mai SHA/MD5
- **Helmet + CORS configurazioni strette** — in produzione, origini CORS esplicite, helmet.on()
- **Logging tentativi di accesso** — registra IP, email, timestamp, success per audit e detection
- **`ValidationPipe` globale** — whitelist rimuove campi extra, previene mass assignment
- **Secret in env, mai hardcodato** — JWT_SECRET, DB_PASSWORD, API_KEY sempre in variabili d'ambiente

## Cross-reference

- [[JS + TS/NestJS/Controller e Routes|NestJS — Controller]] — guard application
- [[JS + TS/NestJS/Services e Providers|NestJS — Services]] — auth service
- [[JS + TS/Node.js/Express/Auth e Sicurezza|Express — Auth e Sicurezza]] — JWT, bcrypt, helmet in Express
- [[JS + TS/TypeScript/Decoratori|TypeScript — Decoratori]] — @UseGuards, custom decorators
