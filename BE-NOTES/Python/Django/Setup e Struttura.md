---
topic: "Setup e Struttura — Django"
tags: [python, django, setup, project-structure]
nav_next: "[[Models e ORM.md]]"
---
Riferimento ufficiale: [docs.djangoproject.com/en/stable/intro/tutorial01](https://docs.djangoproject.com/en/stable/intro/tutorial01/)

Django è un framework **full-stack** (batteries-included): ORM, admin panel, forms, auth, template engine, tutto incluso. A differenza di FastAPI (microframework API-first), Django è pensato per applicazioni web complete con rendering lato server.

### Quando Django vs FastAPI

| FastAPI | Django |
|---|---|
| API REST pure | Full-stack web app |
| Microservizi | Monolite con admin |
| Async nativo | Sync (Async via 3rd party) |
| Massima flessibilità | Convenzioni rigide |
| Type hint-driven | Meno type hints |

Vedi anche: [[BE-NOTES/Python/Django/Models e ORM|Models]], [[BE-NOTES/Python/Django/Django REST Framework|DRF]] per API.

`django-admin startproject` crea la struttura base del progetto. `startapp blog` crea un modulo riutilizzabile (app) dentro il progetto. Una app Django è un pacchetto Python con una struttura predefinita: models, views, admin, migrations. `manage.py` è l'interfaccia CLI — ogni comando Django si esegue con `python manage.py <comando>`.

```bash
pip install django djangorestframework
django-admin startproject myproject
cd myproject
python manage.py startapp blog
```

## Struttura progetto

```
myproject/
├── manage.py              # CLI tool (runserver, migrate, etc.)
├── myproject/
│   ├── __init__.py
│   ├── settings.py        # Configurazione globale
│   ├── urls.py            # URL routing principale
│   ├── asgi.py            # ASGI entry point
│   └── wsgi.py            # WSGI entry point
└── blog/                  # App (modulo riutilizzabile)
    ├── __init__.py
    ├── admin.py           # Config admin panel
    ├── apps.py            # Config app
    ├── models.py          # Modelli DB
    ├── views.py           # Logica delle view
    ├── urls.py            # URL routing dell'app
    ├── serializers.py     # DRF serializers
    └── templates/         # Template HTML
        └── blog/
            └── index.html
```

## settings.py — configurazione base

`INSTALLED_APPS` elenca tutte le app attive. L'ordine conta: Django cerca template, static files, migrations nell'ordine. `django.contrib.admin` deve stare dopo `django.contrib.auth` e `django.contrib.contenttypes` (da cui dipende). La tua app va aggiunta esplicitamente — Django non la scopre automaticamente. `DATABASES` configura la connessione: per SQLite `NAME` è il path del file; per PostgreSQL useresti `ENGINE: django.db.backends.postgresql` con `HOST`, `PORT`, `USER`, `PASSWORD`.

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "rest_framework",     # DRF
    "blog",               # la tua app
]

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}

LANGUAGE_CODE = "it-it"
TIME_ZONE = "Europe/Rome"
```

## Comandi base

`makemigrations` analizza i modelli e crea file di migrazione (differenze rispetto allo stato attuale del DB). `migrate` esegue le migrazioni pendenti. `runserver` avvia il dev server su `http://127.0.0.1:8000`. `createsuperuser` crea un utente admin per `/admin/`. `shell` apre una shell Python con Django già configurato (utile per test rapidi).

```bash
python manage.py runserver          # avvia dev server
python manage.py makemigrations     # crea migrazioni
python manage.py migrate            # applica migrazioni
python manage.py createsuperuser    # crea admin
python manage.py shell              # shell Python con Django
python manage.py test               # esegui test
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `django.core.exceptions.ImproperlyConfigured` | App non in `INSTALLED_APPS` | Aggiungi la tua app a settings.py |
| `No migrations to apply` | Migrazioni già applicate o modelli non rilevati | Controlla che la app sia in INSTALLED_APPS |
| `OperationalError: no such table` | `migrate` non eseguito | Esegui `python manage.py migrate` |
| Template not found | Path template sbagliato | Template in `app/templates/app/nome.html` |
| `ModuleNotFoundError` | Pacchetto non installato | `pip install` o aggiungi a requirements.txt |

## Best practice

- **Una app per responsabilità**: separa blog, utenti, commenti in app diverse
- **settings per ambiente**: usa `settings/base.py`, `settings/dev.py`, `settings/prod.py` invece di un unico file
- **Variabili d'ambiente**: mai hardcodare `SECRET_KEY`, `DATABASE_URL` — usa env var o `python-decouple`
- **Migrations in VCS**: committa sempre i file di migrazione — fanno parte del codice
