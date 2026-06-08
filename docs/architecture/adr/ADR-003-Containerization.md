# ADR-003: Containerization and Deployment Orchestration Strategy

## Status

Accepted

---

## Context

The system is a single-node VPS-hosted deployment consisting of multiple services including a static frontend, REST API, database layer, and scheduled data processing pipelines.

The deployment requires a mechanism for:

- service isolation
- consistent deployment behavior across system updates
- controlled service lifecycle management
- separation of application components into independently managed units

The system operates under the following constraints:

- single-node infrastructure
- low operational overhead requirement
- container-based workload execution for all non-OS-level services
- limited traffic and educational/public-facing usage scope

---

## Decision

**Selected Approach: Containerized deployment using Docker Engine with Docker Compose**

All application components will be deployed as containers managed through Docker Compose.

Service structure:

- NGINX: static frontend serving and reverse proxy routing
- FastAPI: REST API backend service
- PostGIS: persistent spatial database layer
- Pipeline workers: scheduled containerized jobs executed via cron-based orchestration
- External access: only NGINX exposed publicly; internal services remain private within Compose network

---

## Rationale

Containerization was selected to provide a consistent deployment abstraction layer between application components and the host operating system.

Key benefits include:

- operational separation between services
- consistent deployment behavior independent of host OS updates
- isolation of runtime environments per service
- simplified lifecycle management for individual components
- reduced coupling between system services
- predictable deployment and teardown behavior across environments

Docker Compose was selected as the orchestration mechanism due to:

- simplicity of configuration for single-node deployments
- mature ecosystem support and documentation
- widespread industry usage and operational familiarity
- sufficient feature coverage for non-clustered workloads

Kubernetes and similar orchestration systems were rejected due to excessive operational complexity and infrastructure overhead relative to system scale.

Podman was considered but not selected due to lower ecosystem maturity and reduced alignment with common deployment patterns in similar VPS-hosted environments.

---

## Alternatives Considered

### Native host deployment (no containers)

Running services directly on the operating system without containerization.

Rejected due to:

- tighter coupling between services and host environment
- increased risk of dependency conflicts during OS updates
- reduced deployment consistency

---

### Podman + Podman Compose

Daemonless container runtime with rootless-first design.

Considered due to:

- improved security model
- simpler runtime architecture

Rejected due to:

- lower ecosystem maturity compared to Docker Compose
- reduced alignment with common deployment tooling and examples
- potential compatibility gaps in orchestration workflows

---

### Kubernetes

Cluster-based container orchestration platform.

Rejected due to:

- excessive operational complexity for single-node deployment
- unnecessary scaling and scheduling capabilities
- increased system overhead and maintenance burden

---

### Direct system services (systemd services / init-managed processes)

Service management via OS-level init system.

Rejected due to:

- reduced isolation between services
- increased coupling to host OS configuration
- higher maintenance overhead across service dependencies

---

## Consequences

### Positive

- clear separation of system components into isolated runtime units
- consistent deployment model independent of host OS configuration
- simplified service lifecycle management (start, stop, rebuild)
- reduced risk of dependency conflicts between services
- improved maintainability for multi-service architecture
- controlled internal networking between services via Docker Compose

---

### Negative

- dependency on Docker runtime environment
- additional abstraction layer between system and host OS
- need to manage container build and deployment lifecycle
- potential debugging overhead compared to native host processes

---

### Operational Notes

- all inter-service communication occurs over an internal Docker Compose network
- only NGINX is exposed to external network traffic
- pipeline workers are executed as scheduled containers via cron-based triggers
- PostGIS contains persistent data required for system operation
- containers are rebuilt and redeployed as part of update workflows

---

## References

- Docker documentation: https://docs.docker.com
- Docker Compose documentation: https://docs.docker.com/compose
