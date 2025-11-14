## Cosa sono i Join Point  
  
I **Join Point** sono punti specifici durante l'esecuzione del programma dove un aspect può essere applicato. Sono le "opportunità di intercettazione" che il framework AOP offre.[3][4]  
  
> [!info] Definizione Tecnica  
> Un Join Point rappresenta un momento preciso nel flusso di esecuzione dell'applicazione, come l'esecuzione di un metodo, la gestione di un'eccezione, o l'accesso a un campo.  
  
### Join Point in Spring AOP  
  
Spring AOP ha una **limitazione importante** rispetto ad AspectJ completo: supporta **SOLO** method execution join points.[7][3]  
  
```java  
public class UserService {  
  
    public User findById(Long id) {  // ← JOIN POINT  
        return repository.findById(id);  
    }  
  
    public void save(User user) {  // ← JOIN POINT  
        repository.save(user);  
    }  
}  
```  
  
### Tipi di Join Point (AspectJ completo)  
  
AspectJ completo supporta molti tipi di join point, ma Spring AOP solo alcuni:  
  
- [x] **Method execution**: Esecuzione di un metodo (✅ Supportato da Spring)  
- [x] **Method call**: Chiamata di un metodo (❌ Non in Spring AOP)  
- [x] **Constructor execution**: Esecuzione costruttore (❌ Non in Spring AOP)  
- [x] **Field access**: Accesso a un campo (❌ Non in Spring AOP)  
- [x] **Exception handler**: Gestione eccezioni (❌ Non in Spring AOP)  
- [x] **Object initialization**: Inizializzazione oggetto (❌ Non in Spring AOP)  
  
> [!warning] Limitazione Spring AOP  
> Spring AOP funziona tramite proxy dinamici, quindi può intercettare **solo** chiamate a metodi pubblici di bean Spring. Non può intercettare chiamate interne, metodi privati o accessi a campi.  
  
## Differenza tra Join Point e Pointcut  
  
Questa è la distinzione fondamentale da capire:[1][2][7]  
  
```  
JOIN POINT = Punto di esecuzione REALE nel codice  
    ↓  
POINTCUT = Espressione che SELEZIONA quali join point intercettare  
    ↓  
ADVICE = Codice che viene ESEGUITO quando il pointcut matcha  
```  
  
> [!note] Analogia  
> - **Join Point**: Tutte le stazioni della metropolitana (punti disponibili)  
> - **Pointcut**: Il filtro "fermate sulla linea rossa" (selezione)  
> - **Advice**: Cosa fai quando arrivi a quelle fermate (azione)  
  
### Esempio Visivo  
  
```java  
@Service  
public class UserService {  
  
    public User getUser(Long id) { }     // ← JOIN POINT 1  
    public void createUser(User u) { }   // ← JOIN POINT 2  
    public void deleteUser(Long id) { }  // ← JOIN POINT 3  
}  
  
@Service  
public class OrderService {  
  
    public Order getOrder(Long id) { }   // ← JOIN POINT 4  
    public void createOrder(Order o) { } // ← JOIN POINT 5  
}  
  
// POINTCUT: seleziona solo JOIN POINT 2 e 5 (metodi create*)  
@Pointcut("execution(* create*(..))")  
public void createMethods() { }  
```  
  
### In Runtime  
  
```java  
@Aspect  
@Component  
public class LoggingAspect {  
  
    @Before("execution(* com.example.service.*.*(..))")  
    public void logBefore(JoinPoint joinPoint) {  
        //                  └── Oggetto che rappresenta  
        //                      il join point CORRENTE in esecuzione  
  
        String methodName = joinPoint.getSignature().getName();  
        Object[] args = joinPoint.getArgs();  
        Object target = joinPoint.getTarget();  
  
        System.out.println("Metodo: " + methodName);  
        System.out.println("Parametri: " + Arrays.toString(args));  
        System.out.println("Classe target: " + target.getClass());  
    }  
}  
```  
  
> [!tip] Ricorda  
> - **Join Point**: Il concetto (il punto dove può essere applicato un aspect)  
> - **JoinPoint** (oggetto): L'istanza runtime che contiene info sul join point corrente  
> - **Pointcut**: L'espressione che filtra quali join point matchare  
  
## Come si Definisce un Pointcut in Spring  
  
Ci sono **tre modi** principali per definire un pointcut in Spring AOP:[1][3]  
  
### Metodo 1: Inline con l'Advice (Base)  
  
Definisci il pointcut direttamente nell'annotazione dell'advice:  
  
```java  
@Aspect  
@Component  
public class SimpleAspect {  
  
    @Before("execution(* com.example.service.*.*(..))")  
    public void beforeServiceMethod(JoinPoint jp) {  
        System.out.println("Prima del metodo: " + jp.getSignature());  
    }  
  
    @After("execution(* com.example.controller.*.*(..))")  
    public void afterControllerMethod(JoinPoint jp) {  
        System.out.println("Dopo il metodo: " + jp.getSignature());  
    }  
}  
```  
  
**Pro**: Semplice e veloce  
**Contro**: Non riutilizzabile, duplicazione se stesso pointcut serve in più advice  
  
### Metodo 2: @Pointcut Riusabile (Raccomandato)  
  
Definisci pointcut nominati e riusali:[5][3]  
  
```java  
@Aspect  
@Component  
public class ReusableAspect {  
  
    // Definisci il pointcut  
    @Pointcut("execution(* com.example.service.*.*(..))")  
    public void serviceMethods() { }  
    //           └── Nome del pointcut (metodo vuoto)  
  
    @Pointcut("execution(* com.example.controller.*.*(..))")  
    public void controllerMethods() { }  
  
    @Pointcut("execution(* get*(..))")  
    public void getterMethods() { }  
  
    // Usa i pointcut negli advice  
    @Before("serviceMethods()")  
    public void beforeService(JoinPoint jp) {  
        // Codice  
    }  
  
    @After("controllerMethods()")  
    public void afterController(JoinPoint jp) {  
        // Codice  
    }  
  
    // Combina pointcut  
    @Around("serviceMethods() && !getterMethods()")  
    public Object aroundNonGetterService(ProceedingJoinPoint pjp)  
            throws Throwable {  
        // Service methods che NON sono getter  
        return pjp.proceed();  
    }  
}  
```  
  
> [!success] Best Practice  
> Il metodo annotato con `@Pointcut` è **sempre vuoto**. Serve solo come "firma" per dare un nome al pointcut. Il corpo del metodo non viene mai eseguito.[5]  
  
### Metodo 3: Pointcut Centralizzati (Aziendale)  
  
Crea una classe dedicata per tutti i pointcut riusabili:[3]  
  
```java  
@Aspect  
@Component  
public class CommonPointcuts {  
  
    /**  
     * Join point nel web layer  
     */  
    @Pointcut("within(com.example.controller..*)")  
    public void inWebLayer() { }  
  
    /**  
     * Join point nel service layer  
     */  
    @Pointcut("within(com.example.service..*)")  
    public void inServiceLayer() { }  
  
    /**  
     * Join point nel data access layer  
     */  
    @Pointcut("within(com.example.repository..*)")  
    public void inDataAccessLayer() { }  
  
    /**  
     * Metodi pubblici  
     */  
    @Pointcut("execution(public * *(..))")  
    public void publicMethods() { }  
  
    /**  
     * Business service = metodi pubblici nel service layer  
     */  
    @Pointcut("publicMethods() && inServiceLayer()")  
    public void businessService() { }  
}  
```  
  
Poi usali da qualsiasi aspect:  
  
```java  
@Aspect  
@Component  
public class LoggingAspect {  
  
    @Before("com.example.aspect.CommonPointcuts.businessService()")  
    public void logBusinessService(JoinPoint jp) {  
        // Usa il pointcut definito altrove  
    }  
}  
```  
  
## Esempi Pratici di Pointcut con Espressioni AspectJ  
  
### Esempio 1: Selezionare per Nome Metodo  
  
```java  
@Aspect  
@Component  
@Slf4j  
public class MethodNameAspect {  
  
    // Tutti i metodi che iniziano con "find"  
    @Pointcut("execution(* find*(..))")  
    public void findMethods() { }  
  
    // Tutti i metodi che iniziano con "save", "update" o "delete"  
    @Pointcut("execution(* save*(..)) || " +  
              "execution(* update*(..)) || " +  
              "execution(* delete*(..))")  
    public void modificationMethods() { }  
  
    @Before("findMethods()")  
    public void beforeFind(JoinPoint jp) {  
        log.info("🔍 Query: {}", jp.getSignature().getName());  
    }  
  
    @Before("modificationMethods()")  
    public void beforeModification(JoinPoint jp) {  
        log.warn("⚠️ Modifica: {}", jp.getSignature().getName());  
    }  
}  
```  
  
### Esempio 2: Selezionare per Package  
  
```java  
@Aspect  
@Component  
public class PackageAspect {  
  
    // Tutti i metodi in un package specifico  
    @Pointcut("execution(* com.example.service.*.*(..))")  
    public void servicePackage() { }  
  
    // Package e tutti i sotto-package  
    @Pointcut("execution(* com.example.service..*.*(..))")  
    public void servicePackageAndSubpackages() { }  
  
    // Classe specifica  
    @Pointcut("execution(* com.example.service.UserService.*(..))")  
    public void userServiceClass() { }  
  
    @Around("servicePackageAndSubpackages()")  
    public Object measureServicePerformance(ProceedingJoinPoint pjp)  
            throws Throwable {  
        long start = System.currentTimeMillis();  
        Object result = pjp.proceed();  
        long duration = System.currentTimeMillis() - start;  
  
        System.out.println(pjp.getSignature() + " took " + duration + "ms");  
        return result;  
    }  
}  
```  
  
### Esempio 3: Selezionare per Tipo di Ritorno  
  
```java  
@Aspect  
@Component  
public class ReturnTypeAspect {  
  
    // Metodi che ritornano void  
    @Pointcut("execution(void *(..))")  
    public void voidMethods() { }  
  
    // Metodi che ritornano User  
    @Pointcut("execution(com.example.model.User *(..))")  
    public void userReturningMethods() { }  
  
    // Metodi che ritornano qualsiasi tipo  
    @Pointcut("execution(* *(..))")  
    public void anyReturnType() { }  
  
    @AfterReturning(  
        pointcut = "userReturningMethods()",  
        returning = "user"  
    )  
    public void afterReturningUser(JoinPoint jp, User user) {  
        System.out.println("Ritornato user: " + user.getUsername());  
    }  
}  
```  
  
### Esempio 4: Selezionare per Parametri  
  
```java  
@Aspect  
@Component  
public class ParameterAspect {  
  
    // Metodi che accettano un Long  
    @Pointcut("execution(* *(Long)) && args(id)")  
    public void methodsWithLongId(Long id) { }  
  
    // Metodi che accettano Long e String  
    @Pointcut("execution(* *(Long, String)) && args(id, name)")  
    public void methodsWithIdAndName(Long id, String name) { }  
  
    // Metodi con qualsiasi numero di parametri ma primo è Long  
    @Pointcut("execution(* *(Long, ..)) && args(id, ..)")  
    public void methodsStartingWithLong(Long id) { }  
  
    @Before("methodsWithLongId(id)")  
    public void logIdParameter(JoinPoint jp, Long id) {  
        System.out.println("Chiamato con ID: " + id);  
    }  
  
    @Before("methodsWithIdAndName(id, name)")  
    public void logIdAndName(JoinPoint jp, Long id, String name) {  
        System.out.println("ID: " + id + ", Name: " + name);  
    }  
}  
```  
  
### Esempio 5: Selezionare per Annotazioni  
  
```java  
@Aspect  
@Component  
public class AnnotationAspect {  
  
    // Metodi con @Transactional  
    @Pointcut("@annotation(org.springframework.transaction.annotation.Transactional)")  
    public void transactionalMethods() { }  
  
    // Metodi con annotazione custom  
    @Pointcut("@annotation(com.example.annotation.Auditable)")  
    public void auditableMethods() { }  
  
    // Tutti i metodi di classi con @Service  
    @Pointcut("@within(org.springframework.stereotype.Service)")  
    public void serviceBeans() { }  
  
    // Tutti i metodi di classi con @RestController  
    @Pointcut("@within(org.springframework.web.bind.annotation.RestController)")  
    public void restControllers() { }  
  
    @Around("auditableMethods()")  
    public Object auditMethod(ProceedingJoinPoint pjp) throws Throwable {  
        System.out.println("Auditing: " + pjp.getSignature());  
        return pjp.proceed();  
    }  
}  
```  
  
### Esempio 6: Combinazioni Complesse  
  
```java  
@Aspect  
@Component  
public class ComplexPointcutAspect {  
  
    // Metodi pubblici nei service che non sono getter  
    @Pointcut("execution(public * com.example.service.*.*(..)) && " +  
              "!execution(* get*(..))")  
    public void publicNonGetterServiceMethods() { }  
  
    // Metodi di modifica (create/update/delete) solo nei controller  
    @Pointcut("within(com.example.controller..*) && " +  
              "(execution(* create*(..)) || " +  
              " execution(* update*(..)) || " +  
              " execution(* delete*(..)))")  
    public void controllerModificationMethods() { }  
  
    // Metodi transazionali nel service layer che ritornano void  
    @Pointcut("@annotation(org.springframework.transaction.annotation.Transactional) && " +  
              "within(com.example.service..*) && " +  
              "execution(void *(..))")  
    public void voidTransactionalServiceMethods() { }  
  
    @Before("publicNonGetterServiceMethods()")  
    public void beforePublicServiceNonGetter(JoinPoint jp) {  
        System.out.println("Service pubblico non-getter: " + jp.getSignature());  
    }  
}  
```  
  
## Come si Collega un Advice a un Pointcut  
  
Ci sono **due approcci** per collegare un advice a un pointcut:[1]  
  
### Approccio 1: Riferimento Diretto  
  
```java  
@Aspect  
@Component  
public class DirectLinkAspect {  
  
    // Definisci il pointcut  
    @Pointcut("execution(* com.example.service.*.*(..))")  
    public void serviceLayer() { }  
  
    // Collega l'advice al pointcut usando il nome del metodo  
    @Before("serviceLayer()")  
    //       └── Nome del metodo @Pointcut  
    public void beforeServiceMethod(JoinPoint jp) {  
        System.out.println("Prima di: " + jp.getSignature());  
    }  
  
    @After("serviceLayer()")  
    public void afterServiceMethod(JoinPoint jp) {  
        System.out.println("Dopo: " + jp.getSignature());  
    }  
  
    @Around("serviceLayer()")  
    public Object aroundServiceMethod(ProceedingJoinPoint pjp)  
            throws Throwable {  
        System.out.println("Attorno - PRIMA");  
        Object result = pjp.proceed();  
        System.out.println("Attorno - DOPO");  
        return result;  
    }  
}  
```  
  
### Approccio 2: Riferimento Esterno (Fully Qualified)  
  
Quando il pointcut è in un'altra classe:[3]  
  
```java  
// File: CommonPointcuts.java  
@Aspect  
@Component  
public class CommonPointcuts {  
  
    @Pointcut("execution(* com.example.service.*.*(..))")  
    public void serviceLayer() { }  
  
    @Pointcut("execution(* com.example.controller.*.*(..))")  
    public void webLayer() { }  
}  
  
// File: LoggingAspect.java  
@Aspect  
@Component  
public class LoggingAspect {  
  
    // Riferimento con fully qualified name  
    @Before("com.example.aspect.CommonPointcuts.serviceLayer()")  
    //       └── package.classe.metodoPointcut()  
    public void logService(JoinPoint jp) {  
        System.out.println("Service: " + jp.getSignature());  
    }  
  
    @Before("com.example.aspect.CommonPointcuts.webLayer()")  
    public void logController(JoinPoint jp) {  
        System.out.println("Controller: " + jp.getSignature());  
    }  
}  
```  
  
### Collegamento con Cattura Parametri  
  
Puoi passare parametri dal pointcut all'advice:  
  
```java  
@Aspect  
@Component  
public class ParameterPassingAspect {  
  
    // Pointcut che cattura il parametro id  
    @Pointcut("execution(* com.example.service.*.*(Long)) && args(userId)")  
    public void methodsWithUserId(Long userId) { }  
    //                             └── Parametro catturato  
  
    // L'advice riceve il parametro  
    @Before("methodsWithUserId(userId)")  
    public void logUserId(JoinPoint jp, Long userId) {  
        //                                └── Stesso nome!  
        System.out.println("Operazione su user ID: " + userId);  
    }  
}  
```  
  
### Collegamento con Annotazioni  
  
Cattura l'annotazione e usala nell'advice:  
  
```java  
// Annotazione custom  
@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface Auditable {  
    String action();  
    String severity() default "MEDIUM";  
}  
  
@Aspect  
@Component  
public class AuditAspect {  
  
    // Cattura l'annotazione  
    @Around("@annotation(auditable)")  
    //                    └── Nome variabile  
    public Object audit(ProceedingJoinPoint pjp, Auditable auditable)  
            throws Throwable {  
        //                                        └── Riceve l'annotazione  
  
        String action = auditable.action();  
        String severity = auditable.severity();  
  
        System.out.println("Azione: " + action);  
        System.out.println("Severità: " + severity);  
  
        return pjp.proceed();  
    }  
}  
  
// Uso  
@Service  
public class UserService {  
  
    @Auditable(action = "DELETE_USER", severity = "HIGH")  
    public void deleteUser(Long id) {  
        // L'aspect riceve l'annotazione e può leggerne i parametri  
    }  
}  
```  
  
## Schema Completo: Join Point → Pointcut → Advice  
  
```java  
@Aspect  
@Component  
@Slf4j  
public class CompleteExample {  
  
    // 1. DEFINISCI I POINTCUT (filtri per join point)  
  
    @Pointcut("execution(* com.example.service.*.*(..))")  
    public void allServiceMethods() { }  
  
    @Pointcut("execution(* get*(..))")  
    public void getterMethods() { }  
  
    @Pointcut("execution(* set*(..))")  
    public void setterMethods() { }  
  
    @Pointcut("allServiceMethods() && !getterMethods() && !setterMethods()")  
    public void businessMethods() { }  
  
    // 2. COLLEGA ADVICE AI POINTCUT  
  
    @Before("businessMethods()")  
    public void logBefore(JoinPoint jp) {  
        // JOIN POINT fornisce info sul metodo corrente  
        String className = jp.getTarget().getClass().getSimpleName();  
        String methodName = jp.getSignature().getName();  
        Object[] args = jp.getArgs();  
  
        log.info("🚀 [{}] {} chiamato con: {}",  
                 className, methodName, Arrays.toString(args));  
    }  
  
    @AfterReturning(  
        pointcut = "businessMethods()",  
        returning = "result"  
    )  
    public void logAfterReturning(JoinPoint jp, Object result) {  
        log.info("✅ {} completato con: {}",  
                 jp.getSignature().getName(), result);  
    }  
  
    @AfterThrowing(  
        pointcut = "businessMethods()",  
        throwing = "error"  
    )  
    public void logAfterThrowing(JoinPoint jp, Throwable error) {  
        log.error("❌ {} fallito: {}",  
                  jp.getSignature().getName(), error.getMessage());  
    }  
  
    @Around("businessMethods()")  
    public Object measurePerformance(ProceedingJoinPoint pjp)  
            throws Throwable {  
  
        long start = System.currentTimeMillis();  
  
        // Esegui il metodo (il join point reale)  
        Object result = pjp.proceed();  
  
        long duration = System.currentTimeMillis() - start;  
  
        log.info("⏱️ {} eseguito in {} ms",  
                 pjp.getSignature().getName(), duration);  
  
        return result;  
    }  
}  
```  
  
## Riepilogo Visivo  
  
```  
┌─────────────────────────────────────────────────────┐  
│ CODICE APPLICATIVO (Service/Controller)            │  
├─────────────────────────────────────────────────────┤  
│                                                     │  
│  public User getUser(Long id) { }  ← JOIN POINT 1  │  
│  public void saveUser(User u) { }  ← JOIN POINT 2  │  
│  public void deleteUser(Long id) { }←JOIN POINT 3  │  
│                                                     │  
└─────────────────────────────────────────────────────┘  
                        ↓  
┌─────────────────────────────────────────────────────┐  
│ POINTCUT (Filtro/Selezione)                        │  
├─────────────────────────────────────────────────────┤  
│                                                     │  
│  @Pointcut("execution(* save*(..))")               │  
│  public void saveMethods() { }                      │  
│                                                     │  
│  Seleziona: JOIN POINT 2                           │  
│                                                     │  
└─────────────────────────────────────────────────────┘  
                        ↓  
┌─────────────────────────────────────────────────────┐  
│ ADVICE (Azione)                                     │  
├─────────────────────────────────────────────────────┤  
│                                                     │  
│  @Before("saveMethods()")                          │  
│  public void beforeSave(JoinPoint jp) {            │  
│      log.info("Salvataggio...");                   │  
│  }                                                  │  
│                                                     │  
│  Viene eseguito quando JOIN POINT 2 viene chiamato │  
│                                                     │  
└─────────────────────────────────────────────────────┘  
```  
  
## Best Practices Finali  
  
> [!tip] Raccomandazioni  
> 1. **Usa @Pointcut nominati**: Riutilizza invece di duplicare espressioni  
> 2. **Centralizza pointcut comuni**: Crea una classe `CommonPointcuts`  
> 3. **Sii specifico**: Meglio `execution(* com.example.service.*.*(..))` che `execution(* *(..))`  
> 4. **Documenta i pointcut**: Non è ovvio cosa matchano  
> 5. **Preferisci @annotation**: Più esplicito del pattern matching generico  
> 6. **Testa i pointcut**: Verifica che matchino esattamente ciò che vuoi  
> 7. **Usa JoinPoint per info runtime**: Nome metodo, parametri, target  
  
***  
  
Spero questa guida completa ti abbia chiarito la relazione tra Join Point, Pointcut e Advice! Vuoi che ti mostri un esempio pratico completo per il tuo progetto specifico?  
  
[1](https://www.html.it/articoli/spring-aop-ed-aspectj-una-introduzione-alla-programmazione-ad-aspetti-in-java/)  
[2](http://losviluppatore.it/implementare-l-aspect-oriented-programming-con-spring/)  
[3](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/pointcuts.html)  
[4](https://mokabyte.it/2006/12/01/spring-3/)  
[5](https://stackoverflow.com/questions/64584394/whats-the-meaning-of-the-method-use-pointcut-in-spring-aop-just-a-pointcut-sig)  
[6](https://www.emmecilab.net/blog/un-esempio-pratico-di-aop-aspect-oriented-programming-in-spring/)  
[7](https://www.baeldung.com/spring-aop-pointcut-tutorial)  
[8](https://www.youtube.com/watch?v=8OKC3c6ryKg)