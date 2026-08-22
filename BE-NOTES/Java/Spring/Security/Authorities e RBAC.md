---
topic: "Authorities e RBAC"
parent: "[[BE-NOTES/Java/Spring/Security/Spring Security|Spring Security]]"
nav_prev: "[[OAuth2 con Google e GitHub.md]]"
nav_next: "[[Token Blacklist.md]]"
---


Spring Security usa il concetto di **GrantedAuthority** per rappresentare permessi e ruoli. In [[TaskMngr]] il modello autorizzativo è **RBAC semplice** (Role-Based Access Control) con due ruoli: `USER` e `ADMIN`.

## GrantedAuthority

`GrantedAuthority` è un'interfaccia con un solo metodo: `getAuthority()`. Le autorità sono stringhe che Spring Security usa per decidere se una richiesta è autorizzata.

```java
// GrantedAuthority concreto più comune
var auth = new SimpleGrantedAuthority("ROLE_USER");
```

## hasRole vs hasAuthority

| Metodo | Esempio | Authority cercata | Prefisso |
|---|---|---|---|
| `hasRole("ADMIN")` | `.hasRole("ADMIN")` | `ROLE_ADMIN` | Automatico `ROLE_` |
| `hasAuthority("ROLE_ADMIN")` | `.hasAuthority("ROLE_ADMIN")` | `ROLE_ADMIN` | Nessuno |

**Regola:** usa `hasRole` per ruoli (USER, ADMIN, MODERATOR), usa `hasAuthority` per permessi granulari (es. `TASK_READ`, `TASK_DELETE`).

## Modello in TaskMngr

```
┌──────────┐       ┌───────────────────┐       ┌───────────────────────┐
│  User    │       │  CustomUserDetails │       │  SecurityContext      │
│──────────│       │───────────────────│       │───────────────────────│
│ id       │       │ email             │       │ Authentication        │
│ email    │──────→│ password (hash)   │──────→│  ├─ principal: email   │
│ password │       │ role: Role        │       │  └─ authorities:      │
│ role     │       │ authorities:      │       │     "ROLE_USER"       │
└──────────┘       │  "ROLE_" + role   │       │     (o "ROLE_ADMIN")  │
                   └───────────────────┘       └───────────────────────┘
```

### Role enum

```java
public enum Role {
    USER,
    ADMIN
}
```

- `USER` — ruolo predefinito per tutti i nuovi utenti (sia registrazione diretta che OAuth2)
- `ADMIN` — si assegna manualmente via database o script di bootstrap

### GrantedAuthority nel JwtAuthenticationFilter

Il `JwtAuthenticationFilter` estrae il claim `role` dal JWT e crea un `SimpleGrantedAuthority` con prefisso `ROLE_`:

```java
String role = claims.get("role", String.class);
var authorities = List.of(new SimpleGrantedAuthority("ROLE_" + role));
```

Lo stesso fa `CustomUserDetailsService.loadUserByUsername()` per l'autenticazione via form login / basic auth.

## Regole URL-based (`authorizeHttpRequests`)

```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers("/api/admin/**").hasRole("ADMIN")
    .requestMatchers("/api/auth/**").permitAll()
    .anyRequest().authenticated()
)
```

## Method Security (`@PreAuthorize`)

```java
@PreAuthorize("hasRole('ADMIN')")
@DeleteMapping("/users/{id}")
public void deleteUser(@PathVariable Long id) { ... }
```

```java
@PreAuthorize("hasRole('ADMIN') or #id == @securityUtil.getCurrentUserId()")
@PutMapping("/users/{id}")
public UserResponse updateUser(@PathVariable Long id, @RequestBody ...) { ... }
```

La SpEL expression chiama il bean `@securityUtil.getCurrentUserId()` per confrontare l'ID del parametro con l'utente autenticato — evita che un utente modifichi i dati di un altro.

## Perché non permessi granulari

TaskMngr ha scelto ruoli semplici (`USER`/`ADMIN`) invece di un sistema a permessi (es. `TASK_CREATE`, `TASK_DELETE`, `USER_READ`) perché:

- Il dominio non richiede gerarchie di permessi complesse
- Meno overhead di progettazione e manutenzione
- Se in futuro servissero permessi granulari, si può aggiungere una tabella `permissions` con `@ManyToMany` senza rompere l'esistente

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| `hasRole("ROLE_ADMIN")` invece di `hasRole("ADMIN")` | 403 anche per ADMIN | `hasRole` aggiunge già il prefisso `ROLE_` → cerca `ROLE_ROLE_ADMIN` | Scrivi solo il nome del ruolo: `hasRole("ADMIN")`, non `hasRole("ROLE_ADMIN")` |
| `hasAuthority("ADMIN")` invece di `hasAuthority("ROLE_ADMIN")` | 403 — il ruolo non matcha | `hasAuthority` NON aggiunge prefisso | Usa `hasAuthority("ROLE_ADMIN")` o `hasRole("ADMIN")` |
| `@PreAuthorize` senza `@EnableMethodSecurity` | Le annotazioni sui metodi vengono ignorate | Spring non processa `@PreAuthorize` senza l'annotazione di abilitazione | Aggiungi `@EnableMethodSecurity` sulla configurazione |
| SpEL con sintassi errata | `ExpressionEvaluationException` o 403 inaspettato | `@PreAuthorize("hasRole('ADMIN')")` — virgolette errate causano eval fallito | Usa apici singoli per le stringhe SpEL: `'ADMIN'` non `"ADMIN"` |
| Ruolo non coerente tra JWT e DB | Utente ADMIN in DB ma JWT dice USER | Il claim `role` nel JWT non viene aggiornato dopo cambio ruolo | Il JWT riflette il ruolo al momento dell'emissione; per aggiornarlo serve re-login o refresh token |

## In TaskMngr

- `Role.USER` è il default per tutti i nuovi utenti
- `@EnableMethodSecurity` abilita `@PreAuthorize` su controller e service
- `SecurityUtil` fornisce `getCurrentUser()` e `getCurrentUserId()` per confronti in SpEL
- Creazione utente: `new User(email, password, Role.USER)`
