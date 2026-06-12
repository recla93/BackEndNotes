
***  
  
## DRY (Don't Repeat Yourself)  
  
> Evita di duplicare codice o logica. Ogni cosa dovrebbe avere un'unica rappresentazione.  
  
### Esempio:  
  
```java  
public class ValidatorUtils {  
    public static void validateEmail(String email) {  
        if (email == null || !email.contains("@")) {  
            throw new IllegalArgumentException("Email non valida");  
        }  
    }  
}  
```  
  
Usalo ovunque:  
  
```java  
ValidatorUtils.validateEmail(userDTO.getEmail());  
```  
  
***  
  
## SOC (Separation of Concerns)  
  
> Divide il sistema in componenti distinte con responsabilità separate.  
  
### Esempio: Controller, Service, Repository separati  
  
```java  
@RestController  
public class UserController {  
    private final UserService userService;  
    // ...  
}  
@Service  
public class UserService {  
    private final UserRepository userRepository;  
    // ...  
}  
@Repository  
public interface UserRepository extends JpaRepository<User, Long> { }  
```  
  
***  
  
## SSOT (Single Source of Truth)  
  
> Una singola fonte autorevole per ogni dato per evitare inconsistenze.  
  
***  
  
## API (Application Programming Interface) / REST  
  
> Interfacce per comunicare tra moduli software.  
  
### Esempio REST:  
  
```java  
@GetMapping("/users/{id}")  
public ResponseEntity<UserDTO> getUser(@PathVariable Long id) { ... }  
```  
  
***  
  
## DTO (Data Transfer Object)  
  
> Oggetti intermedi per trasferire dati senza esporre direttamente le entità.  
  
***  
  
## CRUD (Create, Read, Update, Delete)  
  
Le 4 operazioni fondamentali per gestire dati.  
  
***  
  
## IoC (Inversion of Control) & DI (Dependency Injection)  
> Gestione automatica delle dipendenze da parte di un container (es. Spring).  
  
```java  
@Service  
public class UserService {  
    private final UserRepository repo;  
  
    @Autowired  
    public UserService(UserRepository repo) {  
        this.repo = repo;  
    }  
}  
```  
  
***  
  
## SOLID: 5 Principi per un codice pulito e manutenibile  
  
***  
  
### S – Single Responsibility Principle (SRP)    
Una classe dovrebbe avere una sola ragione per cambiare, cioè una sola responsabilità.  
  
#### Esempio:  
  
```java  
public class ReportGenerator {  
    public String generate() {  
        return "Report generato";  
    }  
}  
  
public class ReportPrinter {  
    public void print(String report) {  
        System.out.println(report);  
    }  
}  
```  
  
***  
  
### O – Open/Closed Principle (OCP)    
Il software dovrebbe essere aperto alle estensioni, ma chiuso alle modifiche.  
  
#### Esempio:  
  
```java  
public interface PaymentService {  
    void processPayment(double amount);  
}  
  
public class PaypalPaymentService implements PaymentService {  
    @Override  
    public void processPayment(double amount) {  
        System.out.println("Pagamento di " + amount + " tramite Paypal");  
    }  
}  
```  
  
***  
  
### L – Liskov Substitution Principle (LSP)    
Le classi derivate devono poter sostituire le classi base senza alterare il comportamento.  
  
***  
  
### I – Interface Segregation Principle (ISP)    
Meglio molte interfacce piccole e specifiche che una sola grande e generale.  
  
***  
  
### D – Dependency Inversion Principle (DIP)    
Le classi dovrebbero dipendere da astrazioni, non da implementazioni concrete.  
  
#### Esempio:  
  
```java  
interface Developer {  
    void code();  
}  
  
class BackendDeveloper implements Developer {  
    public void code() {  
        System.out.println("Sto scrivendo codice backend");  
    }  
}  
  
class Project {  
    private Developer developer;  
  
    public Project(Developer developer) {  
        this.developer = developer;  
    }  
  
    public void implementFeature() {  
        developer.code();  
    }  
}  
```  
  
***  
  
# Design Pattern Comuni con esempi semplificati  
  
***  
  
## Singleton  
  
> Garantisce una sola istanza di una classe e un accesso globale ad essa.  
  
```java  
public class Logger {  
    private static Logger instance;  
  
    private Logger() {}  
  
    public static synchronized Logger getInstance() {  
        if (instance == null) {  
            instance = new Logger();  
        }  
        return instance;  
    }  
    public void log(String message) {  
        System.out.println(message);  
    }  
}  
```  
  
***  
  
## Factory  
  
> Creare oggetti senza specificare la loro classe esatta.  
  
```java  
public interface Animal {  
    void speak();  
}  
  
public class Dog implements Animal {  
    public void speak() {  
        System.out.println("Bau!");  
    }  
}  
  
public class AnimalFactory {  
    public static Animal createAnimal(String type) {  
        if ("dog".equalsIgnoreCase(type)) return new Dog();  
        // altre condizioni...  
        return null;  
    }  
}  
```  
  
***  
  
## Strategy  
  
> Encapsula algoritmi intercambiabili e li rende usabili in modo intercambiabile.  
  
```java  
public interface PaymentStrategy {  
    void pay(double amount);  
}  
  
public class CreditCardStrategy implements PaymentStrategy {  
    public void pay(double amount) {  
        System.out.println("Pagato " + amount + " con carta di credito");  
    }  
}  
```  
  
***  
  
## Observer  
  
> Permette a oggetti di essere notificati di cambiamenti in un altro oggetto.  
  
***  
  
Questo mix di acronimi, principi e design pattern serve per scrivere codice pulito, scalabile e manutenibile in progetti Spring Boot e non solo. Ogni esempio punta a chiarire il concetto con poche righe di codice facilmente applicabili.  
  
Se vuoi si può aggiungere un approfondimento su specifici pattern o casi d’uso particolari.  
  
[1](https://innovaformazione.net/principi-solid-nella-programmazione/)  
[2](https://www.spacecoding.it/principi-solid-java/)  
[3](https://codegym.cc/it/groups/posts/it.232.solid-cinque-principi-di-base-della-progettazione-di-classi-in-java)  
[4](https://www.reddit.com/r/developersIndia/comments/1g7y3yv/solid_principles_or_design_patterns_what_are_they/)  
[5](https://www.azionadigitale.com/principi-solid-per-il-design-del-software/)  
[6](http://losviluppatore.it/solid-design-principles/)  
[7](https://www.linkedin.com/pulse/mastering-solid-principles-spring-boot-microservices-shaik-ljrjc)