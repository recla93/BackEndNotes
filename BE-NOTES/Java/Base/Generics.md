
È un **tipo parametrizzato** che viene fornito in compilazione e che permette a classe o interfaccia di usare quel parametro come tipo di ritorno o come parametro stesso.
## Vantaggi dei Generics

- Evitano di lavorare con `Object` come tipo formale
- Il tipo viene definito in fase di scrittura del codice
- I generics non sono in runtime, vengono decisi in compilazione
- Permettono di parametrizzare i tipi

## Notazione

- `T`: tipo generico (Type)
- `E`: per liste, rappresenta Element

## Vincoli sui Generics

Possiamo dare dei vincoli da parametrizzare con specializzazioni:

java

`public interface ListEntities<E extends BaseEntity> extends List<E>`

Con quel `BaseEntity` poniamo un limite: il tipo passato deve essere per forza `BaseEntity` o un suo sottotipo.

​