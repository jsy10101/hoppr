# Hoppr

Hoppr is a ride-sharing platform that connects riders with nearby drivers through a set of Go microservices, real-time messaging, and a modern web client. The project focuses on the full trip creation journey—from previewing a route and pricing options to assigning a driver and collecting payment.

## Why Hoppr?

-   Deliver a reliable end-to-end ride request flow for riders and drivers.
-   Provide real-time updates through WebSockets and message queues.
-   Keep pricing, routing, and driver assignment isolated in focused services that can scale independently.

## Architecture at a Glance

-   `services/api-gateway` – Public HTTP and WebSocket entrypoint, forwards trip creation to the trip service over gRPC, and is designed to broker driver commands via RabbitMQ.
-   `services/trip-service` – gRPC service that generates route previews with OSRM, estimates fares, persists rider selections, and prepares trip lifecycle events.
-   `services/driver-service` – Worker service (scaffolded) that will subscribe to trip events, manage driver availability, and push assignments back through the gateway.
-   `web` – Next.js 15 front-end showing map-based trip previews, fare options, and real-time driver state.
-   `shared` – Contracts, protobuf models, and utilities consumed across services.
-   `infra` – Local Kubernetes manifests, Dockerfiles, and scripts (including Tilt) for containerized development.

Supporting diagrams for the trip creation journey and event bus live in `docs/architecture/`.

## Key Integrations

-   **Routing**: OSRM’s public API powers realistic distance and duration estimates.
-   **Messaging**: RabbitMQ will route trip events and driver commands between services.
-   **Payments**: Stripe checkout flow is orchestrated through payment events (service under construction).
-   **Mapping UI**: React Leaflet renders live driver and rider locations in the web client.

## Getting Started

### Prerequisites

-   Docker Engine running locally.
-   Minikube cluster (`minikube start`) with your kube context pointing to it.
-   [Tilt](https://tilt.dev) installed (used to orchestrate all services defined in `Tiltfile`).
-   Optional: Go 1.23+ and Node.js 20+ if you plan to run services or the web client outside of Tilt.

### One-Command Spin Up

```bash
minikube start     # if your cluster is not already running
tilt up            # watches and deploys all services to the cluster
```

Tilt builds each Docker image, applies the manifests under `infra/development`, and streams logs so you can see every service come online in one terminal window.

### Handy Extras

-   Regenerate protobuf contracts after editing `.proto` files:
    ```bash
    make generate-proto
    ```
-   To work on the Next.js front-end in isolation, start it from `web/` with `npm run dev`.

## Additional Documentation

-   Trip creation flow (sequence diagrams): `docs/architecture/trip-creation-flow-v1.md`
-   RabbitMQ routing key map: `docs/architecture/rabbitmq-flow-v0.md`
-   Kubernetes deployments for local clusters: `infra/development/k8s/`

## Roadmap Highlights

-   Flesh out driver-service consumers and command handlers.
-   Stand up the payment service to finalize Stripe checkout sessions.
-   Add persistent storage (MongoDB) behind the trip service repository.
