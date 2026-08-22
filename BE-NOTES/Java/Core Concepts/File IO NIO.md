---
topic: "File I/O e NIO — Java"
nav_prev: "[[Method Class.md]]"
nav_next: "[[Java Best Practices.md]]"
---

Java offre due API per I/O su file: `java.io` (classica, a blocchi) e `java.nio.file` (moderna, introdotta in Java 7). NIO è più espressiva e sicura, con supporto a path, filesystem operations e file attributes.

`java.io` usa classi come `File`, `FileReader`, `BufferedReader`. `java.nio.file` usa `Path`, `Files`, `FileSystem`. Preferisci NIO per codice nuovo.

## Path e Files (NIO)

```java
import java.nio.file.*;

// Creare un path
Path path = Path.of("dati", "input.txt");          // Java 11+
Path path2 = Paths.get("dati", "input.txt");       // Java 7+
Path assoluto = Path.of("/home/user/app/config.properties");

// Informazioni
path.getFileName();       // "input.txt"
path.getParent();         // "dati"
path.toAbsolutePath();    // /current/dir/dati/input.txt
Files.exists(path);
Files.isDirectory(path);
Files.size(path);         // long (bytes)
```

`Path.of()` (Java 11+) sostituisce `Paths.get()`. Usa path relativi o assoluti. `Path` è immutabile e thread-safe.

## Lettura e scrittura (NIO)

```java
import java.nio.file.*;
import java.util.List;

// Leggere tutto il file in una stringa (Java 11+)
String content = Files.readString(path);

// Leggere righe in una lista
List<String> lines = Files.readAllLines(path);

// Scrivere stringa
Files.writeString(path, "Ciao Mondo");              // Java 11+

// Scrivere lista di righe
Files.write(path, List.of("riga1", "riga2"));

// Aggiungere a file esistente
Files.writeString(path, "altra riga\n", StandardOpenOption.APPEND);

// Copiare, spostare, cancellare
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
Files.move(source, target);
Files.delete(path);
```

`readString()`/`writeString()` (Java 11+) semplificano drasticamente le operazioni base. Default encoding: UTF-8. Usa sempre try-with-resources per stream e reader.

## BufferedReader/Writer (IO classico)

```java
import java.io.*;

// Lettura riga per riga (file grandi)
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}

// Scrittura
try (BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {
    writer.write("Ciao Mondo");
    writer.newLine();
}

// Con encoding specifico
try (BufferedReader reader = new BufferedReader(
        new InputStreamReader(new FileInputStream("file.txt"), "UTF-8"))) {
    // ...
}
```

`BufferedReader`/`BufferedWriter` sono utili per file grandi (streaming riga per riga). `FileReader`/`FileWriter` usano il charset di default del sistema. Per encoding specifico, usa `InputStreamReader`/`OutputStreamWriter`.

## Files.walk e directory stream

```java
import java.nio.file.*;
import java.util.stream.Stream;

// Camminare ricorsivamente una directory
try (Stream<Path> stream = Files.walk(Path.of("src"))) {
    stream.filter(Files::isRegularFile)
          .forEach(System.out::println);
}

// Listare file in una directory (non ricorsivo)
try (Stream<Path> stream = Files.list(Path.of("."))) {
    stream.forEach(System.out::println);
}

// Trovare file per pattern
Path start = Path.of("src");
int maxDepth = 10;
try (Stream<Path> stream = Files.find(start, maxDepth,
        (path, attrs) -> path.toString().endsWith(".java"))) {
    stream.forEach(System.out::println);
}
```

`Files.walk()` cammina ricorsivamente una directory (attenzione: depth illimitata di default). `Files.list()` non ricorsivo. Il `try-with-resources` è obbligatorio per chiudere lo stream.

## File Attributes

```java
import java.nio.file.*;
import java.nio.file.attribute.*;

Path path = Path.of("file.txt");

// Attributi di base
FileTime lastModified = Files.getLastModifiedTime(path);
long size = Files.size(path);
boolean isDir = Files.isDirectory(path);
boolean isHidden = Files.isHidden(path);

// Attributi POSIX (Unix)
PosixFileAttributes attrs = Files.readAttributes(
    path, PosixFileAttributes.class);
attrs.permissions();     // rw-r--r--
attrs.owner().getName();
```

`Files.readAttributes()` è più efficiente di chiamate separate per ogni attributo. Usa `BasicFileAttributes` per OS indipendenti, `PosixFileAttributes` per Unix.

## File temporanei

```java
// Creare file temporaneo
Path tempFile = Files.createTempFile("prefisso", ".txt");
Files.writeString(tempFile, "dati temporanei");

// Creare directory temporanea
Path tempDir = Files.createTempDirectory("cache");
tempFile.toFile().deleteOnExit();  // cancellato alla fine della JVM
```

I file temporanei in `java.io.tmpdir` vengono cancellati automaticamente alla chiusura della JVM se usi `deleteOnExit()`. Per sicurezza, cancellali esplicitamente.

## InputStream/OutputStream (binari)

```java
import java.io.*;

// Copia binaria (foto, zip, ecc.)
try (InputStream in = new FileInputStream("source.jpg");
     OutputStream out = new FileOutputStream("dest.jpg")) {
    byte[] buffer = new byte[8192];
    int read;
    while ((read = in.read(buffer)) != -1) {
        out.write(buffer, 0, read);
    }
}

// Copia con NIO (più efficiente)
Files.copy(Path.of("source.jpg"), Path.of("dest.jpg"),
    StandardCopyOption.REPLACE_EXISTING);
```

Per file binari, `InputStream`/`OutputStream` operano su byte. `Files.copy()` con NIO è più efficiente e compatto. Buffer tipico: 8KB (allineato a page size).

## Errori comuni

- **Dimenticare il charset**: `FileReader` usa charset di sistema (non portabile). Specifica sempre UTF-8.
- **Non chiudere risorse**: `try-with-resources` risolve. `Stream` di `Files.walk()` va chiuso.
- **Leggere tutto in memoria per file grandi**: `Files.readAllLines()` carica tutto. Per file grandi, streaming con `BufferedReader` o `Files.lines()`.
- **Path con separatore hard-coded**: `"dati/file.txt"` non funziona su Windows. Usa `Path.of("dati", "file.txt")`.
- **`File.exists()` dopo `File.delete()`**: race condition. API NIO riduce questi problemi.
- **Confondere assoluto e relativo**: verifica con `path.toAbsolutePath()` per debug.

## Best Practices & Conventions

- Preferisci **NIO** (`Path`, `Files`) per codice nuovo.
- Usa **try-with-resources** sempre per risorse I/O.
- Per file piccoli (< 10MB): `Files.readString()` / `Files.writeString()`.
- Per file grandi: streaming con `Files.lines()` o `BufferedReader`.
- Specifica sempre **UTF-8** esplicitamente.
- Per I/O su rete, bufferizza sempre (`BufferedInputStream`, `BufferedReader`).
- Per path, usa `Path.of()` o `Paths.get()` — mai concatenazione di stringhe con separatori.
