---
topic: "Linguaggi e Scenari — scelta tecnologica"
tags: [architecture, decision, java, python, typescript, kotlin, go, csharp, dotnet]
---


Scelta basata su: team, dominio, vincoli di performance, ecosistema, manutenibilità.

## Java
- **Enterprise backend** — banking, e-commerce, SaaS con team ≥ 5 dev
- **Ecosistema Spring** — batch processing (Spring Batch), messaging (JMS/Kafka), security (Spring Security), cloud (Spring Cloud)
- **Performance prevedibile** — JIT compila hotspot, GC tuning maturo, thread pool reali (non GIL)
- **Android nativo** — anche se Kotlin è preferito, Java è ancora onnipresente
- **Quando serve**: team Java-esperti, requisiti enterprise, compliance, decenni di librerie stabili
- **Evita quando**: vuoi prototipare in giornata, scrivere scripting/automazione, ML/AI

Vedi: [[BE-NOTES/Java/Java|Java vault]], [[BE-NOTES/Java/Spring/Spring|Spring ecosystem]]

## Python
- **Backend API** — FastAPI per microservizi (async, type-hint driven, OpenAPI automatico)
- **Full-stack web** — Django per CRUD admin-heavy, CMS, prototipi rapidi con admin panel incluso
- **Scripting e automazione** — glue code, DevOps, ETL, file manipulation
- **Data Science / ML / AI** — Pandas, NumPy, scikit-learn, PyTorch, LangChain (ecosistema dominante)
- **Bot e web scraping** — discord.py, python-telegram-bot, Scrapy, Playwright
- **Prototipazione rapida** — da idea a MVP in giorni, non settimane
- **Evita quando**: devi gestire milioni di richieste concorrenti CPU-bound, hai requisiti di latenza < 10ms, team solo Java-senior senza voglia di imparare Python

Vedi: [[BE-NOTES/Python/Python|Python vault]]

## TypeScript
- **Frontend web** — React, Angular, Vue. TypeScript è lo standard de facto (non JS vanilla)
- **Node.js backend** — NestJS (Spring-like), Express/Fastify (leggeri), tRPC (type-safe API)
- **Full-stack con un linguaggio** — Next.js, Nuxt, SvelteKit — condividi tipi tra client e server
- **Quando serve**: UI ricca, real-time (WebSocket), team full-stack, type safety in frontend
- **Evita quando**: backend enterprise con transaction pesanti, CPU-intensive, ecosistema Java già consolidato

## Kotlin
- **Android moderno** — linguaggio ufficiale Google per Android
- **Spring Boot alternativo** — più conciso di Java, null-safety, coroutine per async, data class
- **Multiplatform (KMP)** — condividi logica tra Android, iOS, web, backend
- **Quando serve**: già in JVM ma vuoi meno boilerplate, coroutine native, team che apprezza FP + OOP
- **Evita quando**: team solo Java-senior (curva di apprendimento), ecosistema librerie meno maturo di Java

## C# / .NET
- **Enterprise Windows** — ASP.NET Core, WPF, WinForms, servizi Windows
- **API REST cross-platform** — ASP.NET Core Minimal API / Controllers, performance top-tier (TechEmpower top 3)
- **Giochi e Real-time** — Unity (C# è il linguaggio primario), SignalR per WebSocket
- **Cloud Azure** — integrazione nativa con Azure Functions, App Service, Cosmos DB
- **Desktop** — WPF (.NET Framework) e WinUI 3 / MAUI (.NET 6+)
- **Performance critiche** — struct value-type, Span<T>, Native AOT, vaste arene non-GC
- **Quando serve**: ecosistema Microsoft, enterprise Windows, giochi Unity, cloud Azure, performance I/O-bound
- **Evita quando**: ML/AI (ecosistema Python domina), scripting veloce, frontend web

Vedi: [[BE-NOTES/C#/C#|C# vault]], [[BE-NOTES/C#/.NET/.NET|.NET framework]]

## Go
- **Microservizi ad alta concorrenza** — goroutine + channel nativi (nessun framework async)
- **CLI tools e DevOps** — single binary, cross-compile facile, startup instantaneo
- **Proxy, gateway, middleware** — perfetto per API gateway, reverse proxy, service mesh
- **Quando serve**: altissima concorrenza I/O, deploy come singolo binario, team che vuole semplicità
- **Evita quando**: business logic complessa con OOP/ereditarietà, team non ha esperienza Go

## Tabella riassuntiva

| Scenario | Scelta primaria | Alternativa |
|---|---|---|
| API REST enterprise | Java (Spring Boot) | Kotlin (Spring Boot), TypeScript (NestJS) |
| API REST rapida / MVP | Python (FastAPI) | TypeScript (Express/Fastify) |
| Full-stack web | TypeScript (Next.js, Angular) | Python (Django), Java (Spring MVC) |
| SPA frontend | TypeScript (React, Vue, Angular) | — |
| Microservizi | Go / Java (Spring Boot) | Python (FastAPI), Kotlin |
| ETL / data pipeline | Python | Java (Spring Batch) |
| ML / AI | Python | — |
| Scripting / automazione | Python | TypeScript (Bun/Deno) |
| Android | Kotlin | Java |
| CLI tools | Go | Python (Click/Typer), C# (Native AOT) |
| Giochi / Real-time 3D | C# (Unity) | — |
| Enterprise Windows | C# / .NET | Java (Spring) |
| Real-time (WebSocket) | TypeScript (Node.js) | Go, Python (FastAPI WebSocket) |
| System programming | Rust / Go | — |
| Mobile cross-platform | Kotlin Multiplatform / Flutter | React Native (TypeScript) |
| DevOps / infra | Go (Terraform, Kubernetes) | Python |

## Regole pratiche

- **Team > linguaggio**: se il team ha 10 anni di Java, scrivere un microservizio in Go non è saggio a meno che non sia critico per performance
- **Ecosistema > hype**: un linguaggio vale quanto le sue librerie. Python per ML, Java per enterprise transaction, TS per frontend
- **Tipo di carico**: I/O-bound → Python asincrono o Go. CPU-bound → Java, Rust, Go. Misto → Java (thread pool reali)
- **Manutenibilità**: codice che vive 10 anni → Java/TS (tipizzazione forte, tooling maturo). Prototipi usa-e-getta → Python
- **Interoperabilità**: se devi integrarti con un ecosistema esistente (JVM, .NET, Node), usa il linguaggio nativo
