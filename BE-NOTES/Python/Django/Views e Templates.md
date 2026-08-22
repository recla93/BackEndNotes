---
topic: "Views e Templates — Django"
tags: [python, django, views, templates, mvt]
nav_prev: "[[Models e ORM.md]]"
nav_next: "[[Django REST Framework.md]]"
---
Riferimento ufficiale: [docs.djangoproject.com/en/stable/topics/http/views](https://docs.djangoproject.com/en/stable/topics/http/views/)

Django segue il pattern **MVT** (Model-View-Template):
- **View**: logica HTTP (simile a Controller in MVC)
- **Template**: HTML con Django Template Language
- **Model**: dati e business logic

Offre sia **Function-Based Views** (FBV, semplici) che **Class-Based Views** (CBV, riutilizzabili via ereditarietà). Le CBV sono più verbose ma riducono boilerplate per CRUD standard.

Vedi anche: [[BE-NOTES/Python/Django/Models e ORM|Modelli]], [[BE-NOTES/Python/Django/Admin e Forms|Forms]] per input utente.

Una view Django riceve una `request` (oggetto che contiene metodo HTTP, parametri, sessioni, utente) e restituisce una `HttpResponse`. `render()` costruisce una risposta HTML usando un template e un contesto (dict di variabili disponibili nel template). `get_object_or_404()` è come `.get()` ma restituisce 404 invece di `DoesNotExist`. `redirect()` restituisce una risposta HTTP 302 al nome della route.

```python
from django.shortcuts import render, get_object_or_404, redirect
from .models import Articolo

def lista_articoli(request):
    articoli = Articolo.objects.filter(pubblicato=True)
    return render(request, "blog/lista.html", {"articoli": articoli})

def dettaglio_articolo(request, id: int):
    articolo = get_object_or_404(Articolo, id=id, pubblicato=True)
    return render(request, "blog/dettaglio.html", {"articolo": articolo})

def crea_articolo(request):
    if request.method == "POST":
        # Valida e salva...
        return redirect("lista_articoli")
    return render(request, "blog/form.html")
```

## Class-Based Views

Le Class-Based Views incapsulano pattern comuni in classi riutilizzabili. `ListView` gestisce automaticamente: query al modello, paginazione (`paginate_by`), e passaggio del contesto al template. `DetailView` carica un singolo oggetto per PK. `CreateView` genera form, valida e salva. Sovrascrivendo `get_queryset()` personalizzi la query senza riscrivere tutta la view. `as_view()` converte la classe in una callable usabile in `urlpatterns`.

```python
from django.views.generic import ListView, DetailView, CreateView
from django.urls import reverse_lazy
from .models import Articolo

class ArticoloListView(ListView):
    model = Articolo
    template_name = "blog/lista.html"
    context_object_name = "articoli"
    paginate_by = 10

    def get_queryset(self):
        return Articolo.objects.filter(pubblicato=True)

class ArticoloDetailView(DetailView):
    model = Articolo
    template_name = "blog/dettaglio.html"

class ArticoloCreateView(CreateView):
    model = Articolo
    fields = ["titolo", "contenuto", "autore"]
    success_url = reverse_lazy("lista_articoli")
```

## Template (Django Template Language)

```html
<!-- blog/lista.html -->
{% extends "base.html" %}

{% block content %}
  <h1>Articoli</h1>
  <ul>
    {% for articolo in articoli %}
      <li>
        <a href="{% url 'dettaglio_articolo' articolo.id %}">
          {{ articolo.titolo }}
        </a>
        <small>di {{ articolo.autore.nome }}</small>
      </li>
    {% empty %}
      <li>Nessun articolo</li>
    {% endfor %}
  </ul>

  {% if is_paginated %}
    <div class="pagination">
      {% for i in page_obj.paginator.page_range %}
        <a href="?page={{ i }}">{{ i }}</a>
      {% endfor %}
    </div>
  {% endif %}
{% endblock %}
```

## URL routing

```python
# blog/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("", views.ArticoloListView.as_view(), name="lista_articoli"),
    path("<int:id>/", views.dettaglio_articolo, name="dettaglio_articolo"),
    path("crea/", views.ArticoloCreateView.as_view(), name="crea_articolo"),
]

# myproject/urls.py
from django.urls import include, path

urlpatterns = [
    path("admin/", admin.site.urls),
    path("articoli/", include("blog.urls")),
]
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `TemplateDoesNotExist` | Template non trovato nel path atteso | Mettilo in `app/templates/app/nome.html` |
| `NoReverseMatch` | `url` o `reverse` non trova il nome della route | Controlla il `name=` in `urlpatterns` |
| `AttributeError` su `request.POST` | Metodo GET su view che usa POST | Controlla `request.method` prima di accedere a `POST` |
| `Paginator` senza page | `page_obj` non esiste nel template | Verifica che `paginate_by` sia impostato |
| CBV restituisce 405 | Metodo HTTP non supportato dalla CBV | Usa la CBV corretta (es. `CreateView` accetta GET+POST) |

## Best practice

- **FBV per logica semplice, CBV per CRUD standard**: FBV è più leggibile per view custom; CBV riduce boilerplate per operazioni standard
- **`get_queryset` vs `model`**: sovrascrivi `get_queryset()` se devi filtrare — non usare `model` da solo
- **Template in `app/templates/app/`**: Django cerca template per app — il namespace dell'app (sottocartella) evita collisioni
- **`include()` per organizzare URL**: separa le route di ogni app nel suo `urls.py`
- **`reverse_lazy` per success_url**: usa `reverse_lazy` (non `reverse`) nei campi classe — `reverse` fallisce se le URL non sono ancora caricate
