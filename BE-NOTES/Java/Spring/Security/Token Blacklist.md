---
topic: "Token Blacklist"
parent: "[[BE-NOTES/Java/Spring/Security/Spring Security|Spring Security]]"
nav_prev: "[[Authorities e RBAC.md]]"
---


Il JWT è **intrinsecamente non revocabile**: una volta emesso, è valido fino alla scadenza. Se un utente fa logout, il token continua a funzionare. La blacklist risolve questo problema: teniamo traccia dei token revocati e li rifiutiamo nel filtro JWT.

## Quando serve

- **Logout esplicito** — l'utente clicca "Logout" → il token corrente non deve più funzionare
- **Cambio password** — tutti i token emessi prima del cambio password devono essere invalidati
- **Sospensione utente** — admin blocca un account → i token attivi devono essere rifiutati
- **Client rubato** — se un dispositivo viene perso, l'utente vuole invalidare tutti i token

## Implementazione in-memory (TaskMngr)

```java
@Component
public class TokenBlacklist {
    // ConcurrentHashMap.newKeySet() = Set thread-safe ottimizzato per concorrenza
    private final Set<String> blacklist = ConcurrentHashMap.newKeySet();

    public void blacklist(String token) {
        blacklist.add(token);
    }

    public boolean isBlacklisted(String token) {
        return blacklist.contains(token);
    }

    @Scheduled(fixedRate = 3600000)  // ogni ora
    public void cleanup() {
        // Opzionale: rimuovi token scaduti (se memorizzi exp)
        log.info("Blacklist size: {}", blacklist.size());
    }
}
```

Nel `JwtFilter`, dopo aver validato il JWT, controlliamo la blacklist:

```java
if (jwtService.isBlacklisted(token)) {
    SecurityContextHolder.clearContext();
    response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Token revocato");
    return;
}
```

## Limiti dell'approccio in-memory

L'implementazione in-memory è semplice, ma:
- **I token si perdono al restart** del server — se l'applicazione viene riavviata, tutti i token diventano validi di nuovo (tranne quelli scaduti)
- **Non scala** — in un cluster con più istanze, ogni istanza ha la propria blacklist
- **Memoria** — potenzialmente molti token in memoria

## Varianti per produzione

| Approccio | Vantaggi | Svantaggi |
|---|---|---|
| **In-memory** | Semplice, nessuna dipendenza esterna | Non persiste, non scala |
| **Redis** | Persistente, TTL nativo, scalabile, condiviso tra istanze | Richiede Redis, latenza di rete |
| **Database** | Persistente, sempre consistente | Lento per ogni richiesta (I/O) |
| **JWT short-lived + refresh** | Nessuna blacklist, token durano pochi minuti | Refresh token deve essere revocabile |

## Soluzione consigliata per produzione

Usa **Redis** con TTL automatico:

```java
// Set con scadenza → Redis elimina automaticamente il token dopo exp
void blacklist(String token, long ttlSeconds) {
    redisTemplate.opsForValue().set("bl:" + token, "true", ttlSeconds, TimeUnit.SECONDS);
}

boolean isBlacklisted(String token) {
    return redisTemplate.hasKey("bl:" + token);
}
```

Il TTL della blacklist dovrebbe essere uguale alla durata massima del token — se un token scade in 24h, non serve tenerlo in blacklist per più di 24h.

## Flusso logout completo (TaskMngr)

```
┌────────────┐  POST /api/auth/logout   ┌────────────────┐
│  Client    │ ────── Authorization: ──→ │ AuthController │
│            │        Bearer <token>     │                │
└────────────┘                           └───────┬────────┘
                                                  │ authService.logout(token)
                                                  ▼
                                         ┌─────────────────┐
                                         │  AuthServiceImpl │
                                         │                  │
                                         │ 1. jwtUtil       │
                                         │    .getExpiration│
                                         │    FromToken()   │
                                         │                  │
                                         │ 2. tokenBlacklist│
                                         │    .blacklist(   │
                                         │     token,expiry)│
                                         └────────┬─────────┘
                                                  │
                                                  ▼
                                         ┌─────────────────┐
                                         │TokenBlacklistSvc │
                                         │                  │
                                         │ token → expiryMs │
                                         │ (ConcurrentHashMap)│
                                         └─────────────────┘

  Ogni richiesta successiva:
  ┌──────────┐  qualsiasi richiesta    ┌────────────────────┐
  │  Client  │ ──────────────────────→ │ JwtAuthentication  │
  │(loggato) │    Authorization: Bearer │ Filter              │
  └──────────┘                         │                     │
                                       │ 1. validate JWT     │
                                       │ 2. tokenBlacklist   │
                                       │    .isBlacklisted() │
                                       │    (se SI → 401)    │
                                       │ 3. SecurityContext  │
                                       └────────────────────┘
```

## Blacklistare un token: significato

"Blacklistare" significa **aggiungere il token a una lista nera** (`ConcurrentHashMap<String, Long>`) in modo che, pur essendo formalmente valido (firma HMAC corretta, non scaduto), il server lo rifiuti comunque. Il JWT non viene "cancellato" o "distrutto" — semplicemente il filtro JWT controlla la blacklist **dopo** aver validato la firma.

### Implementazione reale in TaskMngr

```java
// TokenBlacklistService.java
@Service
public class TokenBlacklistService
{
    private final Map<String, Long> blacklist = new ConcurrentHashMap<>();
    //                token    →  expiration (millis)

    // Aggiunge token + timestamp di scadenza
    public void blacklist(String token, long expiryMs) {
        blacklist.put(token, expiryMs);
    }

    // Controlla e fa cleanup lazy degli scaduti
    public boolean isBlacklisted(String token) {
        cleanup();                    // rimuove token scaduti
        return blacklist.containsKey(token);
    }

    // RemoveIf: scorre la mappa e cancella le entry expired
    private void cleanup() {
        long now = System.currentTimeMillis();
        blacklist.values().removeIf(expiry -> expiry < now);
    }
}
```

Il cleanup è **lazy** — non c'è un thread schedulato. Viene eseguito ogni volta che `isBlacklisted()` viene chiamato (cioè ad ogni richiesta autenticata). I token scaduti vengono rimossi automaticamente, quindi la mappa non cresce all'infinito.

### Logout endpoint

```java
// AuthController.java
@PostMapping("/logout")
public ResponseEntity<ApiResponse<Void>> logout(
        @RequestHeader("Authorization") String authHeader)
{
    String token = authHeader.substring(7);  // toglie "Bearer "
    authService.logout(token);
    return ResponseEntity.ok(
        ApiResponse.success("Logout effettuato con successo"));
}
```

```java
// AuthServiceImpl.java
public void logout(String token)
{
    Date expiry = jwtUtil.getExpirationFromToken(token);
    if (expiry != null)
        tokenBlacklistService.blacklist(token, expiry.getTime());
}
```

### Perché serve l'expiry?

Il `TokenBlacklistService.blacklist(token, expiryMs)` salva il **timestamp di scadenza** del token. Quando `cleanup()` viene eseguito, rimuove solo i token la cui expiration è passata. Così:
- I token in blacklist ma **non ancora scaduti** → vengono rifiutati (logout attivo)
- I token **scaduti** → vengono rimossi dalla mappa (nessun memory leak)

### Perché non basta invalidare il token lato client?

Il client potrebbe cancellare il token dal proprio storage, ma se un attaccante lo ha intercettato (XSS, log, backup), può ancora usarlo. La blacklist server-side è l'unico modo per garantire che il logout sia effettivo.

## Limiti dell'approccio in-memory (TaskMngr attuale)

| Problema | Impatto |
|---|---|
| **Perdita al restart** | Se il server riparte, la blacklist svuota → token tornano validi |
| **Non scala** | Ogni istanza ha la propria mappa → cluster incoerente |
| **Memoria** | Potenzialmente migliaia di token in ConcurrentHashMap |

Per produzione la soluzione è Redis con TTL nativo (come descritto sopra).

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Blacklist non controllata nel JWT filter | Token revocato ancora valido | Il filtro JWT valida la firma ma non controlla la blacklist | Aggiungi check `isBlacklisted(token)` dopo la validazione della firma |
| Pulizia blacklist mai eseguita | `OutOfMemoryError` dopo giorni di attività | La ConcurrentHashMap cresce all'infinito senza rimuovere token scaduti | Implementa cleanup periodico (es. `@Scheduled`) o lazy cleanup in `isBlacklisted()` |
| Token memorizzato interamente in blacklist | Memoria sprecata per token lunghi | Il JWT può essere > 1KB, la blacklist tiene l'intera stringa | Memorizza l'hash del token (SHA-256) invece del token completo |
| Blacklist in-memory in cluster multi-istanza | Token revocato su istanza A ma valido su istanza B | Ogni istanza ha la propria ConcurrentHashMap | Usa Redis condiviso o database per la blacklist in produzione |
| Blacklistare il refresh token ma non l'access token | L'access token continua a funzionare fino a scadenza | Solo il refresh token viene invalidato | Blacklista ANCHE l'access token corrente al logout |

## Vedi anche

- [[BE-NOTES/Java/Spring/Security/JWT - Generazione e Validazione|JWT — Generazione e Validazione]] — come il JWT viene generato e validato
- [[BE-NOTES/Java/Spring/Security/SecurityConfig e Filter Chain|SecurityConfig e Filter Chain]] — dove il filtro JWT è configurato
- [[BE-NOTES/Java/Spring/Security/Authorities e RBAC|Authorities e RBAC]] — autorizzazioni basate su ruolo
