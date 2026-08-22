---
topic: "ASP.NET Core"
nav_prev: "[[.NET.md]]"
nav_next: "[[Entity Framework.md]]"
---

ASP.NET Core è il framework web di Microsoft per costruire API REST, applicazioni MVC, Blazor e minimal API. Cross-platform, open source, e ad alte prestazioni (spesso nei primi 3 del TechEmpower Benchmarks).

## Modelli applicativi

### Minimal API (C# 9+, .NET 6+)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Endpoint dichiarativi
app.MapGet("/api/prodotti", async (AppDbContext db) =>
    await db.Prodotti.ToListAsync());

app.MapGet("/api/prodotti/{id}", async (int id, AppDbContext db) =>
    await db.Prodotti.FindAsync(id) is Prodotto p
        ? Results.Ok(p)
        : Results.NotFound());

app.MapPost("/api/prodotti", async (Prodotto prodotto, AppDbContext db) =>
{
    db.Prodotti.Add(prodotto);
    await db.SaveChangesAsync();
    return Results.Created($"/api/prodotti/{prodotto.Id}", prodotto);
});

app.Run();
```

### Controller-based API

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProdottiController : ControllerBase
{
    private readonly AppDbContext _db;
    
    public ProdottiController(AppDbContext db) => _db = db;
    
    [HttpGet]
    public async Task<ActionResult<List<Prodotto>>> GetAll()
        => await _db.Prodotti.ToListAsync();
    
    [HttpGet("{id}")]
    public async Task<ActionResult<Prodotto>> GetById(int id)
    {
        var prodotto = await _db.Prodotti.FindAsync(id);
        return prodotto is null ? NotFound() : Ok(prodotto);
    }
    
    [HttpPost]
    public async Task<ActionResult<Prodotto>> Create(Prodotto prodotto)
    {
        _db.Prodotti.Add(prodotto);
        await _db.SaveChangesAsync();
        return CreatedAtAction(nameof(GetById), new { id = prodotto.Id }, prodotto);
    }
}
```

## Middleware pipeline

```csharp
var app = builder.Build();

// Ordine della pipeline (critico!)
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();    // 1. Exception handling
}

app.UseAuthentication();                // 2. Authentication
app.UseAuthorization();                 // 3. Authorization
app.UseCors();                          // 4. CORS
app.MapControllers();                   // 5. Endpoint

app.Run();
```

L'ordine conta: exception → auth → authorization → CORS → endpoint. Se metti UseCors dopo MapControllers, CORS non funziona.

## Dependency Injection built-in

```csharp
var builder = WebApplication.CreateBuilder(args);

// Registrazione servizi
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

builder.Services.AddScoped<IProdottiService, ProdottiService>();
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
builder.Services.AddTransient<IEmailService, SmtpEmailService>();

builder.Services.AddControllers();

var app = builder.Build();
```

| Lifetime | Creato | Quando usare |
|----------|--------|--------------|
| `Transient` | Ogni richiesta | Servizi leggeri, stateless |
| `Scoped` | Per richiesta HTTP | DbContext, unit of work |
| `Singleton` | Una volta | Cache, configurazione, log |

## Configurazione

```csharp
// appsettings.json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=MyDb;..."
  },
  "Logging": {
    "LogLevel": { "Default": "Information" }
  },
  "Jwt": {
    "Key": "secret-key",
    "Issuer": "myapp"
  }
}

// Accesso tipizzato
public class JwtOptions
{
    public string Key { get; init; }
    public string Issuer { get; init; }
}

// Program.cs
builder.Services.Configure<JwtOptions>(builder.Configuration.GetSection("Jwt"));
```

## Riferimenti

- [ASP.NET Core documentation](https://learn.microsoft.com/en-us/aspnet/core/)
- [Minimal API overview](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [.NET Dependency Injection](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)
