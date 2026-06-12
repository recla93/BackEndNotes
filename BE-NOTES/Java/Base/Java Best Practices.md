---
topic: "Java Best Practices — codice pulito, sicuro, performante"
parent: "[[BE-NOTES/Java/Java|Java]]"
---

# Java Best Practices

Convenzioni e regole per scrivere Java **leggibile, robusto e performante** — indipendentemente dal framework.

---

## 1. Naming e Stile

### 1.1 Convenzioni

| Cosa | Convenzione | Esempio |
|---|---|---|
| Classi | PascalCase, sostantivi | `UserService`, `TaskRepository` |
| Interfacce | PascalCase, aggettivi o -able | `Serializable`, `Comparable`, `UserMapper` |
| Metodi | camelCase, verbi | `findByEmail()`, `deleteAllExpired()` |
| Variabili | camelCase | `totalCount`, `userName` |
| Costanti | UPPER_SNAKE_CASE | `MAX_POOL_SIZE`, `DEFAULT_PAGE` |
| Enum | UPPER_SNAKE_CASE | `USER`, `ADMIN`, `PENDING` |
| Package | lowercase, dominio inverso | `it.president.taskmngr.*` |
| Parametri tipo | singola lettera maiuscola | `T`, `E`, `K`, `V`, `R` |

### 1.2 Regole

- Nomi **espressivi**: `int elapsedTimeInDays` non `int d`
- Booleani con prefisso: `isActive`, `hasPermission`, `canDelete`
- Evita abbreviazioni: `calculateTotal()` non `calcTot()`
- **Non** usare prefisso `I` per interfacce (`UserService` non `IUserService`)

---

## 2. Gestione Null

### 2.1 Mai restituire null da metodi pubblici

```java
// ❌ Cattivo
public User findById(Long id) {
    return userMap.get(id);  // può restituire null
}

// ✅ Buono: Optional
public Optional<User> findById(Long id) {
    return Optional.ofNullable(userMap.get(id));
}

// ✅ Buono: eccezione se "non trovato" è un errore
public User findById(Long id) {
    User user = userMap.get(id);
    if (user == null) throw new NotFoundException("User " + id + " not found");
    return user;
}
```

### 2.2 Mai accettare null se non necessario

```java
// ❌ Cattivo: accetta null ma non lo gestisce
public void process(String input) {
    input.length();  // NPE se null
}

// ✅ Buono: @NotNull e fail fast
public void process(@NotNull String input) {
    Objects.requireNonNull(input, "input must not be null");
    // ...
}
```

### 2.3 Objects utility

```java
Objects.requireNonNull(obj);           // throw NPE con messaggio default
Objects.requireNonNull(obj, "msg");     // throw NPE con messaggio custom
Objects.isNull(obj);                    // true/false
Objects.nonNull(obj);                   // negato
Objects.toString(obj, "default");       // toString sicuro
```

### 2.4 Optional — quando usarlo

```java
// ✅ OK: valore di ritorno che può essere assente
public Optional<User> findByEmail(String email) { ... }

// ❌ NO: campo di una classe (non serializzabile, viola il contratto)
public class User {
    private Optional<String> middleName;  // ❌
}

// ❌ NO: parametro di metodo (inutile, il chiamante può passare null lo stesso)
public void process(Optional<String> input) { ... }  // ❌
```

---

## 3. Eccezioni

### 3.1 Gerarchia

```java
// Eccezioni checked (il chiamante DEVE gestirle)
IOException, SQLException, InterruptedException

// Eccezioni unchecked (RuntimeException — il chiamante PUÒ ignorarle)
NullPointerException, IllegalArgumentException, IllegalStateException

// Error (non recuperabili — mai catturarle)
OutOfMemoryError, StackOverflowError
```

**Regola:** usa unchecked per errori di programmazione (parametri errati, stato illecito). Usa checked solo se il chiamante può realisticamente recuperare.

### 3.2 Mai ingoiare eccezioni

```java
// ❌ Cattivo: silenzioso
try {
    // ...
} catch (Exception e) {
    // non fa niente
}

// ❌ Cattivo: logga ma continuo come se niente fosse
try {
    // ...
} catch (Exception e) {
    log.error("errore", e);
}

// ✅ Buono: logga e rilancia (o lancia una più specifica)
try {
    // ...
} catch (IOException e) {
    throw new ServiceException("Errore durante il salvataggio", e);
}
```

### 3.3 Fail-fast

```java
// Controlla i presupposti all'inizio del metodo
public void process(User user) {
    if (user == null) throw new IllegalArgumentException("user required");
    if (user.getEmail() == null) throw new IllegalStateException("user.email not set");
    // logica...
}
```

---

## 4. Performance Java

### 4.1 String concatenation

```java
// ❌ Cattivo: crea N oggetti intermedi
String result = "";
for (String s : items) {
    result += s;  // nuova stringa ogni volta!
}

// ✅ Buono: StringBuilder
StringBuilder sb = new StringBuilder();
for (String s : items) {
    sb.append(s);
}
String result = sb.toString();

// ✅ Meglio: String.join (se non serve logica)
String result = String.join(", ", items);
```

### 4.2 Collection con capacità iniziale

```java
// ❌ Cattivo: ArrayList cresce da default 10, reallocando più volte
List<String> list = new ArrayList<>();
for (int i = 0; i < 1000; i++) list.add("item" + i);

// ✅ Buono: specifica capacità iniziale
List<String> list = new ArrayList<>(1000);
Map<String, User> map = new HashMap<>(200);  // load factor 0.75 → 200 * 0.75 = 150 elementi senza rehash
```

### 4.3 Stream: performance

```java
// Collection piccole (< 1000): for-each e stream sono equivalenti
// Collection grandi (> 10000): parallelStream può aiutare
// MA: parallelStream ha overhead di fork/join

// misuro prima di ottimizzare
list.stream()
    .filter(x -> expensiveOperation(x))
    .toList();

list.parallelStream()
    .filter(x -> expensiveOperation(x))
    .toList();  // non sempre più veloce!
```

### 4.4 Boxing/Unboxing

```java
// ❌ Cattivo: autoboxing in loop (crea Integer oggetto ogni iterazione)
Integer sum = 0;
for (int i = 0; i < 100000; i++) {
    sum += i;  // Integer.valueOf() ogni volta!
}

// ✅ Buono: primitiva
int sum = 0;
for (int i = 0; i < 100000; i++) {
    sum += i;
}
```

---

## 5. Collection

### 5.1 Scegli la giusta implementazione

| Interfaccia | Implementazione | Quando usare |
|---|---|---|
| `List` | `ArrayList` | Accesso random, iterazione, aggiunta in coda |
| `List` | `LinkedList` | Inserimenti/rimozioni frequenti in testa/mezzo |
| `Set` | `HashSet` | Unicità, nessun ordine |
| `Set` | `TreeSet` | Unicità + ordinamento naturale |
| `Set` | `LinkedHashSet` | Unicità + ordine inserimento |
| `Map` | `HashMap` | Key-value generico |
| `Map` | `LinkedHashMap` | Key-value + ordine inserimento |
| `Map` | `TreeMap` | Key-value ordinato per chiave |
| `Queue` | `ArrayDeque` | Coda/pila (più veloce di LinkedList) |

### 5.2 Immutabilità

```java
// ❌ Cattivo: lista mutabile esposta
private List<User> users = new ArrayList<>();
public List<User> getUsers() { return users; }  // il chiamante può modificare!

// ✅ Buono: copia difensiva o immutabile
public List<User> getUsers() { return Collections.unmodifiableList(users); }

// Java 9+:
public List<User> getUsers() { return List.copyOf(users); }
```

### 5.3 Iterazione sicura

```java
// ❌ Cattivo: ConcurrentModificationException
for (User u : list) {
    if (u.isExpired()) list.remove(u);
}

// ✅ Buono: Iterator.remove()
Iterator<User> it = list.iterator();
while (it.hasNext()) {
    if (it.next().isExpired()) it.remove();
}

// ✅ Alternativa: removeIf (Java 8+)
list.removeIf(User::isExpired);
```

---

## 6. equals e hashCode

```java
// Regola: se override equals(), DEVI override hashCode()
// Contratto: se a.equals(b) → a.hashCode() == b.hashCode()

@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof User other)) return false;
    return Objects.equals(email, other.email);
}

@Override
public int hashCode() {
    return Objects.hash(email);  // stessi campi di equals
}
```

**Quando NON servono:**
- Enum (già implementati correttamente)
- Record (generati dal compilatore)
- Classi che non finiscono mai in HashSet/HashMap

---

## 7. Programmazione Funzionale

### 7.1 Preferisci lambda a classi anonime

```java
// ❌ Cattivo: classe anonima
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("click");
    }
});

// ✅ Buono: lambda
button.addActionListener(e -> System.out.println("click"));
```

### 7.2 Method reference quando possibile

```java
// ❌ lambda che chiama un solo metodo
list.forEach(s -> System.out.println(s));

// ✅ method reference
list.forEach(System.out::println);

// Altri esempi:
list.stream().map(User::getEmail)
list.stream().filter(Objects::nonNull)
list.stream().sorted(Comparator.comparing(User::getName))
```

### 7.3 Stream: catena pulita

```java
// ❌ Cattivo: tutto su una riga
return users.stream().filter(u -> u.isActive()).map(u -> new UserResponse(u.getId(), u.getEmail())).collect(Collectors.toList());

// ✅ Buono: un'operazione per riga
return users.stream()
    .filter(User::isActive)
    .map(u -> new UserResponse(u.getId(), u.getEmail()))
    .toList();
```

---

## 8. Programmazione Concorrente

### 8.1 Thread safety

```java
// ❌ Cattivo: SimpleDateFormat non è thread-safe
private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

// ✅ Buono: DateTimeFormatter (immutabile e thread-safe)
private static final DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd");

// ✅ Alternativa: ThreadLocal
private static final ThreadLocal<SimpleDateFormat> tl = ThreadLocal.withInitial(
    () -> new SimpleDateFormat("yyyy-MM-dd")
);
```

### 8.2 Variabili condivise

```java
// ❌ Cattivo: race condition
private int counter;

public void increment() {
    counter++;  // non atomico!
}

// ✅ Buono: AtomicInteger
private final AtomicInteger counter = new AtomicInteger(0);

public void increment() {
    counter.incrementAndGet();
}

// ✅ Alternativa: synchronized
public synchronized void increment() {
    counter++;
}
```

### 8.3 ExecutorService

```java
// ❌ Cattivo: creare thread ogni volta
new Thread(() -> doWork()).start();

// ✅ Buono: thread pool
private final ExecutorService executor = Executors.newFixedThreadPool(4);

public void submitTask() {
    executor.submit(() -> doWork());
}

// Ricorda di shutdown!
public void shutdown() {
    executor.shutdown();
    executor.awaitTermination(5, TimeUnit.SECONDS);
}
```

---

## 9. Convenzioni Generali

### 9.1 Ordine membri della classe

```
1. static final fields (costanti)
2. static fields
3. instance fields (final prima)
4. costruttori
5. factory methods (static)
6. metodi pubblici
7. metodi protected/package
8. metodi privati
9. getter/setter (se servono)
10. equals/hashCode/toString
```

### 9.2 Lunghezza metodi

- **< 15 righe:** ideale
- **15-30 righe:** accettabile
- **> 30 righe:** refactoring — estrai sotto-metodi

Se un metodo ha più di 3-4 parametri, crea un oggetto parametro o un builder.

### 9.3 JavaDoc

```java
/**
 * Calcola lo sconto applicabile in base al tipo utente e all'importo.
 * Le regole di sconto sono definite in [link a specifica].
 *
 * @param user        utente (deve avere ruolo e fedeltà impostati)
 * @param baseAmount  importo base positivo
 * @return importo scontato (mai negativo)
 * @throws IllegalArgumentException se baseAmount <= 0
 */
public BigDecimal calculateDiscount(User user, BigDecimal baseAmount) {
```

JavaDoc su: classi pubbliche, metodi pubblici con logica non ovvia.
Niente JavaDoc su: getter, setter, override ovvi.

---

## 10. Anti-pattern Comuni

| Anti-pattern | Problema | Soluzione |
|---|---|---|
| God Class (> 1000 righe) | Troppe responsabilità | Split in classi piccole |
| Switch su tipo | Viola OCP, difficile estendere | Strategy pattern o polimorfismo |
| Catch generico | Nasconde errori specifici | Catch specifici |
| String "magiche" | Non tracciabili, errori di typo | Constanti o enum |
| Mutabilità eccessiva | Bug difficili da trovare | Record, final, immutabili |
| Utilizzare null come valore | NPE, codice difensivo ovunque | Optional o oggetto Null |
| Campi pubblici | Incapsulamento rotto | private + getter |
| Troppi parametri nel costruttore | Difficile da leggere/usare | Builder pattern |

---

## Riferimenti

- [[BE-NOTES/Java/Base/Variabili e Tipi di Dati]]
- [[BE-NOTES/Java/Base/Generics]]
- [[BE-NOTES/Java/OOP/Enum]]
- [[BE-NOTES/Java/OOP/Stato, visibilità, SOC e STATIC]]
- [[BE-NOTES/Java/Tecnologie/Optional e Gestione Null]]
- [[BE-NOTES/Java/Tecnologie/Java Records]]
