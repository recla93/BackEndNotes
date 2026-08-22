---
topic: "Spring Security — autenticazione e autorizzazione"
parent: "[[BE-NOTES/Java/Spring/Boot/Spring Boot|Spring Boot]]"
nav_next: "[[SecurityConfig e Filter Chain.md]]"
---


Modulo di sicurezza Spring. In [[TaskMngr]] gestisce autenticazione tramite **JWT** e **OAuth2** (Google, GitHub), con un'architettura a catena di filtri personalizzata.

## Argomenti

- [[BE-NOTES/Java/Spring/Security/SecurityConfig e Filter Chain|SecurityConfig e Filter Chain]] — configurazione della catena di filtri, CORS, CSRF, route pubbliche, method security
- [[BE-NOTES/Java/Spring/Security/JWT - Generazione e Validazione|JWT - Generazione e Validazione]] — token JWT, firma, claims (incluso `role`), scadenza, refresh token
- [[BE-NOTES/Java/Spring/Security/OAuth2 con Google e GitHub|OAuth2 con Google e GitHub]] — OAuth2 client, resource server, login sociale, linked accounts
- [[BE-NOTES/Java/Spring/Security/Token Blacklist|Token Blacklist]] — revoca token, blacklist in memoria, logout
- [[BE-NOTES/Java/Spring/Security/Authorities e RBAC|Authorities e RBAC]] — modello autorizzativo a ruoli, GrantedAuthority, hasRole vs hasAuthority
