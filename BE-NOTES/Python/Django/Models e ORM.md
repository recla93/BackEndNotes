---
topic: "Models e ORM — Django"
tags: [python, django, orm, database, models]
nav_prev: "[[Setup e Struttura.md]]"
nav_next: "[[Views e Templates.md]]"
---
Riferimento ufficiale: [docs.djangoproject.com/en/stable/topics/db/models](https://docs.djangoproject.com/en/stable/topics/db/models/)

L'ORM di Django è un **active record** (a differenza di SQLAlchemy che è Data Mapper). Ogni modello = tabella, ogni istanza = riga. Le relazioni sono lazy per default (N+1 è un problema comune — usa `select_related` e `prefetch_related`).

Vedi anche: [[BE-NOTES/Python/Django/Admin e Forms|Admin]], [[BE-NOTES/Python/Data/Alembic|Alembic]] per migrazioni, [[BE-NOTES/Python/Django/Django REST Framework|DRF]] per serializzazione API.

Ogni classe che estende `models.Model` diventa una tabella nel database. Ogni `Field` definisce una colonna: `CharField` = VARCHAR, `TextField` = TEXT, `IntegerField` = INTEGER, ecc. `ForeignKey(Autore, on_delete=models.CASCADE)` crea una colonna `autore_id` e un vincolo di chiave esterna — `CASCADE` significa "se l'autore viene eliminato, elimina anche i suoi articoli". `ManyToManyField("Tag")` crea una tabella intermedia articoli_tags. `auto_now_add=True` imposta la data solo alla creazione; `auto_now=True` a ogni salvataggio. `related_name="articoli"` permette di fare `autore.articoli.all()` per leggere tutti gli articoli di un autore.

```python
from django.db import models

class Autore(models.Model):
    nome = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    bio = models.TextField(blank=True)
    data_nascita = models.DateField(null=True, blank=True)
    attivo = models.BooleanField(default=True)
    creato = models.DateTimeField(auto_now_add=True)
    aggiornato = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ["nome"]
        verbose_name_plural = "Autori"

    def __str__(self):
        return self.nome

class Articolo(models.Model):
    titolo = models.CharField(max_length=200)
    contenuto = models.TextField()
    autore = models.ForeignKey(
        Autore, on_delete=models.CASCADE, related_name="articoli"
    )
    tags = models.ManyToManyField("Tag", related_name="articoli")
    pubblicato = models.BooleanField(default=False)
    creato = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.titolo

class Tag(models.Model):
    nome = models.SlugField(unique=True)
```

## Field types comuni

| Field | Tipo DB | Uso |
|---|---|---|
| `CharField(max_length)` | VARCHAR | Testo breve |
| `TextField` | TEXT | Testo lungo |
| `IntegerField` | INTEGER | Numeri interi |
| `FloatField` | FLOAT | Numeri decimali |
| `BooleanField` | BOOLEAN | True/False |
| `DateField` / `DateTimeField` | DATE / DATETIME | Date |
| `EmailField` | VARCHAR | Email (validazione) |
| `FileField` / `ImageField` | VARCHAR (path) | File upload |
| `ForeignKey` | FOREIGN KEY | Relazione N:1 |
| `ManyToManyField` | M2M table | Relazione N:N |
| `OneToOneField` | UNIQUE FK | Relazione 1:1 |

## Query comuni

Le QuerySet di Django sono **lazy**: `Articolo.objects.all()` non esegue SQL fino a quando non accedi ai dati (es. iterazione, `list()`, o chiamate come `.first()`). I filtri si concatenano: `Articolo.objects.filter(pubblicato=True).filter(autore__nome="Mario")`. `__contains` genera SQL `LIKE '%Python%'`, `__year` estrae l'anno (funzione DB), `autore__nome` segue la FK. `.get()` lancia `DoesNotExist` se non trova — usa `.first()` se l'assenza è un caso normale.

```python
# READ
Articolo.objects.all()
Articolo.objects.filter(pubblicato=True)
Articolo.objects.exclude(pubblicato=True)
Articolo.objects.get(id=1)  # se non esiste: DoesNotExist

# Filtri
Articolo.objects.filter(titolo__contains="Python")
Articolo.objects.filter(creato__year=2024)
Articolo.objects.filter(autore__nome="Mario")

# CREATE
autore = Autore.objects.create(nome="Mario", email="mario@test.it")

# UPDATE
articolo = Articolo.objects.get(id=1)
articolo.titolo = "Nuovo titolo"
articolo.save()

# DELETE
articolo.delete()
```

## Aggregazioni e annotazioni

`annotate()` aggiunge un campo calcolato a OGNI riga del risultato (es. numero di articoli per autore). `aggregate()` restituisce un singolo valore (es. somma totale). `select_related("autore")` esegue una JOIN SQL per caricare l'autore insieme all'articolo — risolve il problema N+1 per FK. `prefetch_related("tags")` fa una query separata per i tag e li associa in Python — necessario per ManyToMany (non fattibile con JOIN singola). `Q()` permette OR, AND, NOT tra filtri.

```python
from django.db.models import Count, Avg, Sum, Q

# Count
Autore.objects.annotate(num_articoli=Count("articoli"))

# Filtri complessi con Q
Articolo.objects.filter(Q(pubblicato=True) | Q(autore__nome="Admin"))

# Catene di ForeignKey (select_related per N:1)
articoli = Articolo.objects.select_related("autore").all()

# Prefetch ManyToMany
articoli = Articolo.objects.prefetch_related("tags").all()
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `DoesNotExist` | `.get()` senza risultati | Usa `.first()` che restituisce `None` o cattura eccezione |
| `MultipleObjectsReturned` | `.get()` trova più righe | Usa `.filter()` o aggiungi vincoli univoci |
| N+1 queries | Caricare FK o M2M senza `select_related`/`prefetch_related` | Aggiungi sempre `select_related` per FK nelle liste |
| `FieldError` | Filtro su campo inesistente | Controlla il nome esatto del campo nel modello |
| Operazione lenta su tabella grande | Nessun indice | Aggiungi `db_index=True` nei campi usati per filtro |

## Best practice

- **`select_related` per FK**: sempre in view che restituiscono liste con ForeignKey
- **`prefetch_related` per M2M**: per ManyToMany e reverse FK (related_name)
- **`only()` / `defer()`**: carica solo i campi necessari per performance
- **Migration atomiche**: usa `migrate` in transazioni (default) — se fallisce, rollback
- **`class Meta`**: definisci `ordering`, `verbose_name`, `unique_together` nel modello
