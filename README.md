# Silver Soul Curiosity

Plataforma educativa modular basada en microservicios con Spring Boot y Gradle.

## Estructura del proyecto

```
silver_soul_curiosity/          ← proyecto raíz (:)
├── build.gradle                ← configuración global (group, version, toolchain)
├── settings.gradle             ← declara los submódulos del proyecto
├── gradle.properties           ← propiedades de JVM, JDK y rendimiento
├── gradlew / gradlew.bat      ← entry points del Gradle Wrapper
├── gradle/wrapper/            ← descarga y ejecuta la versión fija de Gradle
├── libs/                       ← submódulo :libs (librerías compartidas)
│   └── build.gradle
├── services/payment/           ← submódulo :payment (servicio de pagos)
│   └── build.gradle
├── services/security/          ← submódulo :security (servicio de seguridad)
│   └── build.gradle
└── docker-compose.yml          ← PostgreSQL, Redis y Kafka para desarrollo
```

## Cómo funciona Gradle en este proyecto

### Proyecto multimódulo

El proyecto está dividido en varios submódulos que Gradle orquesta desde la raíz:

| ID Gradle | Nombre | Carpeta | Rol |
|---|---|---|---|
| `:` | raíz | `./` | Contenedor, configuración global |
| `:libs` | libs | `./libs/` | Librerías y utilidades compartidas |
| `:payment` | payment | `./services/payment/` | Microservicio de pagos |
| `:security` | security | `./services/security/` | Microservicio de autenticación |

### settings.gradle (¿qué proyectos existen?)

Gradle lee este archivo primero para descubrir los submódulos:

```groovy
rootProject.name = 'silver-soul-curiosity'
include 'libs'
// Escanea services/ e incluye cada carpeta como submódulo
file('services').eachDir { dir ->
    include dir.name
    project(":${dir.name}").projectDir = dir
}
```

### build.gradle raíz (configuración que heredan todos)

Usa dos bloques para compartir configuración:

- **`allprojects {}`** — se aplica a la raíz y a todos los submódulos: `group`, `version`, `repositories`
- **`subprojects {}`** — se aplica solo a los submódulos: plugin `java`, toolchain JDK 25

```groovy
allprojects {
    group = 'com.silversoul'
    version = '0.0.1-SNAPSHOT'
    repositories { mavenCentral() }
}

subprojects {
    apply plugin: 'java'
    java { toolchain { languageVersion = JavaLanguageVersion.of(25) } }
}
```

Los plugins `org.springframework.boot` y `io.spring.dependency-management` se declaran en la raíz con `apply false` para que los submódulos puedan usarlos sin repetir la versión:

```groovy
plugins {
    id 'org.springframework.boot' version '4.0.6' apply false
    id 'io.spring.dependency-management' version '1.1.7' apply false
}
```

### Dependencias entre submódulos

Cuando un módulo necesita a otro, se declara con `project()`:

```groovy
// services/payment/build.gradle
dependencies {
    implementation project(':libs')
}
```

Gradle construye automáticamente el **grafo de dependencias** y ejecuta los builds en el orden correcto: libs primero, luego payment y security.

### gradle.properties

Configura el JDK que Gradle usará para compilar y parámetros de memoria:

```properties
org.gradle.java.home=C\:/Program Files/Amazon Corretto/jdk25.0.3_9
org.gradle.jvmargs=-Xmx2g -XX:MaxMetaspaceSize=512m
org.gradle.parallel=true
```

### Gradle Wrapper

Cada módulo tiene su propio wrapper (`gradlew` + `gradle/wrapper/`) apuntando a Gradle **9.0.0**. Esto permite:

- Ejecutar builds sin tener Gradle instalado
- Usar la misma versión en todos los entornos (desarrollo, CI/CD)
- Builds reproducibles: cualquiera que clone el repo obtiene el mismo resultado

```
gradlew / gradlew.bat          ← entry points
        │
        ▼
gradle-wrapper.jar             ← lee gradle-wrapper.properties
        │
        ▼
Descarga Gradle 9.0.0          ← si no está cacheado
        │
        ▼
Ejecuta el build               ← compileJava, test, bootJar, etc.
```

## Requisitos

- **JDK 25** (Corretto recomendado)
- **Docker Desktop** (para PostgreSQL, Redis y Kafka)

## Cómo ejecutar

### 1. Iniciar infraestructura

```bash
docker compose up -d
```

### 2. Build completo

```bash
./gradlew build
```

### 3. Ejecutar un submódulo específico

```bash
./gradlew :payment:bootRun
./gradlew :security:bootRun
```

### Compilar sin tests (desarrollo rápido)

```bash
./gradlew build -x test
```

## Notas

- Los submódulos `payment` y `security` dependen de `libs`. Si haces cambios en `libs`, Gradle lo detecta automáticamente y lo recompila antes.
- El `docker-compose.yml` incluye PostgreSQL, Redis y Kafka. Asegúrate de tener Docker corriendo antes de ejecutar los tests de integración.
- `JAVA_HOME` debe apuntar a JDK 25. Si usas otro JDK, `gradle.properties` tiene `org.gradle.java.home` para forzar la versión correcta.
