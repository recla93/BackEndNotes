### Servlet Base (Java Servlet)

Un **servlet** è una classe Java che riceve richieste e produce risposte HTTP:

```java
@WebServlet("/persone")
public class PersonaServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        // GET /persone
    }
    
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
        // POST /persone
    }
    
    protected void doPut(HttpServletRequest req, HttpServletResponse resp) {
        // PUT /persone
    }
    
    protected void doDelete(HttpServletRequest req, HttpServletResponse resp) {
        // DELETE /persone
    }
}
```

### Controller in Spring

Spring semplifica i servlet con i **controller**:

```java
@Controller  // Restituisce pagine HTML
public class PersonaController {
    @GetMapping("/persone")
    public String listPersone(Model model) {
        List<Persona> persone = service.findAll();
        model.addAttribute("persone", persone);
        return "lista_persone";  // Nome del template
    }
}
```

### REST Controller

Per API REST che ritornano JSON:

```java
@RestController  // Ritorna JSON automaticamente
@RequestMapping("/api")
public class PersonaController {
    @Autowired
    private PersonaService service;
    
    // GET /api/persone
    @GetMapping("/persone")
    public List<Persona> listPersone() {
        return service.findAll();  // Automaticamente convertito a JSON
    }
    
    // GET /api/persone/1
    @GetMapping("/persone/{id}")
    public Persona getOne(@PathVariable int id) {
        return service.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Non trovato"));
    }
    
    // POST /api/persone
    @PostMapping("/persone")
    public Persona create(@RequestBody Persona p) {
        return service.salvaPersona(p);
    }
    
    // PUT /api/persone/1
    @PutMapping("/persone/{id}")
    public Persona update(@PathVariable int id, @RequestBody Persona p) {
        p.setId(id);
        return service.salvaPersona(p);
    }
    
    // DELETE /api/persone/1
    @DeleteMapping("/persone/{id}")
    public void delete(@PathVariable int id) {
        service.delete(id);
    }
}
```

### Annotazioni REST

| Annotazione | Significato | HTTP |
|-------------|------------|------|
| `@GetMapping` | Recuperare risorsa | GET |
| `@PostMapping` | Creare risorsa | POST |
| `@PutMapping` | Aggiornare risorsa | PUT |
| `@PatchMapping` | Aggiornamento parziale | PATCH |
| `@DeleteMapping` | Eliminare risorsa | DELETE |

### @RequestParam

Estrae parametri dalla query string:

```java
@GetMapping("/persone")
public List<Persona> search(@RequestParam String nome) {
    return service.findByNome(nome);
}

// GET /persone?nome=Mario → nome = "Mario"
```

### @PathVariable

Estrae valori dal path:

```java
@GetMapping("/persone/{id}")
public Persona getOne(@PathVariable int id) {
    return service.findById(id).orElseThrow();
}

// GET /persone/5 → id = 5
```

### @RequestBody

Estrae il corpo della richiesta (JSON) → Oggetto Java:

```java
@PostMapping("/persone")
public Persona create(@RequestBody Persona p) {
    return service.salvaPersona(p);
}

// POST /persone
// Body JSON: { "nome": "Mario", "eta": 30 }
// Spring deserializza automaticamente in Persona
```

### HTTP Status Codes

```java
@RestController
public class PersonaController {
    // 200 OK (default)
    @GetMapping("/persone/{id}")
    public Persona getOne(@PathVariable int id) {
        return service.findById(id).orElseThrow();
    }
    
    // 201 Created
    @PostMapping("/persone")
    @ResponseStatus(HttpStatus.CREATED)
    public Persona create(@RequestBody Persona p) {
        return service.salvaPersona(p);
    }
    
    // 204 No Content
    @DeleteMapping("/persone/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable int id) {
        service.delete(id);
    }
    
    // 400 Bad Request
    // 404 Not Found (lanciare eccezione)
    // 500 Internal Server Error
}
```

### REST best practices

```java
@RestController
@RequestMapping("/api/v1/persone")  // Versionamento API
public class PersonaController {
    @Autowired
    private PersonaService service;
    
    // Risorse al plurale
    @GetMapping
    public ResponseEntity<List<PersonaDTO>> listPersone(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size
    ) {
        Page<Persona> pagina = service.findAll(PageRequest.of(page, size));
        List<PersonaDTO> dtos = pagina.getContent()
            .stream()
            .map(p -> new PersonaDTO(p.getId(), p.getNome()))
            .collect(Collectors.toList());
        return ResponseEntity.ok(dtos);
    }
    
    // Sottorisorsa
    @GetMapping("/{idPersona}/presenze")
    public List<PresenzaDTO> getPresenze(@PathVariable int idPersona) {
        return service.getPresenze(idPersona);
    }
    
    // Operazioni custom
    @PostMapping("/{id}/activate")
    public Persona activate(@PathVariable int id) {
        return service.activate(id);
    }
}
```
