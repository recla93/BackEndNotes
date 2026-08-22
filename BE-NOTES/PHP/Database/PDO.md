---
topic: "PDO (PHP Data Objects) — PHP"
tags: [php, database, pdo, sql, crud, prepared-statement, transactions]
nav_prev: "[[REST API.md]]"
---

Riferimento ufficiale: [php.net/manual/en/book.pdo.php](https://www.php.net/manual/en/book.pdo.php) | [php.net/manual/en/pdo.prepared-statements.php](https://www.php.net/manual/en/pdo.prepared-statements.php)

PDO (PHP Data Objects) è l'astrazione database built-in di PHP. Supporta MySQL, PostgreSQL, SQLite, Oracle, SQL Server con la stessa interfaccia. Usa **prepared statement** per prevenire SQL injection.

```php
<?php

declare(strict_types=1);

// Connessione
$dsn = "mysql:host=localhost;dbname=myapp;charset=utf8mb4";
$user = "root";
$pass = "secret";

$pdo = new PDO($dsn, $user, $pass, [
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES   => false,   // prepared statement nativi
]);
```

`ATTR_EMULATE_PREPARES=false` è critico: disabilita l'emulazione e usa prepared statement nativi del driver, con tipi reali.

## DSN per driver

```php
<?php

// MySQL / MariaDB
$dsn = "mysql:host=localhost;port=3306;dbname=myapp;charset=utf8mb4";

// PostgreSQL
$dsn = "pgsql:host=localhost;port=5432;dbname=myapp";

// SQLite (file locale)
$dsn = "sqlite:" . __DIR__ . "/database.sqlite";

// SQL Server
$dsn = "sqlsrv:Server=localhost;Database=myapp";
```

## CRUD con prepared statement

```php
<?php

// CREATE
$stmt = $pdo->prepare("INSERT INTO users (name, email, password) VALUES (?, ?, ?)");
$stmt->execute(["Mario", "mario@example.com", password_hash("secret", PASSWORD_BCRYPT)]);
$id = (int) $pdo->lastInsertId();

// Named parameter (più leggibile)
$stmt = $pdo->prepare("
    INSERT INTO users (name, email, password)
    VALUES (:name, :email, :password)
");
$stmt->execute([
    ":name"     => "Luigi",
    ":email"    => "luigi@example.com",
    ":password" => password_hash("secret", PASSWORD_BCRYPT),
]);

// READ — singolo
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$id]);
$user = $stmt->fetch();  // array associativo o false

// READ — multipli
$stmt = $pdo->prepare("SELECT * FROM users WHERE active = ? ORDER BY name");
$stmt->execute([true]);
$users = $stmt->fetchAll();  // array di array

// UPDATE
$stmt = $pdo->prepare("UPDATE users SET name = ? WHERE id = ?");
$stmt->execute(["Mario Rossi", $id]);

// DELETE
$stmt = $pdo->prepare("DELETE FROM users WHERE id = ?");
$stmt->execute([$id]);
$deleted = $stmt->rowCount();  // numero di righe modificate
```

## Fetch modes

```php
<?php

// FETCH_ASSOC — array associativo (default)
$user = $stmt->fetch(PDO::FETCH_ASSOC);
// ["id" => 1, "name" => "Mario"]

// FETCH_OBJ — oggetto stdClass
$user = $stmt->fetch(PDO::FETCH_OBJ);
// $user->name → "Mario"

// FETCH_COLUMN — singola colonna
$names = $pdo->query("SELECT name FROM users")->fetchAll(PDO::FETCH_COLUMN);
// ["Mario", "Luigi"]

// FETCH_KEY_PAIR — colonna 1 → chiave, colonna 2 → valore
$pairs = $pdo->query("SELECT id, name FROM users")->fetchAll(PDO::FETCH_KEY_PAIR);
// [1 => "Mario", 2 => "Luigi"]

// FETCH_CLASS — idrata in una classe esistente
class User {
    public int $id;
    public string $name;
    public string $email;
}

$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([1]);
$user = $stmt->fetchObject(User::class);
// $user è un'istanza di User con proprietà popolate
```

## Transazioni

```php
<?php

try {
    $pdo->beginTransaction();

    // Operazioni atomiche
    $pdo->prepare("UPDATE accounts SET balance = balance - 100 WHERE id = ?")->execute([1]);
    $pdo->prepare("UPDATE accounts SET balance = balance + 100 WHERE id = ?")->execute([2]);

    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    error_log("Transazione fallita: " . $e->getMessage());
    throw $e;
}
```

### Isolation level (MySQL)

```php
<?php

$pdo->exec("SET TRANSACTION ISOLATION LEVEL READ COMMITTED");
// READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE
```

## Gestione errori

```php
<?php

// ERRMODE_EXCEPTION — lancia PDOException (RACCOMANDATO)
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

// ERRMODE_WARNING — solo warning, non ferma
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_WARNING);

// ERRMODE_SILENT — silenzioso, devi chiamare errorInfo()
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_SILENT);

try {
    $pdo->prepare("SELECT * FROM inesistente")->execute();
    // ERRMODE_EXCEPTION → PDOException: Table 'inesistente' doesn't exist
    // ERRMODE_WARNING   → warning + $stmt->execute() restituisce false
    // ERRMODE_SILENT    → nessun output, $stmt->errorInfo() ha i dettagli
} catch (PDOException $e) {
    echo "Errore: " . $e->getMessage();
}
```

## Connection pooling e singleton

```php
<?php

class Connection
{
    private static ?\PDO $instance = null;

    public static function getInstance(): \PDO
    {
        if (self::$instance === null) {
            self::$instance = new \PDO(
                "mysql:host={$_ENV["DB_HOST"]};dbname={$_ENV["DB_NAME"]};charset=utf8mb4",
                $_ENV["DB_USER"],
                $_ENV["DB_PASS"],
                [
                    \PDO::ATTR_ERRMODE            => \PDO::ERRMODE_EXCEPTION,
                    \PDO::ATTR_DEFAULT_FETCH_MODE => \PDO::FETCH_ASSOC,
                    \PDO::ATTR_EMULATE_PREPARES   => false,
                ]
            );
        }
        return self::$instance;
    }

    // Vieta new Connection() e clonazione
    private function __construct() {}
    private function __clone() {}
}
```

## LIKE e IN con prepared statement

```php
<?php

// LIKE — devi mettere % nel valore, non nella query
$stmt = $pdo->prepare("SELECT * FROM users WHERE name LIKE ?");
$stmt->execute(["%$search%"]);

// IN — placeholder multipli con array
$ids = [1, 2, 3];
$placeholders = implode(",", array_fill(0, count($ids), "?"));
$stmt = $pdo->prepare("SELECT * FROM users WHERE id IN ($placeholders)");
$stmt->execute($ids);
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `SQLSTATE[HY000]: General error: 1366 Incorrect string value` | Charset mismatch DB/app | Usa `charset=utf8mb4` nel DSN e colonne `utf8mb4` |
| `SQLSTATE[23000]: Integrity constraint violation` | DUPLICATE KEY o FK violata | Controlla prima esistenza o usa `ON DUPLICATE KEY UPDATE` |
| `FATAL:  sorry, too many clients already` | Connessioni non chiuse (PostgreSQL) | Usa singleton connection; chiudi esplicitamente in worker |
| `PDOException: could not find driver` | Driver PDO non installato | Installa `php-mysql`, `php-pgsql` ecc. |
| `fetch()` restituisce `false` | Nessuna riga trovata ma non controlli `!== false` | `$user = $stmt->fetch() ?: null` |
| `General error: 2014 Cannot execute queries while other unbuffered queries` | Risultato non consumato prima di nuova query | `$stmt->closeCursor()` o `fetchAll()` prima di riusare connessione |
| `rowCount()` restituisce 0 per SELECT su alcuni driver | rowCount non affidabile per SELECT in MySQL | Conta con `COUNT(*)` o `fetchAll()` + `count()` |
| Transazione non completata | `beginTransaction()` ma manca `commit()` o `rollBack()` | Usa try/finally: `finally { if ($pdo->inTransaction()) $pdo->rollBack(); }` |

## Best practice

- **Prepared statement sempre** — mai concatenare variabili in SQL; anche per valori "sicuri"
- **`ERRMODE_EXCEPTION` sempre** — evita query silenziose che falliscono
- **`EMULATE_PREPARES=false`** — prepared statement nativi (tipi reali, performance)
- **`FETCH_ASSOC` di default** — più prevedibile di `FETCH_BOTH` (default)
- **Singola connessione per request** — connection pooling non serve in PHP (ogni request muore)
- **Named parameter per query lunghe** — `:name` più leggibile di `?` con 10+ parametri
- **Transazioni per operazioni multiple** — atomicità su più scritture correlate
- **`closeCursor()` dopo fetch parziale** — se fai `fetch()` invece di `fetchAll()`, chiudi prima della prossima query
- **Mai fidarsi di `rowCount()` per SELECT** — usa `COUNT(*)` in una query separata
- **Connection DSN da env** — mai hardcodare credenziali; usa `$_ENV` o `getenv()`

## Cross-reference

- [[PHP/Core Concepts/Gestione Errori|Gestione Errori]] — PDOException handling
- [[PHP/Web/REST API|REST API]] — repository pattern con PDO
- [[PHP/Laravel/Eloquent e Database|Laravel — Eloquent e Database]] — ORM su PDO
- [[PHP/Symfony/Doctrine e Database|Symfony — Doctrine e Database]] — DBAL e ORM su PDO
