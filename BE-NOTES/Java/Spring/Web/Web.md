---
topic: "Web"
nav_next: "[[Servlet.md]]"
---

Spring Web (spring-boot-starter-web) è il modulo che gestisce la comunicazione **HTTP**: riceve richieste dai client e produce risposte.

## Architettura di base

```
CLIENT (browser, mobile, Postman)
    │  HTTP Request
    ▼
SERVER (Spring Boot sulla porta 8080)
    │
    ├─ DispatcherServlet ─→ Controller ─→ Service ─→ Repository ─→ DB
    │
    └─ HTTP Response
```

## Fondamenti HTTP

**HTTP** è un protocollo di comunicazione **stateless** — ogni richiesta è indipendente:

| Componente | Esempio | Descrizione |
|---|---|---|
| **URL** | `http://localhost:8080/api/persone/3?page=1` | Indirizzo completo |
| **IP** | `127.0.0.1` (localhost) | Locazione del server fisico |
| **Porta** | `8080` (Spring) / `4200` (Angular) | Server software specifico |
| **URI** | `/api/persone/3` | Risorsa da raggiungere |
| **Path variable** | `/3` | Valore nell'URI (es. ID) |
| **Query string** | `?page=1` | Parametri aggiuntivi |

### Verbi HTTP — CRUD

| Verbo | CRUD | Controller | Esempio |
|---|---|---|---|
| **GET** | Read | `@GetMapping` | `/api/persone` |
| **POST** | Create | `@PostMapping` | `/api/persone` (con body) |
| **PUT** | Update (full) | `@PutMapping` | `/api/persone/{id}` |
| **PATCH** | Update (parziale) | `@PatchMapping` | `/api/persone/{id}` |
| **DELETE** | Delete | `@DeleteMapping` | `/api/persone/{id}` |

I verbi permettono l'**overload** dello stesso URI: `GET /api/persone` ≠ `POST /api/persone` ≠ `DELETE /api/persone`.

### Struttura di una Request

```
Request
├── URL (a chi)
├── Verbo (cosa fare)
├── Header (metadati: Content-Type, Authorization, Accept)
└── Body (dati: JSON, form, file)
```

- **Header**: metadati — tipo di contenuto, token, lingue accettate
- **Body**: dati effettivi — campi form, JSON, file. Può anche essere vuoto (GET, DELETE)

### Struttura di una Response

```
Response
├── Status Code (risultato operazione)
├── Header (metadati)
└── Body (HTML per Controller, JSON per RestController)
```

## Endpoint = URI + Verbo

L'**endpoint** è la combinazione di URI e verbo HTTP. È ciò che identifica univocamente un'operazione:

```
GET  /api/persone    → lista persone
POST /api/persone    → crea persona
GET  /api/persone/5  → persona con ID 5
```

**Endpoint = URI + Verbo** — l'URI da solo non basta, serve anche il verbo.

## @Controller vs @RestController

| Aspetto | @Controller | @RestController |
|---|---|---|
| **Risposta** | Pagine HTML (templates) | Dati JSON |
| **Model** | Usa `Model` per passare dati alla view | Non usa Model |
| **Serializzazione** | Manuale | Automatica (Jackson) |
| **Uso tipico** | Applicazioni web tradizionali | API RESTful |

```java
// @Controller: restituisce una vista HTML
@Controller
public class PaginaController {
    @GetMapping("/home")
    public String home(Model model) {
        model.addAttribute("titolo", "Benvenuto");
        return "home";  // nome del template HTML (Thymeleaf, JSP, etc.)
    }
}

// @RestController: restituisce JSON
@RestController
@RequestMapping("/api/persone")
public class PersonaController {
    @GetMapping
    public List<Persona> getAll() {
        return service.findAll();
        // Jackson serializza automaticamente in JSON
    }
}
```

## Sessioni — HTTP è stateless

HTTP non mantiene stato tra richieste consecutive. La **sessione** risolve questo:

1. Prima request → server crea **session** con un **ID univoco** (JSESSIONID)
2. Response include il session ID
3. Il browser allega il session ID a ogni request successiva
4. Il server riconosce il client e recupera i dati di sessione

```java
@GetMapping("/carrello")
public String carrello(HttpSession session) {
    List<Item> items = (List<Item>) session.getAttribute("carrello");
    if (items == null) {
        items = new ArrayList<>();
        session.setAttribute("carrello", items);
    }
    return "carrello";
}
```

Nelle API REST moderne, le sessioni sono spesso sostituite da **JWT** (JSON Web Token) — il token contiene i dati utente e viene passato nell'header `Authorization`.

## Cookie

I **cookie** sono dati salvati dal browser e inviati automaticamente con ogni request:

```
Server → Response con Set-Cookie → Browser salva → Browser include in ogni request
```

Usati per: session ID, preferenze utente, tracking.

## Hash e Sicurezza

**Hash** = funzione che trasforma un input in una stringa di lunghezza fissa:

```
"password123" → SHA-256 → "a7c3d...b9f"
Stesso input → Stesso hash (deterministico)
```

Usato per:
- **Password**: non salvare mai la password in chiaro, salva l'hash
- **Integrità**: verificare che un file non sia stato modificato
- **Session ID**: generare identificatori univoci

Hash comuni in Java: SHA-256, bcrypt (per password), Argon2.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| `@Controller` restituisce JSON per errore | `404.jsp` o errore "circular view path" | Manca `@ResponseBody` o usi `@Controller` invece di `@RestController` | Usa `@RestController` per API REST; per `@Controller`, aggiungi `@ResponseBody` |
| Path variable vs query param confusi | Endpoint non matcha o parametro vuoto | Mettere `/api/persone/{page}` invece di `/api/persone?page=1` | Path variable per risorsa (`/persone/{id}`), query param per filtro/paginazione |
| Verbo HTTP sbagliato | `405 Method Not Allowed` | Usare GET per creare risorse o POST per cancellare | GET = lettura, POST = creazione, PUT/PATCH = modifica, DELETE = cancellazione |
| `@RequestMapping` senza verbo | Matcha TUTTI i verbi sullo stesso endpoint | Dimenticare GET/POST/PUT/DELETE nello `@RequestMapping` | Usa `@GetMapping`, `@PostMapping` etc. invece di `@RequestMapping` generico |
| Stato HTTP non gestito | `500 Internal Server Error` per errori prevedibili (es. risorsa non trovata) | `findById()` lancia eccezione non gestita | Usa `ResponseEntity` con status code appropriato (404, 400, 201) |
| Session state non thread-safe | Race condition su attributi di sessione | Modificare attributi di sessione senza sincronizzazione | Usa oggetti immutabili o sincronizza l'accesso alla sessione |
| Password in chiaro | Dati sensibili esposti in caso di breccia | Salvare password in chiaro invece dell'hash | Usa sempre bcrypt (o Argon2) per hashare le password |
