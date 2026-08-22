---
topic: "Symfony — Twig e Security"
tags: [symfony, twig, template, security, authentication, firewall, voter]
nav_prev: "[[Doctrine e Database.md]]"
---

Riferimento ufficiale: [twig.symfony.com](https://twig.symfony.com/) | [symfony.com/doc/current/security.html](https://symfony.com/doc/current/security.html)

Twig è il template engine di Symfony (e standard de facto PHP). Compila template in PHP cached e supporta template inheritance, block, macro, e filtri. Il Security component gestisce autenticazione (firewall), autorizzazione (voter), e user provider.

## Twig — Sintassi di base

```twig
{# templates/base.html.twig #}
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My App{% endblock %}</title>
    {% block stylesheets %}{% endblock %}
</head>
<body>
    <header>
        {% if app.user %}
            <p>Ciao, {{ app.user.name }}</p>
            <a href="{{ path('app_logout') }}">Logout</a>
        {% else %}
            <a href="{{ path('app_login') }}">Login</a>
        {% endif %}
    </header>

    <main>
        {% block body %}{% endblock %}
    </main>

    {% block javascripts %}{% endblock %}
</body>
</html>
```

```twig
{# templates/user/index.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}Lista Utenti{% endblock %}

{% block body %}
    <h1>{{ 'users.title'|trans }}</h1>

    {% if users is empty %}
        <p>Nessun utente trovato.</p>
    {% else %}
        <table>
            <thead>
                <tr>
                    <th>{{ knp_pagination_sortable(users, 'Nome', 'u.name') }}</th>
                    <th>Email</th>
                    <th>Azioni</th>
                </tr>
            </thead>
            <tbody>
            {% for user in users %}
                <tr>
                    <td>{{ user.name }}</td>
                    <td>{{ user.email }}</td>
                    <td>
                        <a href="{{ path('users_show', {id: user.id}) }}">Mostra</a>

                        {% if is_granted('ROLE_ADMIN') %}
                            <a href="{{ path('users_edit', {id: user.id}) }}">Modifica</a>
                        {% endif %}
                    </td>
                </tr>
            {% else %}
                <tr><td colspan="3">Nessun record</td></tr>
            {% endfor %}
            </tbody>
        </table>

        <div class="pagination">
            {{ knp_pagination_render(users) }}
        </div>
    {% endif %}
{% endblock %}
```

### Template inheritance

```twig
{# base.html.twig — layout principale #}
{% block body %}{% endblock %}

{# user/index.html.twig #}
{% extends 'base.html.twig' %}
{% block body %}...{% endblock %}

{# Con block padre #}
{% block body %}
    {{ parent() }}  {# include contenuto del block genitore #}
    <p>Contenuto aggiuntivo</p>
{% endblock %}
```

### Filtri e funzioni

```twig
{# Filtri #}
{{ user.createdAt|date('d/m/Y H:i') }}
{{ user.name|upper }}
{{ user.email|lower }}
{{ 'Hello %name%'|trans({"%name%": user.name}) }}
{{ '<script>alert(1)</script>'|escape('html') }}  {# automatico #}
{{ markdown_content|markdown_to_html }}

{# Funzioni #}
{{ path('users_index') }}
{{ asset('css/app.css') }}
{{ absolute_url(path('users_show', {id: user.id})) }}

{# Macro (componenti) #}
{% macro input(name, value, type = "text") %}
    <input type="{{ type }}" name="{{ name }}" value="{{ value|e }}">
{% endmacro %}
{{ _self.input('email', user.email, 'email') }}
```

### Form rendering

```twig
{# Form Theme Bootstrap 5 #}
{% form_theme form 'bootstrap_5_layout.html.twig' %}

{{ form_start(form) }}
    {{ form_row(form.name) }}
    {{ form_row(form.email) }}
    {{ form_row(form.password) }}
    {{ form_row(form.submit) }}
{{ form_end(form) }}
```

## Security — Configurazione

```yaml
# config/packages/security.yaml
security:
    password_hashers:
        App\Entity\User: 'auto'  # bcrypt/argon2i automatico

    providers:
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email   # login con email

    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false

        main:
            lazy: true
            provider: app_user_provider
            json_login:           # API login con JSON
                check_path: app_api_login
                username_path: email
                password_path: password

            form_login:           # login web con form
                login_path: app_login
                check_path: app_login
                enable_csrf: true

            logout:
                path: app_logout
                target: app_home

            # Access control via pattern
            access_denied_handler: App\Security\AccessDeniedHandler

    access_control:
        - { path: ^/api/v1/login, roles: PUBLIC_ACCESS }
        - { path: ^/api/v1/public, roles: PUBLIC_ACCESS }
        - { path: ^/api/v1/admin, roles: ROLE_ADMIN }
        - { path: ^/api/v1/user, roles: ROLE_USER }
        - { path: ^/admin, roles: ROLE_ADMIN }
        - { path: ^/profile, roles: ROLE_USER }
        - { path: ^/login, roles: PUBLIC_ACCESS }
```

## Autenticazione API con JWT

```bash
composer require "symfony/security-bundle"
composer require "lexik/jwt-authentication-bundle"
php bin/console config:dump lexik/jwt-authentication
php bin/console lexik:jwt:generate-keypair  # genera chiavi RSA
```

```yaml
# config/packages/lexik_jwt_authentication.yaml
lexik_jwt_authentication:
    secret_key: '%kernel.project_dir%/config/jwt/private.pem'
    public_key: '%kernel.project_dir%/config/jwt/public.pem'
    pass_phrase: '%env(JWT_PASSPHRASE)%'
    token_ttl: 3600
```

```yaml
# config/packages/security.yaml (firewall main)
firewalls:
    main:
        stateless: true
        provider: app_user_provider
        json_login:
            check_path: app_api_login
            username_path: email
            password_path: password
        jwt: ~
```

## Voter (autorizzazione)

```php
<?php
// src/Security/PostVoter.php

namespace App\Security;

use App\Entity\Post;
use App\Entity\User;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\Voter\Voter;

class PostVoter extends Voter
{
    public const EDIT   = "post.edit";
    public const DELETE = "post.delete";
    public const VIEW   = "post.view";

    protected function supports(string $attribute, mixed $subject): bool
    {
        if (!in_array($attribute, [self::EDIT, self::DELETE, self::VIEW])) {
            return false;
        }
        if (!$subject instanceof Post) {
            return false;
        }
        return true;
    }

    protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool
    {
        $user = $token->getUser();

        if (!$user instanceof User) {
            return false;
        }

        // Admin bypass
        if (in_array("ROLE_ADMIN", $user->getRoles())) {
            return true;
        }

        /** @var Post $post */
        $post = $subject;

        return match ($attribute) {
            self::VIEW   => $post->isPublished() || $post->getUser() === $user,
            self::EDIT   => $post->getUser() === $user,
            self::DELETE => $post->getUser() === $user,
            default      => false,
        };
    }
}
```

```php
<?php
// Uso in controller

use Symfony\Component\Security\Core\Authorization\AuthorizationCheckerInterface;

class PostController extends AbstractController
{
    #[Route("/posts/{id}/edit", methods: ["PUT"])]
    public function edit(Post $post): JsonResponse
    {
        $this->denyAccessUnlessGranted(PostVoter::EDIT, $post);

        // ...
    }

    #[Route("/posts/{id}/delete", methods: ["DELETE"])]
    public function delete(Post $post): JsonResponse
    {
        if (!$this->isGranted(PostVoter::DELETE, $post)) {
            return $this->json(["error" => "Forbidden"], 403);
        }

        // ...
    }
}
```

### Annotation in Twig

```twig
{% if is_granted('post.edit', post) %}
    <a href="{{ path('posts_edit', {id: post.id}) }}">Modifica</a>
{% endif %}
```

## Authenticator custom

```php
<?php
// src/Security/ApiKeyAuthenticator.php

namespace App\Security;

use App\Repository\UserRepository;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Http\Authenticator\AbstractAuthenticator;
use Symfony\Component\Security\Http\Authenticator\Passport\Badge\UserBadge;
use Symfony\Component\Security\Http\Authenticator\Passport\Passport;
use Symfony\Component\Security\Http\Authenticator\Passport\SelfValidatingPassport;

class ApiKeyAuthenticator extends AbstractAuthenticator
{
    public function __construct(
        private readonly UserRepository $userRepository,
    ) {}

    public function supports(Request $request): ?bool
    {
        return $request->headers->has("X-API-Key");
    }

    public function authenticate(Request $request): Passport
    {
        $apiKey = $request->headers->get("X-API-Key");

        return new SelfValidatingPassport(
            new UserBadge($apiKey, function (string $key) {
                return $this->userRepository->findOneBy(["apiKey" => $key]);
            })
        );
    }

    public function onAuthenticationSuccess(Request $request, TokenInterface $token, string $firewallName): ?Response
    {
        return null;  // continua la richiesta
    }

    public function onAuthenticationFailure(Request $request, AuthenticationException $exception): ?Response
    {
        return new JsonResponse(["error" => "API Key non valida"], 401);
    }
}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Unable to find template "user/index.html.twig"` | File twig mancante o path sbagliato | Crea `templates/user/index.html.twig` |
| `Variable "users" does not exist` | Variabile non passata al template | `return $this->render("user/index.html.twig", ["users" => $users])` |
| `The security firewall was not configured` | `security.yaml` non configurato | Definisci firewall in `config/packages/security.yaml` |
| `Full authentication is required to access this resource` | Utente non autenticato su route protetta | Usa `PUBLIC_ACCESS` o reindirizza a login |
| `Access Denied` (403) | Voter nega autorizzazione | Verifica `voteOnAttribute()` o `access_control` |
| `The controller must return a Response` | Controller non restituisce Response | Usa `$this->json()` o `$this->render()` |
| `CSRF token is invalid` | Form senza CSRF o token scaduto | `form_start(form)` include CSRF automaticamente |
| `JWT token not found` | Token JWT non inviato in Authorization header | `Authorization: Bearer {token}` |

## Best practice

- **Twig auto-escaping** — `{{ }}` escapa HTML automaticamente; mai `{{ var|raw }}` su dati utente
- **`is_granted()` in template** — mostra/nascondi UI per ruolo, mai dati sensibili
- **Voter per logica di autorizzazione complessa** — non mettere `if ($post->getUser() === $user)` nei controller
- **`denyAccessUnlessGranted()`** — più leggibile di `if (!$this->isGranted()) throw ...`
- **Firewall stateless per API** — JWT bearer, senza sessione
- **`app.user` in template** — utente corrente disponibile globalmente in Twig
- **Form theme per UI coerente** — Bootstrap 5 o Tailwind layout via `form_theme`
- **Traduzioni via `|trans`** — mai hardcodare stringhe UI
- **CSRF protection su form** — abilitata di default; disabilita solo per API stateless
- **Password hasher `auto`** — sceglie algoritmo migliore disponibile (bcrypt/argon2i)

## Cross-reference

- [[PHP/Symfony/Doctrine e Database|Symfony — Doctrine e Database]] — User entity, repository
- [[PHP/Web/Sessioni e Cookie|Sessioni e Cookie]] — session-based auth
- [[PHP/Laravel/Blade e Template|Laravel — Blade e Template]] — confronto template engine
- [[PHP/Core Concepts/Classi e OOP|Classi e OOP]] — Voter class, interfacce UserInterface
