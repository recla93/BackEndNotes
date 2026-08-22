---
topic: "Lock Ottimistico e Pessimistico"
parent: "[[BE-NOTES/Java/Spring/Data/Spring Data JPA|Spring Data JPA]]"
nav_prev: "[[Hibernate/HQL - Hibernate Query Language.md]]"
nav_next: "[[JPA Auditing.md]]"
---


Quando due utenti modificano la stessa risorsa contemporaneamente, l'ultimo a salvare sovrascrive le modifiche dell'altro (**lost update**). Il locking previene questo problema. La scelta tra ottimistico e pessimistico dipende dalla frequenza delle collisioni e dalla criticità dell'operazione.

## Lock Ottimistico — presuppone che non ci siano conflitti

**Meccanismo**: ogni entità ha un campo `@Version` che viene incrementato a ogni update. Se due richieste leggono la versione 1 e tentano entrambe di aggiornare, la prima riesce (versione → 2), la seconda fallisce perché la versione in DB è 2, non 1.

```java
@Entity
public class Team {
    @Version
    private Long version;   // parte da 0 o 1, incrementato automaticamente
}
```

**Quando usarlo:**
- **Risorse con poche collisioni** — profili utente, task individuali
- **Operazioni frequenti di lettura** — non blocca mai il database
- **Team membership** — due utenti raramente modificano lo stesso team contemporaneamente

**Cosa succede in caso di conflitto:**
```
1. Utente A e B leggono Team(id=5, version=1)
2. A salva: version → 2 (OK)
3. B salva: OptimisticLockException (versione attesa 1, trovata 2)
4. → HTTP 409 Conflict
5. B deve ricaricare i dati e riproporre la modifica
```

**Vantaggi**: nessun lock sul database, massima concorrenza in lettura.
**Svantaggi**: il client deve gestire il 409 e riprovare.

## Lock Pessimistico — blocca la risorsa subito

**Meccanismo**: quando leggi un'entità con `PESSIMISTIC_WRITE`, il database esegue `SELECT ... FOR UPDATE`, che blocca la riga fino al termine della transazione. Altri tentativi di lettura/scrittura restano in attesa.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT t FROM Team t WHERE t.id = :id")
Optional<Team> findByIdWithLock(@Param("id") Long id);
```

**Quando usarlo:**
- **Risorse ad alta contesa** — slot unici, numeri di serie, inventario
- **Operazioni atomiche critiche** — "leggi saldo, verifica, addebita" (transazione finanziaria)
- **Aggiornamenti team in TaskMngr** — quando più admin potrebbero modificare la stessa configurazione

**Cosa succede:**
```
1. A legge Team(id=5) CON LOCK → riga bloccata
2. B tenta di leggere Team(id=5) CON LOCK → B rimane in attesa
3. A completa la transazione → lock rilasciato
4. B può leggere (e vede i dati aggiornati da A)
```

**Vantaggi**: nessun conflitto runtime, comportamento deterministico.
**Svantaggi**: riduce la concorrenza, rischio deadlock se non si rilascia in tempo, richiede connessione DB attiva per tutta la transazione.

## Tabella riassuntiva

| Caratteristica | Ottimistico | Pessimistico |
|---|---|---|
| Quando blocca | Al commit | Alla lettura |
| Collisioni | Lancia eccezione (409) | Attesa automatica |
| Concorrenza | Alta (nessun lock) | Bassa (righe bloccate) |
| Performance lettura | Massima | Media (attesa lock) |
| Complessità client | Deve gestire retry | Trasparente |
| Rischio | Client mal gestito perde dati | Deadlock |

## Transazioni — prerequisite

Entrambi i meccanismi richiedono una transazione **esplicita**:
```java
@Service
public class TeamService {
    @Transactional
    public Team updateTeam(Long id, TeamUpdateRequest request) {
        Team team = teamRepository.findByIdWithLock(id)
            .orElseThrow(() -> new ResourceNotFoundException("Team", id));
        team.setName(request.name());
        return teamRepository.save(team);
        // lock rilasciato quando il metodo termina
    }
}
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| `@Version` su entità senza transazione | Il version non viene mai incrementato | Il lock ottimistico richiede una transazione attiva | Aggiungi `@Transactional` sul metodo che modifica l'entità |
| `OptimisticLockException` non gestita | Errore 500 invece di 409 | Nessun handler cattura l'eccezione | Aggiungi handler nel `@RestControllerAdvice` → HTTP 409 |
| Lock pessimistico senza transazione | Lock non acquisito (la lettura è fuori transazione) | `@Lock(PESSIMISTIC_WRITE)` richiede una transazione attiva | Aggiungi `@Transactional` sul metodo chiamante |
| Deadlock per lock acquisiti in ordine diverso | `PessimisticLockException` o timeout | Due transazioni bloccano risorse in ordine inverso | Acquisisci i lock sempre nello STESSO ordine (es. per ID crescente) |
| Lock pessimistico tenuto per operazioni lente | Pool connessioni esaurito, timeout DB | La transazione include chiamate HTTP lente o I/O su file | Riduci la transazione alla sola operazione DB; sposta logica lenta fuori |
| `@Version` su campo non supportato | Errore a runtime | `@Version` accetta solo `int`, `Integer`, `short`, `Short`, `long`, `Long`, `Timestamp` | Usa `Long` come tipo per la versione |

## In TaskMngr

- `@Version` su tutte le entità principali — rilevamento conflitti per modifiche concorrenti generiche
- `PESSIMISTIC_WRITE` su `TeamRepository.findByIdWithLock()` — l'aggiornamento dei team è un'operazione sensibile
- `OptimisticLockException` → mappata a 409 Conflict nel [[BE-NOTES/Java/Spring/Web/Global Exception Handler|Global Exception Handler]]
- `PessimisticLockException` → 409 o 503 a seconda del contesto
