---
topic: "Decoratori — TypeScript"
tags: [typescript, ts, decorators, experimental, tc39, metadata]
nav_prev: "[[Utility Types.md]]"
---
Riferimento ufficiale: [typescriptlang.org/docs/handbook/decorators.html](https://www.typescriptlang.org/docs/handbook/decorators.html)

I decoratori sono **funzioni che modificano classi, metodi, proprietà o parametri** a tempo di dichiarazione. In TypeScript sono una feature **sperimentale** (basata sulla proposta TC39 stage 2, ora stage 3 con sintassi diversa). NestJS, TypeORM, class-validator ne fanno ampio uso.

```json
// tsconfig.json — serve abilitare
{
    "compilerOptions": {
        "experimentalDecorators": true,  // abilita decoratori legacy TS
        "emitDecoratorMetadata": true    // emette metadata di tipo (usato da NestJS/TypeORM)
    }
}
```

## Tipi di decoratori

```typescript
// 1. Class decorator — riceve il costruttore
function Injectable(target: Function) {
    Reflect.defineMetadata("injectable", true, target);
}

@Injectable
class UserService { }

// 2. Method decorator — riceve target, nome metodo, descriptor
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    descriptor.value = function (...args: any[]) {
        console.log(`Chiamato ${propertyKey} con`, args);
        return original.apply(this, args);
    };
}

class Calculator {
    @Log
    somma(a: number, b: number): number {
        return a + b;
    }
}

// 3. Property decorator
function Format(formatStr: string) {
    return function (target: any, propertyKey: string) {
        Reflect.defineMetadata("format", formatStr, target, propertyKey);
    };
}

class Utente {
    @Format("yyyy-MM-dd")
    dataNascita: Date;
}

// 4. Parameter decorator
function Inject(tag: string) {
    return function (target: any, propertyKey: string, parameterIndex: number) {
        Reflect.defineMetadata(`inject:${tag}`, parameterIndex, target, propertyKey);
    };
}
```

## Decorator factory

I decoratori possono essere **parametrici** (quasi tutti i decoratori reali lo sono): una funzione che restituisce il decoratore vero e proprio.

```typescript
function MinLength(min: number) {
    // Questa è la factory — riceve i parametri di configurazione
    return function (target: any, propertyKey: string) {
        // Questo è il decoratore vero — riceve target e chiave
        Reflect.defineMetadata("minLength", min, target, propertyKey);
    };
}

class Prodotto {
    @MinLength(3)
    nome: string;
}
```

## Reflect Metadata

`emitDecoratorMetadata` fa sì che TS emetta informazioni di tipo a runtime via `Reflect.metadata`. È il meccanismo che permette a NestJS di sapere che `@Injectable()` su una classe la rende un provider, o a TypeORM di capire che `@Column()` su `nome: string` è una colonna VARCHAR.

```typescript
import "reflect-metadata";  // polyfill necessario a runtime

class MyService {
    constructor(
        @Inject("CONNECTION") private db: Database
    ) {}
}

// A runtime: Reflect.getMetadata("design:paramtypes", MyService)
// → [Database] — tipo del parametro del costruttore
```

## Nuova sintassi TC39 (stage 3)

La proposta TC39 per decoratori nativi JS (stage 3, prevista in ES2024+) ha una sintassi diversa, **incompatibile** con quella sperimentale di TS:

```typescript
// Nuova sintassi stage 3 (non ancora in TS per default)
function logged<T extends (...args: any[]) => any>(
    target: T,
    context: ClassMethodDecoratorContext
) {
    return function (this: any, ...args: Parameters<T>) {
        console.log(`called ${String(context.name)}`);
        return target.call(this, ...args);
    } as T;
}
```

La differenza principale: la nuova sintassi riceve un oggetto `context` invece dei tre parametri separati. NestJS non ha ancora migrato — al momento si usa la sintassi sperimentale.

## Uso in framework backend

```typescript
// NestJS — i decoratori sono ovunque
@Controller("users")
export class UserController {
    constructor(private readonly userService: UserService) {}

    @Get(":id")
    @UseGuards(AuthGuard)
    async findOne(@Param("id") id: string) {
        return this.userService.findOne(id);
    }
}

// TypeORM
@Entity("users")
export class User {
    @PrimaryGeneratedColumn("uuid")
    id: string;

    @Column({ length: 100 })
    nome: string;

    @ManyToOne(() => Role)
    @JoinColumn({ name: "role_id" })
    role: Role;
}

// class-validator / class-transformer
export class CreateUserDto {
    @IsString()
    @IsNotEmpty()
    @MinLength(2)
    nome: string;

    @IsEmail()
    email: string;
}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| Decorator non funziona, nessun errore | `experimentalDecorators` non abilitato | Abilita in tsconfig.json |
| `Reflect.metadata` non trovato a runtime | `reflect-metadata` polyfill non importato | `npm install reflect-metadata` e importalo all'entry point |
| `Cannot use decorators on parameters` senza `experimentalDecorators` | Decorator su parametro senza flag abilitato | Abilita experimentalDecorators |
| Decorator non si applica | L'ordine di valutazione conta (bottom-up per parametri) | Controlla l'ordine dei decoratori |
| Metodo decorato perde `this` | Arrow function nel decorator che non preserva contesto | Assicurati che `descriptor.value` usi `function` o `.call(this, ...)` |

## Best practice

- **Decoratori per cross-cutting concerns** — logging, validazione, caching, auth — non per logica di business
- **Non usare decoratori per logica stateful** — un decoratore non dovrebbe mantenere stato tra chiamate
- **`reflect-metadata` importato una volta** — di solito in `main.ts` all'entry point dell'app
- **NestJS style guide** — controller usa decoratori, services no (il più possibile logica pulita senza decoratori)
- **Evita decoratori custom complessi** — se serve logica elaborata, preferisci un middleware o un intercettore
- **Documenta ogni decoratore custom** — cosa fa, cosa modifica, side effect sull'execution context

## Cross-reference

- [[JS + TS/TypeScript/TypeScript|TypeScript]] — setup, tsconfig, experimentalDecorators
- [[JS + TS/NestJS/Setup e Architettura|NestJS — Setup]] — NestJS si basa pesantemente sui decoratori
- [[JS + TS/NestJS/Mappers e DTO|NestJS — Mappers e DTO]] — class-validator decorators
- [[JS + TS/Core Concepts/Classi|JS — Classi]] — sugar syntax, metodi, estensione
