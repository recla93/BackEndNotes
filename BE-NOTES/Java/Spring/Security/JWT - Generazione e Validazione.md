---
topic: "JWT — Generazione e Validazione"
parent: "[[BE-NOTES/Java/Spring/Security/Spring Security|Spring Security]]"
nav_prev: "[[SecurityConfig e Filter Chain.md]]"
nav_next: "[[OAuth2 con Google e GitHub.md]]"
---


Il JWT (JSON Web Token) è il meccanismo di autenticazione stateless di [[TaskMngr]]. Il server firma un token che contiene l'identità dell'utente; il client lo invia a ogni richiesta. Il server verifica la firma e sa chi è l'utente — **nessuna sessione, nessun cookie**.

## Quando usare JWT

- **API stateless** — il server non memorizza sessioni, scala orizzontalmente senza sforzo
- **Microservizi** — un token emesso da un servizio è valido per tutti gli altri
- **Mobile / SPA** — funziona con header HTTP, non dipende dai cookie
- **Separazione frontend-backend** — frontend Angular gestisce il token, backend lo verifica

**Quando NON usarlo:**
- **App server-side tradizionali** (Thymeleaf, JSP) — la sessione HTTP è più semplice e sicura
- **Revoca frequente** — JWT è intrinsecamente non revocabile (serve blacklist → perde il vantaggio stateless)
- **Payload grandi** — il token è inviato a ogni richiesta, peso e larghezza di banda contano

## Struttura del JWT

Un JWT è composto da tre parti separate da punti, codificate in Base64URL:

```
eyJhbGciOiJIUzI1NiJ9.        // header: {"alg":"HS256","typ":"JWT"}
eyJzdWIiOiIxIiwicm9sZSI6IlVTRVIifQ. // payload: {"sub":"1","role":"USER","iat":...,"exp":...}
signature                       // firma HMAC-SHA256
```

| Parte | Contenuto | Scopo |
|---|---|---|
| **Header** | Algoritmo di firma (HS256, RS256) | Dire al server come verificare |
| **Payload** | Claims standard: `sub` (email), `iat`, `exp`; custom: `id` (userId), `role` | Identità, autorizzazione e metadata |
| **Signature** | HMAC(header.payload, secret) | Integrità — chiunque modifichi il payload invalida la firma |

**Importante:** il payload NON è criptato, solo codificato in Base64URL. Non mettere dati sensibili nel JWT (password, dati personali).

### Claim `role`

In [[TaskMngr]] il ruolo (`USER` o `ADMIN`) è un claim custom nel JWT. Alla generazione:

```java
.claim("role", user.getRole().name())
```

Alla validazione, il `JwtAuthenticationFilter` estrae il claim e lo converte in `GrantedAuthority` con prefisso `ROLE_`:

```java
String role = claims.get("role", String.class);
var authorities = List.of(new SimpleGrantedAuthority("ROLE_" + role));
```

Questo permette a Spring Security di usare `hasRole('ADMIN')` e `hasRole('USER')` sia nelle regole url-based che in `@PreAuthorize`.

## Generazione

```java
String token = Jwts.builder()
    .subject(user.getEmail())                      // sub → email (identificatore unico)
    .claim("id", user.getId())                     // id → userId (custom)
    .claim("role", user.getRole().name())          // role → USER | ADMIN (custom)
    .issuedAt(new Date())                          // iat → momento emissione
    .expiration(new Date(System.currentTimeMillis() + 86400000))  // exp → 24 ore
    .signWith(getSigningKey())                     // firma HMAC con chiave segreta
    .compact();
```

`getSigningKey()` legge la chiave da `app.jwt.secret` nelle [[BE-NOTES/Java/Spring/Boot/Application Properties|properties]] — mai hardcodata.

## Validazione

Il `JwtFilter` (un `OncePerRequestFilter`) intercetta ogni richiesta:

```java
@Component
public class JwtFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) {

        // 1. Estrai header Authorization
        String header = request.getHeader("Authorization");
        if (header == null || !header.startsWith("Bearer ")) {
            chain.doFilter(request, response);
            return;  // nessun token → passa al prossimo filter (che richiederà auth)
        }

        // 2. Estrai token (dopo "Bearer ")
        String token = header.substring(7);

        try {
            // 3. Verifica firma e scadenza, estrai claims
            Claims claims = Jwts.parser()
                .verifyWith(getSigningKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();

            // 4. Estrai userId dai claims
            Long userId = Long.parseLong(claims.getSubject());

            // 5. Crea authentication token e impostalo nel SecurityContext
            var authentication = new UsernamePasswordAuthenticationToken(
                userId, null, getAuthorities(claims));
            SecurityContextHolder.getContext().setAuthentication(authentication);

        } catch (JwtException e) {
            // Token non valido o scaduto → SecurityContext vuoto (401)
            SecurityContextHolder.clearContext();
        }

        chain.doFilter(request, response);
    }
}
```

## Refresh Token

Il JWT ha una scadenza breve per limitare i danni in caso di furto. Per non costringere l'utente a rifare login ogni 30 minuti, usiamo un **refresh token**:

| Token | Durata | Dove è memorizzato | Cosa contiene |
|---|---|---|---|
| Access token | 15-30 min | Memoria (Angular) | userId, roles |
| Refresh token | 7-30 giorni | HttpOnly cookie o localStorage | Solo userId + serie |

Flusso:
1. Client chiama `/api/auth/login` → riceve access + refresh token
2. Client usa access token per le richieste normali
3. Quando access token scade (HTTP 401) → client chiama `/api/auth/refresh` con refresh token
4. Server verifica refresh token → emette nuovo access token
5. Se anche refresh token è scaduto → login completo

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Chiave JWT hardcodata nel codice | Secret in VCS, leak in repo pubblico | La chiave è una stringa letterale in Java | Leggi da `application.properties` o variabile d'ambiente; mai hardcodare |
| Dati sensibili nel payload JWT | Password, dati personali in chiaro nel token | Il payload JWT è solo Base64URL, non criptato | Mai mettere dati sensibili nel payload — solo identificatori non sensibili |
| Token scaduto ma nessun refresh | Utente reindirizzato al login ogni 24h | Access token scade ma manca il meccanismo di refresh | Implementa refresh token; configura `expiration` ragionevole per l'uso (15-30 min per alta sicurezza, 24h per task app) |
| `JwtException` non gestita | 500 Internal Server Error per token invalido | Il filtro non cattura `MalformedJwtException`, `ExpiredJwtException` | Avvolgi il parsing in try-catch `JwtException` → `SecurityContextHolder.clearContext()` + 401 |
| Dimenticare il prefisso `Bearer ` | 401: header Authorization non parsato | Il filtro cerca "Bearer " ma l'header arriva come "token" diretto | Il client DEVE inviare `Authorization: Bearer <token>`; il filtro controlla il prefisso |
| Secret troppo corto (< 256 bit per HS256) | `WeakKeyException` o warning di sicurezza | HMAC-SHA256 richiede chiave di almeno 256 bit (32 caratteri) | Usa una chiave di almeno 32 caratteri alfanumerici casuali |

## In TaskMngr

- Access token: 24h (configurabile via `app.jwt.expiration-ms`)
- Refresh token: 7 giorni, memorizzato in localStorage
- `/api/auth/refresh` accetta refresh token nel body, restituisce nuovo access token
- Alla logout, il token viene aggiunto alla [[BE-NOTES/Java/Spring/Security/Token Blacklist|blacklist]]
