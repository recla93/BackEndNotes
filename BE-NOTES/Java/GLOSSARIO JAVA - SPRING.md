## Concetti Fondamentali di Java

## Caratteristiche del Linguaggio

**Java**: Linguaggio a tipizzazione statica, orientato a oggetti, compilato.


## Variabili

**Variabile**:
Nome per manipolare proprietà, oggetti, parametri, collezioni; è sempre "a forma di" qualcosa secondo il tipo di contenuto.

**Variabile di oggetto**: 
Un collegamento a un oggetto in memoria (o a uno spazio in memoria).


**Variabile primitiva**:
Una scatola che contiene un valore.

Le variabili sono applicabili dentro le parentesi graffe e si possono riferire al metodo, all'oggetto o alla classe.

## Stato

**Stato di oggetto**: L'insieme di valori delle sue proprietà in un determinato momento.


**Stato del programma**: L'insieme di valori delle sue variabili in un determinato momento.


**Blank**: Valore vuoto, ad esempio quando in Main diamo invio invece di inserire un valore, oppure in concatenazione delle stringhe diamo il valore iniziale `String res = "";`.

**Null**
valore di una variabile non collegata a nessun oggetto

​

## Tipi di Dati

**Tipi primitivi**:
`int`, `double`, `boolean`, `char`, `short`, `float`, `long` e `byte`. 
Le variabili booleane sono usate come condizioni, assumono valore true o false e si usano con i costrutti `if`, `while`, `switch`.

**Tipi standard/generici**: String.

**Tipi entità**: 
Creati da noi, come Dog, Dish, Meal, Room, House.

## Strutture di Controllo

## Sequenza

**Sequenza**: Esecuzione di un'istruzione dietro l'altra; si applica nel main, non nelle classi tipo/modello.

## Selezione

**Selezione**: Esecuzione condizionale di un programma (si "sceglie" quale istruzione eseguire). Si implementa con `if-else`, `switch`, operatore ternario.

## Iterazione

**Iterazione/Ciclo**: Ripetizione di un'istruzione in base a una condizione. Molto spesso viene usata per lavorare su tutti gli elementi di una collezione uno alla volta; in questo caso la variabile contatore (i) a ogni giro assume il valore che corrisponde all'indice dell'elemento su cui si lavora.

**while**:
Può ripetersi da 0 a n volte.

**do-while**:
Può ripetersi da 1 a n volte.

**for**: 
Scorrimento (un sottotipo di iterazione) con sintassi: variabile di scorrimento, condizione di ripetizione, aggiornamento. Non necessariamente si usa per scorrere le collezioni.

**for-each**: 
`for(User u:users)` significa per ogni elemento 'u' di tipo User nella lista 'users'.

## Classi

## Tipi di Classi

**Main**: Sottoprogramma di avvio.

**Libreria**:
Cassetta di attrezzi adatti per un determinato progetto (es. Console che si occupa di fare input/output verso l'utente).

**Classe tipo**: 
Stampo per creare oggetto, composta dalle proprietà (caratteristiche dell'oggetto) e metodi (comportamento/servizi). Una variabile che rappresenta un oggetto diventa "intelligente" e offre servizi (in quanto si possono invocare i metodi della classe tipo).

## Operatore THIS

**this**: 
Indica l'oggetto in cui sono o sul quale viene chiamato il metodo. Ad esempio: `public void setName(String name) { this.name = name; }` imposta il valore della proprietà name di classe al valore del parametro name.

## Modificatori di Visibilità

**default (non specificato)**: Visibile dentro lo stesso package.

**private**: Visibile solo dentro la classe stessa.
​
**public**: Visibile ovunque.

**protected**: Package + tutti i sottotipi; a prescindere dalla collocazione è sempre visibile ai sottotipi.

**final**: 
Si usa per indicare una variabile, di solito statica, con il valore costante (es. i valori uguali per tutti i rappresentanti della classe tipo).

**static**:
Le variabili possono essere static; sono proprietà di classe, non di oggetto.

## Metodi

## Definizione

**Metodo**: Sottoprogramma composto da diverse istruzioni che possono essere chiamate da altri programmi.

**Firma del metodo**: [visibilità] + [static] + tipo di ritorno/void + nome metodo + ([parametro/i]) + {calcolo/corpo del metodo}.

**Chiamante**: Chi invoca il metodo. I metodi sono contenuti nelle classi e vengono invocati su classi oppure su oggetti (istanze di classi tipo).

**Chiamato**: Metodo invocato.

## Comandi Speciali

**return**: Termina il metodo e restituisce il valore calcolato dal metodo; "uccide" il metodo.

**break**: Termina uno statement (`if`, `switch`, `for`), ad esempio per uscire dal ciclo.

**new**: Operatore di creazione usato per il costruttore; alloca uno spazio in memoria per l'oggetto creato.

## Caratteristiche dei Metodi

**Overload di un metodo**: 
Due metodi con lo stesso nome nella stessa classe che si distinguono secondo i parametri che prendono. Ad esempio: `public void setDob(LocalDate date)` vs `public void setDob(String date)`.

**Parametro**: 
Valore che si passa al metodo dall'esterno; il chiamante passa il parametro al chiamato. I parametri specificano le condizioni di esecuzione di un metodo.

**void**:
Metodo che non restituisce ritorno.

**static (metodo)**: 
Metodo della classe (non si riferisce agli oggetti singoli).

## Repository e CRUD

**Repository**: 
Archivio, oggetto che offre i servizi di CRUD (4 operazioni fondamentali su DB) su entità, gestisce la permanenza.

Lavora su un oggetto alla volta; a ogni entità corrisponde la propria repository.

**Operazioni CRUD**:

- **Create**: Crea oggetto e lo salva su Database, no ID ripetuti, no oggetti ripetuti, no non validi
- **Read**: es. `findAll()`, `findById()`, `findByKey(String key)`
- **Delete**: Rimozione dati
- **Update**: Modifica dati

## Eccezioni

**Eccezione**:
Errore che si verifica durante esecuzione; l'eccezione uccide il metodo.

**Checked**: 
Eccezione a gestione obbligatoria (non la possiamo ignorare), es. `FileNotFoundException`, `SQLException`.

**Unchecked**:
Eccezione a gestione opzionale, es. `NumberFormatException`, `ArrayOutOfBoundsException`.

**Generare eccezione**: 
`throw new NomeEccezione`.

**Propagare eccezione**: 
`throws` (scritto dopo la firma del metodo).

**Gestire eccezione**: 
Costrutto 
`try{codice caldo} catch{eccezione e codice da eseguire nel caso si verifichi} finally {}`.

## Oggetti

**Oggetto**: Variabile composta che sa memorizzare valori ed è in grado di fare i calcoli su se stesso.

Gli oggetti si creano con il costruttore.

## Costruttore

**Costruttore**: Metodo speciale usato per istanziare (creare) oggetti della classe di cui porta il nome.

**Costruttore implicito**: (vuoto) È di default.

**Costruttore esplicito**: Se lo scrivi quello vuoto è disabilitato; prende i parametri che usa come valori da impostare alle proprietà.

## Tipi di Oggetti

**Oggetti generici**: es. String, Array (Vettore), Date, Time. Metodi principali di oggetti String: `toLowerCase()`, `toUpperCase()`, `contains(String s)`, `equals(String s)`, `trim()`.

**Entities**: Riguardano il dominio (ambito di lavoro), es. un piatto in un ristorante.
## Strutture Dati - Collezioni

**Collezione**: È un insieme di valori dello stesso tipo; ogni collezione è un'istanza della sua classe tipo.
## Array (Vettori)

**Array**: Insieme ordinato e finito di elementi dello stesso tipo di dimensione fissa. Ha la forma di uno "scaffale" con dentro elementi di qualsiasi tipo.

Caratteristiche:

- Dimensione fissa
- Ordinato
- Duplicati ammessi
- Tipi primitivi ammessi

Sintassi: `int vet = new int[dim];`
Accesso elemento: `vet = 10;` assegna valore 10 al primo elemento.
## ArrayList

**ArrayList**: Insieme ordinato di elementi dello stesso tipo con dimensione variabile che ammette duplicati.

Caratteristiche:

- Dimensione variabile
- Ordinato
- Duplicati ammessi
- Tipi primitivi NON ammessi perché fanno uso di generics

Metodi: `add(elemento/posizione)`, `remove(elemento/posizione)`, `size()`, `set(int pos, elemento e)`, `contains(Elemento e)`, `get(posizioneDellElemento)`.

Sintassi: `List<TipoElementi> nomeLista = new ArrayList<TipoElementi>();`

## Set

**Set**: Insieme non ordinato di elementi dello stesso tipo che non ammette duplicati.

Caratteristiche:

- Dimensione variabile
- NON ordinato
- Duplicati NON ammessi
- Tipi primitivi NON ammessi


Metodi: Gli stessi delle liste, IN PIÙ `addAll()` (unione di insiemi), `retainAll()` (intersezione tra insiemi), `removeAll()` (sottrarre un insieme dall'altro).

Sintassi: `Set<String> set = new HashSet<String>();`


## Map (Mappe)

**Map**: Insieme non ordinato di coppie chiave-valore; ogni coppia è un entry.

Metodi: `put(chiave, valore)`, `containsKey(chiave)`, `keySet()` (restituisce set contenente tutte le chiavi), `values()` (restituisce set contenente tutti i valori), `entrySet()` (restituisce il set di Entry), `get(chiave)`.

La chiave non può essere duplicata.

Sintassi: `Map<TipoChiave,TipoValore> chiaveToValore = new HashMap<TipoChiave,TipoValore>();`

## Ereditarietà e Polimorfismo

## Ereditarietà

**Ereditarietà**: Una classe estende un'altra classe, ha le stesse proprietà e metodi, a meno che la visibilità non sia private.

Sintassi:
`PC extends Product` (diciamo che PC is_A Product).

Posso estendere solo una classe ma implementare tante interfacce.

**instanceof**: 
Operatore che permette di controllare il tipo concreto.

**Cast**: 
Cambiare il punto di vista, es. `PC p = (PC) product;` abbiamo castato product a pc (sua sottoclasse).

## Classi e Metodi Astratti

**Metodo astratto**: Metodo mancante che deve essere implementato da sottotipi; viene definito in superclasse che deve diventare astratta.

**Classe astratta**: Non può essere istanziata, es. `public abstract class BaseEntity`.

**@Override**: Sovrascrivere = rimpiazzare; il metodo di super viene disabilitato, ma posso richiamarlo con `super.nomeMetodo();`.

I metodi astratti devono essere sovrascritti dalle sottoclassi concrete.

## Interfacce

**Interfacce**: Sono dei contratti che possono essere implementati da altre classi. Sintassi: `public class FileTranslator implements Translator`.

Tutti i metodi sono astratti e hanno la visibilità public predefinita; se uso operatore `default` definisco un metodo concreto che sarà ereditato.

**Interfaccia funzionale**: Un'interfaccia che ha SOLO UNO metodo astratto.

## Tipi

**Tipo concreto**: Tipo dell'oggetto a cui punta; definisce come saranno eseguiti i metodi che chiamiamo.

**Tipo formale**: Tipo della variabile; definisce metodi che posso chiamare su questa variabile.

Spesso: tipo formale (interfaccia) = tipo concreto (classe che la implementa).

Esempio: `List<String> nomelista = new ArrayList<String>()`

## Documentazione e Modellazione

**JavaDoc**: Commenti intelligenti; commentando classe o metodo in questo modo il programmatore lo vedrà da qualsiasi altra classe andando con il mouse sopra.

**Class Diagram**: Illustra i rapporti tra le classi che compongono il nostro programma.

**UML**: Unified Modelling Language.

---

# Spring Framework

## Concetti Fondamentali

## Dependency Injection e Bean

**Dependency Injection**: Funzionalità di Spring che consiste nel fornire a tutti i componenti gli oggetti (bean) di cui essi hanno bisogno per funzionare.


**Bean**: Oggetto condiviso; i bean di default sono singleton, ma possiamo cambiarlo tramite annotazioni. Singleton significa che c'è una sola istanza in Spring context e si usa sempre lo stesso oggetto in tutto il progetto.

**Application Context**: Zona della memoria in cui vengono istanziati i bean all'avvio del programma.

**Application Properties**: File di configurazione Spring che contiene info per collegamento database.

## Annotazioni Spring Core

**@SpringBootApplication**: Esegue component scan: scannerizza il progetto, controllando quali classi sono beanizzabili, le istanzia e mette in application context.

**@Component**: Si mette su intera classe per renderla beanizzabile, i.e. poter usare le sue istanze con @Autowired.

**@Autowired**: Fornisce un bean, i.e. fa dependency injection, mettendo a disposizione istanza dell'oggetto.

**@Service**: Come component ma per servizio; un servizio è un oggetto di cui ci interessano solo i metodi e non lo stato.

**@Bean**: Marca un factory method che istanzia un Spring bean; si mette sul metodo e il return di quel metodo viene beanizzato.

## Annotazioni Lombok

Le annotazioni Lombok permettono di automatizzare la scrittura di boilerplate code (codice necessario per il funzionamento ma senza logica).

**@Data**: Genera metodi getter, setter, toString.

**@Builder**: Genera metodo Builder.

**@SuperBuilder**: Genera builder che funziona anche con ereditarietà.

**@AllArgsConstructor**: Genera costruttore a parametri.

**@NoArgsConstructor**: Genera il costruttore vuoto. N.B. NON funziona se la classe non ha proprietà dichiarate all'interno, ma solo quelle ereditate.

**@Getter**: Genera solo getter; messo sopra la classe vengono generati getter per tutte le proprietà, messo sopra una proprietà ne genera solo per quella proprietà.

**@Setter**: Come sopra ma per setter.

**@ToString**: Genera `toString()`.

**@ToString.exclude**: Omette il valore di proprietà annotata nel `toString()`.


## Pattern Builder

**Builder**: Istanzia un oggetto e assegna valori alle proprietà; è un'alternativa al costruttore a parametri ma permette di imposterne solo alcune e in qualsiasi ordine.

Esempio:

java

`Person newP = Person.builder()     .id(valoredaImpostare)     .name(valore)     .surname(valore)     .dob(valore)     .build();`

## Annotazioni JPA

Le annotazioni JPA permettono di automatizzare ORM (Object Relational Mapping) → traduzione di tabelle SQL in classi Java e trasformazione delle righe del db in oggetti Java.

**@Entity**: Indica che una classe è un'entità; gli oggetti di questa classe saranno salvati in una tabella corrispondente sul database.

**@Table**: Se il nome dell'entità non è uguale al nome della tabella corrispondente su db, si usa questa annotazione per indicare il nome della tabella.

**@Column**: Se il nome di una proprietà non è uguale al nome della colonna corrispondente su db, si usa questa annotazione per indicare il nome della colonna.

**@Id**: Indica che la proprietà viene tradotta in chiave primaria sul db.

**@GeneratedValue(strategy=GenerationType.IDENTITY)**: Indica che il valore della proprietà viene generato automaticamente dal db; il valore non può essere ripetuto all'interno della stessa tabella.

**@JSONIgnore**: La proprietà annotata sarà esclusa dal JSON generato da Rest controller.

**@Transient**: Proprietà annotata non viene mappata a nessuna colonna sul db (con pattern DTO non è necessaria).

**@Repository**: Marca la classe che fornisce funzioni di CRUD su un'entità. Viene implementata implicitamente in classi che implementano un'interfaccia che estende `JpaRepository<Id, Entity>`.

## Annotazioni Web - Controller

## Controller vs RestController

**@Controller**: Si usa con controller che restituisce pagina ftlh, jsp o html (quello che c'è in resources).

**@RestController**: Controller che usa stile REST; restituisce dati grezzi (formato JSON).

## Annotazioni Comuni ai Due Tipi di Controller

**@ModelAttribute**: Si usa per trasformare l'insieme di parametri (dall'URL oppure dal payload) della request in oggetti Java.


**@RequestParam**: Si usa per estrapolare un parametro dalla request, sia query string (`?id=10&nome=mario`) che form-data (parametri passati tramite la form). La chiave del parametro di default corrisponde al nome della variabile; nel caso non lo sia si usa attributo `name=""` di @RequestParam; si può usare attributo `required` che prende il valore boolean.

## Annotazioni per RestController

**@RequestBody**: Si usa per trasformare il body della request in oggetto Java.

**@PathVariable**: Si usa per estrapolare un valore dall'URL in base alla sua posizione.

**@RequestHeader**: Si usa per estrapolare un dato dal header della request (in base a una chiave).

## Mapping HTTP

**@CrossOrigin**: Permette request da qualsiasi URL.

**@GetMapping**: Metodo del controller si esegue in seguito alle request http con il verbo GET; endpoint viene specificato tra le parentesi come String.

**@PostMapping**: Metodo del controller si esegue in seguito alle request http con il verbo POST; endpoint viene specificato tra le parentesi come String; si usa per le operazioni di INSERT.

**@DeleteMapping**: Metodo del controller si esegue in seguito alle request http con il verbo DELETE; endpoint viene specificato tra le parentesi come String.

**@PutMapping**: Metodo del controller si esegue in seguito alle request http con il verbo PUT; endpoint viene specificato tra le parentesi come String; si usa per le operazioni di UPDATE.

## Gestione Eccezioni

**@ExceptionHandler**: Gestisce (come catch) una determinata eccezione (si specifica tra le parentesi) in qualsiasi metodo della classe dove viene scritto il metodo con annotazione exception handler.

**@RestControllerAdvice**: I metodi annotati con @ExceptionHandler all'interno di una classe @RestControllerAdvice catturano eccezioni da TUTTI i Rest controller.

## Relazioni tra Entità

## Relazione 1 a N

**@ManyToOne**: Indica lato N (figlio) della relazione; la tabella corrispondente sul db contiene foreign key.

**@JoinColumn(name="")**: Indica nella tabella lato N il nome della colonna che contiene foreign key.

**@OneToMany(mappedBy="")**: Indica lato 1; mappedBy riporta il nome della proprietà che esprime la relazione nella classe figlio su Java.

**FetchType.EAGER/LAZY**: Determina come si deve comportare Spring Data JPA quando viene letto un oggetto rispetto ai suoi oggetti collegati.

## Relazione N a N

**Opzione 1** (con junction table tradotta in Java): Lato junction table contiene 2 riferimenti ai padri (2 foreign key su db) usando @ManyToOne; lato N (tabella padre verso junction table) usa @OneToMany con mappedBy.

**Opzione 2** (solo per junction table DUMB non tradotta in Java): Usa @ManyToMany con @JoinTable che specifica il nome della tabella associativa, `joinColumns` (chiave esterna della tabella associativa verso di me) e `inverseJoinColumns` (F.K verso l'altro).

## Altri Concetti

**JSON (JavaScript Object Notation)**: Sintassi per rappresentare oggetti anonimi JavaScript (non appartengono a una classe).