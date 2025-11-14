## 3. Reflection API  
  
#java/reflection #advanced  
  
> [!info] Definizione  
> La **Reflection API** permette di ==ispezionare e manipolare== classi, metodi e campi a runtime.  
  
### Funzionalità Principali  
  
| Funzionalità | Descrizione |  
|--------------|-------------|  
| **Ispezione** | Ottenere info su classi, metodi, campi |  
| **Istanziazione** | Creare oggetti dinamicamente |  
| **Invocazione** | Chiamare metodi a runtime |  
| **Modifica** | Cambiare valori di campi privati |  
  
### Ottenere Informazioni  
  
```java  
import java.lang.reflect.*;  
  
public class ReflectionBasics {  
    public static void main(String[] args) throws Exception {  
        Class<?> clazz = String.class;  
          
        System.out.println("Nome: " + clazz.getName());  
          
        Method[] methods = clazz.getMethods();  
        for (Method method : methods) {  
            System.out.println("- " + method.getName());  
        }  
          
        Field[] fields = clazz.getDeclaredFields();  
        for (Field field : fields) {  
            System.out.println("- " + field.getName() + " : " + field.getType());  
        }  
    }  
}  
```  
  
### Invocare Metodi  
  
```java  
import java.lang.reflect.Method;  
  
public class MethodInvocation {  
    public static void main(String[] args) throws Exception {  
        String str = "hello world";  
        Method toUpperCase = str.getClass().getMethod("toUpperCase");  
        String result = (String) toUpperCase.invoke(str);  
        System.out.println(result);  
          
        Method substring = str.getClass()  
            .getMethod("substring", int.class, int.class);  
        String sub = (String) substring.invoke(str, 0, 5);  
        System.out.println(sub);  
    }  
}  
```  
  
### Accedere a Campi Privati  
  
> [!warning] Attenzione  
> Modificare campi privati viola l'incapsulamento. Usare solo quando necessario!  
  
```java  
import java.lang.reflect.Field;  
  
class Person {  
    private String name;  
    private int age;  
      
    public Person(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
}  
  
public class AccessPrivateFields {  
    public static void main(String[] args) throws Exception {  
        Person person = new Person("John", 30);  
          
        Field nameField = Person.class.getDeclaredField("name");  
        nameField.setAccessible(true);  
          
        String name = (String) nameField.get(person);  
        System.out.println("Nome: " + name);  
          
        nameField.set(person, "Jane");  
        System.out.println("Modificato: " + nameField.get(person));  
    }  
}  
```  
  
### Esempio: Dependency Injection  
  
> [!example] Mini Framework DI  
> Implementazione semplificata di dependency injection  
  
```java  
import java.lang.reflect.Field;  
  
@interface Inject {}  
  
class Database {  
    public void connect() {  
        System.out.println("Connesso al database");  
    }  
}  
  
class UserService {  
    @Inject  
    private Database database;  
      
    public void doSomething() {  
        database.connect();  
    }  
}  
  
public class SimpleDI {  
    public static void inject(Object target) throws Exception {  
        Class<?> clazz = target.getClass();  
          
        for (Field field : clazz.getDeclaredFields()) {  
            if (field.isAnnotationPresent(Inject.class)) {  
                field.setAccessible(true);  
                Object instance = field.getType()  
                    .getDeclaredConstructor()  
                    .newInstance();  
                field.set(target, instance);  
            }  
        }  
    }  
      
    public static void main(String[] args) throws Exception {  
        UserService service = new UserService();  
        inject(service);  
        service.doSomething();  
    }  
}  
```  
  
> [!tip] Quando utilizzare Reflection  
> - Framework (Spring, Hibernate)  
> - Serializzazione JSON/XML  
> - Testing (Mockito)  
> - Plugin systems  
> - Annotazioni custom  
  
> [!danger] Limitazioni  
> - **Performance**: 2-3x più lenta  
> - **Sicurezza**: Viola incapsulamento  
> - **Complessità**: Codice meno leggibile  
> - **Type Safety**: Errori solo a runtime  
  