---
preset_name: be-notes-vault
scope: Study Documentation
language: Italiano
one_liner: Vault BE-NOTES per appunti Java, Python, C# — frontmatter con topic + nav_prev/nav_next, link con estensione .md. Risposte in italiano.
---

# Preset: BE-NOTES Vault

> **One-line summary**: Vault di studio back-end con Java, Python, C#. Frontmatter
> con `topic`, navigazione `nav_prev`/`nav_next` con estensione `.md`, nessuna
> navbar Templater. Risposte in italiano.

## Response language

**Italiano.** Lo skill risponde sempre in italiano in questo vault, indipendentemente
dalla lingua dell'input utente, a meno che l'utente non chieda esplicitamente di
cambiare lingua.

## Navbar / templating

**Nessuna navbar Templater.** Il vault usa solo frontmatter per la navigazione:

- `nav_prev: "[[File.md]]"` — link al file precedente nella sequenza didattica
- `nav_next: "[[File.md]]"` — link al file successivo nella sequenza didattica

Non c'è transclusion di navbar nel corpo del file.

## Frontmatter requirements

```yaml
---
topic: "Titolo della Nota"
nav_prev: "[[File Precedente.md]]"
nav_next: "[[File Successivo.md]]"
---
```

| Chiave | Obbligatoria? | Quando presente |
|--------|:---:|---|
| `topic` | Sempre | Nome del concetto trattato |
| `nav_prev` | No | Su file non-primo in una sequenza |
| `nav_next` | No | Su file non-ultimo in una sequenza |
| `nav_home` | No | Su file intro, link alla home del topic genitore |

### Regole sintassi nav

1. **Wikilink tra virgolette con estensione `.md`**: `nav_prev: "[[File.md]]"`
2. **Percorso completo se c'è omonimia**: `nav_prev: "[[Cartella/File.md]]"`
3. **Ometti la chiave se non serve** — niente `nav_prev: ""` o `nav_prev: null`

## Off-limits folders

**Nessuna.** Il vault non ha cartelle off-limits.

## Mandatory folder names

### `Core Concepts/` — nome FISSO e immutabile

Quando si documenta una tecnologia o linguaggio, la cartella per i costrutti
fondamentali NATIVI si chiama obbligatoriamente `Core Concepts/`.

Nomi vietati: `Fondamenti/`, `Concetti Base/`, `Basics/`, `Sintassi/`,
`Base/`, `Fundamentals/`, `Core/`, `Essentials/`.

### `OOP/` — nome FISSO per contenuti OOP

## File and folder naming

- **Formato**: Pascal Case con spazi
- **Estensione**: `.md` nei wikilink di navigazione
- **Nessun titolo H1** — Obsidian usa il nome file come titolo
- **Esempi corretti**: `Classi e Oggetti.md`, `Dependency Injection.md`
- **Esempi sbagliati**: `classi_e_oggetti.md`, `classi-e-oggetti.md`

## Intro-file naming

Il file intro di una cartella prende il **nome della cartella** stessa, oppure
`Intro.md` se il nome cartella è ambiguo o troppo lungo.

## Tooling notes

- Obsidian CLI abilitato (Settings → Command line interface → ON)
- Templater **non** installato / non usato per navbar
- Plugin Dataview non richiesto

## Esempio file corretto

```markdown
---
topic: "Dependency Injection"
nav_prev: "[[Bean e Application Context.md]]"
nav_next: "[[Service Layer.md]]"
---

L'iniezione delle dipendenze (DI) è un pattern in cui un oggetto riceve le sue
dipendenze dall'esterno invece di crearle internamente.
```

## Esempio file SBAGLIATO

```markdown
# Dependency Injection    ← SBAGLIATO: H1 non va messo
nav_prev: Bean e Application Context.md  ← SBAGLIATO: senza virgolette e senza [[ ]]
```

## Tipi di contenuto

### File intro (di cartella o argomento)
- Descrizione introduttiva in prosa, 2-5 frasi
- Nessun code block
- Spiega cosa tratta la sezione e come è organizzata

### File di dettaglio (concetto specifico)
- Struttura pedagogica: Cos'è → Perché esiste → Come funziona → Codice → Errori comuni → Best Practices
- Code block con linguaggio specificato (```csharp, ```java, ecc.)

## Organizzazione gerarchica

```
Linguaggio/
├── Linguaggio.md                    ← Indice principale
├── Core Concepts/
│   ├── Core Concepts.md             ← Intro (se serve)
│   ├── Concetto Uno.md
│   └── Concetto Due.md
├── OOP/
│   └── ...
└── Framework/
    ├── Framework.md                 ← Indice framework
    └── ...
```
