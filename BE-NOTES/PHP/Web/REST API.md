---
topic: "REST API — PHP"
tags: [php, web, rest, api, routing, json, http]
nav_prev: "[[Sessioni e Cookie.md]]"
---

Riferimento ufficiale: [php.net/manual/en/function.header.php](https://www.php.net/manual/en/function.header.php) | [json_encode](https://www.php.net/manual/en/function.json-encode.php) | [PSR-7](https://www.php-fig.org/psr/psr-7/)

PHP nativo può servire API REST senza framework (utile per micro-progetti, prototipi, o understanding). In produzione, usa Laravel o Symfony. Qui il pattern per routing manuale, pars JSON, CORS, e struttura response.

## Entry point: front controller

```
project/
├── public/
│   └── index.php        # front controller (tutte le richieste)
├── src/
│   ├── Router.php
│   ├── Controllers/
│   │   └── UserController.php
│   ├── Services/
│   │   └── UserService.php
│   └── Mappers/
│       └── UserMapper.php
└── .htaccess / nginx.conf
```

### .htaccess (Apache) — rewrite tutto a index.php

```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ index.php [QSA,L]
```

### Nginx

```nginx
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

## Front controller pattern

```php
<?php
// public/index.php

declare(strict_types=1);

require_once __DIR__ . "/../vendor/autoload.php";

use App\Router;
use App\Controllers\UserController;

header("Content-Type: application/json; charset=UTF-8");

// CORS
header("Access-Control-Allow-Origin: https://example.com");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS");
header("Access-Control-Allow-Headers: Content-Type, Authorization");

// Preflight
if ($_SERVER["REQUEST_METHOD"] === "OPTIONS") {
    http_response_code(204);
    exit;
}

// Routing
$router = new Router();
$router->get("/users",         [UserController::class, "index"]);
$router->get("/users/:id",     [UserController::class, "show"]);
$router->post("/users",        [UserController::class, "create"]);
$router->put("/users/:id",     [UserController::class, "update"]);
$router->delete("/users/:id",  [UserController::class, "delete"]);

$router->dispatch($_SERVER["REQUEST_METHOD"], $_SERVER["REQUEST_URI"]);
```

## Router minimale

```php
<?php
// src/Router.php

namespace App;

class Router
{
    private array $routes = [];

    public function get(string $path, array $handler): void
    {
        $this->routes["GET"][] = ["path" => $path, "handler" => $handler];
    }

    public function post(string $path, array $handler): void
    {
        $this->routes["POST"][] = ["path" => $path, "handler" => $handler];
    }

    public function put(string $path, array $handler): void
    {
        $this->routes["PUT"][] = ["path" => $path, "handler" => $handler];
    }

    public function delete(string $path, array $handler): void
    {
        $this->routes["DELETE"][] = ["path" => $path, "handler" => $handler];
    }

    public function dispatch(string $method, string $uri): void
    {
        $uriPath = parse_url($uri, PHP_URL_PATH);
        $routes  = $this->routes[$method] ?? [];

        foreach ($routes as $route) {
            $pattern = preg_replace("/:(\w+)/", "(?P<$1>[^/]+)", $route["path"]);
            $pattern = "#^" . $pattern . "$#";

            if (preg_match($pattern, $uriPath, $matches)) {
                [$class, $action] = $route["handler"];
                $params = array_filter($matches, "is_string", ARRAY_FILTER_USE_KEY);
                $controller = new $class();

                try {
                    $result = $controller->$action($params);
                    echo json_encode($result, JSON_UNESCAPED_UNICODE);
                } catch (\Throwable $e) {
                    http_response_code(500);
                    echo json_encode(["error" => $e->getMessage()]);
                }
                return;
            }
        }

        http_response_code(404);
        echo json_encode(["error" => "Not Found"]);
    }
}
```

## Controller

```php
<?php
// src/Controllers/UserController.php

namespace App\Controllers;

use App\Services\UserService;
use App\Mappers\UserMapper;

class UserController
{
    public function __construct(
        private readonly UserService $userService = new UserService(),
    ) {}

    public function index(): array
    {
        $users = $this->userService->findAll();
        return [
            "data" => array_map([UserMapper::class, "toResponse"], $users),
            "meta" => ["count" => count($users)],
        ];
    }

    public function show(array $params): array
    {
        $id   = (int) ($params["id"] ?? 0);
        $user = $this->userService->findById($id);

        if (!$user) {
            http_response_code(404);
            return ["error" => "User not found"];
        }

        return ["data" => UserMapper::toResponse($user)];
    }

    public function create(): array
    {
        $body = json_decode(file_get_contents("php://input"), true);

        if (!$body || empty($body["email"])) {
            http_response_code(422);
            return ["error" => "Email is required"];
        }

        $user = $this->userService->create($body);

        http_response_code(201);
        return ["data" => UserMapper::toResponse($user)];
    }

    public function update(array $params): array
    {
        $id   = (int) ($params["id"] ?? 0);
        $body = json_decode(file_get_contents("php://input"), true);

        $user = $this->userService->update($id, $body ?? []);

        if (!$user) {
            http_response_code(404);
            return ["error" => "User not found"];
        }

        return ["data" => UserMapper::toResponse($user)];
    }

    public function delete(array $params): array
    {
        $id     = (int) ($params["id"] ?? 0);
        $deleted = $this->userService->delete($id);

        if (!$deleted) {
            http_response_code(404);
            return ["error" => "User not found"];
        }

        http_response_code(204);
        return [];
    }
}
```

## Service e Repository

```php
<?php
// src/Services/UserService.php

namespace App\Services;

use App\Repositories\UserRepository;
use App\Mappers\UserMapper;

class UserService
{
    public function __construct(
        private readonly UserRepository $repository = new UserRepository(),
    ) {}

    public function findAll(): array
    {
        return $this->repository->findAll();
    }

    public function findById(int $id): ?array
    {
        return $this->repository->findById($id);
    }

    public function create(array $data): array
    {
        $this->validate($data);

        if ($this->repository->findByEmail($data["email"])) {
            http_response_code(409);
            throw new \RuntimeException("Email already exists");
        }

        $data["password"] = password_hash($data["password"], PASSWORD_BCRYPT);
        return $this->repository->create($data);
    }

    public function update(int $id, array $data): ?array
    {
        $user = $this->repository->findById($id);
        if (!$user) return null;

        if (isset($data["email"]) && $data["email"] !== $user["email"]) {
            if ($this->repository->findByEmail($data["email"])) {
                throw new \RuntimeException("Email already exists");
            }
        }

        return $this->repository->update($id, $data);
    }

    public function delete(int $id): bool
    {
        return $this->repository->delete($id);
    }

    private function validate(array $data): void
    {
        $errors = [];

        if (empty($data["email"]) || !filter_var($data["email"], FILTER_VALIDATE_EMAIL)) {
            $errors["email"] = "Valid email is required";
        }
        if (empty($data["password"]) || strlen($data["password"]) < 8) {
            $errors["password"] = "Password must be at least 8 characters";
        }

        if ($errors) {
            http_response_code(422);
            throw new \RuntimeException(json_encode($errors));
        }
    }
}
```

### Repository pattern

```php
<?php
// src/Repositories/UserRepository.php

namespace App\Repositories;

use App\Database\Connection;

class UserRepository
{
    public function __construct(
        private readonly \PDO $db = Connection::getInstance(),
    ) {}

    public function findAll(): array
    {
        $stmt = $this->db->query("SELECT * FROM users ORDER BY id");
        return $stmt->fetchAll(\PDO::FETCH_ASSOC);
    }

    public function findById(int $id): ?array
    {
        $stmt = $this->db->prepare("SELECT * FROM users WHERE id = ?");
        $stmt->execute([$id]);
        $result = $stmt->fetch(\PDO::FETCH_ASSOC);
        return $result ?: null;
    }

    public function findByEmail(string $email): ?array
    {
        $stmt = $this->db->prepare("SELECT * FROM users WHERE email = ?");
        $stmt->execute([$email]);
        $result = $stmt->fetch(\PDO::FETCH_ASSOC);
        return $result ?: null;
    }

    public function create(array $data): array
    {
        $stmt = $this->db->prepare(
            "INSERT INTO users (name, email, password) VALUES (?, ?, ?)"
        );
        $stmt->execute([$data["name"], $data["email"], $data["password"]]);
        return $this->findById((int) $this->db->lastInsertId());
    }

    public function update(int $id, array $data): ?array
    {
        $fields = [];
        $values = [];

        foreach (["name", "email", "password"] as $field) {
            if (isset($data[$field])) {
                $fields[] = "$field = ?";
                $values[] = $data[$field];
            }
        }

        if (empty($fields)) return $this->findById($id);

        $values[] = $id;
        $stmt = $this->db->prepare(
            "UPDATE users SET " . implode(", ", $fields) . " WHERE id = ?"
        );
        $stmt->execute($values);

        return $this->findById($id);
    }

    public function delete(int $id): bool
    {
        $stmt = $this->db->prepare("DELETE FROM users WHERE id = ?");
        $stmt->execute([$id]);
        return $stmt->rowCount() > 0;
    }
}
```

## Mapper (DTO → Response)

```php
<?php
// src/Mappers/UserMapper.php

namespace App\Mappers;

class UserMapper
{
    public static function toResponse(array $user): array
    {
        return [
            "id"        => (int) $user["id"],
            "name"      => $user["name"],
            "email"     => $user["email"],
            "createdAt" => $user["created_at"],
            // Esclude: password, updated_at
        ];
    }

    public static function toDTO(array $body): array
    {
        return [
            "name"     => $body["name"] ?? "",
            "email"    => $body["email"] ?? "",
            "password" => $body["password"] ?? "",
        ];
    }

    public static function toList(array $users): array
    {
        return array_map([self::class, "toResponse"], $users);
    }
}
```

## Content negotiation

```php
<?php

// Determina formato response in base all'Accept header
$accept = $_SERVER["HTTP_ACCEPT"] ?? "application/json";

match (true) {
    str_contains($accept, "application/json") => serveJson($data),
    str_contains($accept, "text/xml")         => serveXml($data),
    default                                   => serveJson($data),  // default
};
```

## Error handling centralizzato

```php
<?php

// In index.php
set_exception_handler(function (\Throwable $e): void {
    http_response_code($e->getCode() >= 100 && $e->getCode() < 600
        ? $e->getCode()
        : 500);

    echo json_encode([
        "error"   => $e->getMessage(),
        "code"    => $e->getCode(),
        "trace"   => $_ENV["APP_ENV"] === "dev"
            ? $e->getTraceAsString()
            : null,
    ], JSON_UNESCAPED_UNICODE);
});
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `header()` already sent | Output prima di header() | Nessun echo/html prima di header(); usa output buffering |
| CORS block (preflight fallisce) | OPTIONS non gestito | Rispondi a OPTIONS con 204 e CORS header |
| `json_decode` restituisce null | JSON malformato | Verifica con `json_last_error()` dopo decode |
| `$_POST` vuoto per PUT/PATCH | PHP non pars multipart/form-data per PUT | `file_get_contents("php://input")` per body raw |
| Route non matcha | Pattern regex sbagliato | Testa regex con `preg_match()` separatamente |
| 404 per tutte le route | Rewrite rule mancante | Verifica .htaccess o nginx conf |
| Errore 500 senza messaggio | `display_errors=Off` ma exception non catturata | `set_exception_handler()` o catch global |
| Response JSON non parsabile | BOM UTF-8 o whitespace prima di `<?php` | Nessun carattere prima del PHP tag |

## Best practice

- **Front controller** — unico entry point (`index.php`), rewrite tutto via .htaccess/nginx
- **Service layer** — logica di business fuori dal controller; controller gestisce solo HTTP
- **Repository pattern** — accesso dati isolato in repository; service non sa di SQL
- **Mapper DTO** — mai esporre password o campi interni nella response JSON
- **Input validation** — validare prima di processare; `filter_var()`, regex o libreria (symfony/validator)
- **Error handling globale** — exception handler centralizzato in index.php
- **CORS configurabile per ambiente** — env-specific origin, non `*` in produzione
- **Content negotiation** — Accept header → formato response (JSON default)
- **Rate limiting per IP** — middleware base su REMOTE_ADDR + storage
- **Versioning via URL prefix** — `/api/v1/users`, non header

## Cross-reference

- [[PHP/Web/Superglobali|Superglobali]] — $_SERVER, $_GET, php://input
- [[PHP/Web/Sessioni e Cookie|Sessioni e Cookie]] — token vs session authentication
- [[PHP/Database/PDO|PDO]] — database layer per repository
- [[PHP/Laravel/Routing e Controller|Laravel — Routing e Controller]] — alternativa framework
- [[PHP/Core Concepts/Namespace e Autoload|Namespace e Autoload]] — PSR-4 autoload per src/
