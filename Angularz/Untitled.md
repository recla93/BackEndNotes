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