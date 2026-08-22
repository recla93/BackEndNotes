---
topic: "Transient Keyword"
nav_prev: "[[Reflection API.md]]"
---

#java/serialization #security  
  
> [!info] Definizione  
> `transient` indica che un campo ==NON deve essere serializzato== quando l'oggetto viene convertito in byte stream.  
  
### Serializzazione Base  
  
```java  
public class User implements Serializable {  
    private static final long serialVersionUID = 1L;  
      
    private String username;  
    private String email;  
    private transient String password;  
    private transient int sessionId;  
}  
```  
  
### Casi d'Uso Comuni  
  
#### 1. Dati Sensibili  
  
> [!warning] Sicurezza  
> Usa sempre `transient` per dati sensibili che non devono essere persistiti  
  
```java  
public class BankAccount implements Serializable {  
    private String accountNumber;  
    private String accountHolder;  
    private double balance;  
      
    private transient String pin;  
    private transient String sessionToken;  
}  
```  
  
#### 2. Dati Derivati  
  
```java  
public class Student implements Serializable {  
    private int id;  
    private String name;  
    private Date dob;  
      
    private transient int age;  
    private transient Map<String, Object> cache;  
}  
```  
  
### Esempio Completo  
  
> [!example] Test Serializzazione  
> Dimostrazione di come `transient` previene la serializzazione  
  
```java  
import java.io.*;  
  
public class TransientExample implements Serializable {  
    private static final long serialVersionUID = 1L;  
      
    private String username;  
    private int score;  
    private transient String password;  
      
    public TransientExample(String username, int score, String password) {  
        this.username = username;  
        this.score = score;  
        this.password = password;  
    }  
}  
  
class Main {  
    public static void main(String[] args) throws Exception {  
        TransientExample user = new TransientExample("john", 100, "secret123");  
          
        ObjectOutputStream out = new ObjectOutputStream(  
            new FileOutputStream("user.ser")  
        );  
        out.writeObject(user);  
        out.close();  
          
        ObjectInputStream in = new ObjectInputStream(  
            new FileInputStream("user.ser")  
        );  
        TransientExample loadedUser = (TransientExample) in.readObject();  
        in.close();  
          
        // La password sarÃ  null dopo deserializzazione  
    }  
}  
```  
  
> [!check] Quando usare transient  
> - Dati sensibili (password, PIN, token)  
> - Dati derivati/calcolati  
> - Risorse runtime (connessioni DB, stream)  
> - Cache temporanee  
  
***

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Campo `transient` null dopo deserializzazione | Campo vale `null` dopo `readObject()` anche se aveva un valore | `transient` esclude il campo dalla serializzazione | Inizializza il campo in un metodo `readObject()` personalizzato |
| `serialVersionUID` mancante | `InvalidClassException` dopo modifica della classe | La JVM calcola un UID automatico che cambia se la classe viene modificata | Dichiara sempre `private static final long serialVersionUID = 1L;` |
| `NotSerializableException` a runtime | Classe non implementa `Serializable` | Java non serializza automaticamente oggetti non `Serializable` | Implementa `Serializable` nella classe |
| Oggetti non serializzabili come campi | `NotSerializableException` durante serializzazione | Campi di tipo non `Serializable` referenziati nella classe | Rendi il campo `transient` o rendi serializzabile anche quella classe |
| `transient` usato su campi static | Nessun effetto, campo statico non viene mai serializzato | `static` già esclude dalla serializzazione | Usa solo `transient` su campi di istanza |

***