---
topic: "Method Class"
nav_prev: "[[Generics.md]]"
nav_next: "[[Java Best Practices.md]]"
---

#java/reflection #method

`Method` rappresenta un metodo della JVM a runtime e permette di ottenere informazioni e invocare metodi **dinamicamente**, senza saperli in compilazione.

## Quando usare reflection

- **Framework** — Spring invoca metodi di controller, JUnit esegue `@Test`
- **Serialization** — Jackson, Gson leggono getter per serializzare
- **Plugin system** — carichi classi dinamicamente e invochi metodi per nome
- **Annotation processing** — leggi `@RequestMapping`, `@Transactional` a runtime
- **Proxy/AOP** — creazione di proxy dinamici per log, transaction, security

## Quando NON usare reflection

- **Codice business** — preferisci interfacce tipizzate e chiamate dirette
- **Hot path** — reflection è 2-100x più lento delle chiamate dirette
- **Se puoi usare lambda/method reference** — `Function<String, Integer>` invece di `Method.invoke()`
- **Refactoring** — reflection non dà errori in compilazione se rinominano un metodo

## Ottenere un Method

```java
import java.lang.reflect.Method;

// getMethod() — solo metodi pubblici (inclusi ereditati)
Method m1 = String.class.getMethod("substring", int.class, int.class);

// getDeclaredMethod() — tutti i visibilità, solo dichiarati nella classe
Method m2 = String.class.getDeclaredMethod("substring", int.class);

// getMethods() — tutti i metodi pubblici (inclusi ereditati)
Method[] allPublic = String.class.getMethods();

// getDeclaredMethods() — tutti i metodi dichiarati (qualsiasi visibilità)
Method[] declared = String.class.getDeclaredMethods();
```

## Informazioni sul Method

```java
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;

Method method = HashMap.class.getMethod("put", Object.class, Object.class);

System.out.println("Nome: " + method.getName());
System.out.println("Parametri: " + Arrays.toString(method.getParameterTypes()));
System.out.println("Ritorno: " + method.getReturnType());
System.out.println("Modificatori: " + Modifier.toString(method.getModifiers()));
System.out.println("Annotationi: " + Arrays.toString(method.getAnnotations()));
```

## Invocare un Metodo

```java
import java.lang.reflect.Method;

// Metodo di istanza — senza parametri
String str = "hello";
Method toUpper = str.getClass().getMethod("toUpperCase");
String result = (String) toUpper.invoke(str);  // "HELLO"

// Metodo di istanza — con parametri
Method putMethod = HashMap.class.getMethod("put", Object.class, Object.class);
putMethod.invoke(map, "key1", "value1");

// Metodo statico — primo argomento = null
Method maxMethod = Math.class.getMethod("max", int.class, int.class);
int max = (int) maxMethod.invoke(null, 10, 20);  // 20

// Metodo privato — serve setAccessible(true)
Method privateMethod = MyClass.class.getDeclaredMethod("secret");
privateMethod.setAccessible(true);  // viola incapsulamento!
privateMethod.invoke(instance);
```

## Esempio pratico: Dynamic Calculator

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
        String operation = "multiply";  // da input utente
        Method method = Calculator.class.getMethod(operation, int.class, int.class);
        int result = (int) method.invoke(calc, 10, 5);
        System.out.println(result);  // 50
    }
}
```

## Esempio pratico: Sistema di Plugin

```java
import java.lang.reflect.Method;

@interface Command { String name(); }

class CommandHandler {
    @Command(name = "hello")
    public void sayHello(String name) {
        System.out.println("Hello, " + name + "!");
    }

    @Command(name = "add")
    public int addNumbers(int a, int b) { return a + b; }
}

public class PluginSystem {
    private Map<String, Method> commands = new HashMap<>();
    private Object handler;

    public PluginSystem(Object handler) {
        this.handler = handler;
        for (Method m : handler.getClass().getDeclaredMethods()) {
            if (m.isAnnotationPresent(Command.class)) {
                commands.put(m.getAnnotation(Command.class).name(), m);
            }
        }
    }

    public Object execute(String name, Object... args) throws Exception {
        Method method = commands.get(name);
        if (method == null) throw new IllegalArgumentException("Command not found: " + name);
        return method.invoke(handler, args);
    }
}
```

## Problemi comuni e soluzioni

| Problema | Causa | Soluzione |
|---|---|---|
| **InvocationTargetException** | Il metodo invocato ha lanciato eccezione | Usa `e.getCause()` per vedere l'eccezione originale |
| **NoSuchMethodException** | Nome o parametri sbagliati | Verifica firma esatta (inclusi tipi primitivi vs wrapper) |
| **IllegalAccessException** | Metodo privato/protetto | Chiama `method.setAccessible(true)` (se permesso) |
| **ClassCastException** | Cast errato del risultato | Usa `method.getReturnType()` per sapere cosa restituisce |
| **Performance degradata** | Chiamate reflection in loop | Cache del Method object, o usa `MethodHandle` (Java 7+) |
| **Boxing inaspettato** | `int.class` vs `Integer.class` | `getMethod("foo", int.class)` ≠ `getMethod("foo", Integer.class)` |

## Performance

```
Chiamata diretta:         ~2 ns
Method.invoke():         ~50-100 ns   (25-50x più lento)
MethodHandle.invoke():   ~10-20 ns    (Java 7+, 5-10x più lento)
setAccessible(true):     rimuove il controllo visibilità, migliora performance
```

Nei framework, il lookup del Method viene fatto una volta in fase di init e cacheato — non c'è impatto a runtime.
