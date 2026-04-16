---
name: spring-boot-dockerfile
description: Creates a production-ready multi-stage Dockerfile for a Spring Boot application. Auto-detects Java version, Maven vs Gradle, artifact name, and port from the project's build files and application config. Adds a non-root user, JVM container-aware flags, and an optional Actuator-based health check. Use when the user asks to containerize, dockerize, or create a Dockerfile for a Spring Boot / Java application.
---

# Spring Boot Dockerfile

Generate a reusable, production-minded Dockerfile for a Spring Boot module.

## Preconditions

- Confirm the target directory is a Spring Boot project by finding one of: `pom.xml` (Maven) or `build.gradle` / `build.gradle.kts` (Gradle).
- If a `Dockerfile` already exists in the target directory, show it to the user and ask whether to overwrite before proceeding.

## Phase 1: Auto-detect project facts

Before asking the user anything, read the project and gather:

1. **Build system**: Maven if `pom.xml` exists; Gradle if `build.gradle(.kts)` exists. Detect whether a wrapper (`mvnw` / `gradlew`) is present — prefer it.
2. **Java version**: From `<java.version>` or `<maven.compiler.release>` in `pom.xml`, or `sourceCompatibility` / `java.toolchain` in Gradle. Default to the latest LTS (currently 21) if missing.
3. **Artifact name & packaging**: `<artifactId>` + `<version>` from `pom.xml`, or `rootProject.name` + version from Gradle. Confirm the packaging is `jar` (the default); warn if `war`.
4. **Port**: Look in `src/main/resources/application.yaml`, `application.yml`, or `application.properties` for `server.port`. Default to `8080` if absent.
5. **Actuator presence**: Grep dependencies for `spring-boot-starter-actuator`. Note whether it's already present.
6. **Main class** (rarely needed — Spring Boot's executable jar handles this, but useful if packaging is unusual).

Report what you found in a compact summary so the user can correct any wrong assumptions before the Dockerfile is written.

## Phase 2: Decisions (ask only if ambiguous)

For each decision, use the defaults below unless the project state or the user indicates otherwise. Only prompt the user when the signal is genuinely ambiguous.

| Decision | Default | When to ask |
|---|---|---|
| JRE vs JDK runtime | JRE | User mentions debugging needs |
| Base image | `eclipse-temurin:<version>-jre` | User prefers a different distro (Corretto, Zulu) |
| Build stage | Multi-stage using project's wrapper | Pre-built jar workflow requested |
| Dependency caching | `pom.xml`/`build.gradle`-first layer cache | Never skip unless asked |
| User | `spring:spring` non-root | User requests otherwise |
| JVM flags | `-XX:+UseContainerSupport` | User wants heap tuning or `JAVA_OPTS` |
| Health check | Actuator via `curl` if actuator available | Skip if user declines extra tooling |
| Actuator dependency | Add if missing **and** health check requested | Always confirm before modifying `pom.xml` |
| Port | Value from `application.yaml`, else 8080 | Never — detected |

## Phase 3: Write the Dockerfile

Use the template below, substituting `<JAVA_VERSION>`, `<PORT>`, and Maven vs Gradle commands as appropriate. Place the Dockerfile at the project root (same directory as `pom.xml` / `build.gradle`).

### Maven template

```dockerfile
# ─── Stage 1: Build ───────────────────────────────────────────────────────────
FROM eclipse-temurin:<JAVA_VERSION>-jdk AS build
WORKDIR /workspace

# Cache dependencies — only re-runs when pom.xml changes
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline -q

# Build the jar
COPY src/ src/
RUN ./mvnw clean package -DskipTests -q

# ─── Stage 2: Runtime ─────────────────────────────────────────────────────────
FROM eclipse-temurin:<JAVA_VERSION>-jre

# Install curl for health check (omit block if no health check)
RUN apt-get update && apt-get install -y --no-install-recommends curl && rm -rf /var/lib/apt/lists/*

# Non-root user
RUN groupadd spring && useradd -g spring spring
USER spring:spring

WORKDIR /app
COPY --from=build /workspace/target/*.jar app.jar

EXPOSE <PORT>

HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:<PORT>/actuator/health || exit 1

ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-jar", "app.jar"]
```

### Gradle template

Swap the build stage for:

```dockerfile
FROM eclipse-temurin:<JAVA_VERSION>-jdk AS build
WORKDIR /workspace

COPY gradle/ gradle/
COPY gradlew settings.gradle* build.gradle* ./
RUN ./gradlew --no-daemon dependencies -q || true

COPY src/ src/
RUN ./gradlew --no-daemon bootJar -x test -q
```

And adjust the copy in stage 2 to `COPY --from=build /workspace/build/libs/*.jar app.jar`.

## Phase 4: Optional dependency additions

If the user opts into the Actuator health check and `spring-boot-starter-actuator` is not already a dependency, add it:

- **Maven** (`pom.xml`, inside `<dependencies>`):
  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  ```
- **Gradle** (`build.gradle`):
  ```groovy
  implementation 'org.springframework.boot:spring-boot-starter-actuator'
  ```

Confirm with the user before modifying the build file.

## Phase 5: Report

Tell the user:
1. Where the Dockerfile was written.
2. Any build files that were modified (e.g. actuator added).
3. The build + run commands, e.g.:
   ```bash
   docker build -t <artifactId> .
   docker run -p <PORT>:<PORT> <artifactId>
   ```
4. A one-line verification (curl to `/actuator/health` or a known endpoint).

Do not build the image automatically unless the user asks.

## Notes

- Keep the `.dockerignore` in mind — if none exists, suggest creating one with at minimum `target/`, `build/`, `.git/`, `.idea/` to keep the build context small. Do not create it unprompted.
- If the project has a `war` packaging, warn the user that this template targets executable-jar Spring Boot apps; a war deployment needs a servlet-container base image instead.
- For Spring Boot 2.3+, `spring-boot:build-image` is an alternative (Cloud Native Buildpacks, no Dockerfile needed). Mention it once if the user seems undecided, but don't push it — this skill's job is to produce a Dockerfile.
