---
topic: "Admin e Forms — Django"
tags: [python, django, admin, forms]
nav_prev: "[[Django REST Framework.md]]"
nav_next: "[[Auth e Deploy.md]]"
---
Riferimento ufficiale: [docs.djangoproject.com/en/stable/ref/contrib/admin](https://docs.djangoproject.com/en/stable/ref/contrib/admin/)

L'admin panel di Django è uno dei suoi punti di forza: CRUD generato automaticamente dai modelli, con configurazione avanzata (filtri, ricerca, campi personalizzati).

I **Forms** di Django gestiscono validazione lato server, rendering HTML e protezione CSRF. I `ModelForm` generano automaticamente campi dal modello.

Vedi anche: [[BE-NOTES/Python/Django/Models e ORM|Modelli]], [[BE-NOTES/Python/Django/Views e Templates|Views/Templates]].

`admin.site.register()` espone il modello nell'admin panel su `/admin/`. Da lì puoi fare CRUD senza scrivere una riga di UI. Il registro base mostra solo `__str__` del modello e permette di creare/modificare/eliminare record.

```python
# blog/admin.py
from django.contrib import admin
from .models import Autore, Articolo, Tag

admin.site.register(Autore)
admin.site.register(Articolo)
admin.site.register(Tag)
```

`@admin.register(Articolo)` registra il modello con una classe di configurazione. `list_display` controlla quali colonne mostrare nella lista; `list_filter` aggiunge filtri laterali; `search_fields` abilita la barra di ricerca; `date_hierarchy` aggiunge una navigazione temporale; `fieldsets` organizza i campi in sezioni nella pagina di dettaglio.

## Admin — configurazione avanzata

```python
@admin.register(Articolo)
class ArticoloAdmin(admin.ModelAdmin):
    list_display = ["titolo", "autore", "pubblicato", "creato"]
    list_filter = ["pubblicato", "creato", "autore"]
    search_fields = ["titolo", "contenuto"]
    date_hierarchy = "creato"
    ordering = ["-creato"]
    fieldsets = (
        ("Contenuto", {
            "fields": ("titolo", "contenuto", "autore")
        }),
        ("Stato", {
            "fields": ("pubblicato", "tags")
        }),
    )
```

`ModelForm` genera campi HTML + validazione automaticamente dal modello Django. `widgets` sovrascrive il componente HTML di default (es. `Textarea` invece di `<input type="text">`). `labels` personalizza le etichette. `forms.Form` invece è un form non legato a un modello — utile per filtri, contatti, ecc.

```python
from django import forms
from .models import Articolo

class ArticoloForm(forms.ModelForm):
    class Meta:
        model = Articolo
        fields = ["titolo", "contenuto", "autore", "tags"]
        widgets = {
            "contenuto": forms.Textarea(attrs={"rows": 10}),
            "tags": forms.CheckboxSelectMultiple(),
        }
        labels = {
            "titolo": "Titolo dell'articolo",
        }

class ContactForm(forms.Form):
    nome = forms.CharField(max_length=100)
    email = forms.EmailField()
    messaggio = forms.CharField(widget=forms.Textarea)
```

Il pattern: GET → mostra form vuoto, POST → valida e salva (o mostra errori). `form.is_valid()` esegue tutte le validazioni (campi obbligatori, tipi, lunghezza, ecc.) e popola `form.cleaned_data`. `form.save()` persiste il modello nel DB. Se la validazione fallisce, il form (con errori) viene ripassato al template.

```python
from django.shortcuts import render, redirect
from .forms import ArticoloForm

def crea_articolo(request):
    if request.method == "POST":
        form = ArticoloForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("lista_articoli")
    else:
        form = ArticoloForm()

    return render(request, "blog/form.html", {"form": form})
```

`{% csrf_token %}` inserisce un campo hidden anti-CSRF — Django lo verifica a ogni POST. `form.as_p` renderizza ogni campo dentro un `<p>`; alternative: `as_table`, `as_ul`, o manuale con `form.nome_campo`. Template tag con `.` significa "metodo/proprietà" — Django Template Language, non Python.

```html
{% extends "base.html" %}

{% block content %}
  <h1>Crea Articolo</h1>
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Salva</button>
  </form>
{% endblock %}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `CSRF token missing` | Form senza `{% csrf_token %}` | Aggiungi il tag nel template |
| Form non salva ma non dà errori | `is_valid()` non chiamato | Chiama `is_valid()` PRIMA di accedere a `cleaned_data` |
| `ValidationError` silenziosa | Validatore custom lancia errore ma non catturato | Usa `raise ValidationError(...)` nel `clean()` del form |
| Widget non viene renderizzato | Widget non associato al field corretto | Controlla i nomi in `widgets = {}` |

## Best practice

- **Usa `ModelForm` invece di `Form`** quando il form corrisponde a un modello — meno codice, validazione automatica
- **Personalizza `clean_<campo>()`** per validazione custom a livello di singolo campo
- **Usa `form.non_field_errors`** per errori che coinvolgono più campi (es. date sovrapposte)
- **Non fidarti del frontend**: i Forms di Django sono l'unica barriera di validazione
