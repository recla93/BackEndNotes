---
topic: "Gestione Errori — PHP"
tags: [php, errors, exceptions, try-catch, error-handler, logging]
nav_prev: "[[Namespace e Autoload.md]]"
---
Riferimento ufficiale: [php.net/manual/en/language.exceptions.php](https://www.php.net/manual/en/language.exceptions.php) | [php.net/manual/en/class.throwable.php](https://www.php.net/manual/en/class.throwable.php)

PHP ha due sistemi di segnalazione errori: le **Exception** (OOP, catchabili con try/catch) e gli **Error** (PHP 7+ Throwable, che include Error ed Exception). Tutte le eccezioni non catturate terminano lo script con un Fatal Error.

```php
<?php

declare(strict_types=1);

// Base — try/catch/finally
try {
    $result = dividi(10, 0);
    echo $result;
} catch (DivisionByZeroError $e) {
    echo "Errore: " . $e->getMessage();
} catch (TypeError $e) {
    echo "Tipo sbagliato: " . $e->getMessage();
} finally {
    echo "Eseguito sempre";
    // cleanup: chiudi file, DB, etc.
}
```

## Throwable gerarchia (PHP 7+)

```php
// Throwable (interface)
// ├── Error
// │   ├── TypeError
// │   ├── ArithmeticError
// │   │   └── DivisionByZeroError
// │   ├── ParseError
// │   └── AssertionError
// └── Exception
//     ├── RuntimeException
//     │   ├── OutOfBoundsException
//     │   ├── OverflowException
//     │   └── UnexpectedValueException
//     └── LogicException
//         ├── InvalidArgumentException
//         ├── LengthException
//         └── BadMethodCallException
```

`Error` sono errori interni di PHP (tipo sbagliato, divisione per zero, parse error). `Exception` sono errori applicativi che il programmatore lancia intenzionalmente. Entrambi implementano `Throwable`.

## Custom Exception

```php
<?php

namespace App\Exceptions;

class ValidationException extends \RuntimeException {
    public function __construct(
        string $message = "Validazione fallita",
        private array $errors = [],
        int $code = 422,
        ?\Throwable $previous = null
    ) {
        parent::__construct($message, $code, $previous);
    }

    public function getErrors(): array {
        return $this->errors;
    }

    public function toArray(): array {
        return [
            'message' => $this->getMessage(),
            'errors' => $this->errors,
            'code' => $this->getCode(),
        ];
    }
}

// Uso
throw new ValidationException("Dati non validi", [
    'email' => ['Email già in uso'],
    'password' => ['Minimo 8 caratteri'],
]);
```

## Error handler globale

```php
<?php

// Convertire errori PHP in eccezioni
set_error_handler(function (int $severity, string $message, string $file, int $line): void {
    if (!(error_reporting() & $severity)) {
        return;  // error_reporting silenzia questo errore
    }
    throw new \ErrorException($message, 0, $severity, $file, $line);
});

// Error handler per errori fatali (shutdown)
register_shutdown_function(function (): void {
    $error = error_get_last();
    if ($error !== null && in_array($error['type'], [E_ERROR, E_PARSE, E_CORE_ERROR])) {
        echo "FATAL: {$error['message']} in {$error['file']}:{$error['line']}";
    }
});

// Exception handler globale
set_exception_handler(function (\Throwable $e): void {
    http_response_code($e->getCode() >= 400 ? $e->getCode() : 500);
    header('Content-Type: application/json');
    echo json_encode([
        'error' => $e->getMessage(),
        'file' => $e->getFile(),
        'line' => $e->getLine(),
    ]);
});
```

## Logging

```php
<?php

// error_log — base
error_log("Errore: connessione DB fallita");  // va a error_log di PHP
error_log("Errore", 3, '/var/log/app.log');   // 3 = append a file specifico

// Logger strutturato (Monolog)
use Monolog\Logger;
use Monolog\Handler\StreamHandler;

$log = new Logger('app');
$log->pushHandler(new StreamHandler('/var/log/app.log', Logger::WARNING));
$log->pushHandler(new StreamHandler('php://stdout', Logger::DEBUG));

$log->info('Utente creato', ['id' => 123, 'email' => 'mario@test.it']);
$log->error('Database connection failed', ['exception' => $e]);
```

## Error reporting levels

```php
<?php

// Sviluppo — mostra tutti gli errori
error_reporting(E_ALL);
ini_set('display_errors', '1');
ini_set('display_startup_errors', '1');

// Produzione — logga ma non mostrare
error_reporting(E_ALL);
ini_set('display_errors', '0');
ini_set('log_errors', '1');
ini_set('error_log', '/var/log/php_errors.log');
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Fatal error: Uncaught ...` | Eccezione non catturata | Aggiungi `try/catch` o `set_exception_handler` |
| `Headers already sent` | Output prima di `header()` o `setcookie()` | Sposta output dopo le funzioni header; rimuovi spazi prima di `<?php` |
| `Call to undefined method` | Metodo chiamato non esiste | Controlla nome metodo e namespace |
| `Cannot modify header information` | Output iniziato prima di header | Usa output buffering: `ob_start()` o rimuovi spazi |
| `Class not found` con autoload | Case mismatch o autoload non rigenerato | `composer dump-autoload` e controlla case |
| `E_WARNING: Division by zero` | Divisione per zero senza controllo | Controlla `$divisor !== 0` prima di dividere |

## Best practice

- **Try/catch stretto** — avvolgi solo il codice che PUÒ lanciare, non tutto il file
- **Custom exception per dominio** — `ValidationException`, `NotFoundException`, `PaymentException` — ogni strato con le sue eccezioni
- **`Throwable` nel catch generale** — cattura sia Error che Exception: `catch (\Throwable $e)`
- **Mai ingoiare eccezioni** — `catch (Exception $e) { /* vuoto */ }` nasconde bug
- **Logga sempre** — ogni eccezione catturata va loggata (almeno `error_log()`)
- **Finally per cleanup** — chiudi file, DB connection, stream in finally
- **set_exception_handler globale** — per API, restituisci JSON strutturato anche su errori non catturati
- **display_errors off in produzione** — mai stack trace all'utente; logga su file
- **`error_reporting(E_ALL)` in sviluppo e produzione** — niente warning nascosti

## Cross-reference

- [[PHP/Core Concepts/Funzioni|Funzioni]] — return type, strict_types
- [[PHP/Core Concepts/Classi e OOP|Classi e OOP]] — eccezioni custom, Throwable
- [[PHP/Web/REST API|REST API]] — error response strutturati
- [[PHP/Laravel/Setup e Architettura|Laravel — Setup]] — Handler, debug mode
