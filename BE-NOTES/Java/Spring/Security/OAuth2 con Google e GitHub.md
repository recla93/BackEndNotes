---
topic: "OAuth2 con Google e GitHub"
parent: "[[BE-NOTES/Java/Spring/Security/Spring Security|Spring Security]]"
nav_prev: "[[JWT - Generazione e Validazione.md]]"
nav_next: "[[Authorities e RBAC.md]]"
---


L'OAuth2 login sociale permette agli utenti di autenticarsi con un account esistente (Google, GitHub) invece di creare username/password. [[TaskMngr]] supporta Google (tramite OIDC) e GitHub (tramite OAuth2 standard). Gli utenti possono anche collegare più account social allo stesso profilo.

## Quando usare login sociale

- **Ridurre l'attrito di registrazione** — l'utente non deve compilare form, clicca un bottone
- **Delegare sicurezza a provider esterni** — Google gestisce MFA, password recovery, brute force protection
- **Accesso rapido** — ideale per MVP, demo o app che non richiedono account proprietario
- **Linked accounts** — utente può unire più account social in un unico profilo

**Quando NON usarlo:**
- **App B2B aziendali** — spesso richiedono SSO aziendale (SAML, OIDC provider specifico)
- **Utenti senza account social** — serve comunque auth tradizionale come fallback
- **GDPR/privacy** — attenzione a quali dati richiedi (Google dà email + nome, GitHub dà email pubblica)

## Configurazione (application-prod.yml)

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope: openid, profile, email
          github:
            client-id: ${GITHUB_CLIENT_ID}
            client-secret: ${GITHUB_CLIENT_SECRET}
            scope: user:email, read:user
```

I `client-id` e `client-secret` vengono dalle rispettive console sviluppatori:
- Google: https://console.cloud.google.com/apis/credentials
- GitHub: https://github.com/settings/developers

**Non committarli mai** — sempre in `.env` o variabili d'ambiente.

## Flusso completo

```
Frontend                    Backend                  Google/GitHub
   │                          │                          │
   ├── "Login con Google" ───►│                          │
   │                          ├── redirect 302 ─────────►│
   │◄─── redirect a Google ───┤                          │
   │                                                      │
   │  ──── l'utente fa login su Google ──────────────────►│
   │◄───────────────────── redirect con code ────────────┤
   │                                                      │
   │  ─── POST /login/oauth2/code/google?code=... ──────►│
   │                          │                          │
   │                          ├── scambia code ─────────►│
   │                          │◄─── access token ────────┤
   │                          │                          │
   │                          ├── carica user info ─────►│
   │                          │◄── nome, email, foto ────┤
   │                          │                          │
   │                          ├── cerca/crea utente DB   │
   │                          ├── emette JWT             │
   │◄─── JWT (login OK) ─────┤                          │
```

## CustomOAuth2UserService

Spring Security carica i dettagli dell'utente OAuth2 tramite un `OAuth2UserService`. In TaskMngr, lo personalizziamo per gestire la creazione automatica dell'utente:

```java
@Component
public class CustomOAuth2UserService extends DefaultOAuth2UserService {

    @Override
    public OAuth2User loadUser(OAuth2UserRequest request) {
        OAuth2User oauthUser = super.loadUser(request);

        String provider = request.getClientRegistration().getRegistrationId();  // "google" o "github"
        String providerId = oauthUser.getName();
        String email = oauthUser.getAttribute("email");
        String name = oauthUser.getAttribute("name");

        // Cerca utente per email o linked account
        User user = userRepository.findByEmail(email)
            .orElseGet(() -> createUser(email, name));

        // Collega l'account social se non già collegato
        linkAccount(user, provider, providerId);

        // Emetti JWT (deviato al success handler)
        return oauthUser;
    }
}
```

## Linked Accounts

Un utente può collegare Google **e** GitHub allo stesso profilo:

```java
@Entity
public class LinkedAccount {
    @ManyToOne
    private User user;

    private String provider;       // "google" o "github"
    private String providerId;     // ID univoco del provider
    private String email;
}
```

**Regole di linking:**
1. Se l'email del social matcha un utente → login + link automatico
2. Se l'email NON matcha → nuova registrazione (crea utente + linked account)
3. Utente già loggato può linkare un altro social dal proprio profilo

## Success Handler

Dopo l'autenticazione OAuth2, invece del default redirect a `/`, TaskMngr restituisce un JWT:

```java
@Component
public class OAuth2SuccessHandler extends SimpleUrlAuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request,
                                        HttpServletResponse response,
                                        Authentication authentication) {

        User user = (User) authentication.getPrincipal();
        String jwt = jwtService.generateToken(user);

        // Reindirizza il frontend con il token nell'URL fragment
        response.sendRedirect("http://localhost:4200/oauth2/redirect#token=" + jwt);
    }
}
```

Il frontend estrae il token dal fragment e lo usa per le richieste successive.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Client secret hardcodato in application.yml | Secret in VCS, esposizione pubblica | Le credenziali OAuth2 finite in VCS | Usa `${GOOGLE_CLIENT_ID}` e `${GOOGLE_CLIENT_SECRET}` placeholder con variabili d'ambiente |
| Callback URL non registrata | `400 redirect_uri_mismatch` dal provider OAuth | Google/GitHub hanno la URI una specifica registrata | Registra in console sviluppatori la callback esatta (es. `http://localhost:8080/login/oauth2/code/google`) |
| Scope richiesto non autorizzato | Login OAuth2 funziona ma dati utente mancanti | Lo scope richiesto non è stato approvato dal provider | Allinea gli scope con ciò che effettivamente ti serve; chiedi solo scope necessari |
| Non gestire il linking account esistente | Utente bloccato se email già usata | Login social con email che matcha un account esistente senza collegamento | Controlla email prima di creare nuovo utente: se esiste, collega account sociale |
| Mancata gestione errore OAuth2 | Redirect a pagina di errore generica | L'utente annulla il consenso o il provider restituisce errore | Implementa `OAuth2AuthenticationFailureHandler` per redirect a pagina FE con messaggio |

## In TaskMngr

- Google e GitHub come provider OAuth2
- `CustomOAuth2UserService` per registrazione/login automatico
- `OAuth2SuccessHandler` per restituire JWT al frontend
- `LinkedAccount` entità per utenti multi-provider
- Token JWT emesso dopo login sociale (stesso meccanismo del login tradizionale)
