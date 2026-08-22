---
topic: "Logging — Python"
tags: [python, logging, debugging, observability]
---
Riferimento ufficiale: [docs.python.org/3/library/logging.html](https://docs.python.org/3/library/logging.html)

Il modulo `logging` è il sistema di log standard di Python. A differenza di `print()`, supporta livelli di severità, output su file/console/rete, rotazione dei file, formattazione configurabile e separazione tra logica applicativa e destinazione dei log.

Il logging **non sostituisce il debugging**: serve a registrare eventi durante l'esecuzione in produzione. I cinque livelli standard sono: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`. Per default, solo `WARNING` e superiori vengono mostrati.

Vedi anche:
[[BE-NOTES/Python/Strumenti/Formattazione e Lint|Formattazione e Lint]],
[[BE-NOTES/Python/Tecnologie/Context Manager|Context Manager]].

## Configurazione base

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logging.debug("Non si vede")           # level=INFO, DEBUG è sotto soglia
logging.info("Applicazione avviata")   # Si vede
logging.warning("Memoria bassa")       # Si vede
logging.error("Connessione persa")     # Si vede
```

`basicConfig()` va chiamata **una volta sola** all'avvio. Se chiamata dopo che un logger ha già emesso messaggi, non ha effetto. `%(name)s` è il nome del logger (di default `root`).

## Logger nominativi (raccomandati)

```python
import logging

logger = logging.getLogger(__name__)
# Ogni modulo ha il suo logger col nome del modulo

logger.info("Elaborazione richiesta %s", request_id)
```

`__name__` dà al logger il nome del modulo (es. `mio_package.mio_modulo`). Questo permette di configurare livelli diversi per moduli diversi. Usa sempre **format string con argomenti** (`%s`, valori) invece di f-string: `logging` valuta la stringa solo se il livello è abilitato, risparmiando CPU.

## Handler: dove finiscono i log

```python
import logging

logger = logging.getLogger("mio_app")

# Console
console = logging.StreamHandler()
console.setLevel(logging.DEBUG)

# File con rotazione giornaliera
from logging.handlers import TimedRotatingFileHandler
file_handler = TimedRotatingFileHandler(
    "app.log", when="midnight", backupCount=7
)
file_handler.setLevel(logging.INFO)

# Formattatori diversi per ogni handler
formato_console = logging.Formatter("%(levelname)s: %(message)s")
formato_file = logging.Formatter(
    "%(asctime)s [%(levelname)s] %(name)s: %(message)s"
)
console.setFormatter(formato_console)
file_handler.setFormatter(formato_file)

logger.addHandler(console)
logger.addHandler(file_handler)
```

Ogni logger può avere più handler, ognuno con livello e formato indipendenti. `TimedRotatingFileHandler` crea un nuovo file ogni giorno e tiene 7 backup. `RotatingFileHandler` ruota per dimensione.

## Configurazione da file

```python
import logging.config

logging.config.fileConfig("logging.conf")
```
```ini
[loggers]
keys=root,mio_app

[handlers]
keys=console_handler,file_handler

[formatters]
keys=standard_formatter

[logger_root]
level=WARNING
handlers=console_handler

[logger_mio_app]
level=DEBUG
handlers=console_handler,file_handler
qualname=mio_app

[handler_console_handler]
class=StreamHandler
level=DEBUG
formatter=standard_formatter
args=(sys.stdout,)

[handler_file_handler]
class=handlers.TimedRotatingFileHandler
level=INFO
formatter=standard_formatter
args=('app.log', 'midnight', 1, 7)

[formatter_standard_formatter]
format=%(asctime)s [%(levelname)s] %(name)s: %(message)s
```

Per progetti grandi, meglio un file di configurazione che codice. In alternativa, puoi usare `dictConfig()` con un dict YAML/JSON.

## Catturare eccezioni

```python
import logging

logger = logging.getLogger(__name__)

try:
    1 / 0
except ZeroDivisionError:
    logger.exception("Errore nel calcolo")
    # Aggiunge automaticamente lo stack trace
```

`logger.exception()` è equivalente a `logger.error()` ma include lo stack trace completo. Usalo solo negli `except`. Per loggare eccezioni senza sollevarle esplicitamente, usa `logger.error(msg, exc_info=True)`.

## Filtri e contesto

```python
import logging
from logging import Filter

class SensitiveDataFilter(Filter):
    """Filtra dati sensibili dai log."""

    def filter(self, record: logging.LogRecord) -> bool:
        if hasattr(record, "password"):
            record.password = "***"
        return True

logger = logging.getLogger(__name__)
logger.addFilter(SensitiveDataFilter())

# Aggiungere contesto extra
logger.info("Accesso utente", extra={"user_id": 42})
```

`extra` permette di aggiungere campi personalizzati al log record, accessibili nel formato con `%(user_id)s`. I filtri possono modificare o scartare record prima che arrivino agli handler.

## Errori comuni

- **Usare `print()` per debugging**: in produzione non hai controllo su livelli e destinazioni.
- **Configurare il logger in librerie**: lascia la configurazione all'applicazione finale.
- **Dimenticare `%s` e usare f-string**: `logger.info(f"valore: {x}")` valuta sempre la stringa, anche se il livello è disabilitato.
- **Passare eccezioni come stringa**: usa `logger.exception(msg)` o `exc_info=True`, non `logger.error(str(e))` (perdi lo stack).
- **Root logger configurazione**: `logging.info()` usa il root logger. È meglio usare logger nominativi per controllo granulare.

## Best Practices & Conventions

- Usa sempre **logger nominativi** con `logging.getLogger(__name__)`.
- Non configurare handler in librerie: lascia all'applicazione finale.
- Usa `logger.exception()` negli `except` per catturare stack trace.
- Preferisci formato strutturato (JSON) per ambienti cloud dove i log sono aggregati.
- Imposta `level=WARNING` in produzione, `level=DEBUG` in sviluppo.
- Proteggi dati sensibili con filtri — mai loggare password, token, o dati personali.
- Per applicazioni asincrone, usa `logging.handlers.QueueHandler` per evitare blocchi.
