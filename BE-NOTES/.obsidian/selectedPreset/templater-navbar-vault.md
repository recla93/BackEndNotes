---
preset_name: templater-navbar-vault
scope: Study Documentation
language: Italiano
one_liner: Vault con navbar Templater, nav_prev/nav_next, configs/ off-limits, Core Concepts/ obbligatorio. Risposte in italiano.
---

# Preset: Templater-Navbar Vault

> **One-line summary**: Vault Obsidian con navbar gestita da Templater, frontmatter
> di navigazione `nav_prev`/`nav_next`, cartella `configs/` off-limits e cartella
> `Core Concepts/` obbligatoria per le tecnologie. Risposte in italiano.

## Identification

Apply this preset when the vault:
- Contains a `configs/scripts/navbar-engine` file (transcluded as navbar)
- Uses Templater plugin for templating
- Has the Obsidian CLI enabled (Settings → Command line interface)
- Follows PascalCase-with-spaces naming for files and folders

## Response language

**Italiano.** Lo skill risponde sempre in italiano in questo vault, indipendentemente
dalla lingua dell'input utente, a meno che l'utente non chieda esplicitamente di
cambiare lingua.

## Navbar / templating — SEMPRE inserire manualmente

Il vault usa una navbar basata su Templater. Lo skill **scrive sempre il content
completo** della nota (frontmatter + navbar + corpo) e crea il file con
`obsidian create … content="…"`. La navbar va inserita come prima riga del content
(dopo il frontmatter):

```
![[configs/scripts/navbar-engine#^navbar]]
```

Questa riga **deve** essere presente in ogni file creato. Non ometterla mai — senza
di essa il file non avrà la navigazione.

Perché inserirla a mano e non affidarsi a Templater: lo skill calcola `nav_prev` /
`nav_next` in base alla posizione nella sequenza — valori che un folder-template
statico di Templater non può conoscere. Quindi lo skill deve comunque scrivere il
frontmatter di navigazione, e già che scrive il content completo include anche la
riga della navbar. Nota verificata sul campo: quando `obsidian create` riceve un
`content`, il content vince e l'eventuale trigger "Templater on new file creation"
**non** lo sovrascrive (il trigger agisce solo sui file creati vuoti). Non usare mai
il parametro `template=` di `obsidian create` (vedi gotcha in
`references/cli-commands.md`).

## Frontmatter requirements

Due chiavi opzionali nel frontmatter pilotano i pulsanti laterali della navbar:

| Chiave | Significato | Quando presente |
|--------|-------------|-----------------|
| `nav_prev` | wikilink al file precedente | Sempre, tranne sul primo file di una sequenza o su file isolati |
| `nav_next` | wikilink al file successivo | Sempre, tranne sull'ultimo file di una sequenza o su file isolati |

### Regole di sintassi

1. **Sempre wikilink tra virgolette**: `nav_prev: "[[Nome File]]"`.
   Obsidian aggiorna automaticamente i wikilink nel frontmatter quando rinomini
   il file target — per questo la sintassi stringa semplice non è accettata.
2. **Nome del target senza estensione `.md`** e senza percorso, a meno che ci
   siano omonimi in altre cartelle del vault (in quel caso usa il percorso
   completo: `[[Cartella/Nome File]]`).
3. **Quando una chiave non serve, omettila dal frontmatter** — non scrivere
   `nav_prev: ""` o `nav_prev: null`.

### Tabella dei casi

| Posizione nella sequenza | `nav_prev` | `nav_next` |
|---|---|---|
| Primo file della sequenza | assente | presente |
| File intermedio | presente | presente |
| Ultimo file della sequenza | presente | assente |
| File isolato (nessuna sequenza) | assente | assente |

### File intro (Home, Intro, nome cartella)

I file intro di cartella ricevono `nav_next` **solo se la cartella ha una
sequenza ordinata chiara** (capitoli di un corso, forme normali, step di un
tutorial). Se la cartella è una collezione di concetti paralleli senza ordine
di lettura obbligato, l'intro non ha `nav_next` — i figli si raggiungono dalla
home o dai link interni al testo.

## Off-limits folders

**`configs/` e tutte le sue sottocartelle.**

Contiene file di configurazione critici per i plugin del vault (template script,
impostazioni, navbar engine). Non leggere, creare, modificare o eliminare nessun
file dentro `configs/`. Divieto assoluto, nessuna eccezione.

## Mandatory folder names

### `Core Concepts/` — nome FISSO e immutabile

Quando si documenta una **tecnologia o un linguaggio** nella sua interezza, ogni
tecnologia ha una cartella **obbligatoriamente** chiamata `Core Concepts/` per i
costrutti fondamentali nativi della tecnologia stessa.

> **⚠️ REGOLA INVIOLABILE**: il nome `Core Concepts/` è fisso. Non tradurlo, non
> abbreviarlo, non usare sinonimi. Vale per TUTTE le tecnologie, TUTTI i
> linguaggi, in QUESTO preset.
>
> ❌ NOMI VIETATI: `Concetti Base/`, `Fondamenti/`, `Basics/`, `Sintassi/`,
> `Base/`, `Concetti Fondamentali/`, `Fundamentals/`, `Core/`, `Essentials/`
>
> ✅ UNICO NOME VALIDO: `Core Concepts/`

#### Cosa va in `Core Concepts/`

Un concetto va in `Core Concepts/` se:
- È un costrutto **nativo** della tecnologia (non di una libreria/framework esterno)
- È **fondamentale** — comprendere la tecnologia richiede comprenderlo
- Non appartiene a nessun modulo, libreria o estensione specifica

#### Cosa NON va in `Core Concepts/`

Concetti appartenenti a librerie, framework, estensioni o moduli opzionali — in
quel caso va nella propria sottocartella dedicata.

#### Esempi

| Tecnologia | In `Core Concepts/` | Fuori |
|------------|---------------------|-------|
| Python | variabili, tipi, funzioni, classi, cicli, condizionali, eccezioni, moduli | Django, Flask, NumPy |
| TypeScript | tipi, interfacce, generics, enums, type guards, utility types | Angular, React |
| SQL | SELECT, JOIN, WHERE, GROUP BY, subquery, indici | dialetti specifici |
| RxJS | Observable, Observer, Subscription, Operators, Subject | librerie di integrazione |

## File and folder naming

- **Formato**: Pascal Case con spazi
- **Esempi corretti**: `Behavior Subject`, `Prima Forma Normale`, `Hot Observable`, `DML Instructions`
- **Esempi sbagliati**: `BehaviorSubject`, `prima_forma_normale`, `HOT_OBSERVABLE`
- **Nessun titolo H1 nel body** — Obsidian usa il nome file come titolo

## Intro-file naming

I file intro di una cartella devono avere uno di questi tre nomi (compatibilità
con il navbar engine):

| Nome | Quando usarlo |
|------|---------------|
| `Home` | File intro della root del vault (`Home.md`) |
| `Intro` | File intro generico di una cartella |
| **Nome della cartella** | File intro con lo stesso nome della cartella che contiene (preferito) |

**Preferenza**: usa il nome della cartella quando possibile — più esplicito e
coerente. Usa `Intro` solo quando il nome cartella sarebbe ambiguo o troppo
lungo. Usa `Home` solo per il file radice del vault.

Esempi:
- `DML/DML.md` ✓ oppure `DML/Intro.md` ✓
- `Normalizzazione/Normalizzazione.md` ✓ oppure `Normalizzazione/Intro.md` ✓
- `Home.md` ✓ (solo root)

## Tooling notes

Tooling **specifico di questo preset** (la meccanica generica della CLI —
no-overwrite, scritture asincrone, pattern del file temporaneo, gestione di
`.obsidian/` via filesystem — è in `references/cli-commands.md`):

- Plugin **Templater** installato (per la navbar; vedi sezione navbar — ma lo
  skill non lo invoca mai direttamente: scrive il content completo da sé).
- Questo preset funziona con **un vault aperto alla volta** in Obsidian; verifica
  sempre il vault target con `obsidian vault` prima di scrivere.

## How to create a file with this preset

Costruisci sempre il content completo — (a) il frontmatter di navigazione applicabile
e (b) la navbar come prima riga del content dopo il frontmatter — seguendo il pattern
del file temporaneo in `references/cli-commands.md`. Verifica prima che il path non
esista già.

### Esempio: file intermedio in una sequenza

```bash
cat > /tmp/nota.md << 'EOF'
---
nav_prev: "[[File Precedente]]"
nav_next: "[[File Successivo]]"
---
![[configs/scripts/navbar-engine#^navbar]]

testo del contenuto...
EOF
obsidian create path="percorso/del/File" content="$(cat /tmp/nota.md)"
```

### Esempio: file isolato

```bash
cat > /tmp/nota.md << 'EOF'
![[configs/scripts/navbar-engine#^navbar]]

testo del contenuto...
EOF
obsidian create path="percorso/del/File" content="$(cat /tmp/nota.md)"
```

## Examples

### File corretto

```markdown
---
nav_prev: "[[Prima Forma Normale]]"
nav_next: "[[Terza Forma Normale]]"
---
![[configs/scripts/navbar-engine#^navbar]]

La seconda forma normale (2NF) richiede che ogni attributo non chiave dipenda
dall'intera chiave primaria...
```

### File SBAGLIATO

```markdown
# Normalizzazione              ← SBAGLIATO: H1 non va mai messo

La normalizzazione è...
```

(manca anche la navbar transclusion: doppio errore)

## Tipi di contenuto

### File intro (di cartella o argomento)
- Descrizione introduttiva in prosa, 2-5 frasi
- **Nessun code block**
- Spiega di cosa tratta la sezione e come è organizzata

### File di dettaglio (concetto specifico)
Struttura libera — scegli il formato più efficace:
- Solo testo (concetti teorici)
- Testo + code block (comandi, sintassi)
- Testo + codice + testo (spiegazione → esempio → approfondimento)
- Tabella comparativa (confronti tra varianti)
- Note/warning callout (`> **Nota:** ...`)
- Più esempi consecutivi con titoli `##### Esempio standard:`, `##### Caso speciale:`

## Organizzazione gerarchica tipica

```
Dominio/
└── Tecnologia/
    └── Argomento/
        ├── Argomento.md          ← intro argomento (nome cartella)
        └── Categoria/
            ├── Categoria.md      ← intro categoria (nome cartella)
            └── Sotto Categoria/
                ├── Intro.md      ← OK Intro se il nome cartella è ridondante
                ├── Concetto Uno.md
                └── Concetto Due.md
```
