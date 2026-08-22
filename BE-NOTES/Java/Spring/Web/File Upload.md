---
topic: "File Upload in Spring Boot"
parent: "[[BE-NOTES/Java/Spring/Spring|Spring]]"
---

Spring Boot gestisce file upload tramite `MultipartFile`. Il client invia file con `multipart/form-data`. Spring converte automaticamente la richiesta in un oggetto `MultipartFile`.

Per upload di grandi dimensioni, configura limiti e buffer. Spring Boot usa Apache Commons FileUpload o Servlet 3.0 Part API.

## Configurazione

```properties
# application.properties
spring.servlet.multipart.enabled=true
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=50MB
spring.servlet.multipart.file-size-threshold=2KB
spring.servlet.multipart.location=/tmp/uploads
```

`max-file-size` limite per singolo file. `max-request-size` limite per richiesta totale (multipli file). `file-size-threshold` dimensione oltre la quale il file viene scritto su disco invece di memoria. `location` directory temporanea.

## Controller upload singolo

```java
import org.springframework.web.multipart.MultipartFile;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/files")
public class FileUploadController {

    @PostMapping("/upload")
    public ResponseEntity<String> uploadFile(@RequestParam("file") MultipartFile file) {
        if (file.isEmpty()) {
            return ResponseEntity.badRequest().body("File vuoto");
        }

        String fileName = file.getOriginalFilename();
        String contentType = file.getContentType();
        long size = file.getSize();

        log.info("Ricevuto file: nome={}, tipo={}, size={}",
            fileName, contentType, size);

        // Salva su disco
        file.transferTo(new File("uploads/" + fileName));

        return ResponseEntity.ok("File caricato: " + fileName);
    }
}
```

`@RequestParam("file")` lega il file dalla richiesta multipart. `MultipartFile` fornisce nome originale, tipo, dimensione, content. `transferTo()` salva su filesystem. `getBytes()` restituisce il contenuto in memoria.

## Upload multipli

```java
@PostMapping("/upload-multiple")
public ResponseEntity<List<String>> uploadMultipleFiles(
        @RequestParam("files") List<MultipartFile> files) {

    List<String> uploadedNames = new ArrayList<>();

    for (MultipartFile file : files) {
        if (!file.isEmpty()) {
            String fileName = file.getOriginalFilename();
            file.transferTo(new File("uploads/" + fileName));
            uploadedNames.add(fileName);
        }
    }

    return ResponseEntity.ok(uploadedNames);
}
```

`@RequestParam("files") List<MultipartFile>` accetta multipli file con lo stesso field name. Itera e salva ogni file. Attenzione a `max-request-size` per upload multipli.

## Upload con metadati

```java
@PostMapping("/upload-with-metadata")
public ResponseEntity<FileMetadata> uploadWithMetadata(
        @RequestParam("file") MultipartFile file,
        @RequestParam("description") String description,
        @RequestParam("tags") String tags,
        @RequestParam("visibility") Visibility visibility) {

    FileMetadata metadata = new FileMetadata();
    metadata.setOriginalName(file.getOriginalFilename());
    metadata.setSize(file.getSize());
    metadata.setContentType(file.getContentType());
    metadata.setDescription(description);
    metadata.setTags(List.of(tags.split(",")));
    metadata.setVisibility(visibility);

    fileService.save(file, metadata);

    return ResponseEntity.ok(metadata);
}
```

Combina file e metadati nella stessa richiesta multipart. `@RequestParam` per campi testuali insieme a `MultipartFile`. Utile per caricare documenti con descrizione, categoria, visibilita.

## Upload a Database (BLOB)

```java
@Entity
public class FileEntity {

    @Id
    @GeneratedValue
    private Long id;

    private String fileName;
    private String contentType;

    @Lob
    private byte[] data;

    // getter e setter
}

@Repository
public interface FileEntityRepository extends JpaRepository<FileEntity, Long> {}

@Service
public class FileService {

    @Transactional
    public FileEntity saveFile(MultipartFile file) throws IOException {
        FileEntity entity = new FileEntity();
        entity.setFileName(file.getOriginalFilename());
        entity.setContentType(file.getContentType());
        entity.setData(file.getBytes());
        return repository.save(entity);
    }

    @Transactional(readOnly = true)
    public ResponseEntity<Resource> downloadFile(Long id) {
        FileEntity entity = repository.findById(id).orElseThrow();

        ByteArrayResource resource = new ByteArrayResource(entity.getData());

        return ResponseEntity.ok()
            .contentType(MediaType.parseMediaType(entity.getContentType()))
            .header(HttpHeaders.CONTENT_DISPOSITION,
                "attachment; filename=\"" + entity.getFileName() + "\"")
            .body(resource);
    }
}
```

`@Lob byte[]` salva il file direttamente nel database. Sconsigliato per file grandi (>1MB). Alternativa: salva su filesystem/S3 e tieni solo il path nel DB. `ByteArrayResource` converte byte[] in Resource per download.

## Streaming download

```java
@GetMapping("/download/{id}")
public ResponseEntity<Resource> downloadFile(@PathVariable Long id) {
    FileEntity entity = fileRepository.findById(id).orElseThrow();

    InputStreamResource resource = new InputStreamResource(
        new ByteArrayInputStream(entity.getData()));

    return ResponseEntity.ok()
        .contentType(MediaType.parseMediaType(entity.getContentType()))
        .header(HttpHeaders.CONTENT_DISPOSITION,
            "inline; filename=\"" + entity.getFileName() + "\"")
        .body(resource);
}
```

`CONTENT_DISPOSITION: inline` mostra nel browser (PDF, immagini). `attachment` forza il download. `InputStreamResource` streamma il contenuto senza caricarlo tutto in memoria (se letto da filesystem/S3).

## Upload asincrono

```java
@Service
public class AsyncFileService {

    @Async
    public CompletableFuture<String> processUpload(MultipartFile file) {
        try {
            Thread.sleep(5000); // Simula processamento lungo
            String processedName = "processed_" + file.getOriginalFilename();
            file.transferTo(new File("processed/" + processedName));
            return CompletableFuture.completedFuture(processedName);
        } catch (IOException e) {
            return CompletableFuture.failedFuture(e);
        }
    }
}

@RestController
public class AsyncUploadController {

    @PostMapping("/upload-async")
    public ResponseEntity<String> uploadAsync(@RequestParam("file") MultipartFile file) {
        asyncFileService.processUpload(file);
        return ResponseEntity.accepted()
            .body("File ricevuto, processamento in corso");
    }
}
```

`@Async` processa il file in background. Il controller risponde subito con `202 Accepted`. Il client puo fare polling su un endpoint di stato per sapere quando il processamento e completo.

## Errori comuni

- **File troppo grande**: `MultipartException: SizeLimitExceededException`. Configura `max-file-size` e `max-request-size`.
- **Request non multipart**: se il client non usa `Content-Type: multipart/form-data`, `MultipartFile` e null.
- **transferTo() su file esistente**: sovrascrive senza warning. Genera nome univoco.
- **File in memoria**: file grandi in memoria causano OutOfMemoryError. Imposta `file-size-threshold` per scrivere su disco.
- **Percorso non esistente**: `transferTo(new File("uploads/..."))` fallisce se la directory non esiste. Crea la directory prima.
- **Upload a DB per file grandi**: BLOB grandi degradano le performance del DB. Usa filesystem/S3.
- **Content-Disposition errato**: download corrotto. Verifica `attachment` vs `inline` e quoting del filename.

## Best Practices & Conventions

- Imposta limiti realistici: `max-file-size=10MB`, `max-request-size=50MB`.
- Usa filesystem/S3 per storage, non DB. Salva solo il path nel DB.
- Valida tipo e dimensione prima di salvare.
- Genera nomi file univoci (`UUID` + estensione originale).
- Per upload grandi, usa streaming e processamento asincrono.
- Pulisci file temporanei se usi `multipart.location`.
- Implementa endpoint di download con caching e range request per file grandi.
