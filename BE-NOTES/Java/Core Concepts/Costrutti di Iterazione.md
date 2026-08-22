---
topic: "Costrutti di Iterazione"
nav_prev: "[[Boolean e Condizioni.md]]"
nav_next: "[[Generics.md]]"
---

I cicli permettono di **ripetere un blocco di codice** finché una condizione è vera o per ogni elemento di una collezione.

## For classico — quando sai quante iterazioni

```java
for (int i = 0; i < 10; i++) {
    System.out.println("Iterazione " + i);
}
```

**Quando usarlo:** quando hai un indice numerico, devi iterare in un intervallo specifico, o modificare l'indice durante il ciclo.

**Problemi comuni:**
- **Off-by-one**: `i <= 10` invece di `i < 10` → 11 iterazioni invece di 10
- **Variabile dimenticata**: `for (int i = 0; i < 10; j++)` → loop infinito (i non cambia mai)
- **Modifica dell'indice nel corpo**: `for (int i = 0; i < 10; i++) { i++; }` → salta elementi

## For-each (enhanced for) — quando iteri su collezioni/array

```java
for (String nome : nomi) {
    System.out.println(nome);
}
```

**Quando usarlo:** SEMPRE per iterare su array e `Iterable` (List, Set, Queue). Non puoi sbagliare indice.

**Vantaggi:** più leggibile, nessun rischio off-by-one, funziona con qualsiasi `Iterable`.

**Problemi comuni:**
- **Non puoi modificare la struttura**: `list.remove(elemento)` durante un for-each lancia `ConcurrentModificationException`. Usa `iterator.remove()` o `removeIf()`.
- **Non hai l'indice**: se ti serve la posizione, usa for classico o una variabile contatore esterna.

## While — quando non sai quante iterazioni

```java
while (condizione) {
    // Esecuzione finché la condizione è vera
}

// Esempio tipico: leggere file fino alla fine
while ((linea = reader.readLine()) != null) {
    System.out.println(linea);
}
```

**Quando usarlo:** iterazioni con termine variabile, dipendente da input esterno.

**Problema comune:** **loop infinito** se la condizione non diventa mai falsa. Assicurati che qualcosa cambi la condizione dentro il corpo.

## Do-While — almeno una esecuzione garantita

```java
do {
    // Esecuzione ALMENO UNA VOLTA
} while (condizione);

// Esempio: menu interattivo
int scelta;
do {
    scelta = leggiSceltaUtente();
    eseguiAzione(scelta);
} while (scelta != 0);
```

**Differenza con while:** il corpo viene eseguito prima di controllare la condizione. Utile quando devi fare qualcosa almeno una volta (es. mostrare un menu prima di chiedere se uscire).

## Break e Continue

```java
for (int i = 0; i < 10; i++) {
    if (i == 5) break;         // Esce dal ciclo → stampa 0,1,2,3,4
    if (i == 3) continue;      // Salta all'iterazione successiva → non stampa 3
    System.out.println(i);
}
```

| Istruzione | Cosa fa | Quando usarla |
|---|---|---|
| `break` | Esce immediatamente dal ciclo | Hai trovato ciò che cercavi, interrompi |
| `continue` | Salta alla prossima iterazione | Vuoi saltare un elemento specifico ma continuare |

## ForEach con lambda (Java 8+)

```java
// Internal iteration — Java gestisce il ciclo
nomi.forEach(nome -> System.out.println(nome));
nomi.forEach(System.out::println);  // method reference

// Con stream per operazioni più complesse
nomi.stream()
    .filter(n -> n.startsWith("A"))
    .map(String::toUpperCase)
    .forEach(System.out::println);
```

**Differenza:** il for-each classico è **external iteration** (tu controlli il ciclo). `forEach()` con lambda è **internal iteration** (Java gestisce il ciclo internamente, più espressivo ma meno controllabile).

## Quale ciclo usare?

| Scenario | Ciclo consigliato |
|---|---|
| Iterare su array/lista con indice | For classico |
| Iterare su array/lista senza indice | For-each |
| Iterare su stream con filtri | `stream().forEach()` |
| Numero imprecisato di iterazioni | While |
| Almeno una esecuzione garantita | Do-while |
| Modificare elementi durante iterazione | For classico o Iterator |
| Rimuovere elementi durante iterazione | `removeIf()` o Iterator |