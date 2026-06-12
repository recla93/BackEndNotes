# Aspect Oriented Programming (AOP): Guida Completa  
  
## Il Problema Reale  
  
Prima di capire cos'è AOP, vediamo il problema che risolve. Immagina di avere 10 controller REST e vuoi loggare ogni chiamata:  
  
```java  
@RestController  
@RequestMapping("/api/users")  
public class UserController {  
  
    @GetMapping("/{id}")  
    public User getUser(@PathVariable Long id) {  
        log.info("Chiamata getUser con id: {}", id);  // ← duplicato  
        User user = userService.findById(id);  
        log.info("getUser completato");  // ← duplicato  
        return user;  
    }  
  
    @PostMapping  
    public User createUser(@RequestBody UserDTO dto) {  
        log.info("Chiamata createUser");  // ← duplicato  
        User user = userService.create(dto);  
        log.info("createUser completato");  // ← duplicato  
        return user;  
    }  
}  
```  
  
**Problema**: Codice duplicato ovunque. Se cambi il formato del log, devi modificare centinaia di metodi.  
  
> [!question] La Soluzione: AOP  
> AOP ti permette di scrivere il codice di logging **UNA VOLTA SOLA** e applicarlo automaticamente a tutti i metodi che vuoi. È come un "filtro intelligente" che intercetta le chiamate.  
  
## Cos'è AOP  
  
L'**Aspect Oriented Programming** è un paradigma che separa le **cross-cutting concerns** (preoccupazioni trasversali) dalla logica di business. Invece di duplicare codice come logging, sicurezza, transazioni o gestione errori, l'AOP lo centralizza e lo applica automaticamente.  
  
> [!info] Metafora Semplice  
> Pensa a AOP come a un **interruttore smart** della luce:  
> - **Senza AOP**: Ogni volta che accendi la luce devi anche aprire la finestra, controllare la temperatura, loggare l'orario  
> - **Con AOP**: L'interruttore fa tutto automaticamente. Tu premi solo il pulsante  
  
### Concetti Fondamentali  
  
> [!note] Terminologia AOP  
> - **Aspect**: Modulo che raggruppa le preoccupazioni trasversali (es. LoggingAspect)  
> - **Join Point**: Punto nel programma dove un aspect può intervenire (es. esecuzione di un metodo)  
> - **Pointcut**: Espressione che seleziona quali join point intercettare  
> - **Advice**: Codice eseguito in un join point (before, after, around)  
> - **Weaving**: Processo di applicazione degli aspect al codice target  
  
### Tipi di Advice  
  
- [x] **@Before**: Eseguito prima del metodo target  
- [x] **@After**: Eseguito dopo il metodo (sempre, anche se c'è errore)  
- [x] **@AfterReturning**: Eseguito dopo un ritorno normale (senza eccezioni)  
- [x] **@AfterThrowing**: Eseguito solo in caso di eccezione  
- [x] **@Around**: Avvolge completamente il metodo target (massimo controllo)  
  
## Setup Iniziale  
  
### Dipendenze Maven  
  
Aggiungi questa dipendenza nel tuo `pom.xml`:  
  
```xml  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-aop</artifactId>  
</dependency>  
```  
  
### Configurazione  
  
Con Spring Boot 3.x, AOP è auto-configurato. Se usi versioni precedenti o configurazione manuale:  
  
```java  
@Configuration  
@EnableAspectJAutoProxy  
public class AopConfig {  
}  
```  
  
> [!tip] Auto-Configuration  
> Spring Boot rileva automaticamente la dipendenza AOP e abilita `@EnableAspectJAutoProxy` senza bisogno di configurazione esplicita.  
  
## Percorso di Apprendimento: Dal Base all'Avanzato  
  
### Livello 1: Il Tuo Primo Aspect  
  
Iniziamo con qualcosa di super semplice: loggare TUTTI i metodi dei controller.  
  
```java  
package com.example.aspect;  
  
import lombok.extern.slf4j.Slf4j;  
import org.aspectj.lang.JoinPoint;  
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.annotation.Before;  
import org.springframework.stereotype.Component;  
  
@Aspect      // ← Dice a Spring: "questo è un Aspect"  
@Component   // ← Registra come bean Spring  
@Slf4j       // ← Lombok per il logging  
public class LoggingAspect {  
  
    // Intercetta TUTTI i metodi dei controller  
    @Before("execution(* com.example.controller.*.*(..))")  
    public void logBefore(JoinPoint joinPoint) {  
        String nomeMetodo = joinPoint.getSignature().getName();  
        Object[] parametri = joinPoint.getArgs();  
  
        log.info("🚀 Chiamata metodo: {} con parametri: {}",  
                 nomeMetodo, parametri);  
    }  
}  
```  
  
> [!success] Test Subito  
> Esegui l'app e chiama qualsiasi endpoint. Vedrai il log apparire **AUTOMATICAMENTE** senza aver modificato i controller!  
  
### Livello 2: Catturare il Risultato  
  
Ora aggiungiamo il log del risultato dopo l'esecuzione:  
  
```java  
@Aspect  
@Component  
@Slf4j  
public class LoggingAspect {  
  
    @Before("execution(* com.example.controller.*.*(..))")  
    public void logBefore(JoinPoint joinPoint) {  
        log.info("🚀 Chiamata: {}", joinPoint.getSignature().getName());  
    }  
  
    // Log del risultato dopo l'esecuzione normale  
    @AfterReturning(  
        pointcut = "execution(* com.example.controller.*.*(..))",  
        returning = "result"  
    )  
    public void logAfterReturning(JoinPoint joinPoint, Object result) {  
        log.info("✅ {} completato con risultato: {}",  
                 joinPoint.getSignature().getName(), result);  
    }  
  
    // Log in caso di errore  
    @AfterThrowing(  
        pointcut = "execution(* com.example.controller.*.*(..))",  
        throwing = "error"  
    )  
    public void logAfterThrowing(JoinPoint joinPoint, Throwable error) {  
        log.error("❌ Eccezione nel metodo {}: {}",  
                  joinPoint.getSignature().getName(),  
                  error.getMessage());  
    }  
}  
```  
  
### Livello 3: Performance Monitoring con @Around  
  
`@Around` è il più potente: ti dà controllo completo prima E dopo il metodo.  
  
```java  
@Aspect  
@Component  
@Slf4j  
public class PerformanceAspect {  
  
    @Around("execution(* com.example.controller.*.*(..))")  
    public Object measureExecutionTime(ProceedingJoinPoint joinPoint)  
            throws Throwable {  
  
        String nomeMetodo = joinPoint.getSignature().getName();  
  
        // Tempo di inizio  
        long inizio = System.currentTimeMillis();  
  
        // IMPORTANTE: esegui il metodo vero  
        Object risultato = joinPoint.proceed();  
  
        // Tempo di fine  
        long fine = System.currentTimeMillis();  
        long durata = fine - inizio;  
  
        log.info("⏱️ {} eseguito in {} ms", nomeMetodo, durata);  
  
        return risultato;  
    }  
}  
```  
  
> [!warning] Attenzione Cruciale  
> Con `@Around` devi **sempre** chiamare `joinPoint.proceed()`, altrimenti il metodo originale non viene mai eseguito e l'app si blocca!  
  
### Livello 4: Annotazioni Custom (Professionale)  
  
Invece di applicare l'aspect a tutto, creiamo un'annotazione per decidere dove applicarlo.  
  
**Passo 1 - Crea l'annotazione:**  
  
```java  
package com.example.annotation;  
  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;  
  
@Target(ElementType.METHOD)  // ← Può essere usata sui metodi  
@Retention(RetentionPolicy.RUNTIME)  // ← Disponibile a runtime  
public @interface LoggaMi {  
}  
```  
  
**Passo 2 - Crea l'aspect che la usa:**  
  
```java  
@Aspect  
@Component  
@Slf4j  
public class CustomLoggingAspect {  
  
    // Intercetta SOLO i metodi con @LoggaMi  
    @Around("@annotation(com.example.annotation.LoggaMi)")  
    public Object loggaMetodo(ProceedingJoinPoint joinPoint) throws Throwable {  
        String nomeMetodo = joinPoint.getSignature().getName();  
        Object[] args = joinPoint.getArgs();  
  
        log.info("📝 Chiamata: {} con parametri: {}", nomeMetodo, args);  
  
        long inizio = System.currentTimeMillis();  
        Object risultato = joinPoint.proceed();  
        long durata = System.currentTimeMillis() - inizio;  
  
        log.info("✅ {} completato in {} ms", nomeMetodo, durata);  
  
        return risultato;  
    }  
}  
```  
  
**Passo 3 - Usala dove serve:**  
  
```java  
@RestController  
@RequestMapping("/api/users")  
public class UserController {  
  
    @GetMapping("/{id}")  
    @LoggaMi  // ← Basta questa annotazione!  
    public User getUser(@PathVariable Long id) {  
        return userService.findById(id);  
    }  
  
    @PostMapping  
    // ← Questo NON verrà loggato (niente annotazione)  
    public User createUser(@RequestBody UserDTO dto) {  
        return userService.create(dto);  
    }  
}  
```  
  
### Livello 5: Auditing con Parametri Custom  
  
Annotazione con parametri per tracciare azioni specifiche:  
  
```java  
@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface Auditable {  
    String action() default "";  
}  
```  
  
```java  
@Aspect  
@Component  
@Slf4j  
public class AuditAspect {  
  
    @Around("@annotation(auditable)")  
    public Object auditMethod(ProceedingJoinPoint joinPoint,  
                             Auditable auditable) throws Throwable {  
  
        String username = SecurityContextHolder.getContext()  
                            .getAuthentication().getName();  
        String action = auditable.action();  
  
        log.info("👤 User {} esegue: {}", username, action);  
  
        try {  
            Object result = joinPoint.proceed();  
            log.info("✅ Azione {} completata", action);  
            return result;  
        } catch (Exception e) {  
            log.error("❌ Azione {} fallita: {}", action, e.getMessage());  
            throw e;  
        }  
    }  
}  
```  
  
**Utilizzo:**  
  
```java  
@Service  
public class UserService {  
  
    @Auditable(action = "DELETE_USER")  
    public void deleteUser(Long userId) {  
        userRepository.deleteById(userId);  
    }  
  
    @Auditable(action = "UPDATE_USER")  
    public User updateUser(Long userId, UserDTO dto) {  
        // Logica di aggiornamento  
        return updatedUser;  
    }  
}  
```  
  
### Livello 6: Cache Semplice con AOP  
  
Implementiamo una cache basic usando AOP:  
  
```java  
@Aspect  
@Component  
@Slf4j  
public class CacheAspect {  
  
    private final Map<String, Object> cache = new ConcurrentHashMap<>();  
  
    @Around("@annotation(cacheable)")  
    public Object cacheResult(ProceedingJoinPoint joinPoint,  
                             Cacheable cacheable) throws Throwable {  
  
        String key = generateKey(joinPoint);  
  
        // Controlla se è in cache  
        if (cache.containsKey(key)) {  
            log.info("💾 Cache hit per: {}", key);  
            return cache.get(key);  
        }  
  
        // Esegui il metodo e caccia il risultato  
        log.info("🔍 Cache miss per: {}", key);  
        Object result = joinPoint.proceed();  
        cache.put(key, result);  
  
        return result;  
    }  
  
    private String generateKey(ProceedingJoinPoint joinPoint) {  
        return joinPoint.getSignature().toString() +  
               Arrays.toString(joinPoint.getArgs());  
    }  
}  
```  
  
## La Formula Magica: Pointcut Expressions  
  
Ecco le espressioni che userai di più:  
  
> [!info] Sintassi Pointcut  
> ```java  
> // Tutti i metodi in un package specifico  
> execution(* com.example.service.*.*(..))  
>  
> // Metodi che iniziano con "get"  
> execution(* get*(..))  
>  
> // Metodi che ritornano User  
> execution(com.example.model.User *(..))  
>  
> // Metodi con specifica annotazione  
> @annotation(com.example.annotation.MyAnnotation)  
>  
> // Metodi di classi con @Service  
> @within(org.springframework.stereotype.Service)  
>  
> // Combinazione con AND  
> execution(* com.example.*.*(..)) && @annotation(Auditable)  
>  
> // Combinazione con OR  
> execution(* create*(..)) || execution(* update*(..))  
>  
> // Solo metodi pubblici  
> execution(public * *(..))  
> ```  
  
> [!tip] Decodifica la Formula Base  
> `execution(* com.example.controller.*.*(..))`  
> - Primo `*` = qualsiasi tipo di ritorno  
> - `com.example.controller` = package  
> - Secondo `*` = qualsiasi classe  
> - Terzo `*` = qualsiasi metodo  
> - `(..)` = qualsiasi numero/tipo di parametri  
  
## Esempi Reali: Cosa Hai Già Usato  
  
Hai già usato AOP senza saperlo! Spring usa AOP internamente per:  
  
```java  
// Transaction Management  
@Transactional  // ← AOP gestisce apertura/commit/rollback transazioni  
public void saveUser(User user) {  
    userRepository.save(user);  
}  
  
// Caching  
@Cacheable("users")  // ← AOP controlla cache prima di eseguire  
public User getUser(Long id) {  
    return userRepository.findById(id);  
}  
  
// Security  
@PreAuthorize("hasRole('ADMIN')")  // ← AOP controlla i permessi  
public void deleteUser(Long id) {  
    userRepository.deleteById(id);  
}  
  
// Async  
@Async  // ← AOP esegue il metodo in un thread separato  
public void sendEmail(String to, String subject) {  
    emailService.send(to, subject);  
}  
```  
  
## Quando Usare AOP  
  
- [x] **Logging**: Tracciamento automatico delle chiamate  
- [x] **Security**: Validazione permessi prima dell'esecuzione  
- [x] **Transactions**: Gestione automatica transazioni DB  
- [x] **Caching**: Cache risultati metodi costosi  
- [x] **Error Handling**: Gestione centralizzata eccezioni  
- [x] **Performance Monitoring**: Misurazione tempi esecuzione  
- [x] **Auditing**: Tracciamento modifiche per compliance  
- [x] **Validation**: Validazione input centralizzata  
- [x] **Retry Logic**: Riprova automatica in caso di errore  
  
## Quando NON Usare AOP  
  
- [ ] **Logica di business core**: Va nei service, non negli aspect  
- [ ] **Performance critiche**: AOP ha un overhead (minimo ma presente)  
- [ ] **Operazioni che richiedono visibilità**: Il comportamento deve essere ovvio nel codice  
- [ ] **Team non familiare con AOP**: Curva di apprendimento da considerare  
- [ ] **Debug complesso**: Gli aspect possono rendere il debug più difficile  
  
## Best Practices  
  
> [!tip] Linee Guida  
> 1. **Un aspect = una responsabilità**: Mantieni gli aspect focalizzati su un solo compito  
> 2. **Usa annotazioni custom**: Più chiaro e manutenibile dei pointcut complessi  
> 3. **Documenta bene**: Il comportamento AOP può non essere ovvio leggendo il codice  
> 4. **Testa gli aspect**: Crea unit test per verificare l'intercettazione  
> 5. **Evita pointcut troppo generici**: Possono rallentare l'applicazione  
> 6. **Nomina gli aspect chiaramente**: `LoggingAspect`, non `AspectUno`  
> 7. **Ordina gli aspect**: Usa `@Order` se multipli aspect si applicano allo stesso metodo  
  
### Ordinare Multiple Aspect  
  
```java  
@Aspect  
@Component  
@Order(1)  // ← Eseguito per primo  
public class SecurityAspect {  
    // Verifica permessi  
}  
  
@Aspect  
@Component  
@Order(2)  // ← Eseguito per secondo  
public class LoggingAspect {  
    // Logga dopo il controllo sicurezza  
}  
```  
  
## Struttura Progetto Consigliata  
  
```  
src/main/java/com/example/  
├── aspect/  
│   ├── LoggingAspect.java  
│   ├── PerformanceAspect.java  
│   ├── SecurityAspect.java  
│   ├── AuditAspect.java  
│   └── CacheAspect.java  
├── annotation/  
│   ├── Auditable.java  
│   ├── LoggaMi.java  
│   └── Cacheable.java  
├── controller/  
│   └── UserController.java  
├── service/  
│   └── UserService.java  
└── config/  
    └── AopConfig.java  // ← Solo se serve config custom  
```  
  
## Testing degli Aspect  
  
### Test Unitario dell'Aspect  
  
```java  
@ExtendWith(MockitoExtension.class)  
class LoggingAspectTest {  
  
    @Mock  
    private JoinPoint joinPoint;  
  
    @Mock  
    private Signature signature;  
  
    @InjectMocks  
    private LoggingAspect loggingAspect;  
  
    @Test  
    void testLogBeforeMethod() {  
        // Arrange  
        when(joinPoint.getSignature()).thenReturn(signature);  
        when(signature.getName()).thenReturn("testMethod");  
        when(joinPoint.getArgs()).thenReturn(new Object[]{1, "test"});  
  
        // Act  
        loggingAspect.logBefore(joinPoint);  
  
        // Assert  
        // Verifica che il log sia stato chiamato  
        verify(joinPoint).getSignature();  
        verify(signature).getName();  
    }  
}  
```  
  
### Test di Integrazione  
  
```java  
@SpringBootTest  
class PerformanceAspectIntegrationTest {  
  
    @Autowired  
    private UserService userService;  
  
    @MockBean  
    private UserRepository userRepository;  
  
    @Test  
    void testAspectMeasuresPerformance() {  
        // Arrange  
        when(userRepository.findById(1L))  
            .thenReturn(Optional.of(new User()));  
  
        // Act  
        userService.getUserById(1L);  
  
        // Assert  
        // Verifica che il log delle performance sia presente  
        // (usa un log appender mock o TestAppender)  
    }  
}  
```  
  
## Errori Comuni dei Junior  
  
> [!danger] Attenzione a Questi Errori  
> 1. **Dimenticare `@Component`**: L'aspect non viene rilevato da Spring  
> 2. **Non chiamare `proceed()` in `@Around`**: I metodi non si eseguono mai  
> 3. **Pointcut troppo generici**: Intercetti troppi metodi e rallenti l'app  
> 4. **Usare AOP per logica business**: AOP è per cross-cutting concerns, non logica core  
> 5. **Non gestire le eccezioni in `@Around`**: Le eccezioni vengono ingoiate  
> 6. **Dimenticare `throws Throwable`**: L'aspect non può propagare eccezioni  
  
## Esercizi Pratici  
  
### Livello Base  
- [ ] **Esercizio 1**: Crea un aspect che logga solo i metodi che iniziano con "delete"  
- [ ] **Esercizio 2**: Crea un aspect che conta quante volte viene chiamato ogni metodo  
- [ ] **Esercizio 3**: Logga solo i metodi dei controller che accettano un `@PathVariable`  
  
### Livello Intermedio  
- [ ] **Esercizio 4**: Crea un'annotazione `@RequiresAuth` che verifica se l'utente è autenticato  
- [ ] **Esercizio 5**: Implementa un rate limiter con AOP (max 10 chiamate al minuto)  
- [ ] **Esercizio 6**: Crea un aspect che cattura tutte le eccezioni e le traduce in response HTTP appropriate  
  
### Livello Avanzato  
- [ ] **Esercizio 7**: Implementa un sistema di retry automatico per metodi che falliscono  
- [ ] **Esercizio 8**: Crea un auditing completo che salva nel DB chi, quando e cosa ha modificato  
- [ ] **Esercizio 9**: Implementa una cache LRU (Least Recently Used) con AOP  
  
## Progetto Finale: Sistema di Auditing Completo  
  
Quando ti senti pronto, prova questo progetto che combina tutto:  
  
### Requisiti  
  
```java  
@Auditable(action = "USER_CREATION", severity = "HIGH")  
@PostMapping  
public User createUser(@RequestBody UserDTO dto) {  
    return userService.create(dto);  
}  
```  
  
L'aspect deve:  
- Salvare nel DB chi ha fatto l'azione (username)  
- Timestamp dell'azione  
- Quale azione (dal parametro)  
- Parametri della chiamata (serializzati in JSON)  
- Risultato dell'operazione (successo/fallimento)  
- Livello di severità  
  
### Schema DB  
  
```sql  
CREATE TABLE audit_log (  
    id BIGSERIAL PRIMARY KEY,  
    username VARCHAR(255),  
    action VARCHAR(100),  
    severity VARCHAR(20),  
    parameters TEXT,  
    result TEXT,  
    success BOOLEAN,  
    timestamp TIMESTAMP,  
    execution_time_ms BIGINT  
);  
```  
  
### Implementazione Base  
  
```java  
@Entity  
@Table(name = "audit_log")  
@Data  
public class AuditLog {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
    private String username;  
    private String action;  
    private String severity;  
    private String parameters;  
    private String result;  
    private Boolean success;  
    private LocalDateTime timestamp;  
    private Long executionTimeMs;  
}  
```  
  
```java  
@Aspect  
@Component  
@Slf4j  
@RequiredArgsConstructor  
public class AuditAspect {  
  
    private final AuditLogRepository auditLogRepository;  
    private final ObjectMapper objectMapper;  
  
    @Around("@annotation(auditable)")  
    public Object auditMethod(ProceedingJoinPoint joinPoint,  
                             Auditable auditable) throws Throwable {  
  
        AuditLog auditLog = new AuditLog();  
        auditLog.setTimestamp(LocalDateTime.now());  
        auditLog.setAction(auditable.action());  
        auditLog.setSeverity(auditable.severity());  
  
        // Cattura username  
        Authentication auth = SecurityContextHolder.getContext()  
                                .getAuthentication();  
        auditLog.setUsername(auth != null ? auth.getName() : "anonymous");  
  
        // Serializza parametri  
        try {  
            String params = objectMapper.writeValueAsString(  
                joinPoint.getArgs()  
            );  
            auditLog.setParameters(params);  
        } catch (Exception e) {  
            auditLog.setParameters("Error serializing parameters");  
        }  
  
        long startTime = System.currentTimeMillis();  
  
        try {  
            // Esegui il metodo  
            Object result = joinPoint.proceed();  
  
            // Operazione riuscita  
            auditLog.setSuccess(true);  
            auditLog.setResult(  
                objectMapper.writeValueAsString(result)  
            );  
  
            return result;  
  
        } catch (Exception e) {  
            // Operazione fallita  
            auditLog.setSuccess(false);  
            auditLog.setResult("Error: " + e.getMessage());  
            throw e;  
  
        } finally {  
            // Calcola tempo esecuzione  
            long executionTime = System.currentTimeMillis() - startTime;  
            auditLog.setExecutionTimeMs(executionTime);  
  
            // Salva nel DB (async per non rallentare)  
            auditLogRepository.save(auditLog);  
        }  
    }  
}  
```  
  
## Piano di Studio Completo  
  
### Settimana 1: Fondamentali  
- [x] Leggi questa guida completa  
- [ ] Crea un progetto Spring Boot di test  
- [ ] Implementa logging aspect base con `@Before`  
- [ ] Aggiungi `@AfterReturning` e `@AfterThrowing`  
- [ ] Testa su 2-3 controller diversi  
  
### Settimana 2: Controllo Avanzato  
- [ ] Implementa performance monitoring con `@Around`  
- [ ] Crea la tua prima annotazione custom `@LoggaMi`  
- [ ] Sperimenta con diversi pointcut expressions  
- [ ] Aggiungi gestione errori centralizzata  
  
### Settimana 3: Progetti Reali  
- [ ] Implementa un sistema di caching semplice  
- [ ] Crea auditing base (chi ha fatto cosa)  
- [ ] Aggiungi validazione automatica input  
- [ ] Combina multipli aspect con `@Order`  
  
### Settimana 4: Progetto Finale  
- [ ] Implementa il sistema di auditing completo  
- [ ] Aggiungi test unitari e di integrazione  
- [ ] Documenta il codice e crea README  
- [ ] Studia il codice sorgente di `@Transactional`  
  
## Risorse per Approfondire  
  
> [!info] Documentazione e Tutorial  
> - **Spring AOP Reference**: La documentazione ufficiale (dopo aver fatto pratica)  
> - **AspectJ Documentation**: Il framework completo dietro Spring AOP  
> - **Baeldung Spring AOP**: Tutorial pratici step-by-step  
> - **Spring Source Code**: Studia come Spring implementa `@Transactional` e `@Cacheable`  
  
## Debug degli Aspect  
  
> [!tip] Tecniche di Debug  
> - Metti **breakpoint** negli aspect per vedere quando vengono chiamati  
> - Usa `log.debug` abbondante durante lo sviluppo  
> - Abilita logging Spring AOP: `logging.level.org.springframework.aop=DEBUG`  
> - Usa l'IDE per navigare dai pointcut ai metodi intercettati  
> - Testa aspect isolatamente con unit test  
  
## Cheatsheet Veloce  
  
```java  
// Aspect base  
@Aspect  
@Component  
public class MioAspect {  
  
    // Prima del metodo  
    @Before("execution(* com.example.*.*(..))")  
    public void before(JoinPoint jp) { }  
  
    // Dopo (sempre)  
    @After("execution(* com.example.*.*(..))")  
    public void after(JoinPoint jp) { }  
  
    // Dopo ritorno normale  
    @AfterReturning(pointcut="...", returning="result")  
    public void afterReturning(JoinPoint jp, Object result) { }  
  
    // Dopo eccezione  
    @AfterThrowing(pointcut="...", throwing="error")  
    public void afterThrowing(JoinPoint jp, Throwable error) { }  
  
    // Avvolge completamente  
    @Around("execution(* com.example.*.*(..))")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        // Prima  
        Object result = pjp.proceed();  // Esegui  
        // Dopo  
        return result;  
    }  
}  
```  
  
***  
  
Hai tutto il materiale per diventare competente con AOP! Inizia con gli esempi semplici (Livello 1-2), sperimenta, e poi passa gradualmente ai livelli successivi. Quando hai domande su un esempio specifico o vuoi discutere il tuo progetto di auditing, sono qui!  
  
Preferisci che ti prepari soluzioni dettagliate per gli esercizi o vuoi prima provare da solo?