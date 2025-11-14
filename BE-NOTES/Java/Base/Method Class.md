## 5. Classe Method  
  
#java/reflection #method  
  
> [!info] Definizione  
> `Method` rappresenta un metodo e permette di ==ottenere info== e ==invocare== il metodo dinamicamente.  
  
### Ottenere un Method  
  
```java  
import java.lang.reflect.Method;  
  
public class GetMethodExample {  
    public static void main(String[] args) throws Exception {  
        // getMethod() - solo pubblici  
        Method method1 = String.class  
            .getMethod("substring", int.class, int.class);  
          
        // getDeclaredMethod() - tutti  
        Method method2 = String.class  
            .getDeclaredMethod("substring", int.class);  
          
        // getMethods() - tutti pubblici  
        Method[] allPublic = String.class.getMethods();  
          
        // getDeclaredMethods() - solo dichiarati  
        Method[] declared = String.class.getDeclaredMethods();  
    }  
}  
```  
  
### Informazioni sul Method  
  
```java  
import java.lang.reflect.Method;  
import java.lang.reflect.Modifier;  
  
public class MethodInfo {  
    public static void main(String[] args) throws Exception {  
        Method method = HashMap.class  
            .getMethod("put", Object.class, Object.class);  
          
        System.out.println("Nome: " + method.getName());  
        System.out.println("Parametri: " +   
            Arrays.toString(method.getParameterTypes()));  
        System.out.println("Ritorno: " + method.getReturnType());  
        System.out.println("Modificatori: " +   
            Modifier.toString(method.getModifiers()));  
    }  
}  
```  
  
### Invocare un Metodo  
  
> [!example] Invocazione Dinamica  
  
```java  
import java.lang.reflect.Method;  
import java.util.HashMap;  
  
public class InvokeExample {  
    public static void main(String[] args) throws Exception {  
        // Metodo senza parametri  
        String str = "hello";  
        Method toUpper = str.getClass().getMethod("toUpperCase");  
        String result = (String) toUpper.invoke(str);  
          
        // Metodo con parametri  
        Method putMethod = HashMap.class  
            .getMethod("put", Object.class, Object.class);  
          
        Map<String, String> map = new HashMap<>();  
        putMethod.invoke(map, "key1", "value1");  
          
        // Metodo statico  
        Method maxMethod = Math.class  
            .getMethod("max", int.class, int.class);  
        int max = (int) maxMethod.invoke(null, 10, 20);  
    }  
}  
```  
  
### Calculator Dinamico  
  
```java  
class Calculator {  
    public int add(int a, int b) { return a + b; }  
    public int subtract(int a, int b) { return a - b; }  
    public int multiply(int a, int b) { return a * b; }  
    public int divide(int a, int b) { return a / b; }  
}  
  
public class DynamicCalculator {  
    public static void main(String[] args) throws Exception {  
        Calculator calc = new Calculator();  
          
        String operation = "multiply";  
        int num1 = 10, num2 = 5;  
          
        Method method = Calculator.class  
            .getMethod(operation, int.class, int.class);  
          
        int result = (int) method.invoke(calc, num1, num2);  
        System.out.println(result);  
    }  
}  
```  
  
### Plugin System  
  
> [!example] Sistema di Plugin <  
  
```java  
import java.lang.reflect.Method;  
import java.util.HashMap;  
import java.util.Map;  
  
@interface Command {  
    String name();  
}  
  
class CommandHandler {  
    @Command(name = "hello")  
    public void sayHello(String name) {  
        System.out.println("Hello, " + name + "!");  
    }  
      
    @Command(name = "add")  
    public int addNumbers(int a, int b) {  
        return a + b;  
    }  
}  
  
public class PluginSystem {  
    private Map<String, Method> commands = new HashMap<>();  
    private Object handler;  
      
    public PluginSystem(Object handler) {  
        this.handler = handler;  
        registerCommands();  
    }  
      
    private void registerCommands() {  
        for (Method method : handler.getClass().getDeclaredMethods()) {  
            if (method.isAnnotationPresent(Command.class)) {  
                Command cmd = method.getAnnotation(Command.class);  
                commands.put(cmd.name(), method);  
            }  
        }  
    }  
      
    public Object executeCommand(String name, Object... args)   
            throws Exception {  
        Method method = commands.get(name);  
        if (method == null) {  
            throw new IllegalArgumentException("Command not found");  
        }  
        return method.invoke(handler, args);  
    }  
}  
```  
  
> [!tip] Quando Usare Method  
> - Framework (Spring, JUnit)  
> - Serialization (JSON/XML)  
> - Plugin systems  
> - Command pattern dinamico  
> - Testing frameworks  
  
> [!danger] Limitazioni  
>   
> | Problema | Descrizione |  
> |----------|-------------|  
> | **Performance** | 2-3x più lento |  
> | **Type Safety** | Errori solo a runtime |  
> | **Sicurezza** | Può violare incapsulamento |  
> | **Exception** | Wrapping in InvocationTargetException |  
  
  
  
- [Oracle Java Generics](https://docs.oracle.com/javase/tutorial/java/generics/)  
- [Java Serialization](https://docs.oracle.com/javase/8/docs/technotes/guides/serialization/)  
- [Reflection API](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/package-summary.html)  
- [Spring Best Practices](https://spring.io/guides)