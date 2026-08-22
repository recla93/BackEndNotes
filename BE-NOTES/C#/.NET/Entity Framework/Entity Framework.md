---
topic: "Entity Framework Core"
nav_prev: "[[ASP.NET Core.md]]"
nav_next: "[[Configurazione e DI.md]]"
---

Entity Framework Core (EF Core) è l'ORM (Object-Relational Mapper) di Microsoft per .NET. Mappa oggetti C# a tabelle relazionali, traduce LINQ in SQL, e gestisce change tracking, migration, e caching.

## Perché esiste
Scrivere SQL a mano è ripetitivo e fragile: mapping manuale tra classi e tabelle, concatenazione di stringhe SQL, gestione delle connessioni. EF Core automatizza tutto: le query LINQ diventano SQL sicuro (no injection), le classi diventano tabelle, le property diventano colonne.

## Setup

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

## DbContext e modelli

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Prodotto> Prodotti { get; set; }
    public DbSet<Categoria> Categorie { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configurazione Fluent API (più potente degli attributi)
        modelBuilder.Entity<Prodotto>(entity =>
        {
            entity.ToTable("Prodotti");
            entity.HasKey(p => p.Id);
            entity.Property(p => p.Nome).HasMaxLength(200).IsRequired();
            entity.Property(p => p.Prezzo).HasColumnType("decimal(18,2)");
            entity.HasOne(p => p.Categoria)
                  .WithMany(c => c.Prodotti)
                  .HasForeignKey(p => p.CategoriaId);
        });
    }
}

public class Prodotto
{
    public int Id { get; init; }
    public string Nome { get; set; }
    public decimal Prezzo { get; set; }
    public int CategoriaId { get; set; }
    public Categoria? Categoria { get; set; }
}
```

## Operazioni CRUD

```csharp
// CREATE
var prodotto = new Prodotto { Nome = "Laptop", Prezzo = 999m };
db.Prodotti.Add(prodotto);
await db.SaveChangesAsync();

// READ
var prodotti = await db.Prodotti
    .Where(p => p.Prezzo > 50)
    .OrderBy(p => p.Nome)
    .Include(p => p.Categoria)       // eager loading
    .ToListAsync();

var singolo = await db.Prodotti.FindAsync(id);  // per PK

// UPDATE
var p = await db.Prodotti.FindAsync(id);
if (p is not null)
{
    p.Prezzo = 899m;
    await db.SaveChangesAsync();  // change tracking automatico
}

// DELETE
var toDelete = await db.Prodotti.FindAsync(id);
if (toDelete is not null)
{
    db.Prodotti.Remove(toDelete);
    await db.SaveChangesAsync();
}
```

Il change tracking traccia automaticamente le modifiche: `SaveChangesAsync` genera UPDATE/DELETE/INSERT per le entità modificate.

## Migrations

```bash
# Creare migrazione
dotnet ef migrations add InitialCreate

# Applicare al database
dotnet ef database update

# Script SQL
dotnet ef migrations script -o migrazioni.sql

# Revert
dotnet ef database update NomeMigrazionePrecedente
```

## Query avanzate

```csharp
// Proiezioni
var result = await db.Prodotti
    .Where(p => p.Prezzo > 50)
    .Select(p => new { p.Nome, p.Prezzo, Categoria = p.Categoria!.Nome })
    .ToListAsync();

// Paginazione
var page = await db.Prodotti
    .OrderBy(p => p.Nome)
    .Skip(page * size)
    .Take(size)
    .ToListAsync();

// Raw SQL
var prodotti = await db.Prodotti
    .FromSql($"SELECT * FROM Prodotti WHERE Prezzo > {minPrezzo}")
    .ToListAsync();
```

## EF Core vs Dapper

| | EF Core | Dapper |
|---|---|---|
| Tipo | ORM Full | Micro-ORM |
| Query | LINQ → SQL | SQL manuale |
| Change tracking | ✅ Automatico | ❌ Manuale |
| Performance | Buona | Eccellente |
| Migration | ✅ Built-in | ❌ Manuali |
| Learning curve | Media | Bassa |
| Quando usare | CRUD complessi, domini ricchi, team grandi | Query semplici, performance critiche, legacy |

## Riferimenti

- [EF Core Documentation](https://learn.microsoft.com/en-us/ef/core/)
- [EF Core Performance](https://learn.microsoft.com/en-us/ef/core/performance/)
