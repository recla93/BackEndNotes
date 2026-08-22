---
topic: "Superglobali — PHP"
tags: [php, web, superglobal, http, request, server]
nav_prev: "[[PHP.md]]"
nav_next: "[[Sessioni e Cookie.md]]"
---

Riferimento ufficiale: [php.net/manual/en/language.variables.superglobals.php](https://www.php.net/manual/en/language.variables.superglobals.php) | [php.net/manual/en/reserved.variables.php](https://www.php.net/manual/en/reserved.variables.php)

Le **superglobali** sono array associativi built-in sempre disponibili in ogni scope (funzione, metodo, closure). PHP le popola automaticamente a ogni richiesta HTTP. Contengono dati di input, server, ambiente e sessione.

```php
<?php

// Sempre disponibili — nessun bisogno di global
function esempio(): void {
    echo $_SERVER["REQUEST_METHOD"];  // funziona anche dentro una funzione
}
```

## $_GET — query string (URL)

```php
<?php

// URL: /users?page=2&limit=10&filter=active
$page   = $_GET["page"]   ?? 1;    // "2" (string!)
$limit  = $_GET["limit"]  ?? 10;   // "10" (string!)
$filter = $_GET["filter"] ?? null; // "active"

// Array nella query: /search?tags[]=php&tags[]=laravel
$tags = $_GET["tags"] ?? [];       // ["php", "laravel"]
```

`$_GET` contiene sempre **stringhe**. Converti esplicitamente: `(int) $_GET["page"]`; `declare(strict_types=1)` non aiuta qui perché non passa da parametro di funzione.

## $_POST — corpo richiesta (application/x-www-form-urlencoded o multipart/form-data)

```php
<?php

// Form HTML con method="POST"
$email    = $_POST["email"]    ?? "";
$password = $_POST["password"] ?? "";

// JSON body — NON popola $_POST (serve lettura manuale)
$json   = file_get_contents("php://input");
$data   = json_decode($json, true);
$nome   = $data["nome"] ?? "";
```

`$_POST` è popolato solo per `application/x-www-form-urlencoded` e `multipart/form-data`. Per JSON, GraphQL, XML: usa `php://input`.

## $_SERVER — metadati della richiesta

```php
<?php

// Metodo HTTP
$method = $_SERVER["REQUEST_METHOD"];         // GET, POST, PUT, DELETE, ...

// URL e percorso
$uri     = $_SERVER["REQUEST_URI"];           // /users?page=2
$path    = parse_url($uri, PHP_URL_PATH);     // /users
$host    = $_SERVER["HTTP_HOST"];             // example.com
$scheme  = $_SERVER["REQUEST_SCHEME"];        // http, https
$port    = $_SERVER["SERVER_PORT"];           // 80, 443, 8080

// Client
$ip      = $_SERVER["REMOTE_ADDR"];           // IP del client
$agent   = $_SERVER["HTTP_USER_AGENT"];       // User-Agent header
$referer = $_SERVER["HTTP_REFERER"] ?? "";    // HTTP Referer header

// Script
$self    = $_SERVER["PHP_SELF"];              // /index.php
$docRoot = $_SERVER["DOCUMENT_ROOT"];         // C:/www/myapp
```

`$_SERVER` è popolato dal web server (Apache, Nginx, Caddy). I valori con prefisso `HTTP_` sono header HTTP. Attenzione a spoofing: `REMOTE_ADDR` è l'IP TCP, `HTTP_X_FORWARDED_FOR` è l'header (falsificabile).

## $_FILES — file caricati

```php
<?php

// Form: <input type="file" name="avatar">
$file = $_FILES["avatar"];

$nome    = $file["name"];       // "foto.jpg" (nome originale)
$tmpPath = $file["tmp_name"];   // "C:\tmp\phpABC123" (percorso temporaneo)
$error   = $file["error"];      // UPLOAD_ERR_OK (0)
$size    = $file["size"];       // 123456 (bytes)
$type    = $file["type"];       // "image/jpeg" (MIME dichiarato dal client)

// Spostamento in posizione definitiva
if ($error === UPLOAD_ERR_OK && move_uploaded_file($tmpPath, "uploads/$nome")) {
    echo "File salvato";
}
```

`$_FILES["name"]` è il nome originale (non fidarti per il salvataggio — usa un nome generato). `$_FILES["type"]` è dichiarato dal browser, va validato lato server con `finfo()`.

## $_COOKIE — cookie del client

```php
<?php

$preferenza = $_COOKIE["theme"] ?? "light";

// Impostare un cookie (invierà header Set-Cookie)
setcookie("theme", "dark", time() + 86400 * 30, "/", "", true, true);
// httponly=true → inaccessibile da JS
// secure=true   → solo HTTPS
```

I cookie in `$_COOKIE` sono **solo lettura**: contengono i cookie inviati dal client. Per impostare cookie, usa `setcookie()`. `$_COOKIE` è popolato all'inizio della richiesta; `setcookie()` non modifica `$_COOKIE` nella richiesta corrente.

## $_SESSION — dati di sessione

```php
<?php

session_start();  // avvia la sessione (deve essere prima di qualsiasi output)

$_SESSION["user_id"] = 42;
$_SESSION["roles"]   = ["admin", "editor"];

$userId = $_SESSION["user_id"] ?? null;

// Distrugge sessione
session_destroy();
```

`$_SESSION` richiede `session_start()` esplicito. I dati sono serializzati su server (file, DB, Redis). Non va usato per dati voluminosi.

## $_REQUEST — merge di $_GET, $_POST, $_COOKIE

```php
<?php

// $_REQUEST unisce GET + POST + COOKIE (ordine dipende da request_order in php.ini)
$valore = $_REQUEST["key"] ?? null;

// ⚠ Non usare $_REQUEST in produzione: nasconde la provenienza del dato
```

`$_REQUEST` è comodo ma ambiguo: non sai se il dato viene da GET, POST o COOKIE. Preferisci `$_GET`, `$_POST`, `$_COOKIE` espliciti.

## $_ENV — variabili d'ambiente

```php
<?php

// Popolato se php.ini ha variables_order = "EGPCS" (include E)
$dbHost = $_ENV["DB_HOST"] ?? "localhost";

// Alternativa moderna
$dbHost = getenv("DB_HOST") ?: "localhost";

// $_SERVER contiene anche variabili d'ambiente (se E in variables_order)
$dbHost = $_SERVER["DB_HOST"] ?? "localhost";
```

`$_ENV` non è sempre popolato. Dipende da `variables_order` in `php.ini`. Preferisci `getenv()` o librerie come `vlucas/phpdotenv` per progetti moderni.

## $_GLOBALS — variabili globali

```php
<?php

$config = ["db" => ["host" => "localhost"]];

function debug(): void {
    // $_GLOBALS contiene tutte le variabili globali
    var_dump($GLOBALS["config"]);
}
```

Usa `$_GLOBALS` solo in scenari legacy o debug. In codice moderno, preferisci dependency injection o `use` nelle closure.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Undefined array key "page"` | Accesso a `$_GET["page"]` senza `??` default | Usa `??` — `$_GET["page"] ?? 1` |
| `$_POST` vuoto per JSON | PHP non pars JSON body automaticamente | Leggi con `file_get_contents("php://input")` |
| `move_uploaded_file` fallisce | Directory destinazione non scrivibile | Verifica permessi e usa percorso assoluto |
| `Cannot modify header information` | Output prima di `setcookie()`/`session_start()` | Buffer output o sposta prima di qualsiasi echo |
| `$_FILES` ha `error === UPLOAD_ERR_NO_FILE` | Input file vuoto nel form | Controlla `$_FILES["field"]["error"]` prima di processare |
| `session_start()` non trova la sessione | Cookie di sessione non inviato dal client (o scaduto) | Verifica `session_set_cookie_params()` |
| `Indirect modification of overloaded element` | Tentativo di modificare `$_SESSION["x"]` come array ma non inizializzato | Inizializza `$_SESSION["x"] = []` prima di push |

## Best practice

- **Mai fidarsi di input utente** — valida e sanifica sempre (`filter_input()`, `htmlspecialchars()`, prepared statement)
- **`??` su ogni accesso** — evita `Undefined array key` e stabilisce default espliciti
- **`filter_input()` per validazione** — `filter_input(INPUT_GET, "email", FILTER_VALIDATE_EMAIL)` è più sicuro di `$_GET["email"]` diretto
- **Non usare $_REQUEST** — ambiguità sulla fonte del dato
- **JSON body → `php://input`** — non aspettarti $_POST per API REST moderne
- **File upload: mai fidarsi di `name` e `type`** — genera nome con `uniqid()` e valida MIME con `finfo()`
- **`htmlspecialchars()` in output** — prevenzione XSS: `echo htmlspecialchars($nome, ENT_QUOTES, 'UTF-8')`
- **`variables_order = "EGPCS"`** in php.ini se vuoi $_ENV popolato

## Cross-reference

- [[PHP/Web/Sessioni e Cookie|Sessioni e Cookie]] — gestione stato con session_start, setcookie
- [[PHP/Web/REST API|REST API]] — routing senza framework, gestione input/output
- [[PHP/Core Concepts/Gestione Errori|Gestione Errori]] — gestire Undefined array key con try/catch
- [[PHP/Core Concepts/Stringhe|Stringhe]] — sanificazione output, htmlspecialchars
- [[PHP/Strumenti/Composer e Autoload|Composer]] — librerie per validazione (respect/validation, symfony/validator)
