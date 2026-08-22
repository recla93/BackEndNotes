---
topic: "Configurazione e Dependency Injection in .NET"
nav_prev: "[[Entity Framework.md]]"
---

.NET ha un sistema di configurazione e DI built-in, unificato e indipendente dal framework applicativo (ASP.NET Core, Worker Service, console app).

## Configurazione

## Perché esiste
Le applicazioni hanno bisogno di comportamenti diversi in ambienti diversi (dev, staging, production). Il sistema di configurazione .NET carica impostazioni da **multipli provider** a cascata, dove l'ultimo vince.

```csharp
var builder = WebApplication.CreateBuilder(args);

// L'ordine di caricamento (l'ultimo vince):
// 1. appsettings.json
// 2. appsettings.{Environment}.json
// 3. Variabili d'ambiente (ASPNETCORE_*, MySection__Key)
// 4. Argomenti da riga di comando
// 5. User secrets (sviluppo)

// Accesso tipizzato (Options pattern)
builder.Services.Configure<JwtSettings>(
    builder.Configuration.GetSection("Jwt"));

builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("Smtp"));
```

### Options pattern

```csharp
public class JwtSettings
{
    public string Key { get; init; } = "";
    public string Issuer { get; init; } = "";
    public int ExpiryMinutes { get; init; } = 60;
}

// appsettings.json
{
  "Jwt": {
    "Key": "supersecretkey123!",
    "Issuer": "MyApi",
    "ExpiryMinutes": 120
  }
}

// Uso con DI
public class AuthService
{
    private readonly JwtSettings _jwt;
    
    public AuthService(IOptions<JwtSettings> jwt)
    {
        _jwt = jwt.Value;  // IOptionsSnapshot per ricarica, IOptionsMonitor per notifiche
    }
}
```

| Interfaccia | Ricarica | Scoped | Quando usare |
|-------------|:---:|:---:|---|
| `IOptions<T>` | ❌ | Singleton | Config fissa, letta una volta |
| `IOptionsSnapshot<T>` | ✅ Per richiesta | Scoped | Config che cambia, ricaricata a ogni richiesta |
| `IOptionsMonitor<T>` | ✅ In tempo reale | Singleton | Config che cambia, notifiche di change |

## Dependency Injection

## Perché esiste
La DI built-in elimina la necessità di container DI esterni (Unity, Ninject) per la maggior parte dei casi. Fornisce tre lifetime e supporta factory pattern, open generics, decorator.

```csharp
var services = new ServiceCollection();

// Registrazione base
services.AddScoped<IUserService, UserService>();
services.AddSingleton<ICache, MemoryCache>();
services.AddTransient<IEmailService, SmtpEmailService>();

// Con factory lambda
services.AddScoped<IUserService>(sp =>
{
    var logger = sp.GetRequiredService<ILogger<UserService>>();
    return new UserService(logger, "custom-config");
});

// Open generics
services.AddTransient(typeof(IRepository<>), typeof(EfRepository<>));

// Tutti i tipi che implementano un'interfaccia
services.Scan(scan => scan
    .FromAssemblyOf<Program>()
    .AddClasses(classes => classes.AssignableTo<ITransientService>())
    .AsImplementedInterfaces()
    .WithTransientLifetime());
```

### TryAdd — evita doppia registrazione

```csharp
// Se già registrato, non sovrascrive
services.TryAddScoped<IUserService, UserService>();
```

### Service lifetime rules

| Lifetime | Istanza | Thread-safe? | Disposable? |
|----------|---------|:---:|:---:|
| `Transient` | Ogni risoluzione | Sì (nuova ogni volta) | Sì (se IDisposable) |
| `Scoped` | Una per scope (richiesta HTTP) | No (condivisa) | Sì |
| `Singleton` | Una per applicazione | Sì (DEVE esserlo) | Sì (alla chiusura) |

**Regola d'oro**: non catturare un servizio Scoped in un Singleton — è un **captive dependency**. Il servizio Scoped vivrà per sempre come Singleton.

```csharp
// ❌ Captive dependency: scoped catturato in singleton
services.AddSingleton<IEmailService>(sp =>
{
    var db = sp.GetRequiredService<AppDbContext>();  // scoped!
    return new EmailService(db);  // db vive per sempre!
});

// ✅ Factory: risolvi nel momento d'uso
services.AddSingleton<IEmailService>(sp =>
{
    var factory = sp.GetRequiredService<IDbContextFactory<AppDbContext>>();
    return new EmailService(factory);
});
```

## Worker Services (background jobs)

```csharp
public class DataSyncWorker : BackgroundService
{
    private readonly ILogger<DataSyncWorker> _logger;
    
    public DataSyncWorker(ILogger<DataSyncWorker> logger)
    {
        _logger = logger;
    }
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Sync eseguita alle {Time}", DateTime.UtcNow);
            await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
        }
    }
}

// Program.cs
builder.Services.AddHostedService<DataSyncWorker>();
```

## Serilog (logging strutturato)

```csharp
// Program.cs
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .Enrich.FromLogContext()
    .WriteTo.Console(outputTemplate: "[{Timestamp:HH:mm:ss} {Level}] {Message}{NewLine}")
    .WriteTo.File("logs/app.log", rollingInterval: RollingInterval.Day)
    .CreateLogger();

builder.Host.UseSerilog();

// Uso nei servizi
public class ProdottiService
{
    private readonly ILogger<ProdottiService> _logger;
    
    public ProdottiService(ILogger<ProdottiService> logger) => _logger = logger;
    
    public async Task CreateAsync(Prodotto p)
    {
        _logger.LogInformation("Creazione prodotto {Nome} con prezzo {Prezzo}", p.Nome, p.Prezzo);
        // ...
    }
}
```

## Riferimenti

- [Configuration in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration)
- [Dependency Injection in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)
- [Options pattern in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/options)
- [Serilog](https://serilog.net/)
