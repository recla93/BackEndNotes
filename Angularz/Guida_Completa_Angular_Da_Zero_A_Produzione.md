# 🚀 Guida Completa Angular: Da Zero a Produzione

## Indice
1. [[#⚙️ Setup Progetto da Zero|Setup Progetto da Zero]]
2. [[#📁 Struttura Cartelle Consigliata|Struttura Cartelle]]
3. [[#🔧 Configurazione Ambiente|Configurazione Ambiente]]
4. [[#📊 Flusso Dati in Angular|Flusso Dati in Angular]]
5. [[#🎯 Services: Fondamenti|Services: Fondamenti]]
6. [[#🌐 Backend Detached (API Esterna)|Backend Detached (API Esterna)]]
7. [[#🔗 Backend Integrato (Frontend)|Backend Integrato (Frontend)]]
8. [[#⚡ Angular Signals (Nuovo!)|Angular Signals (Nuovo!)]]
9. [[#✅ Checklist: Nuovo Progetto Angular|Checklist Finale]]

---

# ⚙️ Setup Progetto da Zero

## Passo 1: Installare Node.js e Angular CLI

```bash
# Scarica Node.js da https://nodejs.org (versione LTS)

# Verifica installazione
node --version
npm --version

# Installa Angular CLI
npm install -g @angular/cli

# Verifica
ng version
```

## Passo 2: Creare Nuovo Progetto Angular

```bash
# Crea progetto
ng new mio-progetto
cd mio-progetto

# Scegli opzioni:
# - Routing? Yes
# - Style? CSS/SCSS (dipende da te)

# Avvia il server
ng serve

# Visita: http://localhost:4200
```

## Passo 3: Installare Dipendenze Comuni

```bash
# HttpClient (già incluso di default)

# RxJS (già incluso)

# TypeScript (già incluso)

# Opzionale: TailwindCSS per styling
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

---

# 📁 Struttura Cartelle Consigliata

```
mio-progetto/
├── src/
│   ├── app/
│   │   ├── core/                    # Servizi, guards, interceptors
│   │   │   ├── services/
│   │   │   │   ├── user.service.ts
│   │   │   │   ├── product.service.ts
│   │   │   │   └── api.service.ts   # Servizio generico HTTP
│   │   │   └── guards/
│   │   │       └── auth.guard.ts
│   │   │
│   │   ├── shared/                  # Componenti, pipe, directive condivise
│   │   │   ├── components/
│   │   │   ├── pipes/
│   │   │   └── directives/
│   │   │
│   │   ├── features/                # Feature modules (lazy loaded)
│   │   │   ├── dashboard/
│   │   │   │   ├── dashboard.component.ts
│   │   │   │   ├── dashboard.module.ts
│   │   │   │   └── dashboard-routing.module.ts
│   │   │   ├── products/
│   │   │   └── settings/
│   │   │
│   │   ├── app.component.ts
│   │   ├── app.module.ts
│   │   └── app-routing.module.ts
│   │
│   ├── assets/                      # Immagini, font, etc.
│   ├── styles/                      # CSS globale
│   ├── environments/                # Dev e Prod config
│   │   ├── environment.ts
│   │   └── environment.prod.ts
│   └── main.ts
│
├── angular.json
├── tsconfig.json
├── package.json
└── README.md
```

---

# 🔧 Configurazione Ambiente

## Passo 1: File di Configurazione

### `src/environments/environment.ts` (Sviluppo)
```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api',  // Backend locale
  apiTimeout: 30000
};
```

### `src/environments/environment.prod.ts` (Produzione)
```typescript
export const environment = {
  production: true,
  apiUrl: 'https://api.miodominio.com/api',  // Backend remoto
  apiTimeout: 30000
};
```

## Passo 2: Importare Config nel Servizio

```typescript
import { environment } from '../../environments/environment';

export class UserService {
  private apiUrl = `${environment.apiUrl}/users`;
  
  constructor(private http: HttpClient) {}
}
```

## Passo 3: HttpClientModule in AppModule

```typescript
// src/app/app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    HttpClientModule,  // ← Necessario per http.get(), post(), etc.
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

---

# 📊 Flusso Dati in Angular

## Architettura Completa

```
┌──────────────────────────────────────────────────────────────┐
│                    ANGULAR FRONTEND                           │
├──────────────────────────────────────────────────────────────┤
│                                                                │
│  User clicks button                                            │
│        ↓                                                       │
│  Component method called: loadUsers()                         │
│        ↓                                                       │
│  Component calls Service: userService.getUsers()              │
│        ↓                                                       │
│  Service calls HttpClient: this.http.get(apiUrl)              │
│        ↓                                                       │
│  HttpClient emits Observable                                  │
│                                                                │
└──────────────────────────────────────────────────────────────┘
           ↓ (HTTP GET Request)
      ────────────────────────
           ↓
┌──────────────────────────────────────────────────────────────┐
│                    BACKEND API                                │
├──────────────────────────────────────────────────────────────┤
│                                                                │
│  Route handler: GET /api/users                                │
│        ↓                                                       │
│  Business Logic (queries, validation)                         │
│        ↓                                                       │
│  Database Query                                               │
│        ↓                                                       │
│  Format response as JSON                                      │
│                                                                │
└──────────────────────────────────────────────────────────────┘
           ↓ (HTTP 200 + JSON)
      ────────────────────────
           ↓
┌──────────────────────────────────────────────────────────────┐
│              ANGULAR COMPONENT (CONTINUATION)                 │
├──────────────────────────────────────────────────────────────┤
│                                                                │
│  Observable emits data in .subscribe()                        │
│        ↓                                                       │
│  next() callback: received data                               │
│        ↓                                                       │
│  Component updates: this.users = data                         │
│        ↓                                                       │
│  Change Detection triggers                                    │
│        ↓                                                       │
│  Template renders: *ngFor loop                                │
│        ↓                                                       │
│  UI aggiornata!                                               │
│                                                                │
└──────────────────────────────────────────────────────────────┘
```

---

# 🎯 Services: Fondamenti

## Che Cos'è un Service?

Un **Service** è una classe che:
- Contiene logica di business
- Effettua chiamate HTTP
- Gestisce lo stato dell'app
- È riutilizzabile in più componenti

## Creare un Service

```bash
ng generate service core/services/user
# Crea: src/app/core/services/user.service.ts
```

## Service Completo con CRUD

```typescript
// src/app/core/services/user.service.ts
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, tap } from 'rxjs/operators';
import { environment } from '../../environments/environment';

export interface User {
  id: number;
  nome: string;
  email: string;
  ruolo: 'admin' | 'user';
}

@Injectable({
  providedIn: 'root'  // Singleton disponibile app-wide
})
export class UserService {
  private apiUrl = `${environment.apiUrl}/users`;

  constructor(private http: HttpClient) {}

  // GET tutti gli utenti
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl).pipe(
      tap(data => console.log('Utenti ricevuti:', data)),
      catchError(this.handleError)
    );
  }

  // GET un utente per ID
  getUserById(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`).pipe(
      catchError(this.handleError)
    );
  }

  // POST - Creare nuovo utente
  createUser(user: Omit<User, 'id'>): Observable<User> {
    return this.http.post<User>(this.apiUrl, user).pipe(
      tap(newUser => console.log('Utente creato:', newUser)),
      catchError(this.handleError)
    );
  }

  // PUT - Aggiornare utente
  updateUser(id: number, user: Partial<User>): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user).pipe(
      tap(updated => console.log('Utente aggiornato:', updated)),
      catchError(this.handleError)
    );
  }

  // DELETE - Eliminare utente
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`).pipe(
      tap(() => console.log(`Utente ${id} eliminato`)),
      catchError(this.handleError)
    );
  }

  // Gestione errori centralizzata
  private handleError(error: HttpErrorResponse) {
    let errorMessage = 'Si è verificato un errore';
    
    if (error.error instanceof ErrorEvent) {
      // Errore client
      errorMessage = `Errore: ${error.error.message}`;
    } else {
      // Errore server
      errorMessage = `Errore ${error.status}: ${error.message}`;
    }
    
    console.error(errorMessage);
    return throwError(() => new Error(errorMessage));
  }
}
```

---

# 🌐 Backend Detached (API Esterna)

## Cos'è Backend Detached?

Un backend **separato** dal frontend:
- API corre su un server diverso (e.g., Node.js, Spring Boot, Django)
- Frontend comunica tramite HTTP REST
- Database lontano dal frontend
- Scalabile e indipendente

## Architettura

```
┌────────────────────────┐         ┌──────────────────────┐
│  Angular Frontend      │         │  Node.js Backend     │
│  (localhost:4200)      │────────→│  (localhost:3000)    │
│                        │← ─ ─ ─ ─│                      │
└────────────────────────┘         └──────────────────────┘
                                             ↓
                                    ┌─────────────────┐
                                    │   Database      │
                                    │  (MongoDB/SQL)  │
                                    └─────────────────┘
```

## Esempio Backend Node.js/Express

```bash
# Crea cartella backend
mkdir backend
cd backend
npm init -y
npm install express cors
```

```javascript
// backend/server.js
const express = require('express');
const cors = require('cors');

const app = express();

// Middleware
app.use(cors());  // ← Importante! Permette al frontend di chiamare questo backend
app.use(express.json());

// Sample data
let users = [
  { id: 1, nome: 'Marco', email: 'marco@example.com', ruolo: 'admin' },
  { id: 2, nome: 'Giulia', email: 'giulia@example.com', ruolo: 'user' }
];

// GET tutti
app.get('/api/users', (req, res) => {
  res.json(users);
});

// GET per ID
app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'Non trovato' });
  res.json(user);
});

// POST
app.post('/api/users', (req, res) => {
  const newUser = {
    id: Math.max(...users.map(u => u.id), 0) + 1,
    ...req.body
  };
  users.push(newUser);
  res.status(201).json(newUser);
});

// PUT
app.put('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'Non trovato' });
  
  Object.assign(user, req.body);
  res.json(user);
});

// DELETE
app.delete('/api/users/:id', (req, res) => {
  users = users.filter(u => u.id !== parseInt(req.params.id));
  res.status(204).send();
});

const PORT = 3000;
app.listen(PORT, () => console.log(`Server in ascolto su porta ${PORT}`));
```

```bash
# Avvia backend
node server.js
# Output: Server in ascolto su porta 3000
```

## Frontend Comunica con Backend Detached

```typescript
// src/environments/environment.ts
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api',  // ← Backend esterno
};
```

```typescript
// Il service rimane uguale, ma ora chiama backend esterno
export class UserService {
  private apiUrl = `${environment.apiUrl}/users`;
  
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    // Effettua GET a http://localhost:3000/api/users
    return this.http.get<User[]>(this.apiUrl);
  }
}
```

---

# 🔗 Backend Integrato (Frontend)

## Cos'è Backend Integrato?

Backend "within" il progetto Angular:
- Express server + Angular nel stesso repo
- Utile per app monolitiche
- Deploy più semplice
- Dev: due processi separati

## Struttura Progetto

```
mio-progetto/
├── src/              # Angular Frontend
├── server/           # Express Backend
│   ├── server.js
│   ├── routes/
│   └── package.json
├── package.json      # Scripts per avviare entrambi
└── README.md
```

## Configurazione package.json (Frontend)

```json
{
  "name": "mio-progetto",
  "version": "1.0.0",
  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "frontend": "ng serve --port 4200",
    "backend": "node server/server.js",
    "dev": "concurrently \"npm run backend\" \"npm run frontend\"",
    "prod:build": "ng build --prod && npm start --prefix server"
  },
  "dependencies": {
    "@angular/common": "^17.0.0",
    "@angular/core": "^17.0.0",
    "@angular/platform-browser": "^17.0.0",
    "rxjs": "^7.8.0",
    "tslib": "^2.6.0",
    "zone.js": "^0.14.0"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "^17.0.0",
    "@angular/cli": "^17.0.0",
    "@angular/compiler-cli": "^17.0.0",
    "concurrently": "^8.2.0",
    "typescript": "^5.2.0"
  }
}
```

```bash
# Installa concurrently per avviare frontend+backend insieme
npm install --save-dev concurrently
```

## Backend Integrato (Express)

```javascript
// server/server.js
const express = require('express');
const path = require('path');
const cors = require('cors');

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// API Routes
app.use('/api', require('./routes/users'));

// Serve Angular static files (dopo build)
app.use(express.static(path.join(__dirname, '../dist/mio-progetto')));

// SPA fallback
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, '../dist/mio-progetto/index.html'));
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server in ascolto su porta ${PORT}`));
```

```javascript
// server/routes/users.js
const express = require('express');
const router = express.Router();

let users = [
  { id: 1, nome: 'Marco', email: 'marco@example.com' }
];

router.get('/users', (req, res) => res.json(users));
router.post('/users', (req, res) => {
  const newUser = { id: users.length + 1, ...req.body };
  users.push(newUser);
  res.status(201).json(newUser);
});

module.exports = router;
```

## Avviare Frontend + Backend Integrato

```bash
# Avvia entrambi
npm run dev

# Output:
# Frontend: http://localhost:4200
# Backend: http://localhost:3000
```

---

# ⚡ Angular Signals (Nuovo!)

## Cos'è un Signal?

**Signals** è una nuova API di Angular (v17+) per gestire lo stato reattivo.
- Più performante di RxJS per alcuni casi
- Sintassi più semplice
- Change detection fine-grained

## Signals vs RxJS

| Aspetto | Signals | RxJS Observable |
|---------|---------|-----------------|
| **Sintassi** | `signal()` | `new Observable()` |
| **Lettura** | `value()` | `.subscribe()` |
| **Aggiornamento** | `.set()`, `.update()` | `.next()` |
| **Performance** | Più veloce | Più flessibile |
| **Uso** | State locale | Flussi async |

## Creare Signals

```typescript
import { signal, computed, effect } from '@angular/core';

// Signal semplice
const count = signal(0);

// Leggere valore
console.log(count());  // 0

// Aggiornare
count.set(5);
console.log(count());  // 5

// Update con funzione
count.update(v => v + 1);
console.log(count());  // 6

// Computed: dipende da altri signals
const doubled = computed(() => count() * 2);
console.log(doubled());  // 12

// Effect: side effect quando signal cambia
effect(() => {
  console.log('Count è ora:', count());
});
```

## Service con Signals

```typescript
// src/app/core/services/user-signal.service.ts
import { Injectable, signal, computed } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { environment } from '../../environments/environment';

export interface User {
  id: number;
  nome: string;
  email: string;
}

@Injectable({
  providedIn: 'root'
})
export class UserSignalService {
  private apiUrl = `${environment.apiUrl}/users`;
  
  // Signal: lista di utenti
  users = signal<User[]>([]);
  
  // Signal: loading state
  isLoading = signal(false);
  
  // Signal: errore
  error = signal<string | null>(null);
  
  // Computed: numero di utenti
  userCount = computed(() => this.users().length);
  
  // Computed: utenti admin
  adminUsers = computed(() => 
    this.users().filter(u => u.ruolo === 'admin')
  );

  constructor(private http: HttpClient) {}

  // Fetch users e aggiorna signal
  loadUsers(): void {
    this.isLoading.set(true);
    this.error.set(null);
    
    this.http.get<User[]>(this.apiUrl).subscribe({
      next: (data) => {
        this.users.set(data);
        this.isLoading.set(false);
      },
      error: (err) => {
        this.error.set('Errore nel caricamento');
        this.isLoading.set(false);
      }
    });
  }

  // Aggiungi utente
  addUser(user: Omit<User, 'id'>): void {
    this.http.post<User>(this.apiUrl, user).subscribe({
      next: (newUser) => {
        this.users.update(users => [...users, newUser]);
      }
    });
  }

  // Elimina utente
  removeUser(id: number): void {
    this.http.delete(`${this.apiUrl}/${id}`).subscribe({
      next: () => {
        this.users.update(users => 
          users.filter(u => u.id !== id)
        );
      }
    });
  }
}
```

## Componente con Signals

```typescript
// src/app/features/users/users.component.ts
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { UserSignalService } from '../../core/services/user-signal.service';

@Component({
  selector: 'app-users',
  standalone: true,  // ← Componente standalone (Angular 14+)
  imports: [CommonModule],
  template: `
    <div class="container">
      <h2>Utenti</h2>
      
      <!-- Loading -->
      <p *ngIf="userService.isLoading()">⏳ Caricamento...</p>
      
      <!-- Errore -->
      <p *ngIf="userService.error()" class="error">
        ❌ {{ userService.error() }}
      </p>
      
      <!-- Numero utenti (Computed) -->
      <p>Totale utenti: <strong>{{ userService.userCount() }}</strong></p>
      <p>Admin: <strong>{{ userService.adminUsers().length }}</strong></p>
      
      <!-- Lista utenti -->
      <table *ngIf="!userService.isLoading()">
        <thead>
          <tr>
            <th>ID</th>
            <th>Nome</th>
            <th>Email</th>
            <th>Azioni</th>
          </tr>
        </thead>
        <tbody>
          <tr *ngFor="let user of userService.users()">
            <td>{{ user.id }}</td>
            <td>{{ user.nome }}</td>
            <td>{{ user.email }}</td>
            <td>
              <button (click)="userService.removeUser(user.id)">
                Elimina
              </button>
            </td>
          </tr>
        </tbody>
      </table>
      
      <!-- Form aggiungi utente -->
      <form (ngSubmit)="addUser()">
        <input #nameInput type="text" placeholder="Nome" />
        <input #emailInput type="email" placeholder="Email" />
        <button type="submit">Aggiungi</button>
      </form>
    </div>
  `,
  styles: [`
    .container { max-width: 800px; margin: 20px auto; }
    table { width: 100%; border-collapse: collapse; }
    th, td { border: 1px solid #ddd; padding: 10px; text-align: left; }
    th { background-color: #f5f5f5; }
    button { padding: 5px 10px; cursor: pointer; }
    .error { color: red; }
  `]
})
export class UsersComponent implements OnInit {
  constructor(public userService: UserSignalService) {}

  ngOnInit(): void {
    this.userService.loadUsers();
  }

  addUser(): void {
    // Logica aggiunta utente
  }
}
```

---

# 🎯 Comparazione: RxJS vs Signals vs Entrambi

## Quando Usare RxJS Observable

```typescript
// Flussi asincroni complessi
getUsers(): Observable<User[]> {
  return this.http.get<User[]>(apiUrl).pipe(
    filter(users => users.length > 0),
    map(users => users.sort((a, b) => a.nome.localeCompare(b.nome))),
    debounceTime(300),
    distinctUntilChanged()
  );
}
```

## Quando Usare Signals

```typescript
// Stato locale semplice
export class Component {
  users = signal<User[]>([]);
  selectedUser = signal<User | null>(null);
  
  selectUser(user: User) {
    this.selectedUser.set(user);
  }
}
```

## Hybrid Approach (Consigliato)

```typescript
// Service: RxJS per HTTP, Signals per stato
@Injectable()
export class UserService {
  users = signal<User[]>([]);
  
  // RxJS: gestisce flusso HTTP
  private users$ = this.http.get<User[]>(apiUrl).pipe(
    tap(data => this.users.set(data))  // Aggiorna signal
  );
  
  getUsers$(): Observable<User[]> {
    return this.users$;
  }
}

// Componente: legge signal
@Component({...})
export class UserListComponent {
  constructor(private userService: UserService) {}
  
  // Legge signal direttamente
  users = this.userService.users;
}
```

---

# ✅ Checklist: Nuovo Progetto Angular

## Fase 1: Setup
- [ ] Installa Node.js (LTS)
- [ ] Installa Angular CLI: `npm install -g @angular/cli`
- [ ] Crea progetto: `ng new mio-progetto`
- [ ] Crea struttura cartelle (core, features, shared)

## Fase 2: Ambiente
- [ ] Configura `environment.ts` e `environment.prod.ts`
- [ ] Imposta `apiUrl` corretto
- [ ] Importa `HttpClientModule` in `AppModule`

## Fase 3: Services
- [ ] Crea servizi in `core/services/`
- [ ] Usa `@Injectable({ providedIn: 'root' })`
- [ ] Implementa CRUD methods
- [ ] Gestisci errori con `catchError()`

## Fase 4: Backend (Scegli uno)
- [ ] **Backend Detached**: crea API separata, configura CORS
- [ ] **Backend Integrato**: Express nella stessa repo, configure concurrently

## Fase 5: Componenti
- [ ] Crea componenti in `features/`
- [ ] Inietta servizio via constructor
- [ ] Sottoscrivi Observable in `ngOnInit`
- [ ] Manage loading/error states

## Fase 6: Template
- [ ] Visualizza dati con `*ngFor`
- [ ] Gestisci loading con `*ngIf`
- [ ] Mostra errori all'utente
- [ ] Form per create/update

## Fase 7: Testing
- [ ] Testa servizi mock HTTP
- [ ] Testa componenti con TestBed
- [ ] Esegui test: `ng test`

## Fase 8: Deploy
- [ ] Build per produzione: `ng build --prod`
- [ ] Deploy frontend su Netlify/Vercel/AWS
- [ ] Deploy backend su Heroku/Railway/AWS
- [ ] Aggiorna `environment.prod.ts` con URL reale

---

# 📚 Sommario: Service Pattern

```
┌─ Signal Pattern ─────────────────────────┐
│                                          │
│ users = signal<User[]>([])               │
│ count = computed(() => users().length)   │
│ In template: {{ users() }}               │
│                                          │
└──────────────────────────────────────────┘

┌─ Observable Pattern ─────────────────────┐
│                                          │
│ users$ = this.http.get<User[]>(url)      │
│ In component: subscription               │
│ In template: users$ | async              │
│                                          │
└──────────────────────────────────────────┘

┌─ Hybrid Pattern (Consigliato) ───────────┐
│                                          │
│ Service: RxJS per HTTP + Signals per state
│ Component: legge signals direttamente    │
│ Template: {{ signal() }}                 │
│                                          │
└──────────────────────────────────────────┘
```

---

# 🚀 Prossimi Passi

1. **Crea nuovo progetto**: `ng new my-app`
2. **Setup struttura**: segui le cartelle consigliate
3. **Configura ambiente**: `environment.ts` con apiUrl
4. **Crea servizio**: `ng generate service core/services/user`
5. **Crea backend**: Express + CORS
6. **Crea componente**: `ng generate component features/users`
7. **Test**: `ng serve` e visita `http://localhost:4200`
8. **Deploy**: build e deploy su hosting

---

# 📖 Risorse

- [Angular Docs](https://angular.io/docs)
- [RxJS Guide](https://rxjs.dev/)
- [Angular Signals](https://angular.io/guide/signals)
- [HttpClient](https://angular.io/guide/http)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

