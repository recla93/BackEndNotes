
```java
// For classico
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}

// For-each (enhanced for)
for (String nome : nomi) {
    System.out.println(nome);
}
```

### While Loop

```java
while (condizione) {
    // Esecuzione finché la condizione è vera
}
```

### Do-While Loop

```java
do {
    // Esecuzione almeno una volta
} while (condizione);
```

### Break e Continue

```java
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;  // Esce dal ciclo
    }
    if (i == 3) {
        continue;  // Salta all'iterazione successiva
    }
    System.out.println(i);
}
```