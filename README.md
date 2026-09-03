# Distributed System Design & Event-Driven Architecture

This repository serves as a practical implementation of a **Distributed System** leveraging an **Event-Driven Architecture (EDA)** built on top of **Hexagonal Architecture**. 

The core application logic and building blocks reside in the [Hexagonal Architecture Repository (`github.com/abelgrubio/hexagonal_architecture:v0.1.0`)](https://github.com/abelgrubio/hexagonal_architecture), which provides cleanly decoupled domains, adapters, and ports. This project (`distributed_system_design`) orchestrates those components into a resilient, scalable, and fully observable microservices environment using Docker Compose.

---

## 🏗️ System Architecture

The ecosystem is composed of edge routing, an event-driven worker pipeline communicating asynchronously via a message broker, and a complete observability stack.

```
                  +-----------------------+
                  |    Traefik (Proxy)    |
                  +-----------+-----------+
                              |
                              v
                   [event-architecture-network]
                              |
       +----------------------+----------------------+
       |                      |                      |
       v                      v                      v
+--------------+      +---------------+      +---------------+
| worker-      |      | worker-       |      | worker-       |
| producer     | ---> | inventory     | ---> | order         |
+--------------+      +---------------+      +---------------+
       ^                      |                      |
       |                      v                      v
       |              +---------------+      +---------------+
       |              | worker-       |      | worker-       |
       |              | payment       | ---> | notification  |
       |              +---------------+      +---------------+
       |                      |
       +----------------------+  (RabbitMQ Broker)
                                     ^
                                     |
                             +---------------+
                             |   RabbitMQ    |
                             +---------------+
                                     |
                       (OpenTelemetry / Traces / Metrics)
                                     |
                                     v
                  +-------------------------------------+
                  |       Observability Stack           |
                  |  (OTel Collector, Prometheus,       |
                  |   Tempo, Jaeger, Loki, Grafana)     |
                  +-------------------------------------+
```

### Core Components

1. **Edge Router & Reverse Proxy (`traefik:v3.0`)**: 
   - Acts as the entry point to the system (`port 80`) with an active dashboard (`port 8080`) for automatic Docker container discovery.
2. **Event-Driven Microservices Worker Pipeline (`github.com/abelgrubio/hexagonal_architecture:v0.1.0`)**:
   - **`worker-producer`**: Synthetic cart items producer initiating the event workflow.
   - **`worker-inventory`**: Inventory consumer processing incoming cart states.
   - **`worker-order`**: Order management processing reserved inventory items.
   - **`worker-payment`**: Handles secure payment processing steps.
   - **`worker-notification`**: Final step handling user notification alerts.
3. **Message Broker (`rabbitmq:3-management`)**:
   - Manages asynchronous messaging across workers (`AMQP` on `5672`, Management UI on `15672`).
4. **Observability & Telemetry Stack**:
   - **OpenTelemetry Collector (`otel-collector`)**: Gathers traces, metrics, and logs from workers.
   - **Prometheus (`prom/prometheus:v3`)**: Scrapes performance metrics.
   - **Tempo & Jaeger**: Distributed tracing backends (`port 3200` & `16686`).
   - **Loki**: Log aggregation platform (`port 3100`).
   - **Grafana (`grafana:12.4.2`)**: Unified dashboard UI mapped to `http://localhost:4000`.

---

## 🚀 Getting Started

### Prerequisites

* [Docker](https://docs.docker.com/get-docker/) & [Docker Compose](https://docs.docker.com/compose/) installed on your machine.
* Access to pull the base image from GitHub Packages / Container Registry (`github.com/abelgrubio/hexagonal_architecture:v0.1.0`). Make sure you are authenticated with GitHub Container Registry if required:
  ```bash
  docker login ghcr.io -u <your-github-username>
  ```

### Configuration Files

Ensure you have the required configuration files in your root directory before launching:
* `otel.env`: Environment variables for OpenTelemetry configuration.
* `otel-collector.yaml`: OpenTelemetry Collector pipeline routing configuration.
* `prometheus.yaml`: Prometheus scrape configuration.
* `tempo.yaml`: Tempo storage and tracing configuration.
* `loki.yaml`: Loki log configurations.

### Running the System

1. Clone this repository and navigate into the directory infra.
2. Spin up the distributed system using Docker Compose:
   ```bash
   docker compose up -d
   ```
3. Verify that all containers are healthy and running:
   ```bash
   docker compose ps
   ```

---

## 📊 Accessing Services & Dashboards

Once running, you can access the operational interfaces via your browser:

* **Grafana Dashboards**: [http://localhost:4000](http://localhost:4000) *(Default credentials: `admin` / `admin`)*
* **Traefik Dashboard**: [http://localhost:8080](http://localhost:8080)
* **RabbitMQ Management UI**: [http://localhost:15672](http://localhost:15672) *(Default credentials configured via environment variables)*
* **Jaeger UI**: [http://localhost:16686](http://localhost:16686)

---

## 🔮 Next Steps & Roadmap

To further evolve this distributed system design, the following milestones are planned:

1. **Performance & Latency Tracking**: 
   - Integrate custom OpenTelemetry metrics to accurately measure end-to-end execution times (from cart production to notification delivery) across the hexagonal boundaries.
2. **Interactive API Gateway**: 
   - Develop an external REST/GraphQL API adapter that can interact dynamically with the system, allowing manual injection of events instead of relying solely on the synthetic `worker-producer`.
3. **Chaos & Load Testing**: 
   - Implement automated load scripts (e.g., using Locust or k6) to test system resilience, backpressure management, and queue saturation in RabbitMQ under heavy loads.
4. **Automated Alerting**: 
   - Configure alert rules inside Prometheus/Grafana to trigger notifications if worker error rates spike or latency thresholds are breached.

---

## 📝 License

This project is open-source and available under the [MIT License](LICENSE).