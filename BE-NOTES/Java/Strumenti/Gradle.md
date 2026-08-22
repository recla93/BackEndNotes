---
topic: "Gradle — Build Tool"
parent: "[[BE-NOTES/Java/Java|Java]]"
---

Gradle è un build tool moderno che combina il meglio di Maven (dichiarativo) e Ant (flessibile). Usa un DSL Groovy/Kotlin invece di XML, con build incrementali, dependency caching e task graph ottimizzato.

A differenza di Maven (ciclo di vita fisso), Gradle costruisce un **task graph**: ogni task dichiara input e output, Gradle esegue solo i task necessari quando input/output cambiano. È il build tool ufficiale per Android e sempre piu usato in progetti Spring.

## build.gradle.kts (Kotlin DSL)

```kotlin
plugins {
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.4"
    java
}

group = "com.example"
version = "1.0.0"

java {
    sourceCompatibility = JavaVersion.VERSION_21
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.test {
    useJUnitPlatform()
}
```

Le configurazioni (Maven scope equivalenti): `implementation` (compile), `testImplementation` (test), `compileOnly` (provided), `runtimeOnly` (runtime), `annotationProcessor`.

## Comandi principali

```bash
# Compila
./gradlew build

# Test
./gradlew test

# Pulisce e build
./gradlew clean build

# Skippa test
./gradlew build -x test

# Tasks disponibili
./gradlew tasks

# Albero dipendenze
./gradlew dependencies

# Build cache (riutilizza output tra build)
./gradlew build --build-cache
```

`./gradlew` (Gradle Wrapper) garantisce che tutti usino la stessa versione di Gradle. Il file `gradlew` e `gradlew.bat` vanno versionati. `--build-cache` condivide output compilato tra sviluppatori e CI.

## Multi-modulo

```kotlin
// settings.gradle.kts
rootProject.name = "my-project"
include("core", "api", "persistence")
```

```kotlin
// api/build.gradle.kts
dependencies {
    implementation(project(":core"))
    implementation(project(":persistence"))
    implementation("org.springframework.boot:spring-boot-starter-web")
}
```

`settings.gradle.kts` elenca i moduli. `project(":module")` crea dipendenza tra moduli. Gradle compila i moduli nell'ordine corretto automaticamente.

## Gradle vs Maven

| Caratteristica | Maven | Gradle |
|---------------|-------|--------|
| Configurazione | XML | Groovy/Kotlin DSL |
| Performance | Buona | Migliore (incremental build, cache) |
| Flessibilità | Bassa | Alta (task custom arbitrari) |
| Curva | Bassa | Media |
| Build time | 1x | 0.3-0.5x (incrementale) |
| Diffusione Enterprise | Alta | Media (crescente) |
| Android | No | **Si** (ufficiale) |
| Wrapper | `mvnw` | `gradlew` (standard) |

## Errori comuni

- **`implementation` vs `api`**: `api` espone la dipendenza ai consumatori (come Maven `compile`). `implementation` è privata. Preferisci `implementation`.
- **Dimenticare `useJUnitPlatform()`**: senza, JUnit 5 non funziona.
- **Cache corrotte**: `./gradlew clean` o cancella `~/.gradle/caches/`.
- **Kotlin DSL con sintassi Groovy**: `compile group: 'x', name: 'y'` vs `implementation("x:y")`.
- **Task non eseguiti perché up-to-date**: controlla che gli input dichiarati siano corretti.
- **Configurazione in `buildscript` vs `plugins`**: preferisci `plugins` block per plugin moderni.

## Best Practices & Conventions

- Usa **Gradle Wrapper** (`gradlew`) e versiona `gradle/` e `gradlew`.
- Preferisci **Kotlin DSL** (`build.gradle.kts`) su Groovy (`build.gradle`) per type-safety e autocompletamento IDE.
- Usa `implementation` per dipendenze interne, non `api`.
- Sfrutta **build cache** in CI per build piu veloci.
- Per progetti Spring Boot, usa il plugin `io.spring.dependency-management` invece di specificare versioni manualmente.
- Blocca le versioni in `libs.versions.toml` (Version Catalog, Gradle 7+) per progetti multi-modulo.
