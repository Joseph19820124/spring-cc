# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Spring Boot 3.5.3 application built with:
- Java 17
- Spring Web and WebFlux
- Spring Modulith for modular architecture
- GraphQL code generation via Netflix DGS
- GraalVM native compilation support
- Lombok for reducing boilerplate
- Docker Compose integration

## Development Commands

### Build and Run
```bash
# Make Maven wrapper executable (first time only)
chmod +x ./mvnw

# Clean and compile
./mvnw clean compile

# Run the application
./mvnw spring-boot:run

# Run with specific profile
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# Package as JAR
./mvnw clean package

# Build native image (requires GraalVM)
./mvnw -Pnative native:compile
```

### Testing
```bash
# Run all tests
./mvnw test

# Run a specific test class
./mvnw test -Dtest=SpringCcApplicationTests

# Run tests with coverage
./mvnw test jacoco:report

# Run modulith verification tests
./mvnw test -Dtest=*ModulithTest
```

### GraphQL Development
```bash
# Generate GraphQL types from schema
./mvnw graphqlcodegen:generate

# Schema location: src/main/resources/graphql-client
# Generated code: target/generated-sources (com.example.springcc.codegen package)
```

## Architecture

### Package Structure
- `com.example.springcc` - Root package containing the main application class
- `com.example.springcc.codegen` - Generated GraphQL types (do not edit)
- Future modules should follow Spring Modulith conventions under the root package

### Key Technologies
1. **Spring Modulith**: Enforces modular boundaries within the monolith
   - Each module should be a direct sub-package of the root
   - Inter-module communication through public APIs only
   - Module tests verify architectural constraints

2. **GraphQL Code Generation**: 
   - Place GraphQL schemas in `src/main/resources/graphql-client`
   - Generated types are created during build
   - Use `@DgsComponent` and `@DgsQuery/@DgsMutation` for GraphQL endpoints

3. **Native Compilation**:
   - Application supports GraalVM native image compilation
   - Use `@RegisterReflectionForBinding` for classes needing reflection
   - Test native compatibility with `./mvnw -Pnative test`

### Configuration
- Main configuration: `src/main/resources/application.properties`
- Profile-specific configs: `application-{profile}.properties`
- Docker services defined in `compose.yaml` (currently empty)

## Development Workflow

1. **Adding a new module**:
   - Create package under `com.example.springcc.{module}`
   - Define module's public API in the package root
   - Keep internal implementation in `internal` sub-package
   - Add module verification test

2. **Adding GraphQL schemas**:
   - Place `.graphqls` files in `src/main/resources/graphql-client`
   - Run `./mvnw generate-sources` to generate types
   - Implement resolvers using DGS annotations

3. **Running with Docker Compose**:
   - Define services in `compose.yaml`
   - Spring Boot will auto-start Docker Compose services
   - Use `spring.docker.compose.enabled=false` to disable