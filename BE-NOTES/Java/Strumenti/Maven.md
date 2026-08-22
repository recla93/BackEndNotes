---
topic: "Maven — Build Tool"
parent: "[[BE-NOTES/Java/Java|Java]]"
---

Maven è il build tool standard per progetti Java. Usa un file `pom.xml` dichiarativo per gestire dipendenze, compilazione, test, packaging e deploy. A differenza di Make/Ant (imperativi), Maven segue il principio "convention over configuration".

Ogni progetto Maven ha un ciclo di vita predefinito (compile → test → package → install → deploy) con fasi (phase) e plugin. Le dipendenze vengono scaricate automaticamente da repository remoti (Maven Central).

## pom.xml — struttura base

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>21</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

`groupId` = namespace inverso (es. `com.example`), `artifactId` = nome modulo, `version` = formato semver. Lo `scope` controlla la visibilità della dipendenza: `compile` (default), `test`, `provided`, `runtime`, `system`.

## Ciclo di vita e comandi

```bash
# Compila
mvn compile

# Test
mvn test

# Package (jar/war)
mvn package

# Pulisce + costruisce da zero
mvn clean install

# Salta test
mvn package -DskipTests

# Usa profile specifico
mvn clean install -Pproduction

# Albero delle dipendenze
mvn dependency:tree
```

`install` copia il package nel repository locale (`~/.m2/repository`). `deploy` lo pubblica su un repository remoto (Nexus/Artifactory). `dependency:tree` è essenziale per debug di conflitti.

## Spring Boot parent e starter

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

Il parent `spring-boot-starter-parent` fornisce dependency management per tutte le librerie Spring compatibili. Gli `starter` raggruppano dipendenze coese (web, JPA, security, test). Non serve specificare la versione: la gestisce il parent.

## Multi-modulo

```xml
<!-- pom.xml radice (parent) -->
<groupId>com.example</groupId>
<artifactId>my-project</artifactId>
<version>1.0.0</version>
<packaging>pom</packaging>

<modules>
    <module>core</module>
    <module>api</module>
    <module>persistence</module>
</modules>
```

```xml
<!-- core/pom.xml -->
<parent>
    <groupId>com.example</groupId>
    <artifactId>my-project</artifactId>
    <version>1.0.0</version>
</parent>
<artifactId>core</artifactId>
```

`mvn clean install` sulla root compila tutti i moduli nell'ordine corretto. Ogni modulo eredita dal parent. I moduli possono dipendere tra loro: `mvn dependency:tree` mostra le dipendenze traverse.

## Plugin comuni

```xml
<build>
    <plugins>
        <!-- Spring Boot packaging -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>

        <!-- Checkstyle -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-checkstyle-plugin</artifactId>
        </plugin>

        <!-- JaCoCo coverage -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.11</version>
            <executions>
                <execution>
                    <goals><goal>prepare-agent</goal></goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

I plugin Maven estendono il ciclo di vita. `spring-boot-maven-plugin` crea il fat jar eseguibile. JaCoCo si aggancia ai test per misurare la copertura.

## Errori comuni

- **Conflict di dipendenze**: due librerie portano versioni diverse della stessa dipendenza. Usa `mvn dependency:tree` e `<exclusions>`.
- **Versione non specificata nel parent**: se il parent non fornisce un BOM, devi specificare esplicitamente.
- **`-DskipTests` vs `-Dmaven.test.skip=true`**: il primo compila i test ma non li esegue, il secondo non compila neanche i test.
- **`provided` scope dimenticato**: librerie come Lombok e Servlet API vanno con `provided` (fornite dal container/plugin).
- **Jar eseguibile non creato**: senza `spring-boot-maven-plugin`, il jar non ha il Main-Class nel manifest.
- **`mvn clean` obbligatorio dopo modifiche strutturali**: non sempre Maven rileva cambiamenti in configurazioni e plugin.

## Best Practices & Conventions

- Usa `spring-boot-starter-parent` per progetti Spring Boot.
- Mantieni le versioni in `<properties>` o in un BOM centralizzato.
- Usa `<exclusions>` per rimuovere dipendenze transitive indesiderate.
- Blocca le versioni con `<dependencyManagement>` nel parent per progetti multi-modulo.
- Esegui `mvn dependency:tree` prima di aggiungere nuove dipendenze per verificare conflitti.
- Per build riproducibili, usa `maven-wrapper` (`mvnw`) incluso nel repository.
