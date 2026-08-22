---
topic: "TypeScript — panoramica e setup"
tags: [typescript, ts, setup, tsconfig, backend]
nav_next: "[[Tipi Avanzati.md]]"
---
Riferimento ufficiale: [typescriptlang.org/docs](https://www.typescriptlang.org/docs/)

TypeScript (2012, Anders Hejlsberg — Microsoft) è un **superset** di JS che aggiunge **tipizzazione statica opzionale**. Il codice TS viene **compilato** (transpilato) in JS puro tramite `tsc`. Non esiste un runtime TS nativo: il browser e Node.js eseguono sempre JS.

Rispetto a JS puro:
- **Type safety** — errori catturati a compile-time (typo, shape sbagliata, null check)
- **Tooling superiore** — autocompletamento, refactoring, navigazione molto migliori
- **Documentazione vivente** — il tipo è la documentazione, non serve commentare cosa restituisce una funzione
- **Adozione massiva** — standard de facto per qualsiasi progetto backend con Node.js (NestJS, Fastify, Express moderno)

## Setup

```bash
npm install -D typescript
npm install -D @types/node           # tipi per Node.js API
npx tsc --init                        # genera tsconfig.json
```

`@types/node` è un **pacchetto di definizioni** con le API di Node.js (process, fs, http, etc.). TypeScript non include i tipi di Node.js built-in — vanno installati separatamente da DefinitelyTyped.

## tsconfig.json

```json
{
    "compilerOptions": {
        "target": "ES2022",           // versione JS di output
        "module": "NodeNext",         // sistema moduli (NodeNext = ESM per Node)
        "moduleResolution": "NodeNext",
        "outDir": "./dist",           // cartella output JS
        "rootDir": "./src",           // cartella sorgente TS
        "strict": true,               // abilita TUTTI i flag strict
        "esModuleInterop": true,      // compatibilità import CJS ↔ ESM
        "skipLibCheck": true,         // non controlla .d.ts delle librerie
        "forceConsistentCasingInFileNames": true,
        "resolveJsonModule": true,    // import JSON come moduli
        "declaration": true,          // genera .d.ts per librerie
        "sourceMap": true             // debug: mappa TS → JS
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "dist"]
}
```

`strict: true` abilita: `noImplicitAny`, `strictNullChecks`, `strictFunctionTypes`, `strictBindCallApply`, `noUnusedLocals`, `noUnusedParameters`. È il flag più importante — senza, TypeScript perde gran parte del suo valore.

## Compilazione

```bash
npx tsc                        # compila tutto
npx tsc --watch                # watch mode
npx tsc --noEmit               # solo type-check (senza output)
```

`--noEmit` è usato nei CI/CD e nei lint-staged: controlla i tipi senza generare file. Il vero output JS si produce con build tool (tsup, esbuild, vite) più veloci di `tsc`.

## Strict mode e implicazioni

```typescript
// Senza strict: any implicito
function saluta(nome) {       // nome: any
    return `Ciao ${nome}`;
}

// Con strict: errore — nome ha tipo implicito any
function saluta(nome) {       // Error: Parameter 'nome' implicitly has 'any' type
    return `Ciao ${nome}`;
}

// Fix: tipo esplicito o tipo inferito
function saluta(nome: string) { return `Ciao ${nome}`; }
```

## tsconfig paths e baseUrl

```json
{
    "compilerOptions": {
        "baseUrl": "./src",
        "paths": {
            "@/services/*": ["services/*"],
            "@/models/*":   ["models/*"]
        }
    }
}
```

Permette import puliti: `import { UserService } from "@/services/user"` invece di `../../../services/user`. Richiede configurazione anche nel bundler/runtime (es. `tsconfig-paths` per Node.js, `moduleNameMapper` per Jest).

## Relazione con JS

TS è JS + tipo. **Qualsiasi JS valido è anche TS valido**. Puoi gradualmente aggiungere tipo a un progetto JS esistente:

1. Rinomina `.js` → `.ts`
2. Aggiungi `"strict": false`
3. Attiva `allowJs` e `checkJs` per file misti
4. Aggiungi tipo gradualmente, file per file
5. Attiva `strict` solo quando tutto è migrato

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Cannot find module 'x'` | Tipi non installati | `npm install -D @types/x` |
| `TS18002: 'strict' mode is required` | Progetto strict abilitato | Metti `strict: true` o rimuovi flag ridondanti |
| `TS2307: Cannot find module` | Path non risolto dal resolver | Configura `paths` in tsconfig |
| `TS2322: Type 'x' is not assignable to type 'y'` | Assegnazione di tipo incompatibile | Usa type assertion (`as Type`) solo se sicuro |
| `TS6133: 'x' is declared but never used` | Variabile/mport non usato | Rimuovila o usa `_` prefisso per intenzionale |
| DLL d'ambiente mancanti a runtime | Ambient .d.ts non caricati | Assicurati che il file sia incluso in `include` |

## Best practice

- **`strict: true` sempre** — è il motivo per cui usi TS; senza strict, perdi la maggior parte dei benefici
- **`strictNullChecks` da solo** se non puoi attivare tutto — è il flag che cattura più bug (undefined access)
- **Mai usare `any` esplicitamente** — se devi, usa `unknown` e poi fai narrowing
- **Type inference > type annotation** — lascia che TS inferisca il tipo, annota solo quando serve (parametri, export)
- **`as const`** per literal type su oggetti e array: `const config = { ... } as const`
- **Namespace deprecati** — usa ES modules (import/export), namespace è legacy (pre-ES6)
- **`interface` per API pubbliche, `type` per union/intersection** — interface si estende più chiaramente
- **`readonly` per immutabilità** — `readonly string[]`, `Readonly<{ x: number }>`
- **Non usare `Function` come tipo** — specifica la firma: `(x: number) => string`
- **`@ts-expect-error` per casi eccezionali** — meglio di `@ts-ignore` (si lamenta se la riga sotto non ha errore)

## Cross-reference

- [[JS + TS/TypeScript/Tipi Avanzati|Tipi Avanzati]] — unknown, never, union, intersection
- [[JS + TS/TypeScript/Generics|Generics]] — type parameter, constraints
- [[JS + TS/TypeScript/Utility Types|Utility Types]] — Partial, Pick, Omit, Record
- [[JS + TS/TypeScript/Decoratori|Decoratori]] — decoratori in TS vs TC39
- [[JS + TS/Core Concepts/Moduli|Moduli]] — sistema moduli, ESM vs CJS
