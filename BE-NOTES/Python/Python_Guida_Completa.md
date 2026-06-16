# 🐍 PYTHON - Guida Completa: Scripting, Backend e Bot

## Indice
1. [Fondamenti di Python](#fondamenti)
2. [Struttura del Linguaggio](#struttura)
3. [Scripting per Automazione](#scripting)
4. [Backend con FastAPI](#backend)
5. [Creazione di Bot](#bot)
6. [Checklist Progetto](#checklist)

---

## 1. FONDAMENTI DI PYTHON {#fondamenti}

### Cosa è Python?
Python è un linguaggio **interpretato**, **dinamico** e **orientato agli oggetti**:
- **Interpretato**: Eseguito direttamente senza compilazione
- **Dinamico**: Tipi assegnati a runtime (a differenza di Java)
- **Leggibile**: Sintassi simile all'inglese, perfetto per imparare

### Perché usare Python?
Python è ottimo per: scripting, backend (FastAPI, Django), bot (Discord/Telegram), data science e AI/ML.

---

## 2. STRUTTURA DEL LINGUAGGIO {#struttura}

### Variabili e Tipi di Dato

```python
# Stringhe
nome = "Mario"  # str
messaggio = 'Hello'

# Numeri
eta = 25  # int
prezzo = 19.99  # float

# Booleani
attivo = True  # bool

# Liste (array mutabile)
numeri = [1, 2, 3, 4, 5]
lista_mista = [1, "ciao", 3.14, True]

# Tuple (array immutabile)
coordinate = (10, 20)

# Dizionari (key-value map)
persona = {
    "nome": "Mario",
    "eta": 25,
    "città": "Roma"
}

# Set (valori unici)
colori = {"rosso", "blu", "verde"}

# Tipo dinamico: puoi cambiare tipo alla stessa variabile
valore = 5  # int
valore = "testo"  # string (Python lo permette!)
```

### Operatori

```python
# Aritmetici
10 + 5  # 15
10 - 5  # 5
10 * 5  # 50
10 / 5  # 2.0 (float)
10 // 3  # 3 (divisione intera)
10 % 3  # 1 (modulo/resto)
2 ** 3  # 8 (esponente)

# Comparazione
5 == 5  # True
5 != 3  # True
5 > 3   # True
5 >= 5  # True
5 < 10  # True

# Logici
True and False  # False
True or False   # True
not True        # False

# Membership
5 in [1, 2, 3, 5]  # True
"a" in "ciao"      # True
```

### Stringhe (String Operations)

```python
testo = "Hello World"

# Accesso caratteri
testo[0]  # "H"
testo[-1]  # "d" (ultimo carattere)
testo[0:5]  # "Hello" (slicing: da indice 0 a 4)

# Metodi utili
testo.lower()  # "hello world"
testo.upper()  # "HELLO WORLD"
testo.replace("World", "Python")  # "Hello Python"
testo.split()  # ["Hello", "World"]

# String interpolation
nome = "Mario"
print(f"Ciao {nome}")  # "Ciao Mario" (f-string)
print(f"2 + 2 = {2 + 2}")  # "2 + 2 = 4"
```

### Strutture di Controllo

#### If/Else

```python
eta = 18

if eta >= 18:
    print("Sei maggiorenne")
elif eta >= 13:
    print("Sei adolescente")
else:
    print("Sei bambino")
```

#### For Loop

```python
# Iterare su lista
frutti = ["mela", "banana", "arancia"]
for frutto in frutti:
    print(f"Mangio {frutto}")

# Iterare con indice
for i, frutto in enumerate(frutti):
    print(f"{i}: {frutto}")  # 0: mela, 1: banana, ...

# Range
for i in range(5):  # 0, 1, 2, 3, 4
    print(i)

# Range con step
for i in range(0, 10, 2):  # 0, 2, 4, 6, 8
    print(i)
```

#### While Loop

```python
contatore = 0
while contatore < 5:
    print(contatore)
    contatore += 1

# Break e Continue
for i in range(10):
    if i == 5:
        break  # Esce dal loop
    if i == 2:
        continue  # Salta questa iterazione
    print(i)
```

---

## 3. FUNZIONI E PROGRAMMAZIONE AD OGGETTI

### Funzioni

```python
# Funzione semplice
def saluta(nome):
    return f"Ciao {nome}"

print(saluta("Mario"))  # "Ciao Mario"

# Parametri di default
def presentati(nome, città="Roma"):
    return f"Mi chiamo {nome} e abito a {città}"

presentati("Mario")  # Roma (default)
presentati("Luigi", "Milano")  # Milano

# Parametri variabili
def somma(*numeri):
    total = 0
    for n in numeri:
        total += n
    return total

somma(1, 2, 3, 4, 5)  # 15

# Keyword arguments
def crea_profilo(nome, eta, città):
    return {"nome": nome, "eta": eta, "città": città}

crea_profilo(nome="Mario", città="Roma", eta=25)
# Puoi passare in qualsiasi ordine

# Funzioni lambda (anonime, una riga)
quadrato = lambda x: x ** 2
print(quadrato(5))  # 25

# Lambda con map/filter
numeri = [1, 2, 3, 4, 5]
raddoppiati = list(map(lambda x: x * 2, numeri))  # [2, 4, 6, 8, 10]

pari = list(filter(lambda x: x % 2 == 0, numeri))  # [2, 4]
```

### Classi e Oggetti

```python
class Persona:
    def __init__(self, nome, eta):
        self.nome = nome
        self.eta = eta
    
    def presentati(self):
        return f"Mi chiamo {self.nome} e ho {self.eta} anni"
    
    def compleanno(self):
        self.eta += 1

# Creazione istanza
mario = Persona("Mario", 25)
print(mario.presentati())  # "Mi chiamo Mario e ho 25 anni"
mario.compleanno()
print(mario.eta)  # 26

# Ereditarietà
class Studente(Persona):
    def __init__(self, nome, eta, matricola):
        super().__init__(nome, eta)  # Chiama __init__ della classe padre
        self.matricola = matricola
    
    def presentati(self):
        return f"{super().presentati()} (Matricola: {self.matricola})"

luigi = Studente("Luigi", 20, "2024001")
print(luigi.presentati())  # "Mi chiamo Luigi e ho 20 anni (Matricola: 2024001)"
```

---

## 4. SCRIPTING PER AUTOMAZIONE {#scripting}

### Operazioni su File

```python
# Leggere file
with open("testo.txt", "r") as file:
    contenuto = file.read()
    print(contenuto)

# Leggere riga per riga
with open("testo.txt", "r") as file:
    for riga in file:
        print(riga.strip())  # strip() rimuove spazi/newline

# Scrivere file
with open("output.txt", "w") as file:
    file.write("Ciao, questo è un file scritto da Python!\n")
    file.write("Seconda riga\n")

# Append (aggiungere)
with open("output.txt", "a") as file:
    file.write("Terza riga aggiunta\n")

# Leggere JSON
import json
with open("dati.json", "r") as file:
    dati = json.load(file)
    print(dati["nome"])

# Scrivere JSON
dati = {"nome": "Mario", "eta": 25, "città": "Roma"}
with open("dati.json", "w") as file:
    json.dump(dati, file, indent=2)
```

### Operazioni su Cartelle

```python
import os
from pathlib import Path

# Verificare se esiste file/cartella
os.path.exists("cartella")
os.path.isfile("file.txt")
os.path.isdir("cartella")

# Creare cartella
os.makedirs("nuova_cartella", exist_ok=True)

# Listar file in cartella
files = os.listdir("cartella")
for file in files:
    print(file)

# Iterare file ricorsivamente
for root, dirs, files in os.walk("cartella"):
    for file in files:
        percorso_completo = os.path.join(root, file)
        print(percorso_completo)

# Usando Path (moderno)
from pathlib import Path
cartella = Path("cartella")
for file in cartella.glob("*.txt"):  # Tutti i .txt
    print(file)

for file in cartella.rglob("*.py"):  # Ricorsivamente
    print(file)
```

### Automazione Task

```python
import time
from datetime import datetime

# Eseguire task ogni N secondi
def backup_automatico():
    while True:
        print(f"Backup in corso... {datetime.now()}")
        time.sleep(60)  # Ogni 60 secondi
        
        # Copia file da una cartella all'altra
        import shutil
        shutil.copytree("cartella_originale", "backup_cartella", dirs_exist_ok=True)

# Eseguire comando di sistema
import subprocess
result = subprocess.run(["dir"], capture_output=True, text=True)
print(result.stdout)

# Schedulare task (libreria schedule)
import schedule

def compito_giornaliero():
    print("Task eseguito!")

schedule.every().day.at("10:30").do(compito_giornaliero)

while True:
    schedule.run_pending()
    time.sleep(1)
```

---

## 5. BACKEND CON FASTAPI {#backend}

### Setup e Installazione

```bash
# Creare virtual environment
python -m venv venv

# Attivare venv (Windows)
venv\Scripts\activate

# Attivare venv (Mac/Linux)
source venv/bin/activate

# Installare FastAPI e Uvicorn (server)
pip install fastapi uvicorn

# Installare dipendenze comuni
pip install sqlalchemy pydantic requests
```

### Prima API Semplice

```python
# main.py
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Modello dati (schema)
class Persona(BaseModel):
    nome: str
    eta: int
    città: str = "Roma"  # default

# GET - Recuperare dati
@app.get("/")
def root():
    return {"messaggio": "Benvenuto!"}

@app.get("/persone/{id}")
def get_persona(id: int):
    return {"id": id, "nome": "Mario", "eta": 25}

# POST - Creare dati
@app.post("/persone")
def crea_persona(persona: Persona):
    return {
        "messaggio": "Persona creata",
        "data": persona
    }

# PUT - Aggiornare dati
@app.put("/persone/{id}")
def aggiorna_persona(id: int, persona: Persona):
    return {
        "id": id,
        "messaggio": "Persona aggiornata",
        "data": persona
    }

# DELETE - Eliminare dati
@app.delete("/persone/{id}")
def elimina_persona(id: int):
    return {"messaggio": f"Persona {id} eliminata"}

# Avviare server
# uvicorn main:app --reload
```

### Query Parameters e Validation

```python
from fastapi import FastAPI, Query
from typing import Optional

app = FastAPI()

# Query parameters
@app.get("/search")
def search(q: str = Query(..., min_length=1), skip: int = 0, limit: int = 10):
    return {
        "query": q,
        "skip": skip,
        "limit": limit
    }
# GET /search?q=mario&skip=0&limit=20

# Query parameters opzionali
@app.get("/filtro")
def filtro(categoria: Optional[str] = None, prezzo_max: Optional[float] = None):
    return {
        "categoria": categoria,
        "prezzo_max": prezzo_max
    }
```

### Database con SQLAlchemy

```python
from fastapi import FastAPI
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from pydantic import BaseModel

DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Modello Database
class PersonaDB(Base):
    __tablename__ = "persone"
    
    id = Column(Integer, primary_key=True, index=True)
    nome = Column(String, index=True)
    eta = Column(Integer)
    città = Column(String, default="Roma")

Base.metadata.create_all(bind=engine)

# Schema Pydantic (per request/response)
class PersonaSchema(BaseModel):
    nome: str
    eta: int
    città: str = "Roma"
    
    class Config:
        from_attributes = True

app = FastAPI()

# Dipendenza per sesione DB
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Endpoint per creazione
@app.post("/persone", response_model=PersonaSchema)
def crea_persona(persona: PersonaSchema, db: Session = Depends(get_db)):
    db_persona = PersonaDB(**persona.dict())
    db.add(db_persona)
    db.commit()
    db.refresh(db_persona)
    return db_persona

# Endpoint per lettura
@app.get("/persone/{id}", response_model=PersonaSchema)
def get_persona(id: int, db: Session = Depends(get_db)):
    return db.query(PersonaDB).filter(PersonaDB.id == id).first()

# Endpoint per lista
@app.get("/persone", response_model=list[PersonaSchema])
def lista_persone(db: Session = Depends(get_db)):
    return db.query(PersonaDB).all()

# Endpoint per aggiornamento
@app.put("/persone/{id}")
def aggiorna_persona(id: int, persona: PersonaSchema, db: Session = Depends(get_db)):
    db_persona = db.query(PersonaDB).filter(PersonaDB.id == id).first()
    if not db_persona:
        raise HTTPException(status_code=404, detail="Non trovato")
    
    for key, value in persona.dict().items():
        setattr(db_persona, key, value)
    
    db.commit()
    db.refresh(db_persona)
    return db_persona

# Endpoint per eliminazione
@app.delete("/persone/{id}")
def elimina_persona(id: int, db: Session = Depends(get_db)):
    db_persona = db.query(PersonaDB).filter(PersonaDB.id == id).first()
    if not db_persona:
        raise HTTPException(status_code=404, detail="Non trovato")
    
    db.delete(db_persona)
    db.commit()
    return {"messaggio": "Eliminato"}
```

### Connessione al Frontend (CORS)

```python
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Abilitare CORS per il frontend
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:4200"],  # Angular dev server
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Ora il frontend Angular può fare richieste senza errori CORS
```

---

## 6. CREAZIONE DI BOT {#bot}

### Bot Discord

```python
import discord
from discord.ext import commands
import os
from dotenv import load_dotenv

load_dotenv()
TOKEN = os.getenv("DISCORD_TOKEN")

# Creare bot
bot = commands.Bot(command_prefix="!", intents=discord.Intents.default())

# Evento: Bot online
@bot.event
async def on_ready():
    print(f"{bot.user} è online!")

# Comando semplice
@bot.command(name="ciao")
async def ciao(ctx):
    await ctx.send(f"Ciao {ctx.author.name}!")

# Comando con argomenti
@bot.command(name="somma")
async def somma(ctx, a: int, b: int):
    risultato = a + b
    await ctx.send(f"{a} + {b} = {risultato}")

# Event: Nuovo messaggio
@bot.event
async def on_message(message):
    if message.author == bot.user:
        return
    
    if "ciao" in message.content.lower():
        await message.reply("Ciao! 👋")
    
    await bot.process_commands(message)

# Event: Membro entra
@bot.event
async def on_member_join(member):
    channel = discord.utils.get(member.guild.channels, name="benvenuti")
    if channel:
        await channel.send(f"Benvenuto {member.mention}!")

# Avviare bot
bot.run(TOKEN)
```

**File .env:**
```
DISCORD_TOKEN=your_bot_token_here
```

### Bot Telegram

```python
import logging
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

TELEGRAM_TOKEN = "your_bot_token_here"

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Inviato quando utente fa /start"""
    await update.message.reply_text(f"Ciao {update.effective_user.first_name}! 👋")

async def help(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Inviato quando utente fa /help"""
    await update.message.reply_text(
        "Comandi disponibili:\n"
        "/start - Inizia\n"
        "/help - Aiuto\n"
        "/somma 5 3 - Somma due numeri"
    )

async def somma(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Somma due numeri: /somma 5 3"""
    try:
        a, b = int(context.args[0]), int(context.args[1])
        await update.message.reply_text(f"{a} + {b} = {a + b}")
    except:
        await update.message.reply_text("Usa: /somma <numero1> <numero2>")

async def echo(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Ripeti il messaggio"""
    await update.message.reply_text(f"Hai detto: {update.message.text}")

def main():
    application = Application.builder().token(TELEGRAM_TOKEN).build()
    
    # Registrare handler
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CommandHandler("help", help))
    application.add_handler(CommandHandler("somma", somma))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, echo))
    
    # Avviare bot (polling)
    application.run_polling()

if __name__ == '__main__':
    main()
```

---

## 7. STRUTTURA PROGETTO PYTHON

### Scripting Project

```
mio_script/
├── main.py              # Entry point
├── utils.py             # Funzioni utility
├── config.py            # Configurazioni
├── requirements.txt     # Dipendenze
├── .env                 # Variabili ambiente
├── input/               # File input
└── output/              # File output
```

### Backend Project (FastAPI)

```
mio_backend/
├── main.py              # Entry point
├── app/
│   ├── __init__.py
│   ├── models.py        # Modelli DB (SQLAlchemy)
│   ├── schemas.py       # Schemi (Pydantic)
│   ├── database.py      # Configurazione DB
│   └── routers/
│       ├── __init__.py
│       ├── persone.py   # Endpoint /persone
│       ├── prodotti.py  # Endpoint /prodotti
│       └── auth.py      # Autenticazione
├── tests/
│   ├── __init__.py
│   └── test_api.py
├── requirements.txt
├── .env
└── .gitignore
```

### Bot Project

```
mio_bot/
├── main.py              # Entry point
├── commands/
│   ├── __init__.py
│   ├── admin.py
│   ├── utility.py
│   └── fun.py
├── events/
│   ├── __init__.py
│   └── messages.py
├── utils.py
├── requirements.txt
├── .env
└── .gitignore
```

---

## 8. CHECKLIST PROGETTO PYTHON {#checklist}

### ✅ Prima di Iniziare
- [ ] Python 3.8+ installato (`python --version`)
- [ ] Virtual environment creato (`python -m venv venv`)
- [ ] Virtual environment attivato
- [ ] requirements.txt creato e gestito

### ✅ Struttura Cartelle
- [ ] Cartella principale creata
- [ ] main.py come entry point
- [ ] requirements.txt con dipendenze
- [ ] .env per variabili sensibili
- [ ] .gitignore configurato

### ✅ Per Scripting
- [ ] Moduli organizzati (utils, config, main)
- [ ] Logging configurato
- [ ] Gestione errori con try/except
- [ ] Documentazione stringhe (docstrings)

### ✅ Per Backend (FastAPI)
- [ ] Database schema definito (models.py)
- [ ] Schemi Pydantic per validazione (schemas.py)
- [ ] CORS configurato correttamente
- [ ] Autenticazione implementata (JWT, Bearer token)
- [ ] Dokumentazione API con docstrings
- [ ] Test API scritti

### ✅ Per Bot
- [ ] Token salvato in .env (non nel codice!)
- [ ] Handler per comandi principali
- [ ] Event handler per messaggi/join/etc
- [ ] Logging per debug
- [ ] Rate limiting per evitare ban

### ✅ Deployment
- [ ] Variabili d'ambiente configurate su server
- [ ] requirements.txt aggiornato
- [ ] Virtual environment sul server
- [ ] Processo manager (PM2, systemd, Docker)
- [ ] Monitor e log centrali

---

## COMPARAZIONE QUICK: Scripting vs Backend vs Bot

| Aspetto | Scripting | Backend | Bot |
|---------|-----------|---------|-----|
| **Scopo** | Automazione task | API server | Interazione chat |
| **Trigger** | Timer/cron | HTTP request | Evento (messaggio/comando) |
| **Ciclo vita** | Una volta eseguito | Sempre in ascolto | Sempre in ascolto |
| **Dipendenze** | Minime | FastAPI, SQLAlchemy | discord.py, python-telegram-bot |
| **Testing** | Unit test | API test (pytest) | Handler test |
| **Deploy** | Cron job, Task Scheduler | Uvicorn su server | Bot hosting |

---

## GUIDA RAPIDA: Iniziare un Nuovo Progetto

### 1. Setup base

```bash
mkdir mio_progetto
cd mio_progetto
python -m venv venv
venv\Scripts\activate
pip install fastapi uvicorn sqlalchemy pydantic
```

### 2. Creare main.py

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
def root():
    return {"messaggio": "Hello World"}
```

### 3. Avviare server

```bash
uvicorn main:app --reload
# Server live su http://localhost:8000
```

### 4. Visitare docs

```
http://localhost:8000/docs  # Swagger UI
http://localhost:8000/redoc  # ReDoc
```

---

## RISORSE UTILI

- **Documentazione FastAPI**: https://fastapi.tiangolo.com/
- **Python Docs**: https://docs.python.org/3/
- **discord.py**: https://discordpy.readthedocs.io/
- **python-telegram-bot**: https://python-telegram-bot.readthedocs.io/
- **SQLAlchemy**: https://docs.sqlalchemy.org/

---

**Complimenti!** Adesso hai le basi per:
✅ Automatizzare task con Python  
✅ Creare API backend con FastAPI  
✅ Sviluppare bot per Discord/Telegram  

Inizia con un progetto semplice e scala gradualmente! 🚀
