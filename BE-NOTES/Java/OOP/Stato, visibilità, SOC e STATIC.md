## Stato dell'Oggetto

L'insieme dei valori delle proprietà in un determinato momento è lo **stato dell'oggetto**.

​

L'esterno può modificare lo stato di un oggetto tramite metodi **Setter**, mentre interagisce con lo stato tramite **Getter e Setter**.

​

## Incapsulamento

**Incapsulamento**: Principio per cui un oggetto è responsabile del proprio stato e l'esterno interagisce con esso tramite getter e setter.

​

Vogliamo **accesso controllato alle proprietà** dall'oggetto stesso. L'oggetto deve avere il controllo e non il resto del mondo.

​

## Separation of Concerns (SoC)

La responsabilità deve essere gestita dall'oggetto stesso e non dal chiamante. La logica deve essere concentrata in un punto e quel punto è l'oggetto stesso per poterlo utilizzare in ogni caso secondo i suoi dettami.

​

I controlli è bene farli sempre nel contesto di esecuzione/esistenza dell'oggetto.

​

## Modificatore STATIC

Ciò che non è `static` appartiene allo **scope di oggetto**, mentre con `STATIC` quelle caratteristiche appartengono alla **classe** e non al singolo oggetto.

​

## Caratteristiche di STATIC

- Appartiene direttamente alla classe e non ai singoli oggetti
    

- ​
    
- Non ha accesso al `this`
    
- ​
    
- Può essere chiamato direttamente sulla classe senza creare un'istanza
    
- ​
    
- Esiste una sola copia per l'intera classe, indipendentemente dal numero di oggetti creati
    

- ​
    

**STATIC FINAL**: Valore costante che, nonostante le modifiche, tornerà sempre a quel valore.

​

## Modificatore FINAL

Una volta che una variabile `FINAL` viene assegnata, il suo valore non può più essere cambiato.

​

## Caratteristiche di FINAL

- Può essere usato sia per variabili di oggetto che per variabili di classe (static)
    

- ​
    
- Per le variabili di oggetto, ogni oggetto avrà la sua propria variabile final
    
- ​
    
- È spesso usato per definire costanti
    
- ​
    
- Aiuta a centralizzare valori importanti in un unico punto del codice