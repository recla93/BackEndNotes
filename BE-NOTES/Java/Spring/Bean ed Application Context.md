## Bean e Application Context

### Cos'è un Bean?

Un **Bean** è un oggetto gestito da Spring. Di default è un **Singleton** (un'unica istanza per tutta l'app):

```
┌────────────────────────────────────┐
│    Application Context (memoria)   │
│                                    │
│  ┌──────────────────────────────┐ │
│  │ Bean: PersonaService         │ │
│  │ (Istanza unica per tutta app)│ │
│  └──────────────────────────────┘ │
│  ┌──────────────────────────────┐ │
│  │ Bean: PersonaRepository      │ │
│  └──────────────────────────────┘ │
│  ┌──────────────────────────────┐ │
│  │ Bean: PersonaController      │ │
│  └──────────────────────────────┘ │
└────────────────────────────────────┘
```

### Application Context

L'**Application Context** è una zona di memoria gestita da Spring dove risiedono i **Bean**:

```java
// Spring legge le annotazioni e crea i Bean
@Configuration
public class AppConfig {
    // ...
}

// Il boot dell'app crea l'Application Context
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
        // Spring ha creato l'Application Context con tutti i Bean
    }
}
```

### Creazione di Bean

```java
// 1. Con annotazioni
@Component
public class PersonaService {
    // Spring lo riconosce e lo crea come Bean
}

// 2. Con @Bean in @Configuration
@Configuration
public class AppConfig {
    @Bean
    public PersonaService personaService() {
        return new PersonaService();
    }
}

// 3. Con metodi di factory
@Bean
public PersonaRepository personaRepository(DataSource dataSource) {
    // Spring passa le dipendenze
    return new PersonaRepository(dataSource);
}
```

### Scope dei Bean

```java
// Singleton (default): un'istanza per tutta l'app
@Component
@Scope("singleton")
public class PersonaService {
}

// Prototype: nuova istanza ogni volta
@Component
@Scope("prototype")
public class PersonaService {
}

// Request (web): nuova istanza per ogni HTTP request
@Component
@Scope("request")
public class PersonaService {
}

// Session (web): nuova istanza per ogni sessione HTTP
@Component
@Scope("session")
public class PersonaService {
}
```
