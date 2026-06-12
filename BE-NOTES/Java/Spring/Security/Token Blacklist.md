---
topic: "Token Blacklist"
parent: "[[BE-NOTES/Java/Spring/Security/Spring Security|Spring Security]]"
---

# Token Blacklist

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

## In TaskMngr

- Blacklist in-memory (sufficiente per sviluppo e MVP)
- Logout → aggiunge token corrente alla blacklist
- `@Scheduled` per monitoraggio periodico della dimensione
- Migrazione a Redis pianificata per produzione

## Vedi anche

- [[BE-NOTES/Java/Spring/Security/JWT - Generazione e Validazione|JWT — Generazione e Validazione]] — come il JWT viene generato e validato
