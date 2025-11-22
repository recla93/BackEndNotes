```typescript

import { Injectable } from '@angular/core';  
import {HttpClient} from '@angular/common/http';  
import {inject} from '@angular/core';  
import {Card} from '../models/interfaces/card.interface';  
import {BehaviorSubject, combineLatest, Observable, switchMap} from 'rxjs';  
import {map} from 'rxjs/operators';  
  
@Injectable({  
  providedIn: 'root',  
})  
export class CardsService {  
  http = inject(HttpClient)  
  pageSize$ = new BehaviorSubject<number>(20);  
  pageNumber$ = new BehaviorSubject<number>(1);  
  
  /*  
  Il $ è un indicatore visivo che ti dice immediatamente: "Questa variabile contiene un Observable  (o Subject/BehaviorSubject)". È come mettere un'etichetta sulla variabile per renderla riconoscibile   a colpo d'occhio.  
  Observable: Gli observable contengono funzioni e dati che verranno eseguite LAZY (solo su chiamata) in maniera  asincrona mediante il subscribe, metodo che si ISCRIVE all'observable e lo fa partire.  
  Behavior Subject: Possono essere considerate delle variabili potenziate che permettono di modificare il dato  e la sua visualizzazione / utilizzo in maniera Asincrona.  
  exhaust map: da priorità al primo, annullando tutti i valori che gli arrivano se non ha ancora  terminato il primo processo  concatmap: Integra un sistema di queue, una volta terminato il   primo processo passa al secondo, etc(u) ((salute))  
  switch map: Da priorità all'ultimo, annullando tutti i valori arrivati prima, anche se non completati  (esempio paginator(i'll be back), se spammo il cambio pagina e vado alla successiva prenderà i valori ULTIMI,  quindi quelli che ci interessano)  
   Combine Latest: combina gli ultimi valori di uno o più elementi di RxJS per dare in output    un observable che si aggiornerà in automatico alla modifica degli elementi di partenza  */  
   
  getCards(): Observable<Card[]>{  
    return combineLatest([this.pageSize$, this.pageNumber$])  
      .pipe(switchMap(  
        ([pageSize, pageNumber]) =>  
        {  
        return this.http.get<Card[]>  
        ('https://api.tcgdex.net/v2/en/cards?pagination:itemsPerPage='  
          +pageSize+'&pagination:page='+pageNumber);  
        })  
      /*  
      ALL'ATTO PRATICO:      Lanciamo il metodo getCards, che ritorna un Obs di Cards[], creiamo un Observable      tramite il combine latest (che prende gli ultimi valori dei BS pagesize e pagenumber)      svolgiamo delle operazioni con il pipe      e lanciamo la switchmap per prendere gli ultimi valori disponibili dei BS pageSize e PageNum,      di tipo number, che ritornano un Obs di Cards[], che infine torneranno quindi un Obs tramite http.get      con i valori dell'API che contiene la paginazione, itemsPerPage (del paginator), page size, paginazione e page number.      Per aggiungere un filter, quindi, basterebbe aggiungere un BS nei parametri input del combine, aggiungerlo alla switchmap      ed aggiungerlo all'EndPoint (tutto questo serve per avere una pagina dinamica, se avessimo preso i parametri normali,      avremmo dovuto aggiornare manualmente la pagina.       */  
      
//quando facciamo http.get è SEMPRE un Observable, perchè ci aspettiamo che la richiesta che stiamo scrivendo      
//venga eseguita SOLO quando facciamo il subscribe. Quindi cio' che scriviamo    dopo il this.http.get sarà una
//Questo metodo prende in input tramite il combine latest i 2 behavior           subject e utilizza i valori al suo interno      
//per la paginazione in modo che quando tramite il paginator si cambia pagina    o grandezza, verranno modificati      
//i valori nel BS(behavior sub) che modificheranno Observable e di                conseguenza il risultato dell'endpoint      
//senza dover eseguire nuovamente il subscribe all'observable.    )  
  
  }  
  
  getCardsNumber(): Observable<number>{  
    return this.http.get<Card[]>('https://api.tcgdex.net/v2/en/cards').pipe(  
      map((res:Card[]) => res.length)  
    );  
  }  
}

```

```HTML

<section>  
  <div class="w-3/4 mx-auto grid grid-cols-3 gap-4">  
    @for (card of cards; track card.id) {  
      <mat-card>  
        <mat-card-title>{{ card.name }}</mat-card-title>  
        @if (card.image) {  
          <img mat-card-image src="{{ card.image+'/high.webp' }}">  
        }  
      </mat-card>  
    }  
  </div>  
  <mat-paginator [length]="cardsNumber$ | async"  
                 [pageSize]="cardsService.pageSize$ | async"  
                 [pageSizeOptions]="[20, 50, 100]"  
                 aria-label="Select page"  
                 (page)="  
                  cardsService.pageNumber$.next($event.pageIndex + 1);  
                  cardsService.pageSize$.next($event.pageSize)  
                        "  
  
  >  
                <!--  
                abbiamo 2 tipi di interazioni con i componenti in ang:                @input e @output:                @input ci permettono                di mandare delle informazioni statiche (class) o dinamiche che prende come riferimento qualcosa                di dichiarato nel TS([class]) tramite le parentesi quadre, gli stiamo specificando che possiamo prendere                qualcosa dal TS, DATA BINDING, ([class] = "variabile_con_classi")                (esempio: [length] = "cardsService.pageSize$ | async"  
                @Output invece, sono eventi scatenati dal componente con lo scopo di mandare un segnale e dati al                componente padre così da poter gestire il comportamento nel componente padre                Esempio: il paginatore manda il segnale ( (page) ) al componente home-page nel quale noi                gestiamo le interazioni tra il paginator ed il service.  
  
                -->  </mat-paginator>  
</section>  
  
<!--  
mi servono 2 behavior subject: uno per PageSize, l'altro per Page.  
questi due verranno usati come parametri all'interno dell'URL del  
 GetCards in maniera tale che quando cambio page size,o passo alla page successiva, in automatico farà il fetch dei dati.  
  
  
-->

```
# ANGULAR

API:  
L'API (Application Programming Interface) non è specificamente definita nel contesto fornito. Tuttavia, basandomi sulle informazioni disponibili, posso dedurre che l'API è menzionata come parte del sistema di comunicazione tra il frontend e il backend.

Domenico Farano suggerisce che l'API è utilizzata per l'accesso a Internet. Francesco Pierno menziona che le API gestiscono le richieste e le risposte. Inoltre, viene detto che le repository in Angular comunicano con le API del backend per eseguire operazioni CRUD (Create, Read, Update, Delete).

In sintesi, l'API sembra essere un'interfaccia che permette la comunicazione e lo scambio di dati tra il frontend (lato client) e il backend (lato server) dell'applicazione.

REPOSITORY:  
In Angular, una repository è essenzialmente un servizio specializzato che si occupa della comunicazione con le API del backend. Le repository in Angular svolgono un ruolo simile ai DAO (Data Access Object) nel backend, ma per il frontend.

Ecco i punti chiave sul concetto di repository in Angular:

1. Sono classi che vengono trattate come servizi in Angular.
2. Forniscono funzionalità CRUD (Create, Read, Update, Delete) per il frontend.
3. Comunicano direttamente con le API del backend per eseguire operazioni sui dati.
4. Sono l'equivalente frontend dei DAO utilizzati nel backend.
5. Vengono utilizzate per separare la logica di accesso ai dati dal resto dell'applicazione.
6. Possono essere iniettate nei componenti o in altri servizi dove è necessario l'accesso ai dati.

Le repository in Angular aiutano a mantenere il codice organizzato e a separare le responsabilità all'interno dell'applicazione, rendendo più facile la gestione delle interazioni con il backend.

###  SERVIZI:

In Angular, i servizi sono classi che svolgono un lavoro specifico e vengono istanziati automaticamente. Sono simili ai Bean o all'autowiring in Spring. I servizi vengono iniettati nei componenti, che sono pezzi di pagina web con la propria logica, stile e struttura.

I servizi possono essere iniettati nei componenti in diversi modi:

1. Tramite il costruttore
2. Usando una proprietà con @Injectable()
3. Scrivendo "= inject()" seguito dalla classe del servizio tra parentesi

I servizi sono utilizzati principalmente nei componenti per gestire la logica e l'interazione con il backend. In particolare, le repository in Angular sono un tipo di servizio che comunica con le API del backend per eseguire operazioni CRUD (Create, Read, Update, Delete), simili ai DAO nel backend.

### COMPONENT:

I componenti in Angular sono pezzetti di pagina web con la propria logica, stile e struttura. Più precisamente, un componente è composto da tre file: HTML, CSS e TypeScript. Questi componenti servono a rappresentare e gestire parti specifiche di una pagina, rendendo la struttura del sito web modulare e più facile da gestire. Invece di avere tutta la pagina in un unico blocco, i componenti permettono di dividere e settorializzare i vari elementi, consentendo di lavorare su ciascuno separatamente e di creare una pagina web più flessibile e organizzata.

### INJECT:

In Angular, i servizi possono essere iniettati nei componenti in tre modi principali:

1. Tramite il costruttore: Si può iniettare il servizio direttamente nel costruttore del componente.
2. Usando una proprietà: Si può creare una proprietà nel componente e utilizzare "= inject()" seguito dalla classe del servizio tra parentesi.
3. Utilizzando una funzione simile a un'annotazione: Anche se non è esattamente un'annotazione come in altri framework, Angular ha una funzione che permette di iniettare i servizi in modo simile.

Questi metodi sono equivalenti all'autowiring in Spring, permettendo l'iniezione automatica dei servizi nei componenti dove sono necessari.

### BINDING:

Certo, ti spiego nuovamente i tipi di binding con le informazioni fornite nel contesto:

1. Interpolazione (Mustache Syntax): Usa le doppie parentesi graffe {{ }}. Serve per inserire valori all'interno del testo HTML. Ad esempio, {{nome}} mostrerà il valore della variabile "nome" nel testo.
2. Property Binding: Usa le parentesi quadre . Serve per assegnare valori agli attributi degli elementi HTML. Ad esempio, ="col" assegnerà il valore della variabile "col" all'attributo style.color dell'elemento.
3. Event Binding: Usa le parentesi tonde ( ). Serve per gestire gli eventi. Ad esempio, (click)="metodo()" farà partire il metodo specificato quando si verifica l'evento di click sull'elemento.
4. Two-way Binding: Usa la sintassi "banana in a box" [( )]. Serve per creare un collegamento bidirezionale tra l'HTML e il TypeScript. Ad esempio, [(ngModel)]="nome" legherà il contenuto di un input alla variabile "nome", aggiornando entrambi quando uno dei due cambia.

Questi quattro tipi di binding sono fondamentali per l'interazione tra l'HTML e il TypeScript in Angular.
![[Pasted image 20251122161846.png]]
### DIRETTIVE:

Le direttive in Angular sono strumenti importanti per manipolare il DOM e aggiungere comportamenti agli elementi HTML. Dal contesto, possiamo evidenziare alcuni punti chiave sulle direttive:

1. Esistono due sintassi principali per le direttive:

- La sintassi più vecchia usa un asterisco, come *ngFor
- La nuova sintassi usa la chiocciola (@)

3. Un esempio di direttiva è *ngFor, che viene usata per ripetere elementi HTML basandosi su un array di dati.
4. La sintassi con asterisco (es. *ngFor) si applica direttamente all'elemento HTML e esegue la logica specificata per quell'elemento.
5. La nuova sintassi con chiocciola (es. @for) offre una logica simile ma con una sintassi diversa e richiede l'uso di "track by" per l'ottimizzazione delle prestazioni.
6. Le direttive permettono di eseguire operazioni come cicli e condizioni direttamente nel template HTML, rendendo il codice più dinamico e reattivo.

Queste sono le informazioni principali sulle direttive che possiamo estrarre dal contesto fornito.
![[Pasted image 20251122161906.png]]
Ricapitolando:

Angular è un framework per lo sviluppo di applicazioni web front-end.

Cosa fa:

1. Permette di creare pagine web dinamiche e interattive
2. Facilita la creazione di componenti riutilizzabili
3. Gestisce il binding tra HTML e TypeScript
4. Fornisce direttive per applicare logica di programmazione all'HTML

Come funziona:

1. Utilizza componenti come blocchi di costruzione delle pagine
2. Implementa il binding per collegare HTML e TypeScript in quattro modi:

- Interpolazione ({{ }})
- Property binding ([])
- Event binding (())
- Two-way binding ([()])

4. Usa direttive (come *ngFor, *ngIf) per applicare logica all'HTML
5. Inietta servizi nei componenti per gestire la logica e la comunicazione con il backend
6. Utilizza repository per comunicare con le API del backend

Angular semplifica lo sviluppo di applicazioni web complesse fornendo una struttura organizzata e strumenti potenti per la gestione dell'interfaccia utente e della logica dell'applicazione.

### ROUTING:

Il routing in Angular serve per decidere quale componente grafico mostrare in base alla URL in cui ci troviamo. Viene gestito da un file chiamato app-routing.module.ts, dove si definisce un array di oggetti. Questi oggetti hanno almeno due proprietà fondamentali:

path: che specifica l'URL

component: che indica quale componente deve essere caricato per quel path

Definendo queste rotte nel file app-routing.module.ts, Angular sa quale componente visualizzare quando l'utente naviga a un determinato URL nell'applicazione. Questo permette di creare un'applicazione a singola pagina (SPA) con navigazione tra diverse "pagine" senza ricaricare effettivamente la pagina del browser.

IN MODO DIVERSO:

Il routing in Angular è un meccanismo fondamentale per gestire la navigazione tra diverse viste o componenti dell'applicazione. Ecco un riepilogo delle informazioni fornite sul routing:

1. Il routing viene gestito da un file chiamato app-routing.module.ts.
2. All'interno di questo file, si definisce un array di oggetti che rappresentano le rotte.
3. Ogni oggetto rotta ha almeno due proprietà principali:

- path: specifica l'URL della rotta
- component: indica quale componente deve essere caricato per quella rotta

5. Esempio di definizione di una rotta:  
    { path: 'home', component: HomeComponent }
6. Nel template HTML dell'applicazione, si usa l'elemento <router-outlet> per indicare dove Angular deve inserire il componente corrispondente alla rotta attiva.
7. Quando un utente naviga a un URL specifico (es. http://localhost:4200/home), Angular caricherà il componente associato a quel path all'interno del <router-outlet>.

Questo sistema permette di creare applicazioni a pagina singola (SPA) con navigazione fluida tra diverse viste, senza ricaricare completamente la pagina.

PATH PARAM:

La parametrizzazione dei path significa che è possibile definire parti variabili all'interno dell'URL di una rotta. Questo permette di creare percorsi dinamici che possono accettare valori diversi.

Con questa funzionalità, puoi:

1. Creare URL dinamici che cambiano in base a determinati valori, come ad esempio un ID utente o un nome di prodotto.
2. Passare dati specifici al componente associato a quella rotta. Questi dati possono essere recuperati e utilizzati all'interno del componente stesso.
3. Rendere le tue rotte più flessibili e riutilizzabili, evitando di dover definire una rotta separata per ogni possibile variazione.
4. Recuperare i valori dei parametri nel componente di destinazione, permettendoti di personalizzare il contenuto o il comportamento del componente in base ai valori passati nell'URL.

Per esempio, potresti avere una rotta come '/utente/:id', dove ':id' è un parametro che può essere sostituito con un valore reale, come '/utente/123'. Il componente associato a questa rotta potrebbe quindi utilizzare questo ID per caricare e visualizzare i dettagli specifici dell'utente con quell'ID.



Per far funzionare questa cosa, devi scrivere "withInputBinding" all'interno di app.config dove c'è il router. Specificamente, il contesto menziona che "Per far funzionare questo giochetto, Però, dato che questa intassi è nuova. Vi ricordo che dovete scrivere all'interno. Di up, puntofig. Dove c'è router, dovete scrivere. Wiff Input Banding." Questo è necessario per abilitare la funzionalità di input binding per i parametri di routing.

### CONFIGURAZIONE DI ANGULAR

Ha 2 file di configurazione, 2 json:

Node.js è un runtime environment per JavaScript. È un ambiente di esecuzione che permette di eseguire codice JavaScript al di fuori del browser web (in locale). Node.js consente di utilizzare JavaScript per sviluppare applicazioni lato server, strumenti da riga di comando e altri tipi di software che non necessariamente hanno un'interfaccia grafica. È ampiamente utilizzato per lo sviluppo di applicazioni web e altre applicazioni basate su JavaScript. Usato per eseguire server di sviluppo, non è altro che cio’ che mette Angular alla porta 4200 mettendo a disposizione gli output.

Node.js è un runtime environment che permette di eseguire JavaScript localmente, al di fuori di un browser web. Nel contesto di Angular, Node.js viene utilizzato principalmente per due scopi:

1. Come server di sviluppo: Angular utilizza Node.js per creare un server di sviluppo locale, solitamente sulla porta 4200. Questo server permette agli sviluppatori di vedere le modifiche al loro codice in tempo reale e fornisce informazioni sugli errori durante lo sviluppo.
2. Per la gestione delle dipendenze: Node.js, attraverso il suo gestore di pacchetti npm, viene utilizzato per gestire le dipendenze del progetto Angular. Queste dipendenze sono definite nel file package.json, che è essenzialmente un file di configurazione per il progetto Node.js/Angular.

In sintesi, Node.js è uno strumento fondamentale nell'ecosistema di sviluppo Angular, fornendo un ambiente di esecuzione per JavaScript e facilitando il processo di sviluppo e la gestione delle dipendenze del progetto.

NPM gestore progetti in Node, fa il Maven di note, installa librerie, ecc.  
NPM sta per Node Package Manager. È un software all'interno di Node.js che si occupa di gestire e installare le librerie per progetti basati su JavaScript. Specificamente nel contesto di Angular, NPM viene utilizzato per installare le librerie di base necessarie per il funzionamento del framework e per gestire le dipendenze del progetto. È essenzialmente l'equivalente di Maven per i progetti Node.js e Angular.

Package.json: come il POM Maven.  
Il package.json è un file di configurazione per Angular. Secondo il contesto, esso contiene:

1. La configurazione per Node.js
2. Le librerie che si vogliono utilizzare nel progetto
3. Comandi particolari, come quello del proxy
4. Informazioni necessarie per il server di sviluppo di Angular, che funziona sulla porta 4200 e permette di vedere le modifiche in tempo reale e ricevere notifiche sugli errori durante lo sviluppo del programma Angular.

Il package.json è essenzialmente il file che definisce e gestisce le dipendenze e le impostazioni del progetto Node.js/Angular.

Angular.json:  
Angular.json è un file di configurazione importante per i progetti Angular. Basandomi sulle informazioni fornite nel contesto, ecco alcuni punti chiave su angular.json:

1. È uno dei due file principali di configurazione in un progetto Angular, insieme a package.json.
2. Contiene configurazioni specifiche per Angular, a differenza di package.json che è più generico per progetti Node.js.
3. Nel file angular.json, è possibile specificare quali file CSS e JavaScript devono essere importati globalmente nel progetto Angular.
4. Un esempio menzionato nel contesto è l'inclusione di file Bootstrap. Se si vuole usare Bootstrap in un progetto Angular, si deve specificare i file CSS e JavaScript di Bootstrap in angular.json.
5. Le configurazioni in angular.json sono principalmente utilizzate dal server di sviluppo di Angular (ng serve).
6. Queste configurazioni aiutano a impostare l'ambiente di sviluppo, permettendo funzionalità come l'aggiornamento in tempo reale delle modifiche nel browser.

È importante notare che angular.json gioca un ruolo cruciale nella configurazione dell'ambiente di sviluppo Angular, permettendo agli sviluppatori di personalizzare vari aspetti del loro progetto.

### BUILDING APP ANGULAR:

Certo, ecco una spiegazione schematizzata del processo di build di un'applicazione Angular:

1. Sviluppo dell'applicazione:

- Creazione dei componenti, servizi, e altre parti dell'applicazione Angular
- Utilizzo del server di sviluppo Angular (ng serve) per testare l'applicazione in locale

3. Comando di build:

- Esecuzione del comando "ng build" da terminale  
    DA PACKAGE JSON POSSIAMO FARE IL BUILD PREMENDO PLAY ACCANTO A BUILD:

5. Processo di build:

- Angular compila tutto il codice TypeScript in JavaScript
- Combina tutti i file in bundle ottimizzati
- Genera i file statici necessari per il deployment

7. Output del build:

- Creazione di una cartella "dist" (distribution)
- All'interno di "dist", generazione di file ottimizzati:

- index.html
- JavaScript bundle
- CSS bundle
- Altri asset necessari

9. Risultato finale:

- Un set di file statici (HTML, CSS, JavaScript) pronti per essere deployati su un server web
- L'applicazione può ora essere eseguita senza il server di sviluppo Angular

Questo processo trasforma l'applicazione Angular da un progetto di sviluppo in una versione ottimizzata e pronta per la produzione, che può essere ospitata su qualsiasi server web standard.

E possiamo pure usare tutti i file angular in backend (spring)
-------

Per portare le cose di Angular in Spring, dovresti seguire questi passaggi:

1. Esegui il build del tuo progetto Angular usando il comando "ng build". Questo creerà una cartella "dist" con i file compilati.
2. Prendi i file generati nella cartella "dist" del tuo progetto Angular.
3. Inserisci l'index.html nella cartella "resources/templates" del tuo progetto Spring.
4. Metti tutti gli altri file (JavaScript, CSS, ecc.) nella cartella "resources/static" del progetto Spring.
5. Configura Spring per servire questi file statici e l'index.html come pagina principale.

In questo modo, il tuo frontend Angular sarà integrato nel backend Spring e funzionerà come una singola applicazione. Spring servirà i file statici generati da Angular quando accedi all'applicazione.

### DUCK TYPING

Il duck typing è un concetto in programmazione che si riferisce al fatto che il tipo di un oggetto è determinato dalle sue caratteristiche e comportamenti, piuttosto che dalla sua classe o tipo dichiarato esplicitamente. Come spiegato nel contesto, la definizione scherzosa ma ufficiale nella documentazione di TypeScript era "Se sembra una papera e fa il verso di una papera, allora è una papera". In pratica, questo significa che in linguaggi come TypeScript, un oggetto può essere considerato di un certo tipo se ha le proprietà e i metodi attesi per quel tipo, indipendentemente da come è stato dichiarato o creato.

Certamente, posso spiegarti in dettaglio il concetto di prototipo in JavaScript e TypeScript, con alcuni esempi.

In JavaScript e TypeScript, i tipi concreti funzionano attraverso un meccanismo chiamato "prototipo". Invece di avere una vera e propria idea di tipo, questi linguaggi utilizzano una zona di memoria dove sono contenute le informazioni sul tipo dell'oggetto.

Il prototipo è essenzialmente un oggetto che contiene proprietà e metodi che possono essere condivisi da più istanze. Quando si crea un oggetto, questo eredita automaticamente le proprietà e i metodi del suo prototipo.

Ecco un esempio di come funziona il prototipo in JavaScript:

function Persona(nome, cognome) {

  this.nome = nome;

  this.cognome = cognome;

}

Persona.prototype.saluta = function() {

  console.log("Ciao, sono " + this.nome + " " + this.cognome);

};

let persona1 = new Persona("Mario", "Rossi");

persona1.saluta(); // Output: Ciao, sono Mario Rossi

In questo esempio, saluta è un metodo definito nel prototipo di Persona, e tutte le istanze di Persona possono accedervi.

Per quanto riguarda le classi anonime, in JavaScript e TypeScript è possibile creare oggetti senza definire una classe esplicita. Ecco un esempio:

let personaAnonima = {

  nome: "Luigi",

  cognome: "Verdi",

  saluta: function() {

    console.log("Ciao, sono " + this.nome + " " + this.cognome);

  }

};

personaAnonima.saluta(); // Output: Ciao, sono Luigi Verdi

Questo oggetto personaAnonima è essenzialmente una classe anonima con proprietà e metodi definiti direttamente.

Un oggetto è tale perché è stato creato tramite il costruttore della sua classe. Come spiegato nel contesto, un oggetto persona è di tipo persona perché è stato creato usando il costruttore della classe Persona. Allo stesso modo, un oggetto animale è di tipo animale perché è stato creato usando il costruttore della classe Animale. Questo principio si applica a tutti i tipi, inclusi i tipi primitivi.

Il tipo concreto è il tipo effettivo di un oggetto in memoria, definito al momento della sua creazione tramite il costruttore. Viene spiegato che in Java, quando si crea un oggetto usando un costruttore specifico, quell'oggetto avrà quel tipo concreto per tutta la sua esistenza. Ad esempio, se si crea un oggetto usando il costruttore di "Persona", quell'oggetto sarà di tipo concreto "Persona" per sempre.

OGGETTO PROTOTIPO  
  
L'oggetto prototipo è un concetto fondamentale in JavaScript e TypeScript. Ecco come funziona nel dettaglio:

1. Ogni oggetto in JavaScript ha un prototipo associato.
2. Il prototipo è essenzialmente un altro oggetto che contiene proprietà e metodi.
3. Quando si crea un oggetto da una classe, il prototipo di quell'oggetto viene impostato al prototipo della classe.
4. Il prototipo contiene il costruttore della classe e tutti i metodi definiti nella classe.
5. Quando si accede a una proprietà o un metodo di un oggetto, JavaScript cerca prima nell'oggetto stesso. Se non lo trova, cerca nel suo prototipo, poi nel prototipo del prototipo, e così via lungo la catena dei prototipi.
6. Questo meccanismo permette l'ereditarietà in JavaScript.
7. È possibile modificare il prototipo di un oggetto esistente, aggiungendo nuovi metodi o proprietà che saranno disponibili per tutti gli oggetti creati da quella classe.
8. Il prototipo finale nella catena è solitamente Object.prototype, che contiene metodi comuni a tutti gli oggetti JavaScript.

Questo sistema di prototipi permette a JavaScript di implementare concetti di programmazione orientata agli oggetti come l'ereditarietà senza utilizzare classi nel senso tradizionale.

http client manda request: e viene prodotto observable buoto  
2 la request arriva al backend. backend produce array persone, tale jason viene mappato e trasformato. l'array viene messo in observable, il componente ha definito callback con subscribe, non appena arriva response, parte la callback

### COMUNICAZIONE ANGULAR E BACKEND

{

L'insegnante sta spiegando i seguenti processi relativi alla comunicazione tra Angular e il backend:

1. L'HttpClient crea un Observable vuoto.
2. L'utilizzatore del metodo passa una callback all'Observable, specificando cosa fare quando arriveranno i dati.
3. La risposta arriva dal backend.
4. Il contenuto della risposta viene inserito nell'Observable.
5. La callback viene eseguita, ricevendo come parametro il contenuto dell'Observable.
6. La callback esegue l'azione specificata, in questo caso riempire una proprietà all'interno del componente con i dati ricevuti.

Questi passaggi descrivono come Angular gestisce le richieste HTTP asincrone utilizzando gli Observable.

}

La comunicazione tra Angular e il backend avviene attraverso i seguenti passaggi dettagliati:

1. L'HttpClient di Angular invia una richiesta HTTP al backend.
2. Il metodo della repository (ad esempio getAll()) restituisce immediatamente un Observable vuoto.
3. La richiesta HTTP viaggia verso il server backend.
4. Il backend elabora la richiesta e produce una risposta JSON (ad esempio un array di oggetti persona).
5. La risposta JSON viene automaticamente deserializzata e "duck-typed" in base al tipo specificato (es. array di Person).
6. Il contenuto deserializzato viene inserito nell'Observable precedentemente vuoto.
7. Nel frattempo, il componente Angular ha impostato una callback sull'Observable usando il metodo subscribe().
8. Quando l'Observable viene riempito con i dati, la callback viene eseguita.
9. La callback riceve come parametro il contenuto dell'Observable (l'array di Person).
10. Tipicamente, la callback aggiorna una proprietà del componente con i dati ricevuti, permettendo ad Angular di aggiornare la vista.

Questo meccanismo permette una gestione asincrona efficiente della comunicazione HTTP, consentendo all'applicazione di rimanere reattiva mentre attende la risposta dal server.

  
INTERFACE E MODELLI  
In Angular, le interfacce e i modelli sono utilizzati per definire la struttura dei dati.

Le interfacce in Angular sono utilizzate per definire i tipi di dati. Esse specificano quali proprietà e metodi un oggetto dovrebbe avere. Grazie al concetto di "duck typing" in TypeScript, un oggetto può essere considerato di un certo tipo se ha le proprietà e i metodi definiti nell'interfaccia, anche se non implementa esplicitamente quell'interfaccia.

I modelli, invece, sono utilizzati per rappresentare la struttura dei dati all'interno dell'applicazione. Sono spesso definiti come classi o interfacce e vengono utilizzati per garantire che i dati abbiano una struttura coerente in tutta l'applicazione.

Nel contesto di Angular, i modelli sono particolarmente utili quando si lavora con le API. Quando si effettua una richiesta HTTP, i dati ricevuti possono essere "dactapizzati" (tipizzati) secondo il modello definito, garantendo così che i dati siano nella forma attesa e facilitando l'uso di questi dati all'interno dell'applicazione.  
Le interfacce non sono discusse in dettaglio nel contesto fornito. Tuttavia, ci sono alcune menzioni che possono essere rilevanti:

1. Le interfacce sono utilizzate in TypeScript per definire la struttura degli oggetti.
2. Nel contesto del duck typing, si menziona che un'interfaccia può definire proprietà e metodi che un oggetto dovrebbe avere.
3. Le interfacce sono utilizzate per garantire che gli oggetti abbiano determinate proprietà, specialmente quando si lavora con richieste GET o POST al backend.
4. Un esempio menzionato è un'interfaccia "persona" che potrebbe definire proprietà come nome e cognome.

Non ci sono informazioni più dettagliate sulle interfacce nel contesto fornito. Per una spiegazione più approfondita, sarebbe necessario consultare altre fonti.

###   
PROTOTYPE

I prototipi in JavaScript e TypeScript sono un concetto fondamentale per comprendere come funzionano gli oggetti e l'ereditarietà. Ecco i dettagli principali:

1. Ogni oggetto in JavaScript ha una proprietà interna chiamata [ (spesso indicata come **proto**).
2. Quando si crea una classe, viene definito un prototipo che contiene tutti i metodi e le proprietà di quella classe.
3. Quando si crea un'istanza di una classe, l'oggetto risultante è essenzialmente un oggetto anonimo a cui viene allegato il prototipo della classe.
4. Il prototipo contiene il costruttore della classe e tutti i metodi definiti nella classe.
5. Quando si cerca di accedere a una proprietà o un metodo di un oggetto, JavaScript prima cerca nell'oggetto stesso, poi nel suo prototipo, poi nel prototipo del prototipo, e così via fino a raggiungere Object.prototype.
6. Questo meccanismo permette l'ereditarietà e il condivisione di metodi tra istanze della stessa classe.
7. È possibile modificare il prototipo di una classe anche dopo la sua definizione, aggiungendo nuovi metodi o proprietà che saranno disponibili per tutte le istanze esistenti e future di quella classe.
8. In TypeScript, il sistema di tipi si basa su questo meccanismo di prototipi, ma aggiunge controlli statici dei tipi per garantire una maggiore sicurezza e prevedibilità del codice.

Questo sistema di prototipi è alla base del concetto di "duck typing" in JavaScript e TypeScript, dove un oggetto può essere trattato come di un certo tipo se ha le proprietà e i metodi richiesti, indipendentemente dalla sua effettiva definizione di classe.

È importante notare che questa strutturazione dei dati aiuta a prevenire errori e rende il codice più manutenibile e comprensibile.

### CALLBACK

Le callback sono funzioni che vengono passate come parametro ad un'altra funzione e vengono richiamate all'interno di essa in un momento successivo. Nel contesto di Angular e delle chiamate HTTP, le callback vengono utilizzate con gli Observable.

Specificamente:

1. Una callback è una funzione che viene passata come parametro al metodo subscribe() di un Observable.
2. Questa funzione non viene eseguita immediatamente, ma viene memorizzata per essere eseguita in futuro.
3. La callback viene eseguita solo quando l'Observable viene "riempito" con i dati della risposta HTTP.
4. Quando la risposta arriva dal backend, il contenuto viene inserito nell'Observable.
5. A questo punto, la callback viene richiamata automaticamente, ricevendo come parametro i dati contenuti nell'Observable.
6. Questo meccanismo permette di gestire in modo asincrono le risposte delle chiamate HTTP, eseguendo del codice solo quando i dati sono effettivamente disponibili.

In sintesi, le callback in questo contesto sono un modo per definire cosa fare con i dati una volta che questi sono stati ricevuti dal server, senza bloccare l'esecuzione del resto del codice.

SERVIZI E REPO  
Nel contesto di Angular, i servizi e le repository sono componenti fondamentali per la gestione dei dati e la logica di business dell'applicazione:

Servizi:

- Sono classi che svolgono un lavoro specifico
- Vengono istanziati automaticamente e iniettati dove necessario
- Funzionano in modo simile ai Bean e all'autowiring in Spring
- Possono essere iniettati nei componenti

Repository:

- Sono un tipo specifico di servizio in Angular
- Comunicano con le API del backend
- Forniscono funzionalità CRUD (Create, Read, Update, Delete)
- Sono l'equivalente dei DAO (Data Access Object) nel frontend
- Gestiscono la comunicazione tra frontend e backend

Le repository in particolare si occupano di effettuare le chiamate HTTP al backend e gestire i dati ricevuti, spesso utilizzando Observable per gestire le risposte asincrone. Vengono tipicamente utilizzate all'interno dei componenti per recuperare o inviare dati al server.

### FLUSSO DI DATI  
  

Ecco una descrizione dettagliata del flusso di dati in Angular, in ordine e con procedimenti precisi:

1. L'HttpClient invia una richiesta HTTP al backend.
2. L'HttpClient crea immediatamente un Observable vuoto.
3. Il metodo della repository (ad esempio getPersone()) restituisce questo Observable vuoto.
4. Il componente che ha chiamato il metodo della repository riceve l'Observable vuoto.
5. Il componente si sottoscrive all'Observable usando il metodo subscribe(), passando una funzione di callback.
6. La richiesta HTTP raggiunge il backend.
7. Il backend elabora la richiesta e produce una risposta, tipicamente in formato JSON.
8. La risposta JSON viene inviata dal backend al frontend.
9. L'HttpClient riceve la risposta JSON.
10. L'HttpClient deserializza automaticamente il JSON in un array di oggetti (nel nostro esempio, un array di persone).
11. L'HttpClient inserisce questo array di oggetti nell'Observable precedentemente vuoto.
12. L'Observable, ora pieno, attiva la funzione di callback fornita nel metodo subscribe().
13. La funzione di callback viene eseguita, ricevendo come parametro il contenuto dell'Observable (l'array di persone).
14. La funzione di callback tipicamente aggiorna una proprietà del componente con i dati ricevuti.
15. Angular rileva il cambiamento dei dati e aggiorna automaticamente la vista del componente.

Questo processo permette una gestione asincrona efficiente dei dati, consentendo all'applicazione di rimanere reattiva durante le operazioni di rete.!


<!--

      BINDING = COLLEGARE ELEMENTI NELL'HTML a TYPESCRIPT

      - Interpolazione, BAFFO (mustache) -> {{valore}}, mette come testo nell'html

      - Event Binding, con le tonde -> (click)="metodo()", uguale all'onclick di js

      - Property Binding, Box Binding -> [attributoHtml]="valoreTs", si usa per definire

          attributi html in maniera dinamica, prendendoli da Ts

      - Two-Way Binding, Banana in a box -> [(ngModel)]="variabileTs", collega in maniera bidirezionale

        il valore di un input a un valore in typescript

 -->

<h1 [style.color]="titolo.color">{{titolo.value}}</h1>  
<button (click)="cambiaTitolo()">CAMBIA TITOLO</button>  
<input [(ngModel)]="valoreCasella" placeholder="inserisci valore">

<h2>Hai inserito nella casella il valore {{valoreCasella.toUpperCase()}}</h2>  
<button (click)="cambiaValoreCasella()">CAMBIA CASELLA</button>

<input [(ngModel)]="persona.nome" placeholder="inserisci valore">  
<input [(ngModel)]="persona.cognome" placeholder="inserisci valore">

<h2>Ciao mi chiamo  {{persona.nome}} {{persona.cognome}}</h2>

<input placeholder="cerca per " [(ngModel)]="chiaveRicerca">  
<div *ngFor="let p of filtraNomi()">  
<h3>{{p}}</h3>  
</div>