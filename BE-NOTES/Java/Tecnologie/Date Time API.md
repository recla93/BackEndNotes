---
topic: "Date Time API (java.time) — Java"
parent: "[[BE-NOTES/Java/Java|Java]]"
---

`java.time` (JSR-310, Java 8+) e l'API moderna per date e orari in Java. Sostituisce `java.util.Date` e `java.util.Calendar`, che erano mal progettati (mutabili, non thread-safe, mesi da 0, naming confuso).

L'API si basa su classi **immutabili** e thread-safe. I principi: `LocalDate` (solo data), `LocalTime` (solo ora), `LocalDateTime` (entrambi), `ZonedDateTime` (con fuso orario), `Instant` (timestamp Unix).

## LocalDate, LocalTime, LocalDateTime

```java
import java.time.*;

// Oggi
LocalDate oggi = LocalDate.now();
LocalTime adesso = LocalTime.now();
LocalDateTime ora = LocalDateTime.now();

// Date specifiche
LocalDate data = LocalDate.of(2024, Month.JANUARY, 15);
LocalTime orario = LocalTime.of(14, 30, 0);
LocalDateTime dataOra = LocalDateTime.of(2024, 1, 15, 14, 30);

// Parsing
LocalDate parsed = LocalDate.parse("2024-01-15");           // ISO
LocalDate custom = LocalDate.parse("15/01/2024", DateTimeFormatter.ofPattern("dd/MM/yyyy"));

// Manipolazione (immutabile: restituisce nuova istanza)
LocalDate domani = oggi.plusDays(1);
LocalDate ieri = oggi.minusDays(1);
LocalDate primoDelMese = oggi.withDayOfMonth(1);
```

Tutte le classi `java.time` sono immutabili. `plusDays()`, `minusMonths()`, `withDayOfMonth()` restituiscono nuove istanze. Il default parsing e ISO (`yyyy-MM-dd`). Per formati custom usa `DateTimeFormatter`.

## Duration e Period

```java
import java.time.*;

// Duration: differenza in secondi/nanos (per orari)
Duration durata = Duration.between(startTime, endTime);
durata.toHours();        // ore
durata.toMinutes();      // minuti
durata.getSeconds();     // secondi totali

// Period: differenza in giorni/mesi/anni (per date)
Period periodo = Period.between(startDate, endDate);
periodo.getYears();
periodo.getMonths();
periodo.getDays();

// Aggiungere duration/period
LocalDateTime nuova = ora.plus(Duration.ofHours(2));
LocalDate nuovaData = oggi.plus(Period.ofDays(30));
```

`Duration` misura tempo basato su secondi/nanos (per `LocalTime`, `Instant`). `Period` misura tempo basato su date (per `LocalDate`). Non confonderli: `Period.ofDays(30)` tiene conto dei mesi, `Duration.ofDays(30)` sono 30*24 ore esatte.

## Instant — timestamp Unix

```java
import java.time.*;

// Ora corrente in UTC
Instant ora = Instant.now();          // 2024-01-15T12:30:00Z

// Da timestamp Unix (millisecondi)
Instant fromEpoch = Instant.ofEpochMilli(System.currentTimeMillis());
long millis = ora.toEpochMilli();

// Conversione con ZonedDateTime
ZonedDateTime zdt = ora.atZone(ZoneId.of("Europe/Rome"));
Instant back = zdt.toInstant();

// Parsing ISO 8601
Instant parsed = Instant.parse("2024-01-15T12:30:00Z");
```

`Instant` e un timestamp UTC indipendente dal fuso orario. Ideale per: log, storing in database, comunicazione tra sistemi. `System.currentTimeMillis()` e l'equivalente legacy.

## ZonedDateTime e ZoneId

```java
import java.time.*;

// Fuso orario corrente
ZoneId fusoItalia = ZoneId.of("Europe/Rome");
ZonedDateTime oraItalia = ZonedDateTime.now(fusoItalia);

// Da LocalDateTime con fuso
LocalDateTime ora = LocalDateTime.now();
ZonedDateTime zoned = ora.atZone(ZoneId.of("Europe/Paris"));

// Cambiare fuso
ZonedDateTime aLondra = zoned.withZoneSameInstant(ZoneId.of("Europe/London"));

// Ora legale (gestita automaticamente)
ZonedDateTime change = ZonedDateTime.of(2024, 3, 31, 2, 30, 0, 0,
    ZoneId.of("Europe/Rome"));  // 2:30 non esiste! (passaggio ora legale)
```

La `Z` in `ZoneId` rappresenta UTC. `withZoneSameInstant()` cambia fuso mantenendo lo stesso istante. Le transizioni ora legale/solare sono gestite automaticamente: Java aggiusta l'ora se necessario.

## DateTimeFormatter

```java
import java.time.format.DateTimeFormatter;

// Formattazione
LocalDate oggi = LocalDate.now();
oggi.format(DateTimeFormatter.ISO_DATE);          // 2024-01-15
oggi.format(DateTimeFormatter.ofPattern("dd/MM/yyyy"));   // 15/01/2024
oggi.format(DateTimeFormatter.ofPattern("dd MMM yyyy", Locale.ITALIAN));  // 15 gen 2024

// Parsing
LocalDate parsed = LocalDate.parse("15/01/2024",
    DateTimeFormatter.ofPattern("dd/MM/yyyy"));

// Formatter thread-safe (si puo riusare)
DateTimeFormatter ITALIAN_DATE = DateTimeFormatter
    .ofPattern("dd/MM/yyyy")
    .withLocale(Locale.ITALY);
```

`DateTimeFormatter` e thread-safe (a differenza di `SimpleDateFormat` che non lo era). I pattern usano lettere (`yyyy`, `MM`, `dd`, `HH`, `mm`, `ss`). Predefinisci formatter come costanti statiche per riuso.

## Intervalli e query

```java
import java.time.temporal.*;

// ChronoUnit per differenze in unita specifiche
long giorni = ChronoUnit.DAYS.between(start, end);
long ore = ChronoUnit.HOURS.between(start, end);
long mesi = ChronoUnit.MONTHS.between(start, end);

// TemporalAdjuster: utility per manipolazioni comuni
import static java.time.temporal.TemporalAdjusters.*;

LocalDate primoGiornoMese = oggi.with(firstDayOfMonth());
LocalDate ultimoGiornoMese = oggi.with(lastDayOfMonth());
LocalDate prossimoLunedi = oggi.with(next(DayOfWeek.MONDAY));
LocalDate primoGiornoAnno = oggi.with(firstDayOfYear());

// Verificare date
oggi.isBefore(domani);
oggi.isAfter(ieri);
oggi.isLeapYear();  // bisestile
```

`TemporalAdjusters` offre manipolazioni pronte (inizio/fine mese, prossimo lunedi, ecc.). `ChronoUnit` calcola differenze in unita specifiche senza creare oggetti intermedi.

## Legacy API compatibility

```java
import java.util.Date;
import java.util.Calendar;

// Date → Instant
Date legacyDate = new Date();
Instant instant = legacyDate.toInstant();

// Instant → Date
Date back = Date.from(instant);

// Calendar → ZonedDateTime
Calendar cal = Calendar.getInstance();
ZonedDateTime zdt = ZonedDateTime.ofInstant(
    cal.toInstant(), cal.getTimeZone().toZoneId());

// java.sql.Date → LocalDate
java.sql.Date sqlDate = new java.sql.Date(System.currentTimeMillis());
LocalDate localDate = sqlDate.toLocalDate();
```

Per codice legacy, converti subito in `java.time` appena ricevi un `Date`/`Calendar`. JPA 2.2+ (Hibernate 5.3+) supporta nativamente `java.time`. Spring Boot converte automaticamente le date nei form.

## Errori comuni

- **`DateTimeParseException`**: formato non corrisponde. Usa `DateTimeFormatter.ofPattern(...)` con il pattern esatto.
- **Dimenticare che le classi sono immutabili**: `data.plusDays(1)` non modifica `data`, restituisce una nuova istanza.
- **Duration vs Period**: `Duration.ofDays(1)` sono 24 ore. `Period.ofDays(1)` e 1 giorno (considera ora legale). Usa `Duration` per `Instant`/`LocalTime`, `Period` per `LocalDate`.
- **TimeZone con nomi sbagliati**: `ZoneId.of("CET")` puo non funzionare. Usa `Europe/Rome`, `America/New_York`.
- **SimpleDateFormat in ambiente multi-thread**: non e thread-safe! Usa `DateTimeFormatter`.
- **Parsing di date senza anno**: `LocalDate.parse("15/01")` fallisce. Usa `MonthDay` per date senza anno.

## Best Practices & Conventions

- Usa sempre **`java.time`** per codice nuovo. Mai `java.util.Date` o `Calendar`.
- Per timestamp assoluti: **`Instant`**. Per date/orari: **`LocalDate`**, **`LocalTime`**, **`LocalDateTime`**.
- Per dati con fuso orario: **`ZonedDateTime`** o salva come `Instant` + `ZoneId` separato.
- Per storing in DB: `Instant` (UTC) o `LocalDateTime` (se il fuso e gestito dall'app).
- Per formattazione: definisci `DateTimeFormatter` come costanti statiche thread-safe.
- Per API JSON: configura Jackson per serializzare `LocalDate` come `yyyy-MM-dd` e `Instant` come ISO-8601.
