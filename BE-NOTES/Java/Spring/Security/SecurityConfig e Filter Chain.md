---
topic: "SecurityConfig e Filter Chain"
parent: "[[BE-NOTES/Java/Spring/Security/Spring Security|Spring Security]]"
---

# SecurityConfig e Filter Chain

Spring Security lavora a **catena di filtri**: ogni richiesta HTTP attraversa una serie di filtri (CORS, CSRF, autenticazione, autorizzazione) prima di arrivare al controller. In [[TaskMngr]] la catena è personalizzata per supportare JWT + OAuth2, con route pubbliche e protette.

## Schema base

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // 1. CORS — permetti richieste dal frontend (localhost:4200)
            .cors(Customizer.withDefaults())

            // 2. CSRF — disabilitato (API stateless, nessun cookie di sessione)
            .csrf(AbstractHttpConfigurer::disable)

            // 3. Nessuna sessione HTTP — ogni richiesta è indipendente
            .sessionManagement(sm -> sm.sessionCreationPolicy(STATELESS))

            // 4. Regole di autorizzazione
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**", "/api/oauth2/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated()
            )

            // 5. Login OAuth2 (Google, GitHub)
            .oauth2Login(oauth2 -> oauth2
                .userInfoEndpoint(userInfo -> userInfo
                    .userService(customOAuth2UserService))
                .successHandler(oAuth2SuccessHandler)
            )

            // 6. JWT Filter prima del filter di autenticazione standard
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

## Elementi chiave — perché sono configurati così

### CORS abilitato
Il frontend Angular (`http://localhost:4200`) è un'origine diversa dal backend (`http://localhost:8080`). Senza CORS, il browser blocca le richieste. Spring Boot lo configura leggendo `spring.web.cors.*` o un `@Bean CorsConfigurationSource`.

### CSRF disabilitato
Il CSRF protegge dagli attacchi che sfruttano i cookie di sessione. Ma noi usiamo JWT nell'header `Authorization`, non cookie — il CSRF non serve. Se usassi cookie-based auth, il CSRF diventerebbe obbligatorio.

### Stateless
`STATELESS` dice a Spring Security di **non creare mai una sessione HTTP**. Ogni richiesta è autenticata indipendentemente dal JWT. Alternativa: `IF_REQUIRED` (default) crea sessione se serve, ma non è necessario per API JWT.

### Ordine dei filtri
`jwtFilter` viene inserito **prima** di `UsernamePasswordAuthenticationFilter` perché deve leggere il JWT, validarlo e impostare il `SecurityContext` prima che gli altri filtri decidano se la richiesta è autorizzata.

## Route pubbliche vs protette

| Route | Accesso | Perché pubblica |
|---|---|---|
| `POST /api/auth/login` | Pubblica | Chiunque può fare login |
| `POST /api/auth/register` | Pubblica | Nuovi utenti |
| `POST /api/auth/refresh` | Pubblica | Rinnovo token (ha il refresh token) |
| `/api/oauth2/**` | Pubblica | Callback OAuth2 (Google/GitHub reindirizzano qui) |
| `/swagger-ui/**` | Pubblica | Documentazione API |
| `/v3/api-docs/**` | Pubblica | OpenAPI JSON |
| `GET /api/users/me` | Autenticata | Solo utente loggato |
| `POST /api/tasks` | Autenticata | Solo utente loggato |
| `/api/admin/**` | ADMIN | Solo utenti con ruolo ADMIN |

## Method Security (@EnableMethodSecurity)

Oltre alle regole url-based in `authorizeHttpRequests`, il progetto usa **method security**:

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // abilita @PreAuthorize, @PostAuthorize, etc.
public class SecurityConfig { ... }
```

### Esempi @PreAuthorize

```java
// Solo ADMIN
@PreAuthorize("hasRole('ADMIN')")
@DeleteMapping("/{id}")
public void deleteUser(@PathVariable Long id) { ... }

// ADMIN oppure l'utente stesso
@PreAuthorize("hasRole('ADMIN') or #id == @securityUtil.getCurrentUserId()")
@PutMapping("/{id}")
public UserResponse updateUser(@PathVariable Long id, @RequestBody ...) { ... }
```

La SpEL expression `@securityUtil.getCurrentUserId()` chiama il bean `SecurityUtil` per confrontare l'ID del parametro con l'utente autenticato.

## Modello autorizzativo (RBAC)

```
User
 └── role: Role enum (USER, ADMIN)

JWT claims
 ├── sub: email
 ├── id: userId
 └── role: "USER" | "ADMIN"

GrantedAuthority nel SecurityContext
 └── "ROLE_USER" | "ROLE_ADMIN"  (prefisso ROLE_ automatico con hasRole)
```

| Ruolo | Permessi |
|---|---|
| `USER` (default) | CRUD sui propri dati, task, team |
| `ADMIN` | Gestione utenti (DELETE, update di qualsiasi utente), accesso a route `/api/admin/**` |

## In TaskMngr

- SecurityConfig centralizzato — unico punto per tutte le regole
- `@EnableWebSecurity` + `@EnableMethodSecurity` + `@Configuration`
- `jwtFilter` è un `@Component` iniettato via costruttore
- `customOAuth2UserService` gestisce la creazione/ricerca utenti OAuth2
- `Role.USER` è il default per tutti i nuovi utenti
- `Role.ADMIN` va assegnato manualmente (DB o script)
