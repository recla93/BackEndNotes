---
topic: "Django REST Framework"
tags: [python, django, drf, api, rest]
nav_prev: "[[Views e Templates.md]]"
nav_next: "[[Admin e Forms.md]]"
---
Riferimento ufficiale: [www.django-rest-framework.org](https://www.django-rest-framework.org/)

DRF è il modo standard per creare API REST con Django. Fornisce serializer (simili a Pydantic), ViewSet (CRUD automatico), authentication classes, routing via `DefaultRouter`.

A differenza di FastAPI (che è API-first), DRF si aggiunge a Django come app. Utile se hai già un sito Django e vuoi esporre API.

Vedi anche: [[BE-NOTES/Python/Django/Setup e Struttura|Setup Django]], [[BE-NOTES/Python/Django/Models e ORM|Modelli]], [[BE-NOTES/Python/FastAPI/Setup e Primi Passi|FastAPI]] per confronto.

DRF si installa come app Django. `INSTALLED_APPS` deve contenere `rest_framework`. `REST_FRAMEWORK` nel `settings.py` imposta le policy globali: `DEFAULT_PERMISSION_CLASSES` richiede autenticazione su tutte le API (override per-view), `DEFAULT_PAGINATION_CLASS` abilita paginazione su tutte le liste.

```python
# settings.py
INSTALLED_APPS = [
    ...
    "rest_framework",
]

# Per proteggere API (default)
REST_FRAMEWORK = {
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
    "DEFAULT_PAGINATION_CLASS":
        "rest_framework.pagination.PageNumberPagination",
        "PAGE_SIZE": 10,
}
```

## Serializers

`ModelSerializer` genera automaticamente campi di serializzazione dal modello Django — equivalente di Pydantic ma per Django. `read_only=True` significa il campo è incluso nell'output ma ignorato in input (utile per relazioni nidificate: mostri l'autore completo nella risposta ma non devi passarlo nella request). `write_only=True` è l'opposto: accetti un `autore_id` in input ma non lo restituisci nella risposta.

```python
from rest_framework import serializers
from .models import Articolo, Autore

class AutoreSerializer(serializers.ModelSerializer):
    class Meta:
        model = Autore
        fields = ["id", "nome", "email"]

class ArticoloSerializer(serializers.ModelSerializer):
    autore = AutoreSerializer(read_only=True)
    autore_id = serializers.IntegerField(write_only=True)

    class Meta:
        model = Articolo
        fields = ["id", "titolo", "contenuto", "autore", "autore_id", "creato"]
```

## ViewSets e Router

`ModelViewSet` genera automaticamente 6 operazioni CRUD (list, create, retrieve, update, partial_update, destroy) da un queryset e serializer. `router.register()` mappa ogni operazione a un URL: `GET /articoli/` → list, `POST /articoli/` → create, `GET /articoli/1/` → retrieve, ecc. `perform_create` permette di iniettare logica extra (es. assegnare l'autore corrente) prima del salvataggio.

```python
from rest_framework import viewsets, permissions
from .models import Articolo
from .serializers import ArticoloSerializer

class ArticoloViewSet(viewsets.ModelViewSet):
    queryset = Articolo.objects.filter(pubblicato=True)
    serializer_class = ArticoloSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

    def perform_create(self, serializer):
        serializer.save(autore=self.request.user.autore)

# urls.py
from rest_framework.routers import DefaultRouter
from .views import ArticoloViewSet

router = DefaultRouter()
router.register(r"articoli", ArticoloViewSet)

urlpatterns = router.urls
```

## APIView — controllo manuale

`APIView` è l'equivalente DRF delle FBV di Django: scrivi esplicitamente i metodi HTTP. `many=True` dice al serializer di processare una lista di oggetti (restituisce una lista JSON). `request.data` contiene il body parsato (JSON, form data, multipart). `serializer.errors` restituisce errori di validazione strutturati (campo → messaggio). Se usi ViewSet non serve scrivere tutto questo — ma `APIView` dà controllo totale.

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

class ArticoloList(APIView):
    def get(self, request):
        articoli = Articolo.objects.all()
        serializer = ArticoloSerializer(articoli, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = ArticoloSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

## Permessi custom

`BasePermission` richiede di implementare `has_permission` (a livello di view) e/o `has_object_permission` (a livello di singolo oggetto). `@action(detail=True)` aggiunge un endpoint custom al ViewSet (es. `/articoli/1/modifica/`) senza dover creare una view separata. `permission_classes` locale sovrascrive `DEFAULT_PERMISSION_CLASSES` globale.

```python
from rest_framework.permissions import BasePermission

class IsAutore(BasePermission):
    def has_object_permission(self, request, view, obj):
        return obj.autore == request.user.autore

@action(detail=True, permission_classes=[IsAutore])
def modifica(self, request, pk=None):
    ...
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `AssertionError: .is_valid() must be called before .data` | Accesso a `.data` senza chiamare `.is_valid()` | Chiama `serializer.is_valid(raise_exception=True)` |
| 405 Method Not Allowed | Metodo HTTP non supportato dal ViewSet | Usa `ModelViewSet` o aggiungi `@action(methods=['patch'])` |
| `Object does not exist` con PK valida | ViewSet filtrato esclude l'oggetto | Controlla `get_queryset()` — non chiama `.get()` |
| Campi in nested serializer non scritti | `read_only=True` senza `write_only` counterpart | Aggiungi `autore_id = IntegerField(write_only=True)` |
| `detail` route collide con `list` | Nome action coincide con nome ViewSet | Usa `@action(detail=False)` per action su collezioni |

## Best practice

- **ViewSet per CRUD standard, APIView per logica custom**: ViewSet riduce boilerplate; APIView per operazioni non standard (es. upload, export)
- **`raise_exception=True` in `is_valid()`**: evita di scrivere `if not valid: return 400` esplicitamente
- **`perform_create`/`perform_update` per logica pre-salvataggio**: meglio che sovrascrivere `create()`/`update()` interamente
- **Throttling per API pubbliche**: usa `DEFAULT_THROTTLE_CLASSES` per limitare richieste
- **Versioning**: metti `/api/v1/` nel prefisso URL o usa `Accept header versioning`
