# Cosa sono i PointCut?  
  
I **Pointcut** sono il "cervello selettivo" degli Aspect: decidono **DOVE** applicare il tuo codice (Advice). Sono espressioni/predicati che matchano uno o più Join Point, cioè i punti del programma dove vuoi intercettare l'esecuzione.[1][2][5]  
  
> [!info] Definizione Semplice  
> Un Pointcut è come un **filtro** o una **query** che dice: "Esegui questo Advice su tutti i metodi che soddisfano queste condizioni"  
  
## La Relazione: Join Point → Pointcut → Advice  
  
```  
Join Point: TUTTI i punti possibili dove intercettare  
    ↓  
Pointcut: SELEZIONE di quali join point ti interessano  
    ↓  
Advice: COSA fare quando il pointcut matcha  
```  
  
**Esempio pratico**:[3][5]  
- **Join Point**: Esecuzione del metodo `getUserById()`  
- **Pointcut**: "Tutti i metodi che iniziano con 'get' nei controller"  
- **Advice**: Logga il nome del metodo prima dell'esecuzione  
  
## Anatomia di un Pointcut  
  
```java  
@Before("execution(* com.example.controller.*.*(..))")  
//      └─────────────────┬────────────────────────┘  
//                    POINTCUT EXPRESSION  
```  
  
### Breakdown dell'Espressione  
  
```java  
execution(* com.example.controller.*.*(..))  
```  
  
> [!note] Decodifica Completa  
> - `execution`: Tipo di join point (esecuzione di metodo)  
> - `*` (primo): Qualsiasi tipo di ritorno (void, String, User, ecc.)  
> - `com.example.controller`: Package  
> - `*` (secondo): Qualsiasi classe nel package  
> - `*` (terzo): Qualsiasi metodo della classe  
> - `(..)`: Qualsiasi numero/tipo di parametri  
  
## Tipi di Pointcut Designator  
  
### 1. execution - Il Più Usato  
  
Matcha l'**esecuzione** di un metodo:[4]  
  
```java  
// Tutti i metodi pubblici  
execution(public * *(..))  
  
// Metodi che iniziano con "save"  
execution(* save*(..))  
  
// Metodi che ritornano User  
execution(com.example.model.User *(..))  
  
// Metodo specifico con parametri specifici  
execution(* com.example.service.UserService.findById(Long))  
  
// Tutti i metodi in un package e sotto-package  
execution(* com.example.service..*.*(..))  
```  
  
### 2. @annotation - Per Annotazioni Custom  
  
Matcha metodi con una specifica annotazione:  
  
```java  
@annotation(com.example.annotation.LoggaMi)  
```  
  
```java  
// Uso  
@LoggaMi  
public User getUser(Long id) {  
    // Questo metodo verrà intercettato  
}  
```  
  
### 3. within - Basato su Classi/Package  
  
Matcha tutti i metodi **dentro** certe classi o package:[8]  
  
```java  
// Tutti i metodi in un package  
within(com.example.service.*)  
  
// Tutti i metodi in package e sotto-package  
within(com.example.service..*)  
  
// Tutti i metodi di una classe specifica  
within(com.example.service.UserService)  
```  
  
### 4. @within - Classi con Annotazione  
  
Matcha tutti i metodi di classi annotate:  
  
```java  
// Tutti i metodi di classi con @Service  
@within(org.springframework.stereotype.Service)  
  
// Tutti i metodi di classi con @RestController  
@within(org.springframework.web.bind.annotation.RestController)  
```  
  
### 5. this e target  
  
```java  
// this: il proxy AOP è di tipo UserService  
this(com.example.service.UserService)  
  
// target: l'oggetto target è di tipo UserService  
target(com.example.service.UserService)  
```  
  
### 6. args - Basato sui Parametri  
  
Matcha metodi con parametri specifici:  
  
```java  
// Metodi che accettano un Long come parametro  
args(Long)  
  
// Metodi che accettano Long e String  
args(Long, String)  
  
// Metodi con qualsiasi numero di parametri ma il primo è Long  
args(Long, ..)  
```  
  
## Combinare Pointcut: AND, OR, NOT  
  
Puoi creare espressioni complesse:[2]  
  
```java  
@Aspect  
@Component  
public class ComplexAspect {  
      
    // AND: Metodi nei controller E che iniziano con "delete"  
    @Before("execution(* com.example.controller.*.*(..)) && " +  
            "execution(* delete*(..))")  
    public void beforeDelete(JoinPoint jp) {  
        // Eseguito solo su metodi delete* nei controller  
    }  
      
    // OR: Metodi create O update O delete  
    @Before("execution(* create*(..)) || " +  
            "execution(* update*(..)) || " +  
            "execution(* delete*(..))")  
    public void beforeModification(JoinPoint jp) {  
        // Eseguito su tutti i metodi di modifica  
    }  
      
    // NOT: Tutti i metodi TRANNE quelli che iniziano con "get"  
    @Before("execution(* com.example.service.*.*(..)) && " +  
            "!execution(* get*(..))")  
    public void beforeNonGetter(JoinPoint jp) {  
        // Eseguito su tutti i metodi tranne i getter  
    }  
}  
```  
  
## Pointcut Riusabili con @Pointcut  
  
Invece di ripetere la stessa espressione, definisci pointcut nominati:[5]  
  
```java  
@Aspect  
@Component  
public class ReusablePointcuts {  
      
    // Definisci pointcut riusabili  
    @Pointcut("execution(* com.example.controller.*.*(..))")  
    public void controllerMethods() {}  
      
    @Pointcut("execution(* com.example.service.*.*(..))")  
    public void serviceMethods() {}  
      
    @Pointcut("execution(* get*(..))")  
    public void getterMethods() {}  
      
    @Pointcut("execution(public * *(..))")  
    public void publicMethods() {}  
      
    // Usali negli advice  
    @Before("controllerMethods()")  
    public void beforeController(JoinPoint jp) {  
        // Codice  
    }  
      
    // Combinali  
    @Before("serviceMethods() && !getterMethods()")  
    public void beforeServiceNonGetter(JoinPoint jp) {  
        // Eseguito sui service, ma non sui getter  
    }  
      
    // Combina multipli  
    @Around("(controllerMethods() || serviceMethods()) && publicMethods()")  
    public Object aroundPublicControllerOrService(ProceedingJoinPoint pjp)   
            throws Throwable {  
        return pjp.proceed();  
    }  
}  
```  
  
## Esempi Pratici Reali  
  
### Caso 1: Logga Solo le Operazioni di Scrittura  
  
```java  
@Aspect  
@Component  
@Slf4j  
public class WriteOperationLogger {  
      
    // Pointcut che matcha create, update, delete, save  
    @Pointcut("execution(* create*(..)) || " +  
              "execution(* update*(..)) || " +  
              "execution(* delete*(..)) || " +  
              "execution(* save*(..))")  
    public void writeOperations() {}  
      
    @Before("writeOperations()")  
    public void logWriteOperation(JoinPoint jp) {  
        log.warn("⚠️ WRITE OPERATION: {}", jp.getSignature().getName());  
    }  
}  
```  
  
### Caso 2: Performance Monitoring Solo su Service Lenti  
  
```java  
@Aspect  
@Component  
@Slf4j  
public class SlowServiceMonitor {  
      
    // Solo i metodi dei service che non sono getter  
    @Pointcut("@within(org.springframework.stereotype.Service) && " +  
              "!execution(* get*(..))")  
    public void potentiallySlowServiceMethods() {}  
      
    @Around("potentiallySlowServiceMethods()")  
    public Object monitorPerformance(ProceedingJoinPoint pjp)   
            throws Throwable {  
        long start = System.currentTimeMillis();  
        Object result = pjp.proceed();  
        long duration = System.currentTimeMillis() - start;  
          
        if (duration > 1000) {  
            log.warn("🐌 SLOW: {} took {} ms",   
                     pjp.getSignature().getName(), duration);  
        }  
        return result;  
    }  
}  
```  
  
### Caso 3: Cattura Parametri Specifici  
  
```java  
@Aspect  
@Component  
@Slf4j  
public class UserIdLogger {  
      
    // Metodi che accettano Long come primo parametro  
    @Pointcut("execution(* com.example.service.*.*(Long, ..)) && " +  
              "args(userId, ..)")  
    public void methodsWithUserId(Long userId) {}  
      
    @Before("methodsWithUserId(userId)")  
    public void logUserId(JoinPoint jp, Long userId) {  
        log.info("👤 Operazione su userId: {}", userId);  
    }  
}  
```  
  
## Wildcard e Pattern Matching  
  
```java  
// * = un elemento qualsiasi  
execution(* *(..))  // qualsiasi metodo  
  
// .. = zero o più elementi  
execution(* com.example..*.*(..))  // package e sotto-package  
  
// + = tipo e tutte le sottoclassi  
execution(* com.example.service.BaseService+.*(..))  
  
// Combinazioni  
execution(* com.example.*.service..*Service.find*(..))  
// Package qualsiasi sotto com.example  
// Sotto-package "service"  
// Classi che finiscono con "Service"  
// Metodi che iniziano con "find"  
```  
  
## Pointcut in XML (Legacy)  
  
Prima delle annotazioni, si usava XML:[8]  
  
```xml  
<aop:config>  
    <aop:pointcut id="servicePointcut"  
                  expression="execution(* com.example.service.*.*(..))"/>  
      
    <aop:aspect ref="loggingAspect">  
        <aop:before pointcut-ref="servicePointcut"   
                    method="logBefore"/>  
    </aop:aspect>  
</aop:config>  
```  
  
**Oggi si preferiscono le annotazioni!**  
  
## Best Practices per Pointcut  
  
> [!tip] Consigli Pratici  
> 1. **Sii specifico**: `execution(* com.example.controller.*.*(..))` è meglio di `execution(* *(..))`  
> 2. **Usa @Pointcut nominati**: Riutilizza invece di duplicare  
> 3. **Preferisci @annotation**: Più esplicito e controllato che pattern generici  
> 4. **Testa i pointcut**: Verifica che matchino esattamente ciò che vuoi  
> 5. **Documenta pointcut complessi**: Non è ovvio cosa matchano  
  
## Errori Comuni  
  
> [!danger] Attenzione  
> ```java  
> // ❌ SBAGLIATO: Troppo generico, matcha TUTTO  
> @Before("execution(* *(..))")  
>   
> // ✅ CORRETTO: Specifico al tuo package  
> @Before("execution(* com.example.controller.*.*(..))")  
>   
> // ❌ SBAGLIATO: Sintassi errata  
> @Before("execution(* com.example.service)")  
>   
> // ✅ CORRETTO: Devi specificare classe e metodo  
> @Before("execution(* com.example.service.*.*(..))")  
> ```  
  
## Debug dei Pointcut  
  
Per vedere quali metodi vengono matchati:  
  
```java  
@Aspect  
@Component  
@Slf4j  
public class PointcutDebugger {  
      
    @Pointcut("execution(* com.example.service.*.*(..))")  
    public void serviceMethods() {}  
      
    @Before("serviceMethods()")  
    public void debugPointcut(JoinPoint jp) {  
        log.debug("🎯 MATCHED: {}", jp.getSignature().toLongString());  
    }  
}  
```  
  
Poi abilita il debug logging:  
```properties  
logging.level.com.example.aspect=DEBUG  
```  
  
## Riepilogo Visivo  
  
```  
POINTCUT = FILTRO CHE SELEZIONA DOVE APPLICARE L'ASPECT  
  
Join Points disponibili:  
├─ UserController.getUser()      ← ✅ Matcha  
├─ UserController.createUser()   ← ✅ Matcha  
├─ UserController.deleteUser()   ← ✅ Matcha  
├─ OrderService.processOrder()   ← ❌ Non matcha (non è controller)  
└─ ProductService.findProduct()  ← ❌ Non matcha (non è controller)  
  
Pointcut: execution(* com.example.controller.*.*(..))  
                      └─ Solo metodi nei controller  
```  
  
***  
  
**In sintesi**: I Pointcut sono le "query" o "filtri" che selezionano dove il tuo Aspect deve agire. Sono il collegamento tra "cosa fare" (Advice) e "dove farlo" (Join Point).[1][2][5]  
  