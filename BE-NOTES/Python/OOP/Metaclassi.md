---
topic: "Metaclassi — Python OOP"
tags: [python, oop, metaclassi, type, metaprogrammazione]
---
Riferimento ufficiale: [docs.python.org/3/reference/datamodel.html#metaclasses](https://docs.python.org/3/reference/datamodel.html#metaclasses)

Una metaclasse e la **classe di una classe**: definisce come una classe viene costruita e si comporta. In Python, `type` e la metaclasse predefinita: quando scrivi `class Foo:`, Python chiama `type('Foo', (object,), {})` per crearla.

Le metaclassi intercettano la creazione della classe per modificarla: aggiungere metodi, validare attributi, registrare classi in un registry, applicare decoratori automaticamente. Sono uno strumento di metaprogrammazione potente ma da usare con parsimonia.

Vedi anche:
[[BE-NOTES/Python/OOP/Classi e Oggetti|Classi e Oggetti]],
[[BE-NOTES/Python/OOP/Decoratori|Decoratori]],
[[BE-NOTES/Python/OOP/Magic Methods|Magic Methods]].

## type come metaclasse

```python
# Creare una classe con type (equivalente di class)
MyClass = type('MyClass', (object,), {'x': 10, 'saluta': lambda self: 'Ciao'})

obj = MyClass()
print(obj.x)        # 10
print(obj.saluta()) # Ciao
```

`type(nome, basi, namespace)` crea una classe al volo. `name` e il nome, `basi` e una tupla di superclassi, `namespace` e un dict di attributi e metodi.

## Metaclasse personalizzata

```python
class LogMeta(type):
    """Metaclasse che logga la creazione di ogni classe."""

    def __new__(mcs, name: str, bases: tuple, namespace: dict):
        print(f"Creazione classe: {name}")
        print(f"  Basi: {bases}")
        print(f"  Attributi: {list(namespace.keys())}")
        return super().__new__(mcs, name, bases, namespace)


class MiaClasse(metaclass=LogMeta):
    x = 10

    def metodo(self):
        pass

# Output:
# Creazione classe: MiaClasse
#   Basi: ()
#   Attributi: ['__module__', '__qualname__', 'x', 'metodo']
```

`__new__` nella metaclasse viene chiamato quando la classe viene **definita**, non quando viene istanziata. Puoi modificare `namespace` prima di creare la classe.

## Registry pattern con metaclasse

```python
class PluginRegistry(type):
    registry: dict[str, type] = {}

    def __new__(mcs, name: str, bases: tuple, namespace: dict):
        cls = super().__new__(mcs, name, bases, namespace)
        if not name.startswith('Base'):  # Esclude la base
            mcs.registry[name] = cls
        return cls


class BasePlugin(metaclass=PluginRegistry):
    """Classe base per plugin. Non viene registrata."""

    def execute(self) -> str:
        raise NotImplementedError


class EmailPlugin(BasePlugin):
    def execute(self) -> str:
        return "Invio email"


class SmsPlugin(BasePlugin):
    def execute(self) -> str:
        return "Invio SMS"


print(PluginRegistry.registry)
# {'EmailPlugin': <class ...>, 'SmsPlugin': <class ...>}

# Usare il registry
for nome, cls in PluginRegistry.registry.items():
    plugin = cls()
    print(f"{nome}: {plugin.execute()}")
```

Il registry pattern e l'uso piu comune delle metaclassi: ogni classe che estende `BasePlugin` viene automaticamente registrata senza bisogno di codice esplicito.

## Validazione con metaclasse

```python
class ValidateAttributes(type):
    """Impone che tutti i metodi abbiano docstring."""

    def __new__(mcs, name: str, bases: tuple, namespace: dict):
        for attr_name, attr_value in namespace.items():
            if callable(attr_value) and not attr_value.__doc__:
                raise TypeError(
                    f"{name}.{attr_name} deve avere una docstring"
                )
        return super().__new__(mcs, name, bases, namespace)


class Documented(metaclass=ValidateAttributes):
    def saluta(self):
        """Saluta l'utente."""
        pass

    def calcola(self):
        pass  # TypeError!
```

Utile per imporre standard di codifica a livello di framework: docstring obbligatorie, naming convention, tipizzazione.

## Metaclasse vs decoratore di classe

```python
# Decoratore di classe (alternativa piu semplice)
def add_methods(cls):
    cls.nuovo_metodo = lambda self: "aggiunto"
    return cls

@add_methods
class MioServizio:
    pass

# Metaclasse (piu potente per gerarchie e registry)
class ServizioMeta(type):
    def __new__(mcs, name, bases, ns):
        if 'processa' not in ns:
            raise TypeError(f"{name} deve definire processa()")
        return super().__new__(mcs, name, bases, ns)
```

I decoratori di classe sono piu semplici e coprono l'80% dei casi. Le metaclassi servono quando devi estendere la semantica dell'ereditarieta o mantenere un registry automatico.

## `__init_subclass__` vs metaclasse

```python
# Alternativa moderna (Python 3.6+): senza metaclasse
class Base:
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        print(f"Sottoclasse creata: {cls.__name__}")

class Derivata(Base):  # Stampa: Sottoclasse creata: Derivata
    pass
```

`__init_subclass__` e un'alternativa piu semplice alla metaclasse per molti casi: viene chiamata quando una classe viene sottoclassata. Non sostituisce metaclassi per registry o modifiche alla creazione.

## Errori comuni

- **Metaclassi per casi semplici**: per aggiungere un metodo statico, usa un decoratore di classe.
- **Conflitto di metaclassi**: se due superclassi hanno metaclassi diverse che non collaborano, TypeError.
- **Dimenticare `super().__new__()`**: la metaclasse deve sempre chiamare `super().__new__()` o `type.__new__()`.
- **Modificare la classe dopo `__new__`**: usa `__init__` della metaclasse per modifiche post-creazione.
- **Pensare che la metaclasse influenzi le istanze**: la metaclasse opera sulla classe, non sulle istanze.

## Best Practices & Conventions

- Preferisci **decoratori di classe** o `__init_subclass__` alle metaclassi per la maggior parte dei casi.
- Usa metaclassi solo per: **registry automatici**, **validazione della struttura della classe**, **API framework**.
- Mantieni una singola metaclasse per gerarchia per evitare conflitti di metaclasse.
- Documenta sempre perche stai usando una metaclasse: e un pattern che richiede giustificazione.
- Per Python moderno (3.6+), valuta sempre prima `__init_subclass__` e decoratori.
