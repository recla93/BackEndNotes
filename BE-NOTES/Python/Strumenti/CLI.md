---
topic: "CLI (Interfaccia a riga di comando) — Python"
tags: [python, cli, argparse, click, typer, command-line]
---
Riferimento ufficiale argparse: [docs.python.org/3/library/argparse.html](https://docs.python.org/3/library/argparse.html)
Riferimento ufficiale Click: [click.palletsprojects.com](https://click.palletsprojects.com/)
Riferimento ufficiale Typer: [typer.tiangolo.com](https://typer.tiangolo.com/)

Creare un'interfaccia a riga di comando (CLI) in Python significa trasformare uno script in un vero comando con argomenti, flag, help e gestione errori. Le tre librerie principali sono: `argparse` (stdlib, configurazione dichiarativa), `click` (decoratori, composizione), `typer` (type hints, autocompletamento).

`argparse` è sempre disponibile ma verboso. `click` è più espressivo con decoratori. `typer` (basato su click) usa type hints e genera automaticamente help e validazione.

Vedi anche:
[[BE-NOTES/Python/Core Concepts/Funzioni|Funzioni]],
[[BE-NOTES/Python/Strumenti/Poetry e UV|Poetry e UV]],
[[BE-NOTES/Python/Strumenti/pytest|pytest]].

## Argparse — standard library

```python
# cli_argparse.py
import argparse

def main():
    parser = argparse.ArgumentParser(
        description="Elabora file di input."
    )
    parser.add_argument(
        "input",                # posizionale (obbligatorio)
        help="File di input"
    )
    parser.add_argument(
        "-o", "--output",       # opzionale
        default="output.txt",
        help="File di output (default: output.txt)"
    )
    parser.add_argument(
        "-v", "--verbose",
        action="store_true",    # flag booleano
        help="Output dettagliato"
    )

    args = parser.parse_args()
    print(f"Input: {args.input}")
    print(f"Output: {args.output}")
    print(f"Verbose: {args.verbose}")

if __name__ == "__main__":
    main()
```

```bash
python cli_argparse.py dati.txt -o risultati.txt -v
```

`add_argument()` supporta posizionali (obbligatori per default), opzionali (`--flag`), flag booleani (`store_true`). L'help viene generato automaticamente con `-h`. I namespace sono accessibili come attributi di `args`.

## Click — decoratori e composizione

```bash
pip install click
```

```python
# cli_click.py
import click

@click.command()
@click.argument("input")              # posizionale
@click.option("-o", "--output",       # opzionale
              default="output.txt",
              help="File di output")
@click.option("-v", "--verbose",
              is_flag=True,
              help="Output dettagliato")
@click.option("--mode",
              type=click.Choice(["read", "write", "append"]),
              default="read",
              help="Modalità operativa")
def main(input, output, verbose, mode):
    """Elabora file di input con modalità specifica."""
    click.echo(f"Input: {input}")
    click.echo(f"Output: {output}")
    click.echo(f"Modo: {mode}")

if __name__ == "__main__":
    main()
```

Click usa decoratori per legare opzioni a parametri di funzione. `click.echo()` è multipiattaforma e gestisce Unicode. `click.Choice` limita i valori ammessi. Click supporta nativamente colori, prompt interattivi e paginazione.

## Click — gruppi di comandi

```python
# cli_gruppo.py
import click

@click.group()
def cli():
    """Gestione database."""
    pass

@cli.command()
@click.argument("name")
def create(name):
    """Crea un nuovo utente."""
    click.echo(f"Utente {name} creato.")

@cli.command()
@click.argument("name")
def delete(name):
    """Elimina un utente."""
    click.echo(f"Utente {name} eliminato.")

if __name__ == "__main__":
    cli()
```

```bash
python cli_gruppo.py create Mario
python cli_gruppo.py delete Mario
```

I gruppi Click permettono comandi annidati stile `git commit`, `docker run`. Ogni `@cli.command()` è un sottocomando indipendente. L'help mostra automaticamente l'elenco dei comandi.

## Typer — type hints nativi

```bash
pip install typer
```

```python
# cli_typer.py
import typer

app = typer.Typer()

@app.command()
def saluta(nome: str, cognome: str = "", maiuscolo: bool = False):
    """Saluta una persona."""
    messaggio = f"Ciao {nome} {cognome}".strip()
    if maiuscolo:
        messaggio = messaggio.upper()
    typer.echo(messaggio)

@app.command()
def somma(a: int, b: int = 0):
    """Somma due numeri."""
    risultato = a + b
    typer.echo(f"Risultato: {risultato}")

if __name__ == "__main__":
    app()
```

```bash
python cli_typer.py saluta Mario --cognome Rossi --maiuscolo
python cli_typer.py somma 5 3
```

Typer deduce automaticamente tipo, opzionalità (da default) e help dal type hint. `typer.Option()` e `typer.Argument()` permettono controllo fine. Typer genera anche autocompletamento per shell (bash, zsh, fish).

## Tabella comparativa

| Caratteristica | Argparse | Click | Typer |
|---------------|----------|-------|-------|
| Stdlib | Si | No | No |
| Boilerplate | Alto | Basso | Minimo |
| Type hints | No | No | Si (automatico) |
| Sottocomandi | `add_subparsers` | `@click.group` | `typer.Typer()` |
| Validazione | Manuale | `click.Choice` | Automatica da tipo |
| Colori/Unicode | No | Si | Si |
| Autocompletamento | Manuale | Si | Si |
| Ideale per | Script semplici | CLI mediate/complesse | CLI moderne con tipi |

## Entry point (package installabile)

```python
# pyproject.toml
[project.scripts]
mio-comando = "mio_package.cli:main"
```

```bash
pip install .
mio-comando --help
```

Definendo un entry point in `pyproject.toml`, la CLI diventa un comando globale dopo l'installazione del package. Funziona con tutti e tre i framework (argparse, click, typer).

## Errori comuni

- **Argparse: dimenticare `action="store_true"` per flag**: senza, `-v` aspetta un argomento.
- **Click: decoratori in ordine sbagliato**: `@click.option` va sotto `@click.command()` (più vicino alla funzione).
- **Typer: tipo sbagliato per flag booleano**: usa `bool` per flag, non `str`. Typer converte automaticamente.
- **Non gestire errori di input**: argparse e Typer mostrano help automaticamente; Click solleva `UsageError`.
- **Path con spazi**: usa sempre virgolette. `argparse` e Typer gestiscono nativamente.

## Best Practices & Conventions

- Per script semplici (< 5 opzioni): **argparse** (nessuna dipendenza).
- Per CLI strutturate con sottocomandi: **Click** (decoratori, composizione).
- Per progetti moderni con type hints: **Typer** (meno boilerplate, autocompletamento).
- Segui le convenzioni Unix: opzioni brevi (`-v`), lunghe (`--verbose`), posizionali per argomenti obbligatori.
- L'output `--help` deve essere autoesplicativo. Usa descrizioni chiare.
- Exit code: 0 per successo, 1 per errore generico, 2 per errore di input.
- Per input utente complesso, usa prompt interattivo (Click/FastAPI lo supportano).
