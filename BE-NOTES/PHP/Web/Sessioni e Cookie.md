---
topic: "Sessioni e Cookie — PHP"
tags: [php, web, session, cookie, state, security]
nav_prev: "[[Superglobali.md]]"
nav_next: "[[REST API.md]]"
---

Riferimento ufficiale: [php.net/manual/en/book.session.php](https://www.php.net/manual/en/book.session.php) | [php.net/manual/en/features.cookies.php](https://www.php.net/manual/en/features.cookies.php)

Il web è **stateless** per natura (HTTP). Sessioni e cookie forniscono **persistenza dello stato** tra richieste successive dello stesso utente. I cookie sono lato client (chiavi-valore nel browser), le sessioni sono lato server con un identificativo nel cookie.

## Cookie — dati sul client

```php
<?php

// Impostare un cookie
setcookie(
    name: "theme",           // nome
    value: "dark",           // valore (string)
    expires_or_options: time() + 86400 * 30,  // scadenza (timestamp Unix; 0 = sessione)
    path: "/",               // percorso (tutto il dominio)
    domain: "",              // dominio (vuoto = corrente)
    secure: true,            // solo HTTPS
    httponly: true,          // inaccessibile da JavaScript
);

// Cookie array
setcookie("cart[item_1]", "42", time() + 3600);
setcookie("cart[item_2]", "17", time() + 3600);

// Lettura (solo richieste successive)
echo $_COOKIE["theme"] ?? "light";
```

`setcookie()` prima di qualsiasi output HTML (altrimenti "Cannot modify header information"). Usa **httponly** per prevenire XSS che ruba cookie. Usa **secure** per evitare intercettazione su HTTP.

### Parametri di sicurezza cookie

```php
<?php

// PHP 7.3+ — array associativo
setcookie("token", $value, [
    "expires"  => time() + 3600,
    "path"     => "/",
    "domain"   => ".example.com",
    "secure"   => true,
    "httponly" => true,
    "samesite" => "Strict",   // Lax (default), Strict, None
]);

// SameSite:
// - Strict → non inviato su richieste cross-site (max protezione CSRF)
// - Lax    → inviato su navigazione GET top-level (default browser moderni)
// - None   → inviato sempre (richiede secure)
```

### Cancellare un cookie

```php
<?php

// Imposta scadenza nel passato
setcookie("theme", "", time() - 3600);
```

## Sessioni — dati sul server

```php
<?php

// 1. Avvia sessione (PRIMA di qualsiasi output)
session_start();

// 2. Legge/Scrive dati
$_SESSION["user_id"] = 42;
$_SESSION["cart"] = [
    ["product_id" => 1, "qty" => 2],
    ["product_id" => 3, "qty" => 1],
];

$userId = $_SESSION["user_id"] ?? null;

// 3. Elimina dati specifici
unset($_SESSION["cart"]);

// 4. Distrugge sessione
session_destroy();
```

PHP invia automaticamente un cookie `PHPSESSID` al primo `session_start()`. I dati di sessione sono serializzati su file (default) o su storage configurato (DB, Redis).

### Configurazione sessione

```php
<?php

// Prima di session_start()
ini_set("session.name", "SID");                    // nome cookie (default: PHPSESSID)
ini_set("session.cookie_lifetime", 0);              // 0 = fino a chiusura browser
ini_set("session.cookie_secure", 1);                // solo HTTPS
ini_set("session.cookie_httponly", 1);              // non accessibile da JS
ini_set("session.cookie_samesite", "Strict");       // SameSite
ini_set("session.gc_maxlifetime", 3600);            // durata dati su server (secondi)
ini_set("session.use_strict_mode", 1);              // rifiuta session ID non inizializzati
ini_set("session.use_cookies", 1);                  // usa cookie per trasporto SID
ini_set("session.use_only_cookies", 1);             // solo cookie (no URL)
```

Alternativa moderna in PHP 7.3+:

```php
<?php

session_set_cookie_params([
    "lifetime" => 0,
    "path"     => "/",
    "domain"   => "",
    "secure"   => true,
    "httponly" => true,
    "samesite" => "Strict",
]);
session_start();
```

### Session handler personalizzato

```php
<?php

// Usa Redis per sessioni (esempio con predis)
// composer require predis/predis
use Predis\Client;

class RedisSessionHandler implements SessionHandlerInterface {
    private Client $redis;
    private int $ttl;

    public function __construct() {
        $this->redis = new Client("tcp://localhost:6379");
        $this->ttl   = (int) (ini_get("session.gc_maxlifetime") ?: 3600);
    }

    public function open(string $path, string $name): bool { return true; }
    public function close(): bool { return true; }

    public function read(string $id): string {
        return $this->redis->get("session:$id") ?? "";
    }

    public function write(string $id, string $data): bool {
        $this->redis->setex("session:$id", $this->ttl, $data);
        return true;
    }

    public function destroy(string $id): bool {
        $this->redis->del("session:$id");
        return true;
    }

    public function gc(int $max_lifetime): int { return 0; }
}

session_set_save_handler(new RedisSessionHandler(), true);
session_start();
```

### Flash messages (dati monouso)

```php
<?php

// Imposta messaggio flash
$_SESSION["_flash"]["success"] = "Operazione completata";

// Nella richiesta successiva
$message = $_SESSION["_flash"]["success"] ?? null;
unset($_SESSION["_flash"]["success"]);  // consuma — monouso

// Helper
function flash(string $key, ?string $value = null): ?string {
    if ($value !== null) {
        $_SESSION["_flash"][$key] = $value;
        return null;
    }
    $msg = $_SESSION["_flash"][$key] ?? null;
    unset($_SESSION["_flash"][$key]);
    return $msg;
}
```

### Rigenerazione ID sessione

```php
<?php

session_start();

// Dopo login — previene session fixation
if ($password_verificata) {
    session_regenerate_id(true);  // true = elimina vecchia sessione
    $_SESSION["user_id"] = $user->id;
}

// Periodicamente (es. ogni 30 minuti)
if (!isset($_SESSION["_last_regenerated"]) || $_SESSION["_last_regenerated"] < time() - 1800) {
    session_regenerate_id(true);
    $_SESSION["_last_regenerated"] = time();
}
```

## Differenze chiave: cookie vs sessione

| Aspetto | Cookie | Sessione |
|---|---|---|
| **Dove** | Client (browser) | Server (file/DB/Redis) |
| **Dimensione max** | ~4 KB per cookie | Dipende da storage |
| **Sicurezza** | Leggibile/modificabile dal client | Solo server (client vede solo ID) |
| **Persistenza** | Con `expires` (cookie persistente) | Dipende da `gc_maxlifetime` |
| **Performance** | Nessun carico server (inviato in header) | Lettura/scrittura su storage |
| **Dati sensibili** | NO — cifrare se necessario | SÌ — PIN, ruoli, preferenze |

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Cannot modify header information` | Output HTML prima di session_start/setcookie | Sposta session_start all'inizio dello script; usa output buffering |
| `session_start()` fallisce senza errore | Session file lock conteso (richieste concorrenti sullo stesso SID) | Chiudi sessione presto con `session_write_close()` |
| Cookie non salvato | Scadenza nel passato o path sbagliato | Verifica `time() > expires` e path corrisponde |
| Sessione persa dopo redirect | Cookie non inviato per path diverso | Path cookie deve coprire tutto il sito (`/`) |
| Session fixation | ID sessione non rigenerato dopo login | `session_regenerate_id(true)` dopo autenticazione |
| `session_destroy()` non cancella cookie | Distrugge dati server ma cookie rimane | Cancella esplicitamente: `setcookie(session_name(), "", time()-3600)` |
| Sessione non persiste su HTTPS/HTTP switch | Cookie secure=true su HTTP (non inviato) | Coerenza protocollo o cookie secure dinamico |

## Best practice

- **`session_start()` all'inizio dello script** — prima di qualsiasi output, anche whitespace
- **`session_regenerate_id(true)` dopo login e permessi** — previene session fixation
- **Chiudi sessione appena possibile** — `session_write_close()` dopo aver letto/scritto; evita lock concorrente
- **Cookie: httponly + secure + samesite** — minimo tre flag di sicurezza
- **Sessione: `use_strict_mode=1`** — rifiuta ID di sessione non generati da PHP (previene injection)
- **Dati sensibili mai in cookie** — PIN, token, ruolo vanno in sessione server
- **Non serializzare oggetti complessi in sessione** — problemi di versioning; usa ID + query
- **Redis/Memcached per sessioni in produzione** — scalabile, no lock concorrente, persistenza opzionale
- **Flash messages standard** — pattern monouso con $_SESSION["_flash"]
- **Cookie size: max 4 KB** — non superare; usa sessione per dati grandi

## Cross-reference

- [[PHP/Web/Superglobali|Superglobali]] — $_SESSION, $_COOKIE, $_SERVER
- [[PHP/Web/REST API|REST API]] — autenticazione API (token vs session), stateless design
- [[PHP/Laravel/Auth e Sicurezza|Laravel — Auth e Sicurezza]] — session driver, sanctum
- [[PHP/Core Concepts/Gestione Errori|Gestione Errori]] — errori di header modification
- [[PHP/Database/PDO|PDO]] — salvare sessioni in DB con session_set_save_handler
