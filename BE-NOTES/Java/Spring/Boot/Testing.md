---
topic: "Testing Spring Boot"
parent: "[[BE-NOTES/Java/Spring/Spring|Spring]]"
nav_prev: "[[[[Spring Boot Best Practices.md]]|Spring Boot Best Practices]]"
---

Spring Boot offre annotazioni per testare specifici slice dell'applicazione senza caricare l'intero contesto. I test che caricano tutto il contesto (`@SpringBootTest`) sono necessari solo per integration test.

I test possono essere lenti (30-60s per contesto). Usa le slice annotations per test mirati: controller, repository, service, json.

## Slice annotations

```java
// Solo il contesto web (controller, filtri, advice)
@WebMvcTest(UserController.class)
class UserControllerTest { ... }

// Solo il contesto JPA (repository, entity)
@DataJpaTest
class UserRepositoryTest { ... }

// Solo JSON serialization
@JsonTest
class UserDtoTest { ... }

// Solo REST client
@RestClientTest(UserClient.class)
class UserClientTest { ... }
```

Ogni slice annotation carica solo i bean necessari per quel layer. Risultato: test in secondi invece di minuti. `@DataJpaTest` usa un database embedded (H2) per default.

## @SpringBootTest — full context

```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class ApplicationIntegrationTest {

    @Autowired
    private TestRestTemplate rest;

    @Test
    void fullFlow() {
        ResponseEntity<UserDto> response = rest.getForEntity(
            "/api/users/1", UserDto.class);
        assertThat(response.getStatusCode()).isOk();
    }
}
```

`@SpringBootTest` carica l'intero contesto. `RANDOM_PORT` evita conflitti di porta. `@ActiveProfiles("test")` usa `application-test.properties` con DB e configurazioni di test.

## Testcontainers — database reali

```java
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import org.testcontainers.containers.PostgreSQLContainer;

@Testcontainers
@SpringBootTest
class DatabaseIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    void dbConnectionWorks() {
        assertThat(userRepository.count()).isZero();
    }
}
```

Testcontainers avvia un container Docker reale per PostgreSQL (MySQL, Redis, Kafka) durante i test. `@DynamicPropertySource` sovrascrive le properties Spring con quelle del container. Alternativa piu realistica di H2 embedded.

## MockBean — mock nel contesto

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void getUser_returns200() throws Exception {
        when(userService.getUser(1L)).thenReturn(new UserDto("Alice"));

        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.nome").value("Alice"));
    }
}
```

`@MockBean` sostituisce un bean reale nel contesto Spring con un mock Mockito. Utile per isolare il layer web dal business layer. Dopo il test, il contesto viene ricaricato (ogni test ricrea il mock).

## @DataJpaTest — repository

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void findByEmail_returnsUser() {
        User user = new User("test@example.com", "Alice");
        entityManager.persistAndFlush(user);

        Optional<User> found = userRepository.findByEmail("test@example.com");

        assertThat(found).isPresent();
        assertThat(found.get().getNome()).isEqualTo("Alice");
    }
}
```

`@DataJpaTest` configura solo il layer JPA: repository, entity manager, datasource embedded. `TestEntityManager` permette operazioni dirette sul DB senza passare dal repository. Ogni test e transazionale e viene rollbackato automaticamente.

## Test per service layer

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void getUser_quandoEsiste_ritornaUser() {
        when(userRepository.findById(1L))
            .thenReturn(Optional.of(new User(1L, "Alice")));

        UserDto result = userService.getUser(1L);

        assertThat(result.getNome()).isEqualTo("Alice");
        verify(userRepository).findById(1L);
    }
}
```

Il service non ha bisogno di Spring per i test unitari. `@ExtendWith(MockitoExtension.class)` + `@Mock` + `@InjectMocks` e sufficiente. Test veloci (millisecondi) senza caricare il contesto.

## Test per RestTemplate/WebClient

```java
@SpringBootTest(webEnvironment = WebEnvironment.MOCK)
@AutoConfigureMockMvc
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void createUser_returns201() throws Exception {
        String json = """
            {"nome": "Alice", "email": "alice@test.com"}
            """;

        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(json))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"));
    }
}
```

`webEnvironment = MOCK` (default) usa un servlet container mock (non avvia Tomcat). `@AutoConfigureMockMvc` configura `MockMvc` per testare i controller tramite dispatcher servlet simulato.

## Errori comuni

- **`@SpringBootTest` per ogni test**: lentissimo. Usa slice annotations.
- **H2 in produzione vs Testcontainers**: H2 non e uguale a PostgreSQL. Usa Testcontainers per test che dipendono da features specifiche del DB.
- **MockBean in test non `@WebMvcTest`**: `@MockBean` funziona solo con contesto Spring caricato. Per service test, usa Mockito diretto.
- **Test che condividono stato**: ogni test deve essere indipendente. Usa `@BeforeEach` per setup pulito.
- **Database embedded in CI**: se il CI non ha Docker, Testcontainers non funziona. Usa H2 + `@DataJpaTest` semplice per CI, Testcontainers per locale.
- **Timeout per contesto lento**: i test Spring possono richiedere minuti. Imposta `@TimeOut` su test di integrazione.

## Best Practices & Conventions

- **Unit test**: service layer con Mockito, nessun contesto Spring.
- **Web test**: `@WebMvcTest` per controller.
- **Repository test**: `@DataJpaTest` per repository.
- **Integration test**: `@SpringBootTest` solo per flussi completi (pochi test).
- **Database reali**: Testcontainers per PostgreSQL/MySQL. H2 per test veloci.
- **Organizzazione**: test unitari in `src/test/java/...service/`, web test in `...controller/`, integration test in `...integration/`.
- **MockBean con cautela**: ogni `@MockBean` forza il refresh del contesto. Usane pochi per test.
