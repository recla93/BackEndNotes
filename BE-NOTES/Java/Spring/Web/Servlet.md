---
topic: "Servlet"
nav_prev: "[[Web.md]]"
nav_next: "[[Controller e REST.md]]"
---

Una **Servlet** è una classe Java che riceve richieste HTTP e produce risposte. È il fondamento su cui Spring costruisce i Controller.

## Cos'è una Servlet

| Concetto | Spiegazione |
|---|---|
| **Classe Java** | Estende `HttpServlet` |
| **Scopo** | Ricevere request, produrre response |
| **Metodi** | `doGet()`, `doPost()`, `doPut()`, `doDelete()` |
| **Mapping** | `@WebServlet("/url")` o `web.xml` |
| **Rispetto a Spring** | Versione base del Controller, più limitata e verbosa |

```java
@WebServlet("/persone")
public class PersonaServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        // Legge parametri dalla request
        String id = req.getParameter("id");

        // Prepara risposta
        resp.setContentType("text/html");
        PrintWriter out = resp.getWriter();
        out.println("<h1>Lista Persone</h1>");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        // Crea nuova risorsa
    }
}
```

## DispatcherServlet — il cuore di Spring MVC

Spring non usa le Servlet direttamente. Usa una Servlet **centrale** chiamata `DispatcherServlet`:

```
Request → DispatcherServlet → Controller → Service → Response
                  ↓
           (trova il Controller giusto in base all'URL)
```

`DispatcherServlet` riceve **tutte** le richieste e le inoltra al Controller appropriato basandosi sul mapping (`@GetMapping`, `@PostMapping`, etc.). È il **Front Controller Pattern**.

## Servlet vs Spring Controller

| Aspetto | Servlet | Spring Controller |
|---|---|---|
| **Classe base** | `HttpServlet` | POJO con annotazioni |
| **Mapping** | `@WebServlet` (uno per servlet) | `@RequestMapping` o shortcut |
| **Parametri** | `req.getParameter("nome")` | `@RequestParam`, `@PathVariable`, `@RequestBody` |
| **Response** | `PrintWriter.getWriter()` scrive HTML | Ritorna String (view) o oggetto (JSON) |
| **JSON** | Manuale (es. Gson) | Automatico con Jackson |
| **Validazione** | Manuale | `@Valid` + `BindingResult` |
| **Test** | Complesso (serve server HTTP) | Semplice (MockMvc) |

**Regola pratica:** non scrivere mai Servlet direttamente in un progetto Spring Boot. Usa `@Controller` o `@RestController`.

## Ciclo di vita di una Servlet

1. **Init** — chiamato una volta quando la servlet viene caricata
2. **Service** — chiamato per ogni richiesta, determina `doGet` vs `doPost`
3. **Destroy** — chiamato quando la servlet viene rimossa

```java
@WebServlet("/esempio")
public class EsemploServlet extends HttpServlet {

    @Override
    public void init() {
        // Setup iniziale (una volta)
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) {
        // Gestione richiesta
    }

    @Override
    public void destroy() {
        // Pulizia risorse
    }
}
```

## Quando usare le Servlet ancora oggi

- **Progetti legacy** che usano Servlet/JSP senza Spring
- **Filtri**: `javax.servlet.Filter` (o `jakarta.servlet.Filter`) — ancora usati in Spring per CORS, logging, autenticazione
- **WebSocket** endpoint di basso livello
- **Se stai costruendo** un framework MVC da zero (raro)

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Scrivere servlet direttamente in Spring Boot | Codice non integrato con Spring (no DI, no validazione) | "Tanto funziona" | Usa `@Controller`/`@RestController` sempre in progetti Spring Boot |
| Dimenticare `@WebServlet` | Servlet mai chiamata, 404 | La servlet non è registrata | Aggiungi `@WebServlet("/path")` o registra via `web.xml` |
| Dimenticare `throws` per `IOException` | Errore di compilazione: `PrintWriter.getWriter()` lancia | `doGet`/`doPost` dichiarano le eccezioni nella firma | Usa `@Controller` invece della servlet — Spring gestisce le eccezioni |
| Gestione manuale dei parametri | Boilerplate, null check ovunque | `req.getParameter()` restituisce stringa o null | Spring bootstrappa direttamente a `@RequestParam`, `@PathVariable`, DTO |