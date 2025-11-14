L'override permette di ridefinire un metodo di una classe padre in una classe figlia. Consente di personalizzare il comportamento di un metodo ereditato mantenendo la stessa firma del metodo originale.

​

## Interface (Interfacce)

## Cosa può contenere un'interfaccia

- FINAL STATIC: Sì
- STATIC: No
- METODI STATIC: Sì
- METODI CONCRETI: Sì (con keyword `default` da Java 8)
- METODI ASTRATTI: Sì
- PROPRIETÀ NORMALI: No
## Caratteristiche principali

- Può contenere solo costanti (proprietà final static) e metodi astratti di default
- Non può essere istanziata direttamente
- Serve a massimizzare la differenza tra tipo formale e tipo concreto
- Permette l'implementazione multipla
- Viene utilizzata principalmente come tipo formale
- Tutti gli elementi sono implicitamente public e abstract

Un'interfaccia è un tipo formale che dice al resto del programma che quella classe sa fare determinate cose.
## StringBuilder

Utilizzato per la concatenazione di stringhe di grosse dimensioni, più efficiente rispetto alla concatenazione standard.
## Interface Comparable

L'interfaccia `Comparable` è implementabile agli oggetti per renderli comparabili. Quando gli oggetti sono in lista, saranno comparabili per caratteristica di oggetto.
## Overload

Si ha **overload** quando si hanno versioni diverse dello stesso metodo distinte dal numero/tipo di parametri.

Il chiamante decide quale metodo usare in base ai parametri che passa.

## Best Practice

È bene avere un metodo base (versione principale) e tante altre versioni che prendono input diversi e rimandano al metodo default.

**Principio DRY** (Don't Repeat Yourself): L'overload segue questo principio.
## Casting

**Casting = cambio di tipo formale**.

​