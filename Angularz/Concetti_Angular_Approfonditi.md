# 🎓 Concetti Angular Approfonditi: Pipes, Features, API Service

---

# 📦 PIPES

## Cosa Sono?

Un **Pipe** è uno strumento in Angular che **trasforma dati nel template (HTML)** per la visualizzazione.

**Non modifica i dati originali**, solo come vengono mostrati.

## Cosa Fanno?

I Pipes:
- Formattano dati (date, numeri, valute)
- Trasformano stringhe (maiuscole, minuscole, slice)
- Filtrano array
- Convertono valori per il display

## Sintassi

```html
{{ valore | pipeName }}
{{ valore | pipeName:parametro }}
{{ valore | pipeName1 | pipeName2 }}
```

Il simbolo `|` è il "pipe operator".

---

## Pipes Built-in (Già Inclusi)

### 1. **UpperCase Pipe**
```typescript
// Component
nome = 'marco';

// Template
{{ nome | uppercase }}
// Risultato: MARCO
```

### 2. **LowerCase Pipe**
```typescript
// Component
titolo = 'HELLO WORLD';

// Template
{{ titolo | lowercase }}
// Risultato: hello world
```

### 3. **Currency Pipe** (Formato Valuta)
```typescript
// Component
prezzo = 99.99;

// Template
{{ prezzo | currency }}
// Risultato: $99.99

// Con parametro specifico
{{ prezzo | currency:'EUR' }}
// Risultato: €99.99

{{ prezzo | currency:'EUR':'symbol':'1.2-2' }}
// Risultato: €99.99
```

### 4. **Percent Pipe** (Percentuale)
```typescript
// Component
tasso = 0.75;

// Template
{{ tasso | percent }}
// Risultato: 75%

{{ tasso | percent:'1.2-2' }}
// Risultato: 75.00%
```

### 5. **Date Pipe** (Formattazione Date)
```typescript
// Component
data = new Date('2026-05-18');

// Template
{{ data | date }}
// Risultato: May 18, 2026

{{ data | date:'short' }}
// Risultato: 5/18/26

{{ data | date:'shortDate' }}
// Risultato: 5/18/26

{{ data | date:'fullDate' }}
// Risultato: Sunday, May 18, 2026

{{ data | date:'dd/MM/yyyy HH:mm' }}
// Risultato: 18/05/2026 14:35

{{ data | date:'it-IT' }}  // Locale italiano
// Risultato: 18 mag 2026
```

### 6. **Number Pipe** (Formattazione Numeri)
```typescript
// Component
numero = 1234.567;

// Template
{{ numero | number }}
// Risultato: 1,234.567

{{ numero | number:'1.2-2' }}
// Risultato: 1,234.57  (1 cifra prima, 2-2 dopo)

{{ numero | number:'2.0-0' }}
// Risultato: 1,235  (arrotondato)
```

### 7. **Slice Pipe** (Taglia Stringhe/Array)
```typescript
// Component
testo = 'Angular è fantastico';
numeri = [1, 2, 3, 4, 5];

// Template
{{ testo | slice:0:7 }}
// Risultato: Angular

{{ numeri | slice:1:4 }}
// Risultato: 2, 3, 4
```

### 8. **Async Pipe** (Gestisce Observable/Promise)
```typescript
// Component
users$ = this.userService.getUsers();  // Observable

// Template
// Invece di this.subscribe() nel component:
{{ users$ | async }}  // Pipe async fa subscribe automaticamente

// Nel *ngFor
<div *ngFor="let user of users$ | async">
  {{ user.nome }}
</div>

// IMPORTANTE: Usa async per non fare memory leak
```

### 9. **Json Pipe** (Debug: mostra JSON)
```typescript
// Component
utente = { nome: 'Marco', email: 'marco@example.com' };

// Template
{{ utente | json }}
// Risultato: { "nome": "Marco", "email": "marco@example.com" }

// Utile per debugging!
<pre>{{ utente | json }}</pre>
```

### 10. **KeyValue Pipe** (Itera su oggetti)
```typescript
// Component
settings = {
  tema: 'scuro',
  lingua: 'italiano',
  notifiche: true
};

// Template
<div *ngFor="let item of settings | keyvalue">
  {{ item.key }}: {{ item.value }}
</div>

// Risultato:
// tema: scuro
// lingua: italiano
// notifiche: true
```

---

## Pipes Personalizzati (Custom Pipes)

### Creare un Custom Pipe

```bash
ng generate pipe pipes/truncate
# Crea: src/app/shared/pipes/truncate.pipe.ts
```

### Esempio: Pipe per Troncare Testo

```typescript
// src/app/shared/pipes/truncate.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'truncate',
  standalone: true  // Angular 14+
})
export class TruncatePipe implements PipeTransform {
  transform(valore: string, limite: number = 20): string {
    if (!valore) return valore;
    
    return valore.length > limite 
      ? valore.substring(0, limite) + '...'
      : valore;
  }
}
```

### Usare il Custom Pipe

```typescript
// Component
testo = 'Questo è un testo molto lungo che voglio troncare';

// Template
{{ testo | truncate:15 }}
// Risultato: Questo è un te...
```

### Esempio 2: Pipe per Formattare Telefono

```typescript
// src/app/shared/pipes/phone.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'phone',
  standalone: true
})
export class PhonePipe implements PipeTransform {
  transform(valore: string): string {
    if (!valore || valore.length < 10) return valore;
    
    // Trasforma: 3331234567 → +39 333 123 4567
    return `+39 ${valore.substring(0, 3)} ${valore.substring(3, 6)} ${valore.substring(6)}`;
  }
}
```

```html
<!-- Template -->
{{ telefono | phone }}
<!-- Risultato: +39 333 123 4567 -->
```

### Esempio 3: Pipe Impure (con Parametri Dinamici)

```typescript
// src/app/shared/pipes/filter.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'filter',
  standalone: true,
  pure: false  // ← Importante: ricalcola ogni volta
})
export class FilterPipe implements PipeTransform {
  transform(array: any[], chiave: string, valore: any): any[] {
    if (!array || !chiave) return array;
    
    return array.filter(item => item[chiave] === valore);
  }
}
```

```typescript
// Component
utenti = [
  { id: 1, nome: 'Marco', ruolo: 'admin' },
  { id: 2, nome: 'Giulia', ruolo: 'user' },
  { id: 3, nome: 'Paolo', ruolo: 'admin' }
];

// Template
<div *ngFor="let utente of utenti | filter:'ruolo':'admin'">
  {{ utente.nome }}
</div>

// Risultato: Marco, Paolo
```

---

## Catena di Pipes (Piping Chain)

```html
{{ data | date:'dd/MM/yyyy' | uppercase }}
<!-- Trasforma data, poi converti a maiuscole -->

{{ prezzo | currency:'EUR' | uppercase }}
<!-- Formatta come valuta, poi maiuscole -->

{{ testo | slice:0:50 | uppercase }}
<!-- Taglia a 50 caratteri, poi maiuscole -->
```

---

# 🗂️ FEATURES

## Cosa Sono?

Una **Feature** è un **modulo funzionale dell'applicazione** che raggruppa:
- Componenti correlati
- Servizi specifici della feature
- Routing interno
- Template e stili

**Concetto:** Dividi l'app in pezzi piccoli e indipendenti.

## Cosa Fanno?

Le Features:
- Organizzano il codice logicamente
- Permettono "lazy loading" (carica solo quando necessario)
- Rendono l'app scalabile
- Facilitano il riutilizzo

## Struttura di una Feature

```
src/app/features/
├── dashboard/
│   ├── components/
│   │   ├── dashboard.component.ts
│   │   ├── dashboard.component.html
│   │   └── dashboard.component.css
│   ├── dashboard.module.ts
│   └── dashboard-routing.module.ts
│
├── products/
│   ├── components/
│   │   ├── product-list.component.ts
│   │   └── product-detail.component.ts
│   ├── services/
│   │   └── product.service.ts
│   ├── products.module.ts
│   └── products-routing.module.ts
│
└── settings/
    ├── components/
    │   └── settings.component.ts
    ├── settings.module.ts
    └── settings-routing.module.ts
```

---

## Esempio: Feature Dashboard

### 1. Creare il Modulo della Feature

```bash
ng generate module features/dashboard
ng generate module features/dashboard/dashboard-routing
ng generate component features/dashboard/dashboard
```

### 2. Dashboard Module

```typescript
// src/app/features/dashboard/dashboard.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { DashboardRoutingModule } from './dashboard-routing.module';
import { DashboardComponent } from './dashboard.component';

@NgModule({
  declarations: [DashboardComponent],
  imports: [
    CommonModule,
    DashboardRoutingModule
  ]
})
export class DashboardModule { }
```

### 3. Dashboard Routing

```typescript
// src/app/features/dashboard/dashboard-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { DashboardComponent } from './dashboard.component';

const routes: Routes = [
  { path: '', component: DashboardComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class DashboardRoutingModule { }
```

### 4. Dashboard Component

```typescript
// src/app/features/dashboard/dashboard.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-dashboard',
  templateUrl: './dashboard.component.html',
  styleUrls: ['./dashboard.component.css']
})
export class DashboardComponent implements OnInit {
  statistiche = {
    utenti: 120,
    ordini: 45,
    ricavi: 12500
  };

  ngOnInit() {
    console.log('Dashboard caricato');
  }
}
```

### 5. Dashboard Template

```html
<!-- src/app/features/dashboard/dashboard.component.html -->
<div class="dashboard">
  <h1>Dashboard</h1>
  
  <div class="stats">
    <div class="stat-card">
      <h3>Utenti</h3>
      <p>{{ statistiche.utenti }}</p>
    </div>
    
    <div class="stat-card">
      <h3>Ordini</h3>
      <p>{{ statistiche.ordini }}</p>
    </div>
    
    <div class="stat-card">
      <h3>Ricavi</h3>
      <p>{{ statistiche.ricavi | currency:'EUR' }}</p>
    </div>
  </div>
</div>
```

---

## Lazy Loading di Features

### App Routing Module

```typescript
// src/app/app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () => import('./features/dashboard/dashboard.module')
      .then(m => m.DashboardModule)
  },
  {
    path: 'products',
    loadChildren: () => import('./features/products/products.module')
      .then(m => m.ProductsModule)
  },
  {
    path: 'settings',
    loadChildren: () => import('./features/settings/settings.module')
      .then(m => m.SettingsModule)
  },
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

**Cosa fa il Lazy Loading?**
- **Carica il modulo SOLO quando navighi** a quella rotta
- Riduce il bundle size iniziale
- App più veloce al primo carico
- Bundle size per feature: ~50KB invece di 500KB all'inizio

---

## Feature vs Modulo Condiviso

| Feature Module | Shared Module |
|---|---|
| **Logica specifica** della feature | **Componenti riutilizzabili** in tutta l'app |
| **Lazy loaded** | **Importato ovunque** necessario |
| Componenti, servizi interni | Pipes, directives, componenti generici |
| Esempio: DashboardModule, ProductsModule | Esempio: CommonModule, SharedModule |

---

# 🔌 API SERVICE

## Cosa È?

Un **API Service** è un servizio Angular che:
- **Centralizza tutte le chiamate HTTP**
- **Gestisce la comunicazione con il backend**
- **Fornisce metodi riutilizzabili** per ogni operazione

**Concetto:** Un'interfaccia standardizzata tra frontend e backend.

## Cosa Fa?

L'API Service:
- Effettua GET, POST, PUT, DELETE
- Gestisce errori globalmente
- Imposta headers comuni (Authorization, Content-Type)
- Gestisce intercettori (per token, logging, etc.)
- Centralizza la logica di comunicazione

---

## Esempio: API Service Generico

### 1. Generic API Service

```typescript
// src/app/core/services/api.service.ts
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, tap, retry } from 'rxjs/operators';
import { environment } from '../../environments/environment';

@Injectable({
  providedIn: 'root'
})
export class ApiService {
  private apiUrl = environment.apiUrl;

  constructor(private http: HttpClient) {}

  // GET generico
  get<T>(endpoint: string): Observable<T> {
    return this.http.get<T>(`${this.apiUrl}${endpoint}`).pipe(
      tap(data => console.log('GET riuscito:', data)),
      retry(1),  // Retry automatico
      catchError(this.handleError)
    );
  }

  // GET con parametri
  getWithParams<T>(endpoint: string, params?: any): Observable<T> {
    return this.http.get<T>(`${this.apiUrl}${endpoint}`, { params }).pipe(
      retry(1),
      catchError(this.handleError)
    );
  }

  // POST generico
  post<T>(endpoint: string, data: any): Observable<T> {
    return this.http.post<T>(`${this.apiUrl}${endpoint}`, data).pipe(
      tap(result => console.log('POST riuscito:', result)),
      catchError(this.handleError)
    );
  }

  // PUT generico
  put<T>(endpoint: string, data: any): Observable<T> {
    return this.http.put<T>(`${this.apiUrl}${endpoint}`, data).pipe(
      tap(result => console.log('PUT riuscito:', result)),
      catchError(this.handleError)
    );
  }

  // PATCH generico
  patch<T>(endpoint: string, data: any): Observable<T> {
    return this.http.patch<T>(`${this.apiUrl}${endpoint}`, data).pipe(
      catchError(this.handleError)
    );
  }

  // DELETE generico
  delete<T>(endpoint: string): Observable<T> {
    return this.http.delete<T>(`${this.apiUrl}${endpoint}`).pipe(
      tap(() => console.log('DELETE riuscito')),
      catchError(this.handleError)
    );
  }

  // Gestione errori centralizzata
  private handleError(error: HttpErrorResponse) {
    let errorMessage = 'Errore sconosciuto';

    if (error.error instanceof ErrorEvent) {
      // Errore client
      errorMessage = `Errore: ${error.error.message}`;
    } else {
      // Errore server
      if (error.status === 401) {
        errorMessage = 'Non autorizzato. Effettua il login.';
      } else if (error.status === 403) {
        errorMessage = 'Accesso negato.';
      } else if (error.status === 404) {
        errorMessage = 'Risorsa non trovata.';
      } else if (error.status === 500) {
        errorMessage = 'Errore server interno.';
      } else {
        errorMessage = `Errore ${error.status}: ${error.message}`;
      }
    }

    console.error(errorMessage);
    return throwError(() => new Error(errorMessage));
  }
}
```

### 2. Usare API Service nei Services Specifici

```typescript
// src/app/core/services/user.service.ts
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { ApiService } from './api.service';

export interface User {
  id: number;
  nome: string;
  email: string;
}

@Injectable({
  providedIn: 'root'
})
export class UserService {
  constructor(private api: ApiService) {}

  getUsers(): Observable<User[]> {
    return this.api.get<User[]>('/users');
  }

  getUserById(id: number): Observable<User> {
    return this.api.get<User>(`/users/${id}`);
  }

  createUser(user: Omit<User, 'id'>): Observable<User> {
    return this.api.post<User>('/users', user);
  }

  updateUser(id: number, user: Partial<User>): Observable<User> {
    return this.api.put<User>(`/users/${id}`, user);
  }

  deleteUser(id: number): Observable<void> {
    return this.api.delete<void>(`/users/${id}`);
  }
}
```

```typescript
// src/app/core/services/product.service.ts
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { ApiService } from './api.service';

export interface Product {
  id: number;
  nome: string;
  prezzo: number;
  categoria: string;
}

@Injectable({
  providedIn: 'root'
})
export class ProductService {
  constructor(private api: ApiService) {}

  getProducts(): Observable<Product[]> {
    return this.api.get<Product[]>('/products');
  }

  getProductsByCategory(categoria: string): Observable<Product[]> {
    return this.api.getWithParams<Product[]>('/products', { categoria });
  }

  getProductById(id: number): Observable<Product> {
    return this.api.get<Product>(`/products/${id}`);
  }

  createProduct(product: Omit<Product, 'id'>): Observable<Product> {
    return this.api.post<Product>('/products', product);
  }

  updateProduct(id: number, product: Partial<Product>): Observable<Product> {
    return this.api.put<Product>(`/products/${id}`, product);
  }

  deleteProduct(id: number): Observable<void> {
    return this.api.delete<void>(`/products/${id}`);
  }
}
```

### 3. Usare nei Componenti

```typescript
// src/app/features/users/users.component.ts
import { Component, OnInit } from '@angular/core';
import { UserService } from '../../core/services/user.service';

@Component({
  selector: 'app-users',
  templateUrl: './users.component.html'
})
export class UsersComponent implements OnInit {
  users: any[] = [];
  loading = false;

  constructor(private userService: UserService) {}

  ngOnInit() {
    this.loadUsers();
  }

  loadUsers() {
    this.loading = true;
    this.userService.getUsers().subscribe({
      next: (data) => {
        this.users = data;
        this.loading = false;
      },
      error: (err) => {
        console.error('Errore:', err);
        this.loading = false;
      }
    });
  }
}
```

---

## API Service con Interceptors

### Cos'è un Interceptor?

Un **Interceptor** intercetta tutte le richieste HTTP e le modifica prima di inviarle (o dopo riceverle).

**Casi d'uso:**
- Aggiungere Authorization header
- Gestire token JWT
- Logging richieste
- Gestire errori globalmente

### Creare un Interceptor

```bash
ng generate interceptor core/interceptors/auth
```

```typescript
// src/app/core/interceptors/auth.interceptor.ts
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor() {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Ottieni il token da localStorage
    const token = localStorage.getItem('auth_token');

    // Se esiste token, aggiungi l'header Authorization
    if (token) {
      req = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`
        }
      });
    }

    // Prosegui con la richiesta
    return next.handle(req);
  }
}
```

### Registrare l'Interceptor

```typescript
// src/app/app.module.ts
import { NgModule } from '@angular/core';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
import { AuthInterceptor } from './core/interceptors/auth.interceptor';

@NgModule({
  imports: [HttpClientModule],
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true
    }
  ]
})
export class AppModule { }
```

**Cosa fa?**
- Ogni richiesta HTTP automaticamente avrà l'header `Authorization: Bearer <token>`
- Non devi ripetere il token in ogni servizio
- Gestione centralizzata

---

## Flusso Completo: Component → Service → API Service → Backend

```
┌──────────────────────┐
│   User Component     │
│  loadUsers() method  │
└──────────┬───────────┘
           │ chiama
           ↓
┌──────────────────────┐
│   UserService        │
│ getUsers() method    │
└──────────┬───────────┘
           │ chiama
           ↓
┌──────────────────────┐
│   ApiService         │
│ get('/users')        │
└──────────┬───────────┘
           │ chiama
           ↓
┌──────────────────────┐
│   HttpClient         │
│ with Interceptor     │
└──────────┬───────────┘
           │ aggiunge token
           ↓
┌──────────────────────┐
│   HTTP Request       │
│ GET /api/users       │
│ Authorization: ...   │
└──────────┬───────────┘
           │
           ↓ (Internet)
           
┌──────────────────────┐
│   Backend API        │
│ Valida token         │
│ Query database       │
│ Response: JSON       │
└──────────┬───────────┘
           │
           ↓ (Internet)
           
┌──────────────────────┐
│   Interceptor        │
│ Log response         │
└──────────┬───────────┘
           │
           ↓
┌──────────────────────┐
│   ApiService         │
│ Gestisce errori      │
└──────────┬───────────┘
           │
           ↓
┌──────────────────────┐
│   UserService        │
│ Ritorna Observable   │
└──────────┬───────────┘
           │
           ↓
┌──────────────────────┐
│   Component          │
│ .subscribe() riceve  │
│ Aggiorna this.users  │
└──────────┬───────────┘
           │
           ↓
┌──────────────────────┐
│   Template           │
│ *ngFor su users      │
│ UI AGGIORNATA!       │
└──────────────────────┘
```

---

# 📚 Riassunto Visivo

## Pipes

```
Dato originale          Pipe                Risultato nel Template
─────────────          ────                ──────────────────────

'marco'          →  | uppercase       →     MARCO

new Date()       →  | date:'short'    →     5/18/26

99.99            →  | currency:'EUR'  →     €99.99

[1,2,3,4,5]      →  | slice:1:3       →     [2,3]
```

## Features

```
Feature = Modulo funzionale indipendente

App
├── Feature: Dashboard
│   ├── dashboard.component
│   ├── dashboard.service
│   └── dashboard-routing
│
├── Feature: Products
│   ├── product-list.component
│   ├── product-detail.component
│   ├── product.service
│   └── products-routing
│
└── Feature: Settings
    ├── settings.component
    └── settings-routing
```

## API Service

```
Component
    ↓
UserService (chiama ApiService)
    ↓
ApiService.get('/users')
    ↓
HttpClient + Interceptor
    ↓
Backend API
    ↓
Database
```

---

# ✅ Checklist di Comprensione

- [ ] Ho capito che i **Pipes trasformano dati nel template**
- [ ] Conosco i **Pipes built-in** comuni (date, currency, uppercase, etc.)
- [ ] So come creare **Custom Pipes** personalizzati
- [ ] Ho capito che le **Features** sono moduli funzionali indipendenti
- [ ] Ho capito il concetto di **Lazy Loading**
- [ ] So che **API Service centralizza tutte le richieste HTTP**
- [ ] Ho capito il **flusso Component → Service → ApiService → Backend**
- [ ] Conosco il concetto di **Interceptors** per aggiungere headers

