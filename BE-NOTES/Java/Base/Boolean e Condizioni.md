
```java
boolean x = true;
boolean y = false;
```

### Operatori di Confronto

| Operatore | Significato | Esempio |
|-----------|------------|---------|
| `==` | Uguale a | `5 == 5` → true |
| `!=` | Diverso da | `5 != 3` → true |
| `<` | Minore di | `5 < 10` → true |
| `>` | Maggiore di | `5 > 3` → true |
| `<=` | Minore o uguale | `5 <= 5` → true |
| `>=` | Maggiore o uguale | `5 >= 3` → true |

### Operatori Logici

```java
// AND (&&) - Entrambe le condizioni devono essere true
if (eta > 18 && patente == true) {
    System.out.println("Puoi guidare");
}

// OR (||) - Almeno una condizione deve essere true
if (giorno.equals("sabato") || giorno.equals("domenica")) {
    System.out.println("È weekend");
}

// NOT (!) - Nega la condizione
if (!piove) {
    System.out.println("Usciamo");
}
```

### Costrutto If-Else

```java
if (condizione) {
    // Esecuzione se true
} else if (altraCondizione) {
    // Esecuzione se else if è true
} else {
    // Esecuzione se tutte le precedenti sono false
}
```

### Costrutto Switch

```java
switch(giorno) {
    case "lunedì":
        System.out.println("Inizio settimana");
        break;
    case "venerdì":
        System.out.println("Fine settimana");
        break;
    default:
        System.out.println("Giorno normale");
}
```

### Operatore Ternario

```java
String risultato = (eta >= 18) ? "Maggiorenne" : "Minorenne";
```