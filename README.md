# CSI5347 Distributed Systems Course Project

This project is a distributed systems implementation for the CSI 5347 course. It consists of multiple microservices that interact through REST APIs and shared configurations. The system includes an **Inventory Service**, a **Loyalty Program Service**, a **Point of Sale (POS) Service**, **service discovery with Eureka**, centralized routing through a **Gateway Server**, and authentication/authorization using **Keycloak**. The project is containerized using **Docker** and can be orchestrated through **Docker Compose** for testing and deployment.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Clone the Repository](#clone-the-repository)
- [Project Structure](#project-structure)
- [Compiling](#compiling)
- [Running the Application](#running-the-application)
- [Configuration Management](#configuration-management)
- [Testing](#testing)
- [Observability and Security](#observability-and-security)

---

## Project Overview

This project implements a microservices-based distributed system with the following components:

- **Configuration Server (`configuration-server/`)**
  - A Spring Cloud Config Server that centralizes configuration management.
  - Loads configuration from a separate repository located in `configuration-repository/`.

- **Configuration Repository (`configuration-repository/`)**
  - Stores the externalized configuration files used by the Config Server.

- **Service Discovery (`service-discovery/`)**
  - Provides service registration and discovery using **Spring Cloud Eureka**.

- **Gateway Server (`gateway-server/`)**
  - Serves as the entry point for routing requests to backend services.

- **Inventory Service (`inventory-service/`)**
  - Manages vendors, products, and purchase orders.
  - Uses a PostgreSQL database for persistence.

- **Loyalty Program Service (`loyalty-program-service/`)**
  - Manages customer accounts and loyalty rewards.
  - Uses a separate PostgreSQL database for persistence.

- **Point of Sale (POS) Service (`point-of-sale-service/`)**
  - Handles sales transactions, registers, and employees.
  - Uses another PostgreSQL database for persistence.

- **Docker Compose (`docker-compose.yml`)**
  - Orchestrates the microservices, databases, and supporting infrastructure.

- **Submission Artifacts (`submission-artifacts/`)**
  - Contains documentation and required artifacts for the project submission.

---

## Clone the Repository

Since this repository contains multiple submodules, it should be cloned using the following command:

```bash
git clone --recurse-submodules https://github.com/Matt-Hays/csi-5347-course-project.git
cd csi-5347-course-project
```

If the repository was already cloned without submodules:

```bash
git submodule sync --recursive
git submodule update --init --recursive
```

To pull updates from the parent repository and synchronize submodules:

```bash
git pull --recurse-submodules
git submodule update --init --recursive
```

---

## Project Structure

```text
./csi-5347-course-project
│── configuration-server/                 # Spring Cloud Config Server
│── configuration-repository/             # External configuration files
│── inventory-service/                    # Inventory management microservice
│── loyalty-program-service/              # Loyalty program microservice
│── point-of-sale-service/                # POS system microservice
│── service-discovery/                    # Eureka service discovery server
│── gateway-server/                       # API gateway / request routing
│── gatling-maven-plugin-demo-java-main/  # Gatling stress/performance tests
│── submission-artifacts/                 # Documentation and submission files
│── docker-compose.yml                    # Docker Compose setup for running the system
```

---

## Compiling

Each microservice can be compiled separately, or all can be built at once.

To build all services at once:

```bash
mvn clean package spring-boot:build-image -DskipTests
```

To build an individual service, navigate to its directory and run Maven there. For example:

```bash
cd inventory-service
mvn clean package spring-boot:build-image -DskipTests
```

---

## Running the Application

The system is designed to run as Docker containers orchestrated with Docker Compose. Use the following command to start all services together:

```bash
CONFIG_PORT=8071 INVENTORY_DB_PORT=5432 LOYALTY_DB_PORT=5433 POS_DB_PORT=5434 \
INVENTORY_SERVICE_PORT=8080 INVENTORY_SERVICE_PROFILE=dev LOYALTY_SERVICE_PORT=8081 \
LOYALTY_SERVICE_PROFILE=dev POS_SERVICE_PORT=8082 POS_SERVICE_PROFILE=dev \
GATEWAY_PORT=8072 \
docker compose up -d
```

This will:

- Start the **Configuration Server** at port `8071`.
- Start **PostgreSQL instances** for each microservice (`5432`, `5433`, `5434`).
- Start the **Inventory Service** (`8080`), **Loyalty Program Service** (`8081`), and **POS Service** (`8082`).
- Start the **Gateway Server** at port `8072`.
- Start supporting infrastructure defined in `docker-compose.yml`, including **Keycloak**.

To stop all running containers:

```bash
docker compose down
```

To check logs for a specific service:

```bash
docker logs -f <container_name>
```

---

## Configuration Management

The microservices retrieve their configurations from the **Configuration Server** (`configuration-server/`), which in turn loads properties from the **Configuration Repository** (`configuration-repository/`).

To access the configuration properties for a service, make a GET request to:

```text
http://localhost:8071/{application}/{profile}
```

For example, to fetch configurations for the **Inventory Service** in the `dev` profile:

```text
http://localhost:8071/inventory-service/dev
```

---

## Testing

Unit and integration tests can be executed with:

```bash
mvn test
```

For individual services, navigate to the service directory and run:

```bash
cd inventory-service
mvn test
```

A Postman collection exists in **`submission-artifacts/`** for REST API executions.

Example Gatling tests are included in `gatling-maven-plugin-demo-java-main/` for performance and load testing.

---

## Observability and Security

- **Keycloak**
  - Used for authentication and authorization.
  - Started as part of the Docker Compose environment.

- **Spring Cloud Eureka**
  - Used for service discovery between distributed services.

- **ELK Stack and Zipkin**
  - Used for centralized logging and distributed tracing.