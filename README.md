# Hello World Spring Boot App

This repository contains a minimal Spring Boot hello world service built with Maven and packaged with Docker.

## Requirements

- Java 11
- Maven 3.9+
- Docker

## Run locally

```bash
mvn spring-boot:run
```

The app starts on `http://localhost:8080` and returns:

```text
Hello, World!
```

## Build the jar

```bash
mvn clean package
```

The packaged application will be created under `target/`.

## Build the Docker image

```bash
docker build -t hello-world-springboot .
```

## Run the Docker image

```bash
docker run -p 8080:8080 hello-world-springboot
```

## Project layout

- `src/main/java` contains the Spring Boot application and controller
- `src/test/java` contains a basic smoke test
- `Dockerfile` builds a lightweight runtime image for the packaged jar
