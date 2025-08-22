# Bank Account Microservices App

A small Spring Cloud microservices system demonstrating service discovery, config server, API gateway, and two domain services: account-service and customer-service.

## Architecture
- **Discovery Service (Eureka)**: service registry on `8761`.
- **Config Service (Spring Cloud Config Server)**: central configuration on `8888`.
- **Gateway Service (Spring Cloud Gateway)**: edge router on `9999`.
- **Account Service**: domain API on `8083`.
- **Customer Service**: domain API on `8084`.

Services register with Eureka and (where enabled) fetch configuration from the Config Server. The gateway discovers services via Eureka and forwards requests.

## Prerequisites
- JDK 17+
- Maven 3.9+
- Docker & Docker Compose (optional for containerized run)

## Quickstart (Docker Compose)
```bash
# From repo root
docker compose up --build
```

- Eureka Dashboard: `http://localhost:8761`
- Config Server: `http://localhost:8888`
- API Gateway: `http://localhost:9999`

Example API calls via the gateway (service ID based routing):
- Accounts list: `http://localhost:9999/ACCOUNT-SERVICE/api/accounts`
- Account by ID: `http://localhost:9999/ACCOUNT-SERVICE/api/accounts/{accountId}`
- Customers list: `http://localhost:9999/CUSTOMER-SERVICE/api/customers`

If routes are case-sensitive in your environment, also try lowercase service IDs:
- `http://localhost:9999/account-service/api/accounts`
- `http://localhost:9999/customer-service/api/customers`

Direct service access (bypassing gateway):
- Accounts list: `http://localhost:8083/api/accounts`
- Account by ID: `http://localhost:8083/api/accounts/{accountId}`
- Customers list: `http://localhost:8084/api/customers`

## Run Locally (without Docker)
In separate terminals, start services in this order:
```bash
# 1) Discovery
cd discovery-service && ./mvnw spring-boot:run

# 2) Config Server (needs discovery)
cd config-service && ./mvnw spring-boot:run

# 3) Domain services (need discovery + config)
cd customer-service && ./mvnw spring-boot:run
cd account-service && ./mvnw spring-boot:run

# 4) Gateway (needs discovery + config)
cd gateway-service && ./mvnw spring-boot:run
```

## Configuration
Local default ports (overridable via properties/env):
- discovery-service: `8761`
- config-service: `8888`
- gateway-service: `9999`
- account-service: `8083`
- customer-service: `8084`

Services use these env variables in Docker:
- `DISCOVERY_SERVICE_URL` (e.g., `http://bank-discovery-service:8761/eureka`)
- `CONFIG_SERVER_URL` (e.g., `http://bank-config-service:8888/`)

## Troubleshooting
- If you open `http://localhost:8888/...` and see metadata instead of data: this is the Config Server. Use gateway `:9999` or the service ports instead.
- Ensure all services are healthy before testing APIs. Check Docker healthchecks or Spring Boot `/actuator/health` endpoints:
  - `http://localhost:8761/actuator/health`
  - `http://localhost:8888/actuator/health`
  - `http://localhost:8083/actuator/health`
  - `http://localhost:8084/actuator/health`
  - `http://localhost:9999/actuator/health`
- If gateway routes 404:
  - Wait until services appear in Eureka (`http://localhost:8761`).
  - Try both uppercase and lowercase service IDs in the path.
  - Confirm `spring.cloud.config.enabled` for gateway is set appropriately (defaults to disabled in this repo) and that discovery is reachable.

## Repository Layout
- `discovery-service/`
- `config-service/`
- `gateway-service/`
- `account-service/`
- `customer-service/`
- `docker-compose.yml`

## License
MIT or as provided by the original author. 