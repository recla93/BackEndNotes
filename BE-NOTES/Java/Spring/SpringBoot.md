### Framework vs Libreria

| Framework | Libreria |
|-----------|----------|
| **Struttura completa** già pronta | **Componenti singoli** |
| Tu inserisci il tuo codice | Tu usi quello che ti serve |
| **Impone una logica** | **Massima libertà** |
| Esempio: Spring | Esempio: JDBC |

### Cos'è Spring?

**Spring** è un framework che **automatizza operazioni comuni** nella programmazione Java.

```
┌─────────────────────────────────────┐
│     Spring Framework                │
│ ✓ Gestisce creazione oggetti        │
│ ✓ Inietta dipendenze                │
│ ✓ Gestisce il ciclo di vita         │
│ ✓ Facilita web e database           │
│ ✓ Gestisce transazioni              │
└─────────────────────────────────────┘
```

### Spring Boot

**Spring Boot** è una distribuzione di Spring che semplifica la configurazione iniziale (auto-configuration, embedded server, etc).

## Inversion of Control

### Inversion of Control (IoC)

**IoC** significa che **Spring controlla** il programma, non il contrario:

```
Senza IoC (traditional):
main() → crea DAO → esegue query
main() ha il controllo

Con IoC (Spring):
Spring → crea DAO → inietta in Controller
Spring ha il controllo (Inversion)
```
