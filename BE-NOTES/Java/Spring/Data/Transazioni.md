---
topic: "Transazioni in Spring Boot"
parent: "[[BE-NOTES/Java/Spring/Spring|Spring]]"
---

Spring gestisce transazioni dichiarative via `@Transactional`. Ogni metodo marcato esegue in una transazione: se fallisce, tutto viene rollbackato; se ha successo, tutto viene committato.

Spring usa proxy AOP per avvolgere il metodo: apre la transazione prima, committa/rollbacka dopo. Il proxy funziona solo per chiamate esterne (non chiamate interne allo stesso bean).

## @Transactional base

```java
import org.springframework.transaction.annotation.Transactional;
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    @Transactional
    public Order createOrder(CreateOrder request) {
        Order order = orderRepository.save(new Order(request));
        paymentService.charge(request.getPayment());
        inventoryService.reserve(request.getItems());
        return order;
    }
}
```

`@Transactional` apre una transazione all'inizio del metodo e la committa alla fine. Se una qualsiasi eccezione non controllata (`RuntimeException`) viene lanciata, tutto viene rollbackato. Le operazioni dentro il metodo condividono la stessa connessione DB e isolamento.

## Rollback

```java
// Rollback per tutte le eccezioni
@Transactional(rollbackFor = Exception.class)
public void createOrder(CreateOrder request) throws Exception {
    orderRepository.save(new Order(request));
    paymentService.charge(request.getPayment());
}

// Rollback per eccezioni specifiche
@Transactional(rollbackFor = {DataAccessException.class, PaymentException.class})
public void processPayment(PaymentRequest request) {
    paymentRepository.save(new Payment(request));
    paymentGateway.charge(request);
}

// No rollback per eccezioni specifiche
@Transactional(noRollbackFor = BusinessWarningException.class)
public void updateInventory(UpdateRequest request) {
    inventoryRepository.decrement(request.getItems());
    if (request.isPartial()) {
        throw new BusinessWarningException("Scorte parziali");
    }
}
```

Per default, `@Transactional` rollbacka solo per `RuntimeException` ed `Error`. `rollbackFor` estende il rollback a checked exception. `noRollbackFor` esclude eccezioni specifiche dal rollback (warning, notifiche non critiche).

## Propagazione

```java
// Default: usa transazione esistente, ne crea una nuova se assente
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() { methodB(); }

// Crea sempre una nuova transazione, sospende l'esistente
// ATTENZIONE: sospende anche il rollback
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodB() { ... }

// Esegue senza transazione
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void logOperation() { ... }

// Esegue obbligatoriamente dentro una transazione esistente (fallisce se assente)
@Transactional(propagation = Propagation.MANDATORY)
public void nestedOperation() { ... }

// Nesting (savepoint, rollback parziale)
@Transactional(propagation = Propagation.NESTED)
public void subOperation() { ... }
```

`REQUIRED` (default) riutilizza la transazione corrente. `REQUIRES_NEW` crea una nuova transazione indipendente: se fallisce, non impatta la transazione chiamante. `MANDATORY` fallisce se non c'e transazione. `NESTED` usa JDBC savepoint (rollback parziale).

## Isolamento

```java
// Evita dirty read
@Transactional(isolation = Isolation.READ_COMMITTED)
public Order getOrder(Long id) {
    return orderRepository.findById(id).orElseThrow();
}

// Evita dirty read e non-repeatable read
@Transactional(isolation = Isolation.REPEATABLE_READ)
public List<Order> getOrdersByStatus(String status) {
    return orderRepository.findByStatus(status);
}

// Isolamento massimo (serializzabile, piu lento)
@Transactional(isolation = Isolation.SERIALIZABLE)
public void reserveConcurrentStock(Long productId, int quantity) {
    Stock stock = stockRepository.findById(productId).orElseThrow();
    if (stock.getQuantity() >= quantity) {
        stock.setQuantity(stock.getQuantity() - quantity);
        stockRepository.save(stock);
    }
}
```

`READ_COMMITTED` (default in PostgreSQL, SQL Server). `REPEATABLE_READ` (default MySQL). `SERIALIZABLE` massimo isolamento ma peggior concorrenza. Scegli il livello minimo sufficiente: `READ_COMMITTED` basta per la maggior parte dei casi.

## Timeout e readonly

```java
// Timeout dopo 5 secondi
@Transactional(timeout = 5)
public void slowBatchOperation() {
    // Se impiega >5s, TransactionTimedOutException
    processLargeDataset();
}

// Read-Only (ottimizzazione per repliche DB)
@Transactional(readOnly = true)
public UserDto getUser(Long id) {
    // Hint per Hibernate: non carica dirty checking
    return repository.findById(id).orElseThrow();
}

// Combinazione
@Transactional(readOnly = true, timeout = 10)
public Page<Order> searchOrders(String query, Pageable pageable) {
    return orderRepository.search(query, pageable);
}
```

`timeout` annulla la transazione se supera il limite. `readOnly = true` ottimizza le query (Hibernate skip dirty checking, DB puo usare replica). Non rende la lettura immutabile: puoi comunque scrivere, ma perdi l'ottimizzazione.

## Chiamata interna (proxy bypass)

```java
@Service
public class OrderService {

    // ❌ Chiamata diretta — bypassa proxy, nessuna transazione
    public void processOrder(Long id) {
        updateOrder(id);  // @Transactional IGNORATO!
    }

    @Transactional
    public void updateOrder(Long id) {
        // Non esegue in transazione se chiamato da processOrder
        orderRepository.updateStatus(id, "PROCESSED");
    }

    // ✅ Soluzione: auto-iniezione
    @Autowired
    private OrderService self; // proxy iniettato

    public void processOrderFixed(Long id) {
        self.updateOrder(id);  // Passa dal proxy -> transazione attiva
    }
}
```

Proxy AOP di Spring funziona solo per chiamate esterne. Una chiamata interna allo stesso bean bypassa il proxy e `@Transactional` viene ignorato. Soluzione: auto-iniezione del bean (proxy iniettato), o estrai il metodo in un bean separato.

## Test transazionali

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    @Transactional
    void saveAndFind() {
        User user = new User("Alice");
        userRepository.save(user);

        assertThat(userRepository.findAll()).hasSize(1);
    }
    // Rollback automatico dopo il test
}
```

`@DataJpaTest` e transazionale per default: ogni test viene rollbackato alla fine. `@Transactional` su test garantisce isolamento. Non devi pulire i dati dopo ogni test — Spring lo fa automaticamente.

## Transazioni e lock pessimistico

```java
import jakarta.persistence.LockModeType;
import org.springframework.data.jpa.repository.Lock;
import org.springframework.data.jpa.repository.Query;

public interface StockRepository extends JpaRepository<Stock, Long> {

    // Lock pessimistico: blocca la riga fino al commit
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT s FROM Stock s WHERE s.productId = :productId")
    Optional<Stock> findByIdLocked(@Param("productId") Long productId);
}

@Service
public class InventoryService {

    @Transactional
    public void reserveProduct(Long productId, int quantity) {
        Stock stock = stockRepository.findByIdLocked(productId)
            .orElseThrow();
        // Lock acquisito: nessun altro thread modifica questa riga
        if (stock.getQuantity() >= quantity) {
            stock.setQuantity(stock.getQuantity() - quantity);
            stockRepository.save(stock);
        }
    }
}
```

`@Lock(PESSIMISTIC_WRITE)` blocca la riga nel DB (`SELECT ... FOR UPDATE`). Previene race condition su risorse contese. Usalo solo quando necessario: riduce la concorrenza. Alternativa: `@Version` per optimistic locking (piu performante).

## Errori comuni

- **@Transactional ignorato**: chiamata interna allo stesso bean bypassa il proxy.
- **Rollback non attivo per checked exception**: `@Transactional` rollbacka solo `RuntimeException`. Usa `rollbackFor`.
- **Aprire transazione per sola lettura**: costo inutile. Usa `readOnly = true` per ottimizzare.
- **REQUIRES_NEW inaspettato**: la nuova transazione non vede modifiche della transazione padre (non committate).
- **Timeout default infinito**: transazioni lunghe bloccano connessioni e lock. Imposta sempre `timeout`.
- **Stale data in transactioni lunghe**: READ_COMMITTED non rilegge i dati. Usa REPEATABLE_READ se necessario.
- **Dimenticare @EnableTransactionManagement**: necessario solo se non usi Spring Boot auto-configuration.

## Best Practices & Conventions

- Usa `@Transactional(readOnly = true)` su metodi di sola lettura.
- Imposta `timeout` su transazioni che chiamano API esterne o processano batch.
- Usa `rollbackFor` per checked exception che devono causare rollback.
- Non chiamare metodi `@Transactional` internamente allo stesso bean.
- Preferisci `REQUIRES_NEW` per operazioni che devono essere isolate (log, audit).
- Per alta concorrenza, usa optimistic locking (`@Version`) invece di pessimistico.
- Mantieni le transazioni brevi: il lock sulle risorse e costoso.
