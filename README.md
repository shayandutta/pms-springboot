# Patient management (Spring microservices)

## Technology

- **Java** and **Spring Boot** services: API Gateway (**Spring Cloud Gateway**), **Auth** (Spring Security, JWT), **Patient**, **Billing**, and **Analytics**.
- **PostgreSQL** for persistent data; **Apache Kafka** for async domain events; **gRPC** and **Protocol Buffers** for synchronous service calls (e.g. patient ↔ billing).
- **Docker**-ready services; **LocalStack**-based AWS-style infrastructure in `infrastructure/` for local cloud-style deployment and testing.
- **Maven** builds; **SpringDoc OpenAPI** for API documentation behind the gateway.

## Methodology

The system is split by bounded context: each service owns its data and API. External traffic enters through the **API Gateway**, which routes to backend services and runs **JWT validation** on protected routes. **Patient** events are published to **Kafka** for **Analytics**; **Billing** is invoked via **gRPC** when patient flows require it. This keeps concerns separated while allowing integration tests across services (`integration-tests`).

## Project startup (local)

**Prerequisites:** **JDK 21**, **Maven**, and **Docker** (for dependencies such as databases, Kafka, and peer services when you run in containers).

1. **Infrastructure:** Start the dependencies your `application.properties` / `application.yml` expect (PostgreSQL, Kafka, and any other containers your environment uses). Configure JDBC URLs, Kafka bootstrap servers, and gRPC host/ports to match that environment (see each service’s `src/main/resources`).

2. **Service ports (defaults in repo):** Auth `4005`, Patient `4000`, Billing `4001` (gRPC often on `9001`), Gateway `4004`, Analytics as configured. In Docker, hostnames in gateway routes match service names; locally you may use `localhost` and matching ports in overrides.

3. **Build each service:** from the service directory (e.g. `auth-service/`), run:
   `mvn clean install`
   (or `mvn spring-boot:run` to start). Repeat for `patient-service`, `billing-service`, `analytics-service`, and `api-gateway` in an order that satisfies dependencies: typically databases and Kafka first, then auth and domain services, then the gateway last.

4. **HTTP tryouts:** use the sample requests under `api-requests/` and `grpc-requests/` with your base URL (e.g. gateway on port `4004`).

5. **Optional LocalStack / CDK stack:** with LocalStack available, you can use `infrastructure/localstack-deploy.sh` and the `infrastructure` project as documented in that module’s POM and scripts.

Adjust env vars and Spring profiles (e.g. `application-prod.yml` on the gateway) to match your deployment; there is no single `docker-compose` in this repository, so wire containers or IDE run configurations to the ports and hosts above.
