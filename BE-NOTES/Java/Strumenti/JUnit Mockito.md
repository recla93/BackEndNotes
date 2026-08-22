---
topic: "JUnit 5 e Mockito — Testing"
parent: "[[BE-NOTES/Java/Java|Java]]"
---

JUnit 5 è il framework di testing standard per Java. Mockito è la libreria per creare oggetti mock, ideale per isolare il codice sotto test dalle sue dipendenze.

JUnit 5 si compone di: **JUnit Platform** (engine launcher), **JUnit Jupiter** (API + implementazione per Java 8+), **JUnit Vintage** (per test JUnit 4). Mockito si integra nativamente con JUnit 5 via `@ExtendWith(MockitoExtension.class)`.

## JUnit 5 — test base

```java
import org.junit.jupiter.api.*;

class CalcolatriceTest {

    @BeforeAll
    static void setupAll() {
        // Una volta prima di tutti i test
    }

    @BeforeEach
    void setup() {
        // Prima di ogni test
    }

    @Test
    @DisplayName("2 + 3 deve fare 5")
    void testSomma() {
        Calcolatrice calc = new Calcolatrice();
        assertEquals(5, calc.somma(2, 3));
    }

    @Test
    void testDivisionePerZero() {
        Calcolatrice calc = new Calcolatrice();
        assertThrows(ArithmeticException.class,
            () -> calc.dividi(10, 0));
    }

    @AfterEach
    void cleanup() {
        // Dopo ogni test
    }
}
```

`@Test`, `@BeforeEach`, `@AfterEach`, `@BeforeAll` (static), `@AfterAll` (static) sono i lifecycle hook. `assertThrows()` verifica eccezioni. `@DisplayName` personalizza il nome nel report. I metodi di test non devono essere `public` (package-private è sufficiente).

## Assertions

```java
import static org.junit.jupiter.api.Assertions.*;

assertEquals(4, calc.moltiplica(2, 2));
assertNotEquals(5, calc.moltiplica(2, 2));
assertTrue(risultato > 0);
assertFalse(risultato < 0);
assertNull(risultato);
assertNotNull(obj);
assertThrows(IllegalArgumentException.class, () -> metodo(null));
assertDoesNotThrow(() -> metodo(5));
assertIterableEquals(expected, actual);  // per collezioni
assertLinesMatch(expectedLines, actualLines);  // per linee di testo

// Con messaggio personalizzato
assertEquals(4, risultato, "Il risultato dovrebbe essere 4");
```

Ogni assert accetta un messaggio opzionale come ultimo parametro. `assertAll()` raggruppa asserzioni che vengono eseguite tutte anche se qualcuna fallisce (utile per validare piu campi di un oggetto).

## Mockito — mock e verifica

```java
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void trovaUtenteExistente() {
        // Arrange
        User expected = new User(1L, "Alice");
        when(userRepository.findById(1L)).thenReturn(Optional.of(expected));

        // Act
        User result = userService.getUser(1L);

        // Assert
        assertEquals("Alice", result.getNome());
        verify(userRepository).findById(1L);  // verifica chiamata
    }

    @Test
    void trovaUtenteNonExistente() {
        when(userRepository.findById(99L)).thenReturn(Optional.empty());

        assertThrows(UserNotFoundException.class,
            () -> userService.getUser(99L));
    }
}
```

`@Mock` crea il mock. `@InjectMocks` crea l'oggetto reale e inietta i mock. `when()` definisce il comportamento. `verify()` controlla che il metodo sia stato chiamato. `@ExtendWith(MockitoExtension.class)` abilita Mockito in JUnit 5.

## Mockito — matchers e argomenti

```java
// Matchers generici
when(repository.save(any())).thenReturn(savedEntity);
when(repository.findById(anyLong())).thenReturn(Optional.of(user));

// Argument captor (per argomenti complessi)
ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
verify(repository).save(captor.capture());
User captured = captor.getValue();
assertEquals("Alice", captured.getNome());

// Verifica ordine chiamate
InOrder inOrder = inOrder(repository, service);
inOrder.verify(repository).findById(1L);
inOrder.verify(service).process(user);
```

`ArgumentCaptor` cattura l'argomento passato al mock per ispezionarlo. `InOrder` verifica l'ordine delle chiamate. `any()`, `anyLong()`, `anyString()` sono matchers per ignorare argomenti specifici.

## Mockito — comportamento avanzato

```java
// Mock con risposta diversa a chiamate successive
when(repository.findById(anyLong()))
    .thenReturn(Optional.of(user1))
    .thenReturn(Optional.of(user2));

// Lancio eccezione
when(repository.findById(100L)).thenThrow(new DatabaseException("timeout"));

// Spy (oggetto reale con alcuni metodi mockati)
@Spy
List<String> list = new ArrayList<>();
```

`thenReturn()` in catena risponde a chiamate successive. `thenThrow()` per simulare errori. `@Spy` avvolge un oggetto reale permettendo di mockare solo metodi specifici.

## MockMvc — test controller

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void getUserReturns200() throws Exception {
        when(userService.getUser(1L)).thenReturn(new UserDto("Alice"));

        mockMvc.perform(get("/api/users/1")
                .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.nome").value("Alice"));
    }
}
```

`@WebMvcTest` carica solo il contesto web (controller, filtri, ecc.), non l'app completa. `@MockBean` sostituisce bean reali con mock nel contesto Spring. `jsonPath()` estrae valori dalla response JSON.

## @SpringBootTest

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class ApplicationIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void fullIntegration() throws Exception {
        // Test completo con contesto Spring caricato
    }
}
```

`@SpringBootTest` carica l'intero contesto applicativo. `RANDOM_PORT` evita conflitti di porta in CI parallelo. Importante: e lento (>30s). Usalo solo per test di integrazione, non per test unitari.

## Errori comuni

- **Mock non inizializzato**: dimenticare `@ExtendWith(MockitoExtension.class)` o `MockitoAnnotations.openMocks(this)`.
- **`when()` su metodi `void`**: per `void` usa `doThrow()` o `doNothing()` invece di `when()`.
- **Test che dipendono dall'ordine**: ogni test deve essere indipendente. Usa `@TestMethodOrder` solo se strettamente necessario.
- **`@SpringBootTest` per ogni test**: e lentissimo. Usa slice annotations (`@WebMvcTest`, `@DataJpaTest`).
- **Mockare tutto**: se il mock diventa complesso, probabilmente il design della classe va rivisto.
- **Assert su toString()**: fragile. Usa assert specifici su campi.

## Best Practices & Conventions

- Segui **AAA**: Arrange (prepara), Act (esegui), Assert (verifica).
- Usa **`@WebMvcTest`** per controller, **`@DataJpaTest`** per repository, **`@SpringBootTest`** solo per integration test.
- Testa **comportamento**, non implementazione. `verify()` va usato con moderazione.
- Dai nomi descrittivi ai test: `getUser_shouldReturnUser_whenFound()`.
- Usa `assertAll()` per validare piu campi di un oggetto.
- Per test di integrazione reali, considera **Testcontainers** (database, Redis, Kafka reali in Docker).
