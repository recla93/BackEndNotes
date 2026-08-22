---
topic: "Auth e Deploy — Django"
tags: [python, django, auth, security, deploy]
nav_prev: "[[Admin e Forms.md]]"
---
Riferimento ufficiale: [docs.djangoproject.com/en/stable/topics/auth](https://docs.djangoproject.com/en/stable/topics/auth/)

Django ha autenticazione **built-in**: User model, session-based auth, password hashing, decorator `@login_required`, CBV `LoginRequiredMixin`. Per API token-based, si usa DRF + `rest_framework.authtoken` o JWT.

### Security checklist produzione

- `DEBUG = False`
- `SECRET_KEY` in environment variable
- Database PostgreSQL/MySQL (non SQLite)
- HTTPS obbligatorio
- `SECURE_SSL_REDIRECT = True`
- `SESSION_COOKIE_SECURE = True`
- `CSRF_COOKIE_SECURE = True`
- Esegui `python manage.py check --deploy`

Vedi anche: [[BE-NOTES/Python/FastAPI/Auth e Sicurezza|FastAPI Auth]] per confronto, [[BE-NOTES/Python/Strumenti/Docker per Python|Docker]] per deploy.

`create_user()` hasha la password automaticamente — mai usare `User.objects.create()` con password in chiaro. `authenticate()` verifica le credenziali e restituisce `None` se fallisce. `login(request, user)` crea la sessione lato server (cookie di sessione). `@login_required` reindirizza al login se `request.user` è anonimo. `LoginRequiredMixin` fa lo stesso per CBV.

```python
from django.contrib.auth.models import User
from django.contrib.auth import login, logout, authenticate
from django.contrib.auth.decorators import login_required
from django.contrib.auth.mixins import LoginRequiredMixin

# Registrazione
def registrazione(request):
    if request.method == "POST":
        user = User.objects.create_user(
            username=request.POST["username"],
            email=request.POST["email"],
            password=request.POST["password"],
        )
        login(request, user)
        return redirect("home")
    return render(request, "registration/register.html")

# Login
def login_view(request):
    user = authenticate(
        username=request.POST["username"],
        password=request.POST["password"],
    )
    if user:
        login(request, user)
    ...

# Decorator per proteggere view
@login_required
def profile(request):
    return render(request, "profile.html", {"user": request.user})

# CBV equivalent
class ProfileView(LoginRequiredMixin, TemplateView):
    template_name = "profile.html"
```

## Deployment — produzione

`DEBUG = False` disabilita le pagine di errore dettagliate (che espongono configurazione e codice) — obbligatorio in produzione. `ALLOWED_HOSTS` è la whitelist dei domini che possono servire il sito — Django rifiuta richieste con header `Host` non presente. `STATIC_ROOT` è la directory dove `collectstatic` copia i file statici; `STATIC_URL` è il prefisso URL. `MEDIA_ROOT`/`MEDIA_URL` fanno lo stesso per upload utente (immagini, documenti).

```python
# settings.py
DEBUG = False
ALLOWED_HOSTS = ["miodominio.com"]

# Static files
STATIC_ROOT = BASE_DIR / "staticfiles"
STATIC_URL = "/static/"

# Media files (upload utente)
MEDIA_ROOT = BASE_DIR / "media"
MEDIA_URL = "/media/"
```

## Sicurezza

`SECURE_SSL_REDIRECT` reindirizza HTTP → HTTPS (301). `SESSION_COOKIE_SECURE` e `CSRF_COOKIE_SECURE` impediscono l'invio del cookie su connessioni non HTTPS. `HSTS` dice al browser di usare sempre HTTPS per questo dominio — `31536000` secondi = 1 anno, `includeSubdomains` lo estende a tutti i sottodomini, `preload` permette l'inclusione nel preload list dei browser. `SECRET_KEY` in variabile d'ambiente evita che finisca nel repository.

```python
# settings.py (produzione)
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# Non hardcodare SECRET_KEY
import os
SECRET_KEY = os.environ.get("DJANGO_SECRET_KEY")
```

## WSGI + Gunicorn

`gunicorn myproject.wsgi:application` avvia il server WSGI in produzione. `myproject.wssi` è il modulo Python generato da `startproject`; `application` è la callable WSGI. `--bind 0.0.0.0:8000` ascolta su tutte le interfacce (necessario dentro Docker). In sviluppo si usa `runserver` (non adatto a produzione — monothread, senza security headers).

```bash
# Produzione
pip install gunicorn
gunicorn myproject.wsgi:application --bind 0.0.0.0:8000
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `DisallowedHost` | Host header non in `ALLOWED_HOSTS` | Aggiungi il dominio alla lista |
| `SECRET_KEY` vuoto in prod | Variabile d'ambiente non impostata | Controlla env su server |
| Static files 404 | `collectstatic` non eseguito | Esegui `manage.py collectstatic` |
| CSRF token non valido su HTTPS | `CSRF_COOKIE_SECURE=True` ma richiesta via HTTP | Usa HTTPS |
| Gunicorn dà 502 | Worker crashato o timeout | Aggiungi `--timeout 120` o aumenta workers |

## Best practice

- `SECRET_KEY` in variabile d'ambiente
- DEBUG=False in produzione
- Database PostgreSQL/MySQL in prod (non SQLite)
- Static files su CDN/S3 (whitenoise per small deploy)
- Logging configurato
- Test coverage >80%
- `python manage.py check --deploy` per audit
