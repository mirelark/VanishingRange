# System Architecture

## 1. Introduction

### 1.1 Purpose

This document describes the system architecture of VanishingRange: the runtime components, deployment environment, operational model, and non-functional characteristics of the system. It is intended to support development, maintenance, and operational understanding of the platform.

### 1.2 Scope

This document covers the hosting environment, runtime component architecture, deployment configuration, operational processes, security boundaries, and quality attributes of the VanishingRange system. Application-level software architecture is addressed in software-architecture.md. Data pipeline and data model architecture is addressed in data-architecture.md.

### 1.3 Audience

This document is intended for:

- Developers contributing to or maintaining VanishingRange
- System operators responsible for deployment and maintenance
- Educators or administrators performing technical due diligence before classroom adoption

### 1.4 References

- [ADR-001: VPS Hosting Provider Selection](../decisions/ADR-001-VPS.md)
- [ADR-002: Operating System Selection](../decisions/ADR-002-OS.md)
- [ADR-003: Containerization and Deployment Orchestration Strategy](../decisions/ADR-003-Containerization.md)
- [data-architecture.md](data-architecture.md)
- [software-architecture.md](software-architecture.md)

---

## 2. System Overview

### 2.1 Business Purpose

VanishingRange is a free, open educational tool that visualizes the historical and current geographic range of IUCN threatened and extinct species. It is built to help K12 students understand biodiversity loss through explorable data, with a middle school reading level as its baseline audience.

The system aggregates species range and observation data from authoritative public sources (IUCN, GBIF, Encyclopedia of Life, Catalogue of Life, and Protected Planet) and presents it as an interactive map visualization with supporting species information and AI-generated educational content.

### 2.2 System Objectives

- Deliver accurate, attributed species range visualizations for IUCN threatened and extinct species
- Serve K12 educational users with age-appropriate, plainly communicated content
- Operate sustainably on minimal infrastructure within a single-node, cost-constrained hosting environment
- Protect sensitive species location data to reduce poaching and trafficking risk
- Maintain full data provenance and transparency for all presented content
- Remain available as a free, open, and freely forkable educational resource

### 2.3 Architectural Principles

**Precompute for performance.** All GeoJSON polygon outputs and enriched species content are computed during ETL processing and stored in the site database schema. Request-time compute is limited to database reads and API response construction.

**Constraint-aware design.** Every architectural decision is made with awareness of single-node infrastructure limits: CPU, RAM, storage, and bandwidth. Design choices that reduce operational overhead or defer compute to low-traffic periods are preferred.

**Separation of concerns.** Each system component has a clearly defined and bounded responsibility. Components communicate through defined interfaces and do not share runtime environments.

**Transparency by default.** All data presented to users is attributed to its source. AI-generated content is explicitly labeled. Processing methodology is documented and publicly accessible.

**Recoverable by design.** No irreplaceable state exists in the system. All source data is recoverable from upstream APIs. All application code and configuration is version controlled in Git. Recovery from a full system failure is a redeploy and re-run of the ETL pipeline.

---

## 3. System Context

### 3.1 Stakeholders

| Stakeholder | Role | Primary Interest |
|-------------|------|-----------------|
| K12 Students | Primary end users | Accessible, accurate species range visualization |
| Teachers and Educators | Secondary users and adoption decision-makers | Curriculum alignment, data accuracy, age-appropriateness |
| School Administrators | Adoption approvers | Data privacy, platform reliability, citation support |
| Developers and Contributors | Technical maintainers | Code quality, architecture clarity, documentation |
| Conservation Professionals | Domain reviewers | Data accuracy, source attribution, methodology transparency |

### 3.2 External Systems

| System | Type | Interaction |
|--------|------|-------------|
| IUCN Red List API | Data source | Species range polygons, conservation status |
| GBIF API | Data source | Georeferenced species occurrence records |
| Encyclopedia of Life API | Data source | Species descriptions and common names |
| Catalogue of Life API | Data source | Taxonomy normalization |
| Protected Planet API | Data source | Habitat and protected area geometry |
| Ollama (local) | ML inference | AI-generated fun facts and plain language summaries |

### 3.3 Context Diagram

*[To be added -- Mermaid diagram showing system boundary, external data sources, and user interaction points]*

---

## 4. Runtime Architecture

### 4.1 High-Level Architecture

```
[User Browser]
      |
      v
[NGINX Container] -- serves static frontend assets
      |
      v
[FastAPI Container] -- REST API for species, polygon, and metadata requests
      |
      v
[PostGIS Container] -- site schema (precomputed outputs) + source schema (raw ingested data)
      ^
      |
[EL Pipeline Container] -- scheduled extraction and load from external APIs to source schema
[ML Pipeline Container] -- scheduled enrichment and transformation from source schema to site schema
```

### 4.2 Runtime Components

| Component | Container | Technology | Responsibility |
|-----------|-----------|------------|----------------|
| Frontend | nginx | NGINX, HTML/CSS/JS, Leaflet | Static asset serving, map visualization, species UI |
| API | fastapi | Python, FastAPI | REST endpoints for frontend data requests |
| Database | postgis | PostgreSQL + PostGIS | Persistent storage for source and site schemas |
| EL Pipeline | etl | Python | Scheduled extraction and load from external APIs |
| ML Pipeline | ml | Python, Ollama, sentence-transformers | Scheduled enrichment, metadata tagging, fun fact generation |

### 4.3 Component Responsibilities

**NGINX** serves the static frontend and acts as the single public-facing entry point. All external HTTP and HTTPS traffic enters the system through NGINX. Internal services are not exposed to the public network.

**FastAPI** handles all data requests from the frontend. It reads from the site schema in PostGIS and returns structured JSON responses. It has no write access to the database at request time.

**PostGIS** maintains two schemas: the source schema for raw ingested data with full provenance tracking, and the site schema for precomputed GeoJSON polygons and enriched species content ready for API delivery.

**EL Pipeline** runs on a scheduled cron basis, retrieving updated records from external APIs and loading them into the source schema. It performs deduplication across sources and flags updated records for downstream processing.

**ML Pipeline** runs on a scheduled cron basis, processing records flagged as updated by the EL pipeline. It performs metadata tagging using sentence-transformers for related species associations, and generates fun facts and plain language summaries using Ollama with Mistral 7B. Outputs are written to the site schema.

### 4.4 Runtime Interactions

*[To be added -- sequence diagram for request lifecycle and pipeline execution flow]*

---

## 5. Deployment Architecture

### 5.1 Hosting Environment

VanishingRange is deployed on a dedicated Hetzner VPS instance. See [ADR-001](../decisions/ADR-001-VPS.md) for hosting provider selection rationale.

Key hosting characteristics:

- Dedicated instance (no shared compute or network with other tenants)
- Full VM control (custom OS installation and configuration)
- 20TB monthly bandwidth allocation
- Predictable fixed monthly cost (~$35 USD)
- Single geographic region deployment

### 5.2 Network Boundaries

External network access is restricted to NGINX on ports 80 (HTTP) and 443 (HTTPS). All other services communicate over an internal Docker Compose network and are not reachable from the public internet.

SSH access is restricted to authorized keys. Password authentication is disabled.

### 5.3 Infrastructure Topology

*[To be added -- network topology diagram showing public/private boundary, container network, and port exposure]*

### 5.4 Technology Stack

| Layer | Technology |
|-------|------------|
| Hosting | Hetzner VPS |
| Operating System | Void Linux |
| Containerization | Docker Engine + Docker Compose |
| Reverse Proxy | NGINX |
| API Framework | FastAPI (Python) |
| Database | PostgreSQL + PostGIS |
| ML Inference | Ollama (Mistral 7B) |
| Semantic Similarity | sentence-transformers |
| Frontend Map | Leaflet.js + OpenStreetMap |
| IaC (planned) | Terraform |

---

## 6. Operational Model

### 6.1 Processing Schedule

| Process | Schedule | Description |
|---------|----------|-------------|
| EL Pipeline | Periodic cron | Extract updated species records from external APIs, load to source schema |
| ML Pipeline | Periodic cron (after EL) | Enrich updated records with metadata tags and AI-generated content |
| NGINX | Continuous | Serve static frontend assets |
| FastAPI | Continuous | Respond to frontend data requests |

Pipeline containers are not continuously running. They execute their scheduled job and exit, reducing attack surface and resource consumption during idle periods.

### 6.2 Operational Constraints

- Single-node deployment (no horizontal scaling)
- CPU-bound ML inference (Ollama runs on CPU, not GPU)
- Storage bounded by VPS disk allocation
- All pipeline processing must complete within cron scheduling windows without impacting frontend availability
- No budget for external managed services or third-party APIs beyond source data providers

### 6.3 Monitoring and Logging

*[To be added -- monitoring approach, log aggregation, alert thresholds, pending implementation decisions]*

### 6.4 Backup and Recovery

VanishingRange does not maintain traditional backups. Recovery strategy is based on the following:

- All application code and configuration is version controlled in Git and recoverable from the repository
- All source data is recoverable by re-running the EL pipeline against external APIs
- Full system recovery from a catastrophic failure is: redeploy from Git, re-run ETL pipeline to repopulate database
- No user data exists and therefore no user data recovery is required

This approach is appropriate given that VanishingRange is not the master of record for any data it presents. All authoritative data remains available at source.

---

## 7. Security Architecture

### 7.1 Trust Boundaries

*[To be added -- trust boundary diagram pending implementation]*

### 7.2 Data Access Model

*[To be added -- database role and permission model pending schema implementation]*

### 7.3 Security Assumptions

- The system has no authenticated users and presents no user-facing login surface
- No user data is collected, stored, or transmitted
- All external data sources are public APIs accessed over HTTPS
- Internal services communicate over a private Docker Compose network not reachable from the public internet
- SSH access is key-based only; password authentication is disabled
- Container images are sourced from official upstream repositories

### 7.4 Privacy and Data Handling

VanishingRange collects no user data. No cookies are set. No analytics services are integrated. No user accounts exist. The system has no mechanism to identify, track, or store information about individual users.

The only sensitive data category present in the system is recent species occurrence location data, handled under the 90-day exclusion policy documented in data-architecture.md section 14.1.

---

## 8. Quality Attributes

### 8.1 Availability

The system targets reasonable availability for a single-node educational tool with no SLA commitments. Planned maintenance windows are acceptable. Unplanned downtime is mitigated by the simplicity of the deployment and recovery from most failure scenarios is a container restart.

### 8.2 Integrity

Data integrity is maintained by:

- Treating IUCN and GBIF as authoritative sources and not modifying their data
- Tracking provenance for all records through the pipeline
- Distinguishing authoritative, derived, and generated content explicitly
- Labeling interpolated range data as estimated rather than authoritative

### 8.3 Transparency

All data presented to users is attributable. Processing methodology is documented in this repository. AI-generated content is explicitly labeled. Source attribution links are provided in context.

### 8.4 Maintainability

The system is designed for solo maintenance with minimal operational overhead:

- All configuration is version controlled
- Docker Compose provides consistent deployment and teardown
- Pipeline containers are stateless and independently restartable
- Documentation is maintained alongside code in the repository

### 8.5 Performance

Frontend performance is optimized through precomputed GeoJSON delivery. The API layer performs only database reads at request time. No real-time computation occurs in the request path. ML inference runs in scheduled batch processes during low-traffic periods.

---

## 9. Dependencies\

### 9.1 External Data Providers\

### 9.2 Third-Party Services\

### 9.3 Models and Processing Dependencies

---

## 10. Data Publication Philosophy\

### 10.1 Authoritative Sources\

### 10.2 Derived Content\

### 10.3 AI-Generated Content\

### 10.4 Provenance Requirements

---

## 11. Risks and Assumptions

### 11.1 Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| External API structural changes break ingestion pipeline | Medium | High | Monitor API changelogs; design EL pipeline for schema resilience |
| Single-node failure results in full outage | Medium | Medium | Recovery via Git redeploy and ETL re-run; acceptable for educational use case |
| ML inference produces inappropriate content for K12 audience | Low | High | Audience suitability screening prior to publication; human review workflow for generated content |
| Source data gaps produce misleading range visualizations | Medium | Medium | Minimum observation threshold; interpolation labeled as estimated; Known Limitations documentation |
| Rolling OS update introduces system instability | Low | Medium | Staged updates; minimal host customization to reduce configuration drift |

### 11.2 Assumptions

- The system will operate at low to moderate traffic consistent with K12 educational use
- External source APIs will remain publicly accessible without authentication or rate limiting that prevents scheduled ingestion
- Single-node infrastructure is sufficient for the anticipated usage profile
- Hetzner VPS uptime is sufficient for an educational tool without formal SLA requirements

### 11.3 Known Limitations

- IUCN Red List polygon coverage varies by species and taxonomic group. Not all in-scope species have complete historical range polygon coverage.
- Species with sparse GBIF observation histories may have interpolated range polygons for years with insufficient data. Interpolated polygons are labeled as estimated.
- A minimum observation threshold is applied. Years with fewer observations than the threshold default to the prior year's range rather than generating potentially misleading polygon artifacts.
- The 90-day data recency exclusion means the most recent sighting locations are not reflected in the visualization.
- Ollama with Mistral 7B running on CPU produces lower inference throughput than GPU-based deployment. This is acceptable for scheduled batch processing but means enrichment pipeline runs are slower than they would be on GPU hardware.
- The system is a solo-maintained open source project. Response times for bug reports and contributions are not guaranteed.
