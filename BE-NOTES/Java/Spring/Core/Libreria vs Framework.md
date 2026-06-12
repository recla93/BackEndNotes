# Libreria vs Framework

Entrambi sono **codice riutilizzabile** scritto da altri, ma cambia **chi controlla il flusso** dell'applicazione.

## Libreria — chiami tu la libreria

```java
// JDBC è una libreria: tu chiami i suoi metodi
Connection conn = DriverManager.getConnection(url);
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM users");
// Tu decidi quando e come chiamare ogni metodo
```

**Caratteristiche:**
- **Tu hai il controllo** — chiami la libreria quando serve
- **Funzionalità mirate** — una cosa specifica (es. parsing JSON, connessione DB)
- **Leggera** — includi solo ciò che usi
- **Esempi:** JDBC, Jackson, Gson, Apache Commons, JUnit

## Framework — il framework chiama te

```java
// Spring è un framework: definisci componenti e Spring li chiama
@RestController
public class TaskController {
    @GetMapping("/tasks")
    public List<Task> getAll() {
        // Tu scrivi solo questa logica
        // Spring chiama questo metodo quando arriva una richiesta HTTP
    }
}
```

**Caratteristiche:**
- **Inversion of Control (IoC)** — il framework gestisce il flusso, tu inserisci il tuo codice nei punti giusti
- **Struttura imposta** — devi seguire le convenzioni del framework (annotazioni, interfacce, configurazioni)
- **Completo** — risolve problemi trasversali (sicurezza, transazioni, serializzazione, DI)
- **Esempi:** Spring Boot, Hibernate, Angular, JUnit (sì, JUnit è un framework — chiama i tuoi `@Test`)

## Regola mnemonica: "Hollywood Principle"

> "Don't call us, we'll call you"
> 
> — Con una **libreria** sei tu a chiamare il codice.
> — Con un **framework** è il framework che chiama il tuo codice.

## Tabella comparativa

| Aspetto | Libreria | Framework |
|---|---|---|
| **Chi controlla il flusso?** | Tu (il tuo codice chiama la libreria) | Il framework (chiama il tuo codice) |
| **Complessità** | Bassa (usi solo ciò che serve) | Alta (curva di apprendimento) |
| **Integrazione** | Semplice (importi e usi) | Richiede configurazione |
| **Flessibilità** | Massima | Vincolata alle regole del framework |
| **Esempi in TaskMngr** | Lombok, MapStruct, JJWT | Spring Boot, Spring Data JPA, Spring Security |

## Quando usare cosa

**Libreria** quando:
- Hai bisogno di una funzionalità specifica (es. parsare JSON con Jackson)
- Vuoi mantenere il controllo totale del flusso
- Hai un progetto piccolo o legacy

**Framework** quando:
- Costruisci un'applicazione complessa (API, web, enterprise)
- Vuoi soluzioni già pronte per problemi comuni (transazioni, sicurezza, DI)
- Il team segue standard consolidati

## Problemi comuni

| Problema | Libreria | Framework |
|---|---|---|
| **Version mismatch** | Dipendenze conflittuali | Versioni dei moduli devono essere compatibili |
| **Lock-in** | Facile sostituire | Difficile migrare ad altro framework |
| **Debugging** | Semplice (segui le chiamate) | Complesso (stack trace profondi, proxy, AOP) |
| **Performance overhead** | Minimo | Può essere significativo (reflection, proxy, context loading) |