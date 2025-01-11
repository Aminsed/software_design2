## High-Level Architecture

Below is a Mermaid diagram illustrating the main components and how they interact.  
Use the “Markdown → Mermaid” format to render the diagram.

```mermaid
flowchart LR
    subgraph Clients
        Provider[Signal Provider (UI or API)] --> FE[Front-end]
        Subscriber[Subscriber (UI or API)] --> FE[Front-end]
    end
    
    FE --(HTTPS/REST)--> Gateway[API Gateway / Main FastAPI App]
    Gateway --> Auth[Authentication & Authorization Module]
    Gateway --> Payment[Payment & Subscription Module]
    Gateway --> ModelRegistry[Model Registry & Versioning DB]
    Gateway --> DataInterface[Data Access Layer (Restricted)]
    Gateway --> Scheduler[Scheduler / Queue Manager]
    Gateway --> ContainerOrchestrator[Container Orchestrator (Docker/K8s)]
    
    subgraph ContainerOrchestration
        ContainerOrchestrator --> C1[Container: Model #1]
        ContainerOrchestrator --> C2[Container: Model #N]
        C1-- No internet access --> DataDB[(Historical Data)]
        C2-- No internet access --> DataDB[(Historical Data)]
    end

    Scheduler --> Queue[Task Queue (e.g., Celery/RQ/RabbitMQ)]
    Queue --> C1
    Queue --> C2
    
    C1 --> ForecastStore[Forecast Results DB]
    C2 --> ForecastStore[Forecast Results DB]
    
    FE <--> ForecastStore
    Payment <--> Auth
    Payment --> BillingDB[(Billing/Subscription Records)]
```

### Architectural Components, Explained

1. **Front-end (FE)**  
   - A web-based client for both Signal Providers and Subscribers.  
   - Allows Providers to upload code (requirements.txt, app.py) and manage signal metadata.  
   - Allows Subscribers to browse signals, see performance metrics, and subscribe/unsubscribe.  
   - Interacts with the API Gateway via HTTPS/REST.

2. **API Gateway / Main FastAPI App (Gateway)**  
   - Central point for handling all HTTP requests (both provider and subscriber).  
   - Routes requests to specialized modules (Auth, Payment, etc.).  
   - Orchestrates container deployment, model versioning, scheduling trigger management, etc.  
   - Ensures code isolation and compliance with the “no modification once deployed” rule (by controlling model versions).

3. **Authentication & Authorization Module (Auth)**  
   - Handles user login, token issuance, role-based access (Provider vs. Subscriber).  
   - Manages entitlements (i.e., which Subscriber has access to which signal).

4. **Payment & Subscription Module (Payment)**  
   - Integrates with a billing platform (can be a simple PayPal/Stripe on-prem if feasible or just local manual payment tracking initially).  
   - Manages subscription states (free trial, active, expired).  
   - Ensures only paying (or trial) subscribers can request forecasts.

5. **Model Registry & Versioning DB (ModelRegistry)**  
   - Stores metadata about each model/signal: version, creation date, provider ID, status (live/offline), etc.  
   - Enforces immutability (once a model is “live,” it can’t be changed; a new version must be published).

6. **Data Access Layer (DataInterface)**  
   - Provides restricted read-only access to historical/technical data.  
   - No direct internet connection from containers.  
   - Allows containers to query necessary data for forecasts without exposing external networking.

7. **Container Orchestrator (ContainerOrchestrator)**  
   - Could be either Docker or Kubernetes-based.  
   - Spawns separate containers for each model version.  
   - Ensures isolation in runtime environments (Python/R versions, libraries).  
   - Network policies block outbound internet traffic.  
   - Can scale horizontally (spawn more containers if load increases).

8. **Scheduler / Queue Manager (Scheduler and Queue)**  
   - Runs tasks at a fixed interval (e.g., every hour).  
   - Places “run forecast” tasks in a queue (using Celery, RQ, or any job queue library).  
   - Each container listens for tasks (or is triggered by the orchestrator) to execute the forecast code.  
   - This design optimizes resource usage by scheduling forecast runs without manual intervention.

9. **Forecast Results DB (ForecastStore)**  
   - Persists forecast outputs (predictions, timestamps, metadata).  
   - Stores performance metrics for historical analysis and UI display.

10. **Billing/Subscription Records DB (BillingDB)**  
   - Maintains subscription/payment histories.  
   - Can be combined with Auth or kept separate for clarity (depending on design preference).

11. **Historical Data (DataDB)**  
   - A local on-prem database (e.g., PostgreSQL) containing time-series/market data.  
   - Accessible strictly through the DataInterface layer (no direct container-to-database credentials leak).


## Step-by-Step Implementation Phases

Below is a practical, 100% achievable roadmap for a single developer, aligned with your skill set (ML engineering, Flask/FastAPI, Docker, some Kubernetes).

### Phase 1: Core Infrastructure Setup

1. **On-Prem Server Preparation**  
   - Start with a single on-prem server (e.g., Linux machine), 16–32 GB RAM, 8+ CPU cores, and sufficient SSD storage.  
   - Install Docker (or prepare a lightweight Kubernetes distro if you have experience, like k3s or microk8s).  
   - Keep the system behind a firewall with SSL termination for secure connections.

2. **Container Orchestrator**  
   - If starting small, Docker Compose + local Docker Registry is enough.  
   - Or, if you prefer orchestration from the start, set up k3s (lightweight K8s).  
   - Ensure the orchestrator is configured so containers can only communicate with the internal network (no outbound internet).

3. **Databases Setup**  
   - Deploy PostgreSQL (or similar) for:  
     - User/Auth data (Auth).  
     - Model registry metadata (ModelRegistry).  
     - Subscription/Billing data (BillingDB).  
     - Forecast results (ForecastStore).  
     - Historical/technical data (DataDB).  
   - Alternatively, split them into separate schemas or separate DB instances based on your preference and resources.

4. **Queue & Scheduler**  
   - Deploy Celery with Redis/RabbitMQ or any scheduler (e.g., RQ, Chronos) for task scheduling.  
   - The scheduler triggers “forecast runs” every hour (or configured interval).  
   - The queue system ensures each container is triggered to run its forecast code, collecting results upon completion.

### Phase 2: API & Auth Foundations

1. **API Gateway (FastAPI/Flask)**  
   - Implement a monolithic service with separate route modules (e.g., /auth, /providers, /subscribers, /container, etc.).  
   - Build an Authentication layer (JWT tokens or session-based).  
   - Implement role-based access (Provider vs. Subscriber).

2. **User Management**  
   - Endpoints:  
     - POST /register (create user),  
     - POST /login (authenticate),  
     - GET /profile, etc.  
   - Initialize basic roles (Provider, Subscriber).

3. **Subscription Management**  
   - Payment can be mocked early with a “fake” payment success flow.  
   - Endpoints:  
     - POST /subscribe (start subscription/trial),  
     - POST /cancel (stop subscription).  
   - Store subscription states in BillingDB.

### Phase 3: Container Build & Model Registry

1. **Signal Provider Workflow**  
   - Create endpoints:  
     - POST /signals/create: Creates a new signal entry in ModelRegistry.  
     - PUT /signals/{id}/upload: Uploads or updates code (requirements.txt, app.py).  
     - POST /signals/{id}/go-live: Triggers container build/publish.  

2. **Immutable Versioning**  
   - Once a model is “go live,” mark that version immutable in ModelRegistry.  
   - If a provider modifies code, it creates a new version with a new container/DC image.

3. **Container Build Pipeline**  
   - For each new version, the platform automatically:  
     - Reads the user-provided Dockerfile or auto-generates one from (requirements.txt + base Python image).  
     - Builds an image (tagged with version).  
     - Pushes the image to an on-prem Docker registry.  
     - Deploys the container through ContainerOrchestrator.

4. **No Internet Access**  
   - Enforce network policies (either Docker network isolation or Kubernetes NetworkPolicies) to block outbound traffic.  
   - The container can only communicate with the DataInterface or queue system.

### Phase 4: Scheduling & Forecast Execution Flow

1. **Hourly Forecast Runs**  
   - Setup a Celery (or RQ) beat scheduler that dispatches tasks every hour.  
   - For each active signal container, the scheduler queues a “run_forecast” job.

2. **Execution in Container**  
   - Each container (model code) queries historical data from the DataDB through DataInterface’s internal endpoint.  
   - The model returns a JSON forecast result to the queue worker, or stores it directly in ForecastStore via an API call.

3. **Result Storage**  
   - The container or orchestrator writes results to ForecastStore, tagged with:  
     - Signal ID / version  
     - Timestamp  
     - Forecast details (predicted values, confidence intervals, etc.).

### Phase 5: Front-end Development

1. **Basic UI**  
   - Set up a simple React/Vue/Angular app (or even server-side templating with Jinja2 if minimal).  
   - Pages for:  
     - Provider Dashboard (My Signals, Edit, Go Live, Publish).  
     - Subscriber Dashboard (Browse Signals, Performance Stats, Subscribe/Unsubscribe).  

2. **Performance Visualization**  
   - Read from ForecastStore to display aggregated performance charts (accuracy, PnL backtest, etc.).  
   - Possibly replay historical forecasts to show how signals changed over time.

3. **Trial & Subscription Flows**  
   - Visible “Trial ends in X days” or “Subscribe for full access.”  
   - Handle subscription statuses using Payment & BillingDB endpoints.

### Phase 6: Payment & Monetization (Production)

1. **Integration with Real Payment Processor**  
   - Hook up to Stripe/PayPal, or an on-prem payment gateway if you must remain offline.  
   - Implement subscription logic (monthly or yearly).  
   - Manage free trial periods (up to one month) with automatic cancellation or conversion to paid plan.

2. **Entitlement Check**  
   - Each forecast request or result access will verify subscription status in Auth/Payment modules.  
   - If a user’s subscription is active or in trial, they can access current forecasts.

### Phase 7: Logging, Monitoring, and Auditing

1. **Application Logging**  
   - Centralize logs (API requests, container build logs, forecast execution logs).  
   - Provide a minimal admin dashboard to check system health.

2. **Performance Metrics**  
   - Collect container CPU/memory usage.  
   - Monitor daily forecast runs to ensure the scheduler is firing properly.

3. **Fraud Prevention & Auditing**  
   - Keep a history of container images and internal checksums.  
   - If a provider tries to swap code under the same version, the system rejects it (must create a new version).


## Hardware Requirements & Scaling Strategy

1. **Initial Setup (Single Machine)**  
   - 1 server with 8+ cores, 16–32 GB RAM, enough SSD (1–2 TB).  
   - Runs Docker (or k3s), the databases, and microservices.  
   - This is sufficient for initial usage (~100 users, with ~50–100 daily forecast tasks).

2. **Scaling Out**  
   - As usage grows:  
     - Separate the database to a dedicated machine (PostgreSQL server).  
     - Move queue & scheduler to a separate instance if needed for CPU-intensive tasks.  
     - Add additional worker nodes for container execution.  
   - Use load balancers (HAProxy or Traefik in K8s) in front of the API gateway.  
   - The container orchestrator can scale container replicas based on CPU usage/queue length.

3. **Storage Expansion**  
   - For large data volumes, add network-attached storage or scale out the data store (sharding or partitioning).  
   - Keep historical forecasts in a separate warehouse schema or in a time-series database (e.g., TimescaleDB).


## Emphasis on Modularity & Clear Interfaces

1. **Monolithic vs. Microservices**  
   - Start monolithic for simplicity: a single FastAPI app controlling modules.  
   - Migrate to microservices (Auth, Payment, Orchestrator Manager, etc.) as scale or complexity grows.

2. **Container Isolation**  
   - Each signal version runs in its own container, pinned to the provider’s specified environment (Python 3.8, 3.9, R, etc.).  
   - Use Dockerfiles that install only the dependencies declared in requirements.txt.  
   - No external network connectivity ensures IP/trade secret protection and prevents abuse.

3. **API Endpoints** (Illustrative Examples)  
   - Authentication:  
     - POST /auth/register  
     - POST /auth/login  
     - GET /auth/me (fetch profile, roles)  
   - Provider:  
     - GET /signals (list own signals)  
     - POST /signals (create new)  
     - PUT /signals/{signal_id}/upload (requirements, app.py)  
     - POST /signals/{signal_id}/go-live (build container, version immutability)  
     - POST /signals/{signal_id}/publish (make publicly discoverable)  
   - Subscriber:  
     - GET /signals (browse public signals)  
     - GET /signals/{signal_id}/performance  
     - POST /signals/{signal_id}/subscribe (trial or paid subscription)  
     - POST /signals/{signal_id}/unsubscribe  
   - Forecast Results:  
     - GET /signals/{signal_id}/forecasts (requires active subscription)  
   - Admin/Monitoring (optional advanced use):  
     - GET /admin/logs  
     - GET /admin/containers/status  

4. **Data Access**  
   - The container calls an internal endpoint (e.g., GET /data/historical?symbol=XYZ&start=...&end=...) to retrieve required time-series.  
   - This endpoint enforces provider entitlements: each model can only see the symbols/data it’s allowed to see.

5. **Scheduling**  
   - A separate process (Celery beat) triggers “run_forecast” tasks every hour.  
   - The tasks are pushed to a job queue.  
   - Each container picks up tasks relevant to its signal ID, runs the forecast, and returns logs/results.


## Putting It All Together

This plan is designed to be implementable by a single developer, leveraging Docker/Kubernetes knowledge, FastAPI for the main service, and minimal front-end frameworks for UI. You can start small (monolithic container + local DB + Docker Compose) and gradually expand to multiple servers as your user base grows and compute demands increase.

1. Build the core FastAPI API (including Auth, Provider, Subscriber modules).  
2. Set up a local Docker registry for building and storing signal images.  
3. Implement scheduling with Celery + Redis/RabbitMQ, ensuring each container can pick up tasks.  
4. Restrict container networking to your internal data interface.  
5. Develop the front-end (even a minimal single-page app) that interacts with these endpoints.  
6. Integrate payments when you’re ready to go production.  
7. Monitor, log, and refine.  

By following these phases carefully, you’ll have a fully on-premises platform that securely hosts multiple forecast models (in Python, R, etc.), manages their versions, schedules their runs, stores the results, and offers a clean UI for subscribers and providers alike. This design naturally accommodates future growth (both horizontal scaling of containers and vertical scaling of your databases/queues).
