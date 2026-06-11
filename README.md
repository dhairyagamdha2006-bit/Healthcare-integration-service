# Healthcare Data Integration Service

A secure, Spring Boot-based clinical data integration service for ingesting, validating, normalizing, and exposing healthcare data through REST APIs.

This project demonstrates how heterogeneous healthcare data sources can be transformed into a unified clinical data layer using modern backend architecture, asynchronous processing, and healthcare interoperability standards such as HL7® FHIR®.

---

## Table of Contents

* [Overview](#overview)
* [Key Features](#key-features)
* [Architecture](#architecture)
* [MVP Scope](#mvp-scope)
* [Tech Stack](#tech-stack)
* [Data Model](#data-model)
* [API Endpoints](#api-endpoints)
* [Security](#security)
* [Getting Started](#getting-started)
* [Messaging Pipeline](#messaging-pipeline)
* [Testing](#testing)
* [Roadmap](#roadmap)
* [Project Structure](#project-structure)
* [FHIR Notice](#fhir-notice)

---

## Overview

The **Healthcare Data Integration Service** is a reference implementation of a hospital-grade backend system designed to ingest clinical data from multiple sources, normalize it into structured healthcare resources, and expose patient-centered data through secure APIs.

The service is built with **Java 21** and **Spring Boot 3**, following a modular architecture that separates API, domain, application, and infrastructure concerns.

It supports clinical workflows such as:

* Patient data aggregation
* Observation and vitals ingestion
* Encounter history retrieval
* Clinical data normalization
* Audit-friendly backend operations
* Secure API access for downstream systems

This project is designed for learning, demonstration, and portfolio use, while following patterns commonly used in healthcare integration platforms.

---

## Key Features

* **Clinical Data Ingestion**

  * Accepts batched observation payloads
  * Stores raw incoming messages
  * Publishes normalization events asynchronously

* **FHIR-Inspired Normalization**

  * Maps clinical payloads into patient, encounter, and observation resources
  * Supports future projection into HL7® FHIR® resources such as `Patient`, `Encounter`, and `Observation`

* **Patient Snapshot API**

  * Retrieves patient demographics
  * Returns recent encounters
  * Returns recent clinical observations and vitals

* **Asynchronous Processing**

  * Uses Spring Cloud Stream and Kafka-compatible messaging
  * Separates API ingestion from normalization workflow
  * Improves scalability and fault tolerance

* **Security**

  * API key authentication for protected endpoints
  * Environment-based configuration
  * Designed with healthcare data protection principles in mind

* **Observability**

  * Actuator endpoints
  * Structured application logs
  * Metrics-ready architecture for Prometheus and OpenTelemetry integration

* **Database Reliability**

  * PostgreSQL persistence
  * Flyway schema migrations
  * Soft-delete and timestamp-ready entity design

---

## Architecture

The service follows a layered backend architecture:

| Layer               | Responsibilities                                          | Technologies                          |
| ------------------- | --------------------------------------------------------- | ------------------------------------- |
| API Layer           | REST controllers, request validation, API security        | Spring MVC, Spring Security           |
| Application Layer   | Business workflows, command handlers, orchestration       | Spring Boot services                  |
| Domain Layer        | Core healthcare models and business rules                 | Java domain entities                  |
| Data Pipeline       | Async ingestion, normalization, retries, event publishing | Spring Cloud Stream, Kafka / Redpanda |
| Persistence Layer   | Database access, schema migrations, entity storage        | PostgreSQL, Spring Data JPA, Flyway   |
| Observability Layer | Health checks, logs, metrics, tracing readiness           | Spring Boot Actuator, Micrometer      |

---

## MVP Scope

The current MVP focuses on the core workflow of ingesting clinical observations and retrieving patient data.

### Implemented / Planned MVP Capabilities

1. **Observation Ingestion API**

   * Accepts batched observation payloads
   * Validates incoming request data
   * Stores raw message records
   * Publishes normalization events

2. **Normalization Worker**

   * Consumes observation normalization events
   * Maps incoming data to internal clinical records
   * Persists normalized observations

3. **Patient Snapshot API**

   * Retrieves patient demographics
   * Returns recent encounters
   * Returns recent observations

4. **Audit and Metrics Foundation**

   * Request logging
   * Health endpoints
   * Metrics-ready configuration

---

## Tech Stack

| Category      | Technology                              |
| ------------- | --------------------------------------- |
| Language      | Java 21                                 |
| Framework     | Spring Boot 3                           |
| API           | Spring MVC, REST                        |
| Security      | Spring Security, API Key Authentication |
| Database      | PostgreSQL                              |
| ORM           | Spring Data JPA / Hibernate             |
| Migrations    | Flyway                                  |
| Messaging     | Spring Cloud Stream                     |
| Broker        | Kafka-compatible Redpanda               |
| Build Tool    | Maven                                   |
| DevOps        | Docker Compose                          |
| Observability | Spring Boot Actuator, Micrometer        |
| Documentation | OpenAPI / Swagger UI                    |

---

## Data Model

The MVP data model includes the following core entities:

### Patient

Stores patient identity and demographic information.

Example fields:

* Patient ID
* Medical Record Number, or MRN
* First name
* Last name
* Date of birth
* Gender
* Consent status
* Facility or tenant identifier

### Encounter

Represents a patient visit or clinical interaction.

Example fields:

* Encounter ID
* Patient reference
* Encounter type
* Start and end time
* Facility
* Status

### Observation

Stores clinical measurements such as vitals or lab results.

Example fields:

* Observation ID
* Patient reference
* Encounter reference
* Observation code
* Display name
* Value
* Unit
* Effective timestamp

### RawMessage

Stores the original ingestion payload for traceability and replay support.

Example fields:

* Raw message ID
* Source system
* Payload body
* Processing status
* Error details
* Created timestamp

---

## API Endpoints

### Observation Ingestion

```http
POST /api/v1/ingest/observations
```

Ingests a batch of patient observations, stores the raw payload, and publishes an asynchronous normalization event.

#### Example Request

```json
{
  "sourceSystem": "hospital-vitals-system",
  "patient": {
    "mrn": "MRN-10045",
    "firstName": "John",
    "lastName": "Doe",
    "dateOfBirth": "1985-04-12",
    "gender": "male"
  },
  "observations": [
    {
      "code": "heart-rate",
      "display": "Heart Rate",
      "value": 78,
      "unit": "beats/min",
      "effectiveDateTime": "2026-01-15T10:30:00Z"
    }
  ]
}
```

#### Example Response

```json
{
  "message": "Observation batch accepted for processing",
  "status": "ACCEPTED"
}
```

---

### Patient Snapshot

```http
GET /api/v1/patients/{patientId}
```

Returns a patient-centered clinical snapshot including demographics, recent encounters, and recent observations.

#### Example Response

```json
{
  "patientId": "123",
  "mrn": "MRN-10045",
  "firstName": "John",
  "lastName": "Doe",
  "dateOfBirth": "1985-04-12",
  "encounters": [],
  "observations": []
}
```

---

## Security

All protected API endpoints require an API key.

The API key must be passed using the following request header:

```http
X-API-Key: local-dev-key
```

The default local development key is:

```text
local-dev-key
```

You can override it using an environment variable:

```bash
APP_API_KEY=your-secure-api-key
```

Or through application configuration:

```properties
app.auth.api-key=your-secure-api-key
```

Public endpoints such as Actuator health checks and Swagger documentation may be excluded from authentication depending on the active profile.

---

## Getting Started

### Prerequisites

Make sure the following tools are installed:

* Java 21
* Maven 3.9+
* Docker
* Docker Compose
* Git

---

### 1. Clone the Repository

```bash
git clone https://github.com/dhairyagamdha2006-bit/Healthcare-integration-service.git
cd Healthcare-integration-service
```

---

### 2. Start Required Services

Start PostgreSQL and Redpanda using Docker Compose:

```bash
docker compose up -d
```

---

### 3. Build the Project

```bash
./mvnw clean verify
```

If you are using Windows:

```bash
mvnw.cmd clean verify
```

---

### 4. Run the Application

```bash
SPRING_PROFILES_ACTIVE=dev APP_API_KEY=local-dev-key ./mvnw spring-boot:run
```

For Windows PowerShell:

```powershell
$env:SPRING_PROFILES_ACTIVE="dev"
$env:APP_API_KEY="local-dev-key"
./mvnw spring-boot:run
```

---

### 5. Open API Documentation

After the application starts, open:

```text
http://localhost:8080/swagger-ui.html
```

You can also check application health at:

```text
http://localhost:8080/actuator/health
```

---

## Messaging Pipeline

Observation ingestion uses an asynchronous event-driven workflow.

### Flow

```text
Client Request
    ↓
Observation Ingestion API
    ↓
Raw Message Stored
    ↓
Normalization Event Published
    ↓
Kafka / Redpanda Topic
    ↓
Normalization Worker
    ↓
Normalized Clinical Records Stored
```

The application publishes `ObservationNormalizationEvent` messages using Spring Cloud Stream.

Default topic:

```text
observation-normalization
```

Default output binding:

```text
observationNormalization-out-0
```

---

## Testing

Run all tests with:

```bash
./mvnw test
```

Run full verification:

```bash
./mvnw clean verify
```

Recommended future testing improvements:

* Unit tests for domain and application services
* Integration tests with Testcontainers
* Security tests for protected endpoints
* API contract tests for ingestion and patient snapshot workflows
* Kafka consumer and producer tests

---

## Roadmap

Planned improvements include:

* Add full HL7® FHIR® resource projection
* Implement HL7 v2 message ingestion
* Add CSV batch upload support
* Add vendor-specific REST API connectors
* Implement Master Patient Index matching
* Add deterministic and probabilistic patient matching
* Add Redis caching for patient snapshot queries
* Add FHIR `AuditEvent` logging
* Add OpenTelemetry distributed tracing
* Add Testcontainers-based integration tests
* Add role-based access control using OAuth2 / OIDC
* Add configurable facility-level consent policies
* Add message replay and dead-letter queue handling

---

## Project Structure

Recommended package layout:

```text
src/main/java/com/example/healthcare
├── api
│   ├── controller
│   └── dto
├── application
│   ├── service
│   └── command
├── domain
│   ├── model
│   └── repository
├── infrastructure
│   ├── persistence
│   ├── messaging
│   └── security
└── config
```

### Package Responsibilities

| Package          | Responsibility                                        |
| ---------------- | ----------------------------------------------------- |
| `api`            | REST controllers and DTOs                             |
| `application`    | Business workflows and orchestration                  |
| `domain`         | Core healthcare models and rules                      |
| `infrastructure` | Database, messaging, and external system integrations |
| `config`         | Spring configuration and application setup            |

---

## Environment Variables

| Variable                     | Description                        | Default          |
| ---------------------------- | ---------------------------------- | ---------------- |
| `SPRING_PROFILES_ACTIVE`     | Active Spring profile              | `dev`            |
| `APP_API_KEY`                | API key used for secured endpoints | `local-dev-key`  |
| `SPRING_DATASOURCE_URL`      | PostgreSQL connection URL          | Profile-specific |
| `SPRING_DATASOURCE_USERNAME` | Database username                  | Profile-specific |
| `SPRING_DATASOURCE_PASSWORD` | Database password                  | Profile-specific |

---

## Example cURL Requests

### Ingest Observations

```bash
curl -X POST http://localhost:8080/api/v1/ingest/observations \
  -H "Content-Type: application/json" \
  -H "X-API-Key: local-dev-key" \
  -d '{
    "sourceSystem": "hospital-vitals-system",
    "patient": {
      "mrn": "MRN-10045",
      "firstName": "John",
      "lastName": "Doe",
      "dateOfBirth": "1985-04-12",
      "gender": "male"
    },
    "observations": [
      {
        "code": "heart-rate",
        "display": "Heart Rate",
        "value": 78,
        "unit": "beats/min",
        "effectiveDateTime": "2026-01-15T10:30:00Z"
      }
    ]
  }'
```

### Get Patient Snapshot

```bash
curl -X GET http://localhost:8080/api/v1/patients/123 \
  -H "X-API-Key: local-dev-key"
```

---

## Compliance Notes

This project is a technical reference implementation and is not production-ready for real patient data without additional compliance, security, and privacy review.

Before using this system with real protected health information, additional safeguards would be required, including:

* HIPAA compliance review
* PHI encryption and key management
* Access control and authorization policies
* Production audit logging
* Data retention and deletion policies
* Secure infrastructure configuration
* Vulnerability scanning and penetration testing

---

## FHIR Notice

FHIR® is a registered trademark of HL7 and is used with the permission of HL7.

This project is not affiliated with or endorsed by HL7.
