
## 🌟 Terminologia Essenziale  
  
- **REST**: architettura per API basata su HTTP e risorse.  
- **Controller**: gestisce le richieste HTTP.  
- **Service**: contiene la logica di business.  
- **Repository**: accede al database (es. JPA).  
- **DTO (Data Transfer Object)**: oggetti per trasferire dati tra client e server senza esporre entità.  
- **Exception Handling**: gestione centralizzata degli errori con `@ControllerAdvice`.  
- **Validation**: verifica dei dati in ingresso con annotazioni (`@Valid`, `@NotBlank`, ...).  
  
***  
  
## 🚀 Pratiche Fondamentali con Esempi  
  
***  
  
### 1. Architettura a Strati  
  
Mantieni il codice organizzato in livelli separati: Controller, Service e Repository.  
  
```java  
@RestController  
@RequestMapping("/api/users")  
public class UserController {  
  
    private final UserService userService;  
  
    public UserController(UserService userService) {  
        this.userService = userService;  
    }  
  
    @GetMapping("/{id}")  
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {  
        UserDTO user = userService.findById(id);  
        return ResponseEntity.ok(user);  
    }  
}  
```  
  
```java  
@Service  
public class UserService {  
  
    private final UserRepository userRepository;  
    private final UserMapper userMapper;  
  
    public UserService(UserRepository userRepository, UserMapper userMapper) {  
        this.userRepository = userRepository;  
        this.userMapper = userMapper;  
    }  
  
    public UserDTO findById(Long id) {  
        User userEntity = userRepository.findById(id)  
            .orElseThrow(() -> new ResourceNotFoundException("User non trovato"));  
        return userMapper.toDTO(userEntity);  
    }  
}  
```  
  
```java  
@Repository  
public interface UserRepository extends JpaRepository<User, Long> {  
}  
```  
  
**Spiegazione:**    
Separare le responsabilità rende il codice più modulare e facile da mantenere. Il controller riceve richieste, il service applica la logica, il repository accede ai dati.  
  
***  
  
### 2. Uso di DTO e Validazione  
  
Usa DTO per accettare e restituire dati, proteggendo l'entità interna e validandoli correttamente.  
  
```java  
public class UserDTO {  
  
    @NotBlank(message = "Il nome è obbligatorio")  
    private String name;  
  
    @Email(message = "Email non valida")  
    @NotBlank(message = "Email obbligatoria")  
    private String email;  
  
    // getter e setter ...  
}  
```  
  
```java  
@PostMapping  
public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserDTO userDTO) {  
    UserDTO createdUser = userService.createUser(userDTO);  
    return new ResponseEntity<>(createdUser, HttpStatus.CREATED);  
}  
```  
  
**Spiegazione:**    
L'annotazione `@Valid` verifica che i dati siano corretti prima di passare al service. Aiuta a prevenire errori e a restituire messaggi chiari in caso di dati invalidi.  
  
***  
  
### 3. Gestione Centralizzata degli Errori  
  
Definisci un gestore globale per uniformare la risposta agli errori.  
  
```java  
@ControllerAdvice  
public class GlobalExceptionHandler {  
  
    @ExceptionHandler(ResourceNotFoundException.class)  
    public ResponseEntity<ApiError> handleResourceNotFound(ResourceNotFoundException ex) {  
        ApiError error = new ApiError(HttpStatus.NOT_FOUND, ex.getMessage());  
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);  
    }  
  
    @ExceptionHandler(MethodArgumentNotValidException.class)  
    public ResponseEntity<ApiError> handleValidationException(MethodArgumentNotValidException ex) {  
        String errors = ex.getBindingResult().getFieldErrors().stream()  
            .map(err -> err.getField() + ": " + err.getDefaultMessage())  
            .collect(Collectors.joining(", "));  
        ApiError error = new ApiError(HttpStatus.BAD_REQUEST, "Validazione fallita: " + errors);  
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);  
    }  
}  
```  
  
```java  
public class ApiError {  
    private HttpStatus status;  
    private String message;  
  
    public ApiError(HttpStatus status, String message) {  
        this.status = status;  
        this.message = message;  
    }  
    // getter e setter  
}  
```  
  
**Spiegazione:**    
Con `@ControllerAdvice` centralizzi l’handling delle eccezioni. Le risposte sono sempre coerenti e chiare per il client.  
  
***  
  
### 4. Uso di ResponseEntity  
  
Per risposte HTTP più flessibili (stato, header, corpo).  
  
```java  
@GetMapping("/hello")  
public ResponseEntity<String> hello() {  
    return ResponseEntity  
        .status(HttpStatus.OK)  
        .header("Custom-Header", "Valore")  
        .body("Ciao dal server!");  
}  
```  
  
**Spiegazione:**    
`ResponseEntity` permette di personalizzare ogni aspetto della risposta HTTP.  
  
***  
  
### 5. Documentazione API con OpenAPI/Swagger  
  
Aggiungi la dipendenza:  
  
```xml  
<dependency>  
    <groupId>org.springdoc</groupId>  
    <artifactId>springdoc-openapi-ui</artifactId>  
    <version>1.7.0</version>  
</dependency>  
```  
  
Visita `http://localhost:8080/swagger-ui.html` per la documentazione interattiva.  
  
***  
  
## 💡 Consigli Extra  
  
- Usa paginazione nelle GET di collezioni.  
- Proteggi le API con Spring Security (OAuth2/OIDC).  
- Configura logging strutturato (SLF4J).  
- Externalizza le configurazioni per differenti ambienti.  
- Segui convenzioni REST (es. nomi plurali per risorse).  
- Testa con MockMvc, Mockito e integrazione completa.  
- Non loggare dati sensibili.  
  
***  
  