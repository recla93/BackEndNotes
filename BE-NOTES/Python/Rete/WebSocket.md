---
topic: "WebSocket — Python"
tags: [python, websocket, realtime, websockets, asyncio]
---
Riferimento ufficiale websockets: [websockets.readthedocs.io](https://websockets.readthedocs.io/)
Riferimento ufficiale FastAPI WebSocket: [fastapi.tiangolo.com/advanced/websocket](https://fastapi.tiangolo.com/advanced/websocket/)

WebSocket e un protocollo di comunicazione full-duplex su una singola connessione TCP. A differenza di HTTP (richiesta-risposta), WebSocket permette al server di inviare dati al client in qualsiasi momento, senza che il client lo richieda.

In Python si usa principalmente `websockets` (server/client standalone) o l'integrazione con FastAPI/Starlette. Le librerie `websocket-client` e `aiohttp` offrono alternative per il lato client.

Vedi anche:
[[BE-NOTES/Python/Tecnologie/Async e AsyncIO|Async e AsyncIO]],
[[BE-NOTES/Python/FastAPI/Setup e Primi Passi|FastAPI]],
[[BE-NOTES/Python/Rete/Requests e HTTP|Requests e HTTP]].

## Server WebSocket (libreria websockets)

```bash
pip install websockets
```

```python
import asyncio
import websockets

async def echo(websocket):
    """Handler: riceve un messaggio e lo rispedisce indietro."""
    async for message in websocket:
        print(f"Ricevuto: {message}")
        await websocket.send(f"Echo: {message}")

async def main():
    async with websockets.serve(echo, "localhost", 8765):
        print("Server WebSocket su ws://localhost:8765")
        await asyncio.Future()  # Run forever

asyncio.run(main())
```

La funzione handler riceve automaticamente la connessione. `async for message in websocket:` itera sui messaggi ricevuti. `websocket.send()` invia un messaggio. Il server rimane in ascolto finche `asyncio.Future()` non viene risolta.

## Client WebSocket

```python
import asyncio
import websockets

async def client():
    uri = "ws://localhost:8765"
    async with websockets.connect(uri) as websocket:
        await websocket.send("Ciao server!")
        response = await websocket.recv()
        print(f"Ricevuto: {response}")

asyncio.run(client())
```

`websockets.connect()` restituisce un WebSocket client. `send()` invia, `recv()` riceve (bloccante fino al prossimo messaggio). Usa `async with` per chiusura automatica.

## WebSocket in FastAPI

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"Ricevuto: {data}")
    except WebSocketDisconnect:
        print("Client disconnesso")
```

FastAPI gestisce WebSocket nativamente. `websocket.accept()` e obbligatorio per stabilire la connessione. `receive_text()`, `receive_bytes()`, `receive_json()` per diversi formati. `WebSocketDisconnect` cattura la disconnessione del client.

## WebSocket broadcast (chat room)

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import Set

app = FastAPI()
connections: Set[WebSocket] = set()

@app.websocket("/chat")
async def chat(websocket: WebSocket):
    await websocket.accept()
    connections.add(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            # Broadcast a tutti i client connessi
            for conn in connections:
                if conn != websocket:
                    await conn.send_text(data)
    except WebSocketDisconnect:
        connections.discard(websocket)
```

Il broadcast e un pattern comune: ogni messaggio ricevuto viene inoltrato a tutti gli altri client connessi (escluso il mittente). Attenzione: in produzione, proteggi da slow consumer e gestispi il backpressure.

## Client WebSocket con reconnect

```python
import asyncio
import websockets

async def listen_with_reconnect():
    uri = "ws://localhost:8765"
    retry_delay = 1
    while True:
        try:
            async with websockets.connect(uri) as ws:
                print("Connesso")
                retry_delay = 1  # Reset
                async for message in ws:
                    print(f"Ricevuto: {message}")
        except (websockets.ConnectionClosed, OSError) as e:
            print(f"Disconnesso: {e}. Riconnessione tra {retry_delay}s...")
            await asyncio.sleep(retry_delay)
            retry_delay = min(retry_delay * 2, 30)  # Backoff esponenziale
```

Il reconnect pattern con backoff esponenziale evita il thundering herd su riconnessioni massive. Raddoppia l'attesa a ogni tentativo fino a 30 secondi.

## WebSocket heartbeat e timeout

```python
import asyncio
import websockets

async def heartbeat(websocket, interval: int = 30):
    """Invia ping periodico per mantenere viva la connessione."""
    try:
        while True:
            await asyncio.sleep(interval)
            pong = await websocket.ping()
            await asyncio.wait_for(pong, timeout=5)
    except asyncio.TimeoutError:
        print("Heartbeat fallito")
        await websocket.close()

async def handler(websocket):
    # Avvia heartbeat in background
    asyncio.create_task(heartbeat(websocket))
    async for message in websocket:
        await websocket.send(f"OK: {message}")
```

Il heartbeat previene la chiusura della connessione da parte di proxy/load balancer per inattivita. `ping()` invia un frame ping, `pong.wait()` aspetta la risposta del client.

## State management e room

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from collections import defaultdict

app = FastAPI()
rooms: dict[str, set[WebSocket]] = defaultdict(set)

@app.websocket("/room/{room_name}")
async def room(websocket: WebSocket, room_name: str):
    await websocket.accept()
    rooms[room_name].add(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            for conn in rooms[room_name]:
                await conn.send_text(f"[{room_name}] {data}")
    except WebSocketDisconnect:
        rooms[room_name].discard(websocket)
        if not rooms[room_name]:
            del rooms[room_name]
```

Le chat room raggruppano client per canale. Ogni messaggio viene inoltrato solo ai membri della stessa room. Pattern usato in applicazioni real-time: chat, notifiche, collaborazione.

## Tabella comparativa librerie

| Libreria | Server | Client | Async | Integrazione |
|----------|--------|--------|-------|-------------|
| `websockets` | Eccellente | Eccellente | Si | Standalone |
| `FastAPI/Starlette` | Eccellente | No | Si | FastAPI nativo |
| `aiohttp` | Buono | Buono | Si | Web framework |
| `websocket-client` | No | Buono | No | Script sincroni |

## Errori comuni

- **Dimenticare `websocket.accept()` in FastAPI**: la connessione rimane in attesa. Sempre chiamare `accept()`.
- **Non gestire `WebSocketDisconnect`**: eccezione non catturata fa crashare l'handler.
- **Connessioni accumulate**: rimuovi sempre i WebSocket disconnessi dal set. Usa `discard()` (non rimuove se assente).
- **Send senza receive**: se il client smette di ricevere, il buffer si riempie. Gestisci backpressure.
- **WebSocket in ambiente serverless**: WebSocket richiede connessione persistente. Non funziona con Lambda/Cloud Functions.
- **Non gestire la chiusura lato client**: `async with` garantisce chiusura pulita.

## Best Practices & Conventions

- Usa **`websockets`** per server WebSocket standalone e microservizi.
- Usa **FastAPI WebSocket** quando sei gia in un'app FastAPI.
- Implementa **heartbeat** per mantenere connessioni attive attraverso proxy/load balancer.
- Usa **backoff esponenziale** lato client per riconnessioni.
- Pulisci sempre le connessioni morte dal registry.
- Per broadcast, gestisci **slow consumer**: se un client e lento, bufferizza o scarta messaggi.
- Proteggi i WebSocket con **autenticazione** (token in query string o al primo messaggio).
