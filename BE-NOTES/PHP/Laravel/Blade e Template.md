---
topic: "Laravel — Blade e Template"
tags: [laravel, blade, template, views, components, layout]
nav_prev: "[[Eloquent e Database.md]]"
nav_next: "[[Services e Repository.md]]"
---

Riferimento ufficiale: [laravel.com/docs/blade](https://laravel.com/docs/blade) | [laravel.com/docs/views](https://laravel.com/docs/views)

Blade è il template engine di Laravel. Compila i template in PHP cached (`storage/framework/views/`). Supporta component, layout, slot, e direttive custom. I file hanno estensione `.blade.php` e vivono in `resources/views/`.

## Sintassi di base

```blade
{{-- resources/views/users/index.blade.php --}}

{{-- Echo con htmlspecialchars --}}
<h1>{{ $title }}</h1>
<p>{{ $user->name }}</p>

{{-- Echo non escapato (ATTENZIONE XSS) --}}
{!! $rawHtml !!}

{{-- Direttive di controllo --}}
@if (count($users) > 0)
    <p>Trovati {{ count($users) }} utenti</p>
@elseif (isset($noResults))
    <p>Nessun risultato</p>
@else
    <p>Caricamento...</p>
@endif

{{-- Loop --}}
@foreach ($users as $user)
    <div>{{ $user->name }} ({{ $user->email }})</div>
@endforeach

@for ($i = 0; $i < 10; $i++)
    <span>{{ $i }}</span>
@endforelse ($users as $user)
    <p>Nessun utente</p>
@endforelse

{{-- Variabili del loop --}}
@foreach ($users as $user)
    @if ($loop->first) <ul> @endif
    <li class="{{ $loop->even ? 'even' : 'odd' }}">
        {{ $loop->iteration }}. {{ $user->name }}
    </li>
    @if ($loop->last) </ul> @endif
@endforeach
```

## Layout (template inheritance)

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html>
<head>
    <title>@yield("title", "Default Title")</title>
    @stack("styles")
</head>
<body>
    <nav>
        @section("sidebar")
            <ul>
                <li><a href="/">Home</a></li>
                <li><a href="/users">Users</a></li>
            </ul>
        @show
    </nav>

    <main>
        @yield("content")
    </main>

    @stack("scripts")
</body>
</html>
```

```blade
{{-- resources/views/users/index.blade.php --}}
@extends("layouts.app")

@section("title", "Lista Utenti")

@section("sidebar")
    @parent  {{-- include il sidebar genitore --}}
    <li><a href="/users/create">Nuovo Utente</a></li>
@endsection

@section("content")
    <h1>Utenti</h1>
    @include("users._table", ["users" => $users])
@endsection

@push("styles")
    <link rel="stylesheet" href="/css/users.css">
@endpush

@push("scripts")
    <script src="/js/users.js"></script>
@endpush
```

## Component (Blade X — moderno)

```blade
{{-- resources/views/components/alert.blade.php --}}
@props([
    "type" => "info",
    "message" => "",
    "dismissible" => false,
])

<div {{ $attributes->merge(["class" => "alert alert-$type"]) }}>
    {{ $message }}
    @if ($dismissible)
        <button type="button" class="close">&times;</button>
    @endif
    {{ $slot }}  {{-- contenuto tra i tag --}}
</div>
```

```blade
{{-- Uso del component --}}
<x-alert type="success" message="Operazione completata!" dismissible />

<x-alert type="error">
    <strong>Attenzione!</strong> Si è verificato un errore.
    {{-- $slot contiene questo HTML --}}
</x-alert>
```

### Component con render (più complesso)

```php
<?php
// app/View/Components/UserCard.php

namespace App\View\Components;

use App\Models\User;
use Illuminate\View\Component;

class UserCard extends Component
{
    public function __construct(
        public User $user,
        public bool $showEmail = false,
    ) {}

    public function render(): string
    {
        return view("components.user-card");
    }

    public function initials(): string
    {
        $words = explode(" ", $this->user->name);
        return collect($words)->map(fn($w) => strtoupper($w[0]))->implode("");
    }
}
```

```blade
{{-- resources/views/components/user-card.blade.php --}}
@props(["user", "showEmail" => false])

<div class="user-card">
    <div class="avatar">{{ $initials() }}</div>
    <h3>{{ $user->name }}</h3>
    @if ($showEmail)
        <p>{{ $user->email }}</p>
    @endif
</div>
```

```blade
{{-- Uso --}}
<x-user-card :user="$user" :showEmail="true" />
```

## Include e sub-view

```blade
{{-- resources/views/users/_table.blade.php --}}
<table>
    <thead>
        <tr><th>Nome</th><th>Email</th></tr>
    </thead>
    <tbody>
        @foreach ($users as $user)
            <tr>
                <td>{{ $user->name }}</td>
                <td>{{ $user->email }}</td>
            </tr>
        @endforeach
    </tbody>
</table>
```

```blade
{{-- Include con variabili aggiuntive --}}
@include("users._table", ["users" => $users, "showActions" => true])

{{-- Include condizionale (se view esiste) --}}
@includeIf("partials.analytics")

{{-- Include per collection --}}
@each("users._row", $users, "user", "users._empty")
```

## Direttive custom

```php
<?php
// AppServiceProvider::boot()

use Illuminate\Support\Facades\Blade;

Blade::directive("datetime", function (string $expression): string {
    return "<?php echo e(\$expression?->format('d/m/Y H:i')); ?>";
});

Blade::if("admin", function () {
    return auth()->user()?->isAdmin() ?? false;
});

Blade::if("role", function (string $role) {
    return auth()->user()?->hasRole($role) ?? false;
});
```

```blade
@datetime($user->created_at)   {{-- 19/06/2026 15:30 --}}

@admin
    <a href="/admin/users">Pannello Admin</a>
@endadmin

@role("editor")
    <button>Modifica</button>
@endrole
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `View [users.index] not found` | File blade mancante | Crea `resources/views/users/index.blade.php` |
| `Undefined variable: users` | Variabile non passata alla view | `return view("users.index", ["users" => $users])` |
| `Class "App\Http\Controllers\View" not found` | `View` facade importata male | `use Illuminate\Support\Facades\View;` |
| `Component [alert] not found` | Component class mancante | `php artisan make:component Alert` |
| `Only variables should be passed by reference` | Funzione direttamente in `@lang()` | Assegna a variabile prima |
| `stack con push duplicato` | `@push` chiamato due volte per stesso name | Usa `@prepend` o controlla logica |
| `parametro mancante in component` | Props non passate ma richieste | Verifica `@props` e attributi del tag |

## Best practice

- **Component per UI ricorrente** — alert, card, button, form input come component riutilizzabili
- **Layout inheritance** — `@extends` + `@section` per struttura pagina coerente
- **`{{ }}` sempre** — mai `{!! !!}` con dati utente (XSS)
- **Partial con underscore** — `_table.blade.php`, `_form.blade.php` per sub-view (convenzione)
- **`@props` con default** — ogni componente deve funzionare anche senza attributi
- **`$attributes->merge()`** — per classi CSS aggiuntive da chi usa il componente
- **Niente logica in template** — query e calcoli nel controller; template solo presentazione
- **Cache dei view compiled** — `php artisan view:cache` in produzione
- **`@push` / `@stack` per CSS/JS** — evita duplicazione; impila risorse per sezione
- **Component con class per logica** — se serve computazione (initials, badge color), usa classe invece di direttiva

## Cross-reference

- [[PHP/Laravel/Routing e Controller|Laravel — Routing e Controller]] — passare dati a view dai controller
- [[PHP/Symfony/Twig e Security|Symfony — Twig e Security]] — template engine alternativo
- [[PHP/Core Concepts/Classi e OOP|Classi e OOP]] — component classe, ereditarietà layout
