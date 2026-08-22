---
topic: "Metodi"
nav_prev: "[[Classi ed Oggetti.md]]"
nav_next: "[[Stato, visibility, SOC e STATIC.md]]"
---

## Firma del Metodo

```
[visibilità] [static] tipoRitorno nomeMetodo ([parametri]) {
    // corpo
}
```

```java
public static int somma(int a, int b) {
    return a + b;
}
```

## Visibilità

| Modificatore | Stessa classe | Stesso package | Sottoclasse | Ovunque |
|---|---|---|---|---|
| `public` | ✓ | ✓ | ✓ | ✓ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| *(default)* | ✓ | ✓ | ✗ | ✗ |
| `private` | ✓ | ✗ | ✗ | ✗ |

**Regola:** usa sempre il modificatore **più restrittivo** possibile. Default = `private`, aumenta solo se serve.

## Parametri e Return

```java
// Con ritorno
public int somma(int a, int b) {
    return a + b;
}

// Senza ritorno (void)
public void stampa(String msg) {
    System.out.println(msg);
    // nessun return necessario
}

// Early return — esce subito
public double dividi(int a, int b) {
    if (b == 0) return 0;  // caso difensivo
    return (double) a / b;
}
```

### Pass by Value (sempre)

Java passa **sempre** i parametri per **valore**:

```java
// Primitivi: passaggio per COPIA
void modifica(int x) { x = 10; }
int a = 5;
modifica(a);            // a rimane 5, non 10!

// Reference: passaggio per COPIA DEL RIFERIMENTO
void modifica(Persona p) { p.setNome("Luigi"); }
Persona p = new Persona("Mario");
modifica(p);            // p.nome diventa "Luigi" (l'oggetto è modificato!)
// MA:
void reassign(Persona p) { p = new Persona("Luigi"); }
reassign(p);           // p rimane "Mario" (il riferimento originale non cambia!)
```

## Metodi Static vs Istanza

```java
public class Matematica {
    // Metodo di classe (static)
    public static int somma(int a, int b) {
        return a + b;
    }

    // Metodo di oggetto (istanza)
    public int moltiplicazione(int a, int b) {
        return a * b;
    }
}

// Uso
Matematica.somma(5, 3);   // Sulla classe
new Matematica().moltiplicazione(5, 3);  // Sull'istanza
```

| | Static | Istanza |
|---|---|---|
| Appartiene a | Classe | Oggetto |
| Accesso a `this` | No | Sì |
| Accesso a campi istanza | No | Sì |
| Override | No (nasconde) | Sì |
| Chiamata | `Classe.metodo()` | `oggetto.metodo()` |

## Overload

Stesso nome, **parametri diversi** (numero, tipo, ordine):

```java
public class Calcolatore {
    public int somma(int a, int b) { return a + b; }
    public int somma(int a, int b, int c) { return a + b + c; }
    public double somma(double a, double b) { return a + b; }
}
```

**Best Practice:** un metodo principale + overload che chiamano il principale con default.

## Varargs (Java 5+)

Numero variabile di argomenti:

```java
public int somma(int... numeri) {
    int totale = 0;
    for (int n : numeri) totale += n;
    return totale;
}

somma(1, 2);           // 3
somma(1, 2, 3, 4, 5);  // 15
```

**Regole:**
- Un solo varargs per metodo
- Deve essere l'ultimo parametro
- Equivalente a `int[]` ma chiamata più comoda

## Problemi comuni

| Problema | Esempio | Soluzione |
|---|---|---|
| **Confondere return con print** | `metodo()` non stampa nulla | `return` ≠ `System.out.println()` |
| **Overload ambiguo** | `foo(null)` con `foo(int)` e `foo(String)` | Il compilatore segnala ambiguità, evita overload con tipi imparentati |
| **Modificare parametri** | Riassignare parametri nel metodo | Dichiarali `final` se necessario |
| **Metodo troppo lungo** | 100+ righe | Spezza in metodi più piccoli (SRP) |
| **Side effect** | Il metodo modifica lo stato globale | Preferisci metodi puri (input → output) |
