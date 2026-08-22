---
topic: "Package Manager, Linting e Testing — JS/TS"
tags: [tools, npm, yarn, pnpm, eslint, prettier, jest, vitest]
---
Riferimento ufficiale: [docs.npmjs.com](https://docs.npmjs.com/) | [eslint.org](https://eslint.org/) | [prettier.io](https://prettier.io/) | [jestjs.io](https://jestjs.io/) | [vitest.dev](https://vitest.dev/)

## Package Manager

```bash
# npm — built-in con Node.js, il più usato
npm init -y                      # inizializza progetto
npm install express              # installa come dipendenza
npm install -D typescript jest   # installa come devDependency
npm install -g @nestjs/cli       # installa globalmente
npx tsc --init                   # esegue senza installare globalmente

# yarn — più veloce di npm, cache offline
yarn add express
yarn add -D typescript

# pnpm — il più veloce, disk space efficient (hard link)
pnpm add express
pnpm add -D typescript
```

## package.json

```json
{
    "name": "my-api",
    "version": "1.0.0",
    "type": "module",
    "scripts": {
        "dev": "tsx watch src/main.ts",
        "build": "tsup src/main.ts --out-dir dist",
        "start": "node dist/main.js",
        "lint": "eslint src/ --ext .ts",
        "format": "prettier --write src/",
        "test": "jest --passWithNoTests",
        "test:watch": "jest --watch",
        "test:cov": "jest --coverage",
        "test:e2e": "jest --config ./test/jest-e2e.json"
    },
    "dependencies": {
        "express": "^4.18.0"
    },
    "devDependencies": {
        "@types/express": "^4.17.0",
        "@types/jest": "^29.0.0",
        "eslint": "^8.0.0",
        "jest": "^29.0.0",
        "prettier": "^3.0.0",
        "ts-jest": "^29.0.0",
        "tsup": "^8.0.0",
        "typescript": "^5.3.0"
    }
}
```

## ESLint — linting

```javascript
// eslint.config.js (flat config, ESLint 9+)
import typescript from "@typescript-eslint/eslint-plugin";
import parser from "@typescript-eslint/parser";

export default [
    {
        files: ["src/**/*.ts"],
        languageOptions: { parser },
        plugins: { "@typescript-eslint": typescript },
        rules: {
            "@typescript-eslint/no-explicit-any": "warn",
            "@typescript-eslint/explicit-function-return-type": "off",
            "@typescript-eslint/no-unused-vars": ["error", { argsIgnorePattern: "^_" }],
            "no-console": "warn",
            "prefer-const": "error",
            "eqeqeq": ["error", "always"],
        },
    },
];
```

## Prettier — formattazione

```json
// .prettierrc
{
    "semi": true,
    "singleQuote": false,
    "tabWidth": 4,
    "trailingComma": "all",
    "printWidth": 100,
    "arrowParens": "always",
    "endOfLine": "lf"
}
```

## tsx — TypeScript runtime per sviluppo

```bash
npm install -D tsx

npm run dev  # tsx watch src/main.ts
# tsx esegue TypeScript direttamente (senza compilare), watch ricarica a ogni modifica
```

## tsup — builder rapido

```bash
npm install -D tsup

# tsup compila TS in JS con esbuild (molto più veloce di tsc)
npx tsup src/main.ts --out-dir dist --target node20 --format esm,cjs
```

## Husky + lint-staged (pre-commit hooks)

```bash
npm install -D husky lint-staged
npx husky init

// .husky/pre-commit
npx lint-staged

// package.json
{
    "lint-staged": {
        "*.ts": ["eslint --fix", "prettier --write"],
        "*.json": ["prettier --write"]
    }
}
```

## Node.js version manager

```bash
# nvm-windows — gestisce versioni Node.js
nvm install 20.11.0
nvm use 20.11.0
nvm list
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Module not found` | Pacchetto non installato o path sbagliato | `npm install` o controlla import path |
| `ESLint: Parsing error: Unexpected token` | ESLint non configurato per TS | Installa `@typescript-eslint/parser` |
| `Prettier: File ignored` | File non nelle glob patterns | Aggiungi pattern in `.prettierignore` |
| `Jest: Cannot find module 'supertest'` | Dev dep mancante | `npm install -D supertest @types/supertest` |
| `tsx: command not found` | tsx non installato | `npm install -D tsx` |
| `ERR_REQUIRE_ESM` | Misto CJS/ESM nella configurazione | Unifica a ESM (`"type": "module"`) o CJS |

## Best practice

- **npm scripts standardizzati** — `dev`, `build`, `start`, `lint`, `format`, `test`, `test:cov`, `test:e2e`
- **Dev dependencies separate** — tutto ciò che serve solo in sviluppo (test, lint, type definitions) in `devDependencies`
- **`^` per versioni** — `"express": "^4.18.0"` permette patch e minor update (ma non major)
- **Lock file in VCS** — `package-lock.json` (o `yarn.lock`, `pnpm-lock.yaml`) va committato per riproducibilità
- **ESLint + Prettier insieme** — ESLint per regole di qualità, Prettier per formattazione (mai conflitto: eslint-config-prettier)
- **Husky per pre-commit** — lint + format + type-check prima di ogni commit (nessun errore stupido in CI)
- **`overrides` per risolvere vulnerabilità** — in package.json per forzare versione di una dipendenza transitiva
- ****npx** per esecuzioni one-shot** — non installare globalmente ciò che usi una volta (`npx create-next-app`)
- **`"engines"` in package.json** — specifica versione Node.js richiesta: `"node": ">=18"`
- **`npm audit` regolarmente** — controlla vulnerabilità; `npm audit fix` per correzioni automatiche

## Cross-reference

- [[JS + TS/TypeScript/TypeScript|TypeScript]] — tsconfig, compilazione
- [[JS + TS/Node.js/Express/Testing|Express — Testing]] — Jest, Supertest per Express
- [[JS + TS/NestJS/Testing|NestJS — Testing]] — Test Bed, e2e in NestJS
