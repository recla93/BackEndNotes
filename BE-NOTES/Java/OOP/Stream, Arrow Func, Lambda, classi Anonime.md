## Lambda e Arrow Function

Modi corti per scrivere funzioni, invece di scrivere tutta la funzione, la scrivi in riga unica.

In Java, una lambda è un'**interfaccia funzionale**.

## Classe Anonima

La classe anonima è una classe a tutti gli effetti ma non ha una firma. All'atto pratico, è una classe Singleton.

La lambda è fare la stessa cosa di una classe anonima ma con la sintassi delle arrow function.

## Esempio

java

`Condizione<String> c = s -> s.length() > 3;`

È una condizione creata usando una classe anonima.

## Stream in Java

Stream è una collezione amorfa in cui tutte le altre collezioni possono essere trasformate e viene usata per operazioni di lavoro sui dati.

## Filter

java

`List<String> array = new ArrayList<>(); array.stream().filter(s -> s.length() > 3)`

## Operazioni comuni

- **Optional**: Indica che c'è o non c'è un oggetto nelle stream. Usa `.orElse(null)` per gestire l'assenza
- **Map**: Trasformazione di qualcosa (come in JavaScript)
- **MapToInt**: Trasforma in int un qualcosa che contenga oggetto

## Singleton Pattern

Nel contesto di framework come **Spring**, quando un bean ha scope singleton significa che può esistere **una ed una sola istanza** di quel bean all'interno del contesto di Spring (IoC container).

## Terminologia Spring

| Termine                | Descrizione                                                                                                                 |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Dependency Injection   | Funzionalità di Spring che consiste nel fornire a tutti i componenti gli oggetti (bean) di cui hanno bisogno per funzionare |
| Bean                   | Oggetto condiviso; i bean di default sono singleton, ma possiamo cambiarlo tramite annotazioni                              |
| Application Context    | Zona della memoria in cui vengono istanziati i bean all'avvio del programma                                                 |
| Application Properties | File di configurazione Spring che contiene info per collegamento database                                                   |
## Interfaccia Funzionale

Un'interfaccia funzionale (come **Predicate**) è un'interfaccia con un solo metodo astratto che può essere implementata con lambda.

​