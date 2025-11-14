
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
          
        // La password sarà null dopo deserializzazione  
    }  
}  
```  
  
> [!check] Quando usare transient  
> - Dati sensibili (password, PIN, token)  
> - Dati derivati/calcolati  
> - Risorse runtime (connessioni DB, stream)  
> - Cache temporanee  
  
***  
  