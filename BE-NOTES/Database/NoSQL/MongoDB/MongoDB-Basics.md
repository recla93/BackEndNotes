# MongoDB - NoSQL Database

## Cos'è MongoDB?

**MongoDB** è un database **NoSQL** (Not Only SQL) che memorizza dati in formato JSON invece di tabelle SQL.

**NoSQL** = Flessibilità, scalabilità, struttura variabile

## Struttura

| SQL | MongoDB |
|-----|---------|
| Database | Database |
| Tabella | Collezione |
| Riga | Documento |
| Colonna | Campo |

## Documenti JSON

MongoDB memorizza **documenti** in formato JSON:

```json
{
    "_id": ObjectId("507f1f77bcf86cd799439011"),
    "nome": "Mario",
    "cognome": "Rossi",
    "età": 30,
    "indirizzo": {
        "via": "Via Roma 1",
        "città": "Milano"
    },
    "hobby": ["calcio", "tennis", "lettura"]
}
```

## Vantaggi di MongoDB

- **Flessibilità**: documenti con strutture diverse nella stessa collezione
- **Oggetti Annidati**: supporta strutture complesse
- **Array**: supporta liste di dati
- **Scalabilità**: facile scalare orizzontalmente

## Operazioni Basiche

### Insert
```javascript
db.persona.insertOne({
    nome: "Mario",
    cognome: "Rossi",
    eta: 30
})
```

### Read
```javascript
db.persona.find()                              // Tutti
db.persona.find({ nome: "Mario" })            // Dove nome = Mario
db.persona.find({ eta: { $gt: 25 } })         // Dove eta > 25
```

### Update
```javascript
db.persona.updateOne(
    { nome: "Mario" },
    { $set: { eta: 31 } }
)
```

### Delete
```javascript
db.persona.deleteOne({ nome: "Mario" })
```

## Query Avanzate

### Regex
```javascript
db.persona.find({ nome: /^M/ })  // Nomi che iniziano con M
```

### Aggregazione
```javascript
db.persona.aggregate([
    { $match: { eta: { $gt: 25 } } },
    { $group: { _id: "$citta", count: { $sum: 1 } } }
])
```

## Differenze da SQL

```javascript
// SQL
SELECT * FROM persona WHERE eta > 25 ORDER BY nome;

// MongoDB
db.persona.find({ eta: { $gt: 25 } }).sort({ nome: 1 })
```

## Quando Usare MongoDB

✅ Dati semi-strutturati
✅ Grandi volumi di dati
✅ Strutture variabili
✅ Sviluppo rapido

❌ Dati altamente strutturati
❌ Transazioni complesse
❌ Relazioni rigide
