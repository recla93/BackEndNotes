---
topic: "Laravel — Eloquent e Database"
tags: [laravel, eloquent, orm, database, migration, relation, query]
nav_prev: "[[Routing e Controller.md]]"
nav_next: "[[Blade e Template.md]]"
---

Riferimento ufficiale: [laravel.com/docs/eloquent](https://laravel.com/docs/eloquent) | [laravel.com/docs/migrations](https://laravel.com/docs/migrations)

Eloquent è l'ORM di Laravel: implementa **Active Record** (ogni model = riga DB). Ogni tabella ha un Model corrispondente che incapsula query, relazioni, mutator/accessor, e scope.

```bash
php artisan make:model User -m          # Model + migration
php artisan make:model Post -mc         # Model + migration + controller
php artisan make:model Category -mfsc   # Model + migration + factory + seeder + controller
```

## Migration

```php
<?php
// database/migrations/xxxx_create_users_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create("users", function (Blueprint $table) {
            $table->id();
            $table->string("name", 100);
            $table->string("email", 255)->unique();
            $table->timestamp("email_verified_at")->nullable();
            $table->string("password");
            $table->rememberToken();
            $table->foreignId("role_id")->constrained("roles")->cascadeOnDelete();
            $table->timestamps();              // created_at, updated_at
            $table->softDeletes();             // deleted_at
            $table->index(["email", "role_id"]);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists("users");
    }
};
```

```bash
php artisan migrate                      # esegue migration pendenti
php artisan migrate:fresh                # drop tutte tabelle + migrate
php artisan migrate:fresh --seed         # fresh + seed dati iniziali
php artisan migrate:rollback             # annulla ultimo batch
php artisan make:migration add_phone_to_users_table  # alter table
```

## Model

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use SoftDeletes;

    protected $fillable = [
        "name", "email", "password", "role_id",
    ];

    protected $hidden = [
        "password", "remember_token",
    ];

    protected $casts = [
        "email_verified_at" => "datetime",
        "is_admin"          => "boolean",
        "config"            => "array",   // JSON column
    ];

    // Relazioni
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class, "user_id", "id");
    }

    public function role(): BelongsTo
    {
        return $this->belongsTo(Role::class);
    }

    // Accessor (attributo computato)
    public function getFullNameAttribute(): string
    {
        return "{$this->name} ({$this->email})";
    }

    // Mutator (modifica prima del save)
    public function setPasswordAttribute(string $value): void
    {
        $this->attributes["password"] = bcrypt($value);
    }

    // Scope
    public function scopeActive(Builder $query): Builder
    {
        return $query->whereNull("deleted_at");
    }

    public function scopeVerified(Builder $query): Builder
    {
        return $query->whereNotNull("email_verified_at");
    }

    // Local scope
    public function scopeByRole(Builder $query, string $role): Builder
    {
        return $query->whereHas("role", fn($q) => $q->where("name", $role));
    }
}
```

### Fillable vs Guarded

```php
<?php

// WHITELIST — solo questi campi possono essere mass-assigned (SICURO)
protected $fillable = ["name", "email", "password"];

// BLACKLIST — tutto è fillable tranne questi (RISCHIOSO)
protected $guarded = ["id", "is_admin"];
```

`$fillable` è più sicuro: esplicita quali campi possono essere impostati via `create()` o `update()` da input utente.

## CRUD con Eloquent

```php
<?php

// CREATE
$user = User::create([
    "name"     => "Mario",
    "email"    => "mario@example.com",
    "password" => "secret123",
]);

$user = new User(["name" => "Luigi"]);
$user->email = "luigi@example.com";
$user->save();

// READ
$user   = User::find(1);                // per PK
$user   = User::findOrFail(1);          // 404 se non trovato
$user   = User::where("email", "mario@example.com")->first();
$active = User::active()->get();        // scope custom

// Aggregazione
$count = User::where("role_id", 1)->count();
$maxId = User::max("id");

// Paginazione
$users = User::paginate(15);            // paginazione con links
$users = User::simplePaginate(15);      // solo prev/next

// UPDATE
User::where("id", 1)->update(["name" => "Mario Rossi"]);

$user = User::find(1);
$user->name = "Mario Verdi";
$user->save();

// DELETE
User::destroy(1);
User::where("active", false)->delete();
$user->delete();                        // soft delete se trait SoftDeletes
$user->forceDelete();                   // hard delete anche con soft
```

## Query Builder (non-ORM)

```php
<?php

use Illuminate\Support\Facades\DB;

$users = DB::table("users")
    ->select("users.*", "roles.name as role_name")
    ->join("roles", "users.role_id", "=", "roles.id")
    ->where("users.active", true)
    ->whereIn("users.role_id", [1, 2, 3])
    ->orderBy("users.name")
    ->paginate(15);

// Subquery
$posts = DB::table("posts")
    ->select("posts.*",
        DB::raw("(SELECT COUNT(*) FROM comments WHERE post_id = posts.id) as comments_count")
    )
    ->get();
```

## Relazioni

```php
<?php

// 1:1
public function profile(): HasOne
{
    return $this->hasOne(Profile::class);
}

// 1:N
public function posts(): HasMany
{
    return $this->hasMany(Post::class, "user_id");
}

// N:1
public function role(): BelongsTo
{
    return $this->belongsTo(Role::class);
}

// N:M
public function roles(): BelongsToMany
{
    return $this->belongsToMany(Role::class)
        ->withPivot("assigned_at")
        ->withTimestamps();
}

// HasManyThrough
public function comments(): HasManyThrough
{
    return $this->hasManyThrough(
        Comment::class,
        Post::class,
        "user_id",      // FK su posts
        "post_id",      // FK su comments
        "id",           // PK su users
        "id"            // PK su posts
    );
}
```

### Eager loading (N+1)

```php
<?php

// SENZA eager — N+1 query (lento)
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->user->name;  // 1 query per ogni post
}

// CON eager — 2 query totali
$posts = Post::with("user", "comments")->get();
foreach ($posts as $post) {
    echo $post->user->name;  // già caricato
}

// Eager loading condizionale
$users = User::with(["posts" => fn($q) => $q->where("published", true)])->get();

// Lazy eager loading (dopo la query iniziale)
$posts = Post::all();
if ($includeAuthor) {
    $posts->load("user");
}
```

## Mutator / Accessor (attributi computati)

```php
<?php

// Accessor — $user->full_name
protected function fullName(): Attribute
{
    return Attribute::make(
        get: fn(mixed $value, array $attributes) => "{$attributes["name"]} ({$attributes["email"]})",
    );
}

// Mutator — $user->password = "new" → hash automatico
protected function password(): Attribute
{
    return Attribute::make(
        set: fn(string $value) => bcrypt($value),
    );
}

// Accessor + Mutator
protected function username(): Attribute
{
    return Attribute::make(
        get: fn(mixed $value) => strtolower($value),
        set: fn(string $value) => ["username" => strtolower($value)],
    );
}
```

## Observer / Eventi

```php
<?php
// app/Observers/UserObserver.php

class UserObserver
{
    public function creating(User $user): void
    {
        $user->uuid = (string) Str::uuid();
    }

    public function created(User $user): void
    {
        Log::info("Nuovo utente: {$user->email}");
    }
}

// AppServiceProvider::boot()
User::observe(UserObserver::class);
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `MassAssignmentException` | Tentativo di fill su campo non in `$fillable` | Aggiungi campo a `$fillable` |
| `Relation "comments" not found` | Metodo relazione inesistente o errore nome | Verifica metodo `comments()` nel model |
| `N+1 query problem` | Caricamento relazione senza eager loading | Usa `with("relation")` |
| `Column not found: 1054 Unknown column 'deleted_at'` | Model con SoftDeletes ma tabella senza deleted_at | Migration con `->softDeletes()` |
| `Class "App\Models\Role" not found` | Model import sbagliato | `use App\Models\Role;` |
| `Call to undefined method ...::paginate()` | `get()` prima di `paginate()` | `Model::where(...)->paginate()` senza `get()` intermedio |
| `Integrity constraint violation: 1452` | FK violata per relazione | Crea prima il parent record |
| `Trying to get property 'name' of non-object` | Relazione null non gestita | `$user?->role?->name` (nullsafe) |

## Best practice

- **`$fillable` sempre** — mai `$guarded` (protezione mass assignment)
- **Eager loading per relazioni in vista/API** — evita N+1
- **Scope per query ricorrenti** — `scopeActive`, `scopeVerified` riusabili
- **Accessor vs Mutator** — accessor (get) per formattazione output; mutator (set) per normalizzazione input
- **Observer per side-effect** — log, eventi, cache clear su creazione/update
- **Resource per API response** — `php artisan make:resource UserResource` trasforma model
- **Migration atomiche** — una migration per modifica; mai modificare migration già eseguita
- **`findOrFail()` per risorse** — 404 automatico se non trovato
- **Chunk per grandi dataset** — `User::chunk(100, fn($users) => ...)` per memoria
- **Lazy loading desabilitato in produzione** — `Model::preventLazyLoading()` in AppServiceProvider

## Cross-reference

- [[PHP/Laravel/Services e Repository|Laravel — Services e Repository]] — repository pattern su Eloquent
- [[PHP/Database/PDO|PDO]] — Eloquent usa PDO sotto; connessione raw con DB facade
- [[PHP/Laravel/Artisan e Testing|Laravel — Artisan e Testing]] — model factory per test
- [[PHP/Core Concepts/Classi e OOP|Classi e OOP]] — trait, ereditarietà nei model
