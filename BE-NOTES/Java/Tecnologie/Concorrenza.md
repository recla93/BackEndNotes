---
topic: "Concorrenza — Java"
parent: "[[BE-NOTES/Java/Java|Java]]"
---

Java offre supporto nativo alla concorrenza con thread, synchronized, Lock, ExecutorService, CompletableFuture, e strutture dati thread-safe (`ConcurrentHashMap`, `CopyOnWriteArrayList`).

A differenza di JavaScript (single-thread con event loop), Java permette esecuzione parallela reale su piu core. A differenza di Python (GIL), i thread Java sono paralleli per CPU-bound.

## Thread — base

```java
// Con Thread (sconsigliato per codice nuovo)
Thread t = new Thread(() -> {
    System.out.println("Ciao da thread: " + Thread.currentThread().getName());
});
t.start();           // avvia
t.join();            // aspetta che finisca (lancia InterruptedException)

// Thread con nome
Thread named = new Thread(() -> lavoro(), "worker-1");
```

Creare thread direttamente e sconsigliato per carichi di lavoro. I thread costano risorse (stack ~1MB ciascuno). Preferisci `ExecutorService`.

## ExecutorService — pool di thread

```java
import java.util.concurrent.*;

// Pool fisso: 4 thread
ExecutorService executor = Executors.newFixedThreadPool(4);

// Pool variabile: crea thread su richiesta, ricicla quelli inattivi
ExecutorService cached = Executors.newCachedThreadPool();

// Singolo thread (esecuzione sequenziale)
ExecutorService single = Executors.newSingleThreadExecutor();

// Submit task
Future<String> future = executor.submit(() -> {
    Thread.sleep(1000);
    return "Risultato";
});

// Bloccante fino al risultato
String result = future.get(5, TimeUnit.SECONDS);

// Spegnimento
executor.shutdown();            // non accetta nuovi task, aspetta quelli in corso
executor.awaitTermination(10, TimeUnit.SECONDS);  // aspetta terminazione
// executor.shutdownNow();      // interrompe forzatamente
```

`ExecutorService` separa la creazione di task dall'esecuzione. `Future.get()` e bloccante. Principali pool: `fixedThreadPool` (carichi prevedibili), `cachedThreadPool` (picchi brevi), `singleThreadExecutor` (coda FIFO).

## Synchronized — sezione critica

```java
public class Contatore {
    private int count = 0;

    // Metodo sincronizzato (lock sull'istanza)
    public synchronized void incrementa() {
        count++;
    }

    // Blocco sincronizzato (lock esplicito)
    public void decrementa() {
        synchronized (this) {
            count--;
        }
    }

    // Blocco con lock object separato
    private final Object lock = new Object();
    public void operazione() {
        synchronized (lock) {
            // solo questo blocco e protetto
        }
    }
}
```

`synchronized` garantisce mutua esclusione e visibilita della memoria. `synchronized` su metodo statico usa il lock della classe. Usa lock object separati per proteggere risorse diverse con lock diversi.

## Lock esplicito (java.util.concurrent.locks)

```java
import java.util.concurrent.locks.*;

private final ReentrantLock lock = new ReentrantLock();

public void operazione() {
    lock.lock();
    try {
        // sezione critica
    } finally {
        lock.unlock();  // sempre nel finally!
    }
}

// ReadWriteLock: letture multiple, scrittura esclusiva
private final ReadWriteLock rwLock = new ReentrantReadWriteLock();

public void leggi() {
    rwLock.readLock().lock();
    try { /* lettura */ }
    finally { rwLock.readLock().unlock(); }
}

public void scrivi() {
    rwLock.writeLock().lock();
    try { /* scrittura */ }
    finally { rwLock.writeLock().unlock(); }
}
```

`ReentrantLock` e piu flessibile di `synchronized`: tryLock, lock interruptible, fair ordering. `ReadWriteLock` permette letture parallele quando nessuno scrive. `synchronized` e sufficiente per l'80% dei casi; Lock serve per esigenze avanzate.

## Strutture dati thread-safe

```java
// HashMap thread-safe
Map<String, String> map = new ConcurrentHashMap<>();
map.put("key", "value");
map.computeIfAbsent("key2", k -> compute(k));  // atomico

// Lista thread-safe
List<String> list = new CopyOnWriteArrayList<>();  // per molti read, pochi write

// Set thread-safe
Set<String> set = ConcurrentHashMap.newKeySet();

// Coda thread-safe per produttore-consumatore
BlockingQueue<String> queue = new LinkedBlockingQueue<>(100);
queue.put("item");           // bloccante se piena
String item = queue.take();  // bloccante se vuota
```

`ConcurrentHashMap` e la scelta predefinita per mappe condivise tra thread. `CopyOnWriteArrayList` e ottima per listener/list (iterazione frequente, modifica rara). `BlockingQueue` e il pattern produttore-consumatore standard.

## CompletableFuture — async moderno

```java
import java.util.concurrent.CompletableFuture;

// Esecuzione asincrona
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "risultato";
}, executor);

// Pipeline di trasformazioni (non bloccanti)
CompletableFuture<Integer> pipeline = CompletableFuture
    .supplyAsync(() -> "42")
    .thenApply(Integer::parseInt)
    .thenApply(n -> n * 2)
    .exceptionally(ex -> {
        log.error("Errore", ex);
        return 0;
    });

// Combinare due future
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "A");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "B");
f1.thenCombine(f2, (a, b) -> a + b);  // "AB"

// Aspettare tutti
CompletableFuture.allOf(f1, f2).join();

// Timeout (Java 9+)
future.orTimeout(5, TimeUnit.SECONDS);
```

`CompletableFuture` permette pipeline asincrone senza thread bloccati. `thenApply()` trasforma, `thenCompose()` appiattisce future annidati, `exceptionally()` gestisce errori. `allOf()` e `anyOf()` combinano piu future.

## Parallel Stream

```java
// Stream parallelo (usa ForkJoinPool common pool)
List<Integer> risultati = lista.parallelStream()
    .filter(x -> x > 10)
    .map(x -> computeExpensive(x))  // eseguito in parallelo
    .collect(Collectors.toList());

// ForkJoinPool custom (Java 8+)
ForkJoinPool customPool = new ForkJoinPool(8);
try {
    customPool.submit(() ->
        lista.parallelStream().forEach(this::processa)
    ).get();
} finally {
    customPool.shutdown();
}
```

`parallelStream()` usa il common ForkJoinPool (con tanti thread quanti core CPU). **Non usare** per I/O-bound (rete, DB, file): blocca i thread del pool. Usa `CompletableFuture` con executor custom per I/O.

## volatile e atomic

```java
// volatile: visibilita garantita (non atomicita)
private volatile boolean running = true;

public void stop() { running = false; }
public void run() {
    while (running) { /* lavoro */ }
}

// Atomic: operazioni atomiche senza lock
private AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();   // atomico: thread-safe senza synchronized
counter.addAndGet(5);
counter.compareAndSet(10, 20);  // CAS (Compare-And-Swap)
```

`volatile` garantisce che le letture/scritture siano visibili tra thread (previene caching CPU). Non garantisce atomicita per operazioni composte (es. `count++`). `AtomicInteger`, `AtomicLong`, `AtomicBoolean`, `AtomicReference` forniscono operazioni CAS atomiche.

## Errori comuni

- **Thread.start() dimenticato**: `new Thread(runnable)` senza `start()` non esegue nulla.
- **Dimenticare `try-finally` per Lock**: se `lock()` e prima del try, e `unlock()` va sempre in `finally`.
- **Deadlock**: due thread che si aspettano lock a vicenda. Usa sempre lo stesso ordine di acquisizione lock.
- **Shared mutable state senza sincronizzazione**: due thread modificano la stessa variabile senza `synchronized`/`Lock`/`Atomic`.
- **`parallelStream()` per I/O**: blocca tutto il pool comune. Usa CompletableFuture con executor dedicato.
- **ForkJoinPool common pool esausto**: task bloccanti (sleep, I/O) consumano tutti i thread. Usa pool separato.

## Best Practices & Conventions

- Preferisci **`ExecutorService`** a thread espliciti.
- Usa **`CompletableFuture`** per async pipeline moderne.
- Per I/O-bound: CompletableFuture con executor dedicato (`newFixedThreadPool` dimensionato sulle risorse I/O).
- Per CPU-bound: parallelStream o `newFixedThreadPool` con `Runtime.getRuntime().availableProcessors()`.
- Usa **`ConcurrentHashMap`** invece di `HashMap` con `synchronized`.
- Mantieni le sezioni critiche piu piccole possibile.
- Preferisci strutture dati immutabili condivise (nessuna sincronizzazione necessaria).
