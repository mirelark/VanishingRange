# Documentation Index

## Overview

This repository is structured into two primary layers:

- /docs contains planning, architecture, decisions, design, and development artifacts
- /code contains the production system implementation

The documentation is designed to support traceability from planning through research, design, development, and final implementation.

---

## Documentation Structure

```
VanishingRange/
├─ index.md
├─ LICENSE.txt
├─ README.md
├─ CONTRIBUTING.md
├─ .gitignore
├─ code/
|   ├─ api/
|   |   ├─ .env.example
|   |   ├─ DOCKERFILE
|   |   ├─ requirements.txt
|   |   └─ main.py
|   ├─ database/
|   |   ├─ DOCKERFILE
|   |   ├─ site_schema.sql
|   |   └─ source_schema.sql
|   ├─ frontend/
|   |   ├─ DOCKERFILE
|   |   ├─ index.html
|   |   ├─ options.json
|   |   ├─ style.css
|   |   └─ interface.js
|   ├─ infra/
|   |   ├─ .env.example
|   |   ├─ docker-compose.yaml
|   |   ├─ nftables.conf
|   |   └─ nginx.conf
|   └─ pipeline/
|       ├─ extract-load/
|       |   ├─ .env.example
|       |   ├─ DOCKERFILE
|       |   ├─ requirements.txt
|       |   └─ el_pipe.py
|       ├─ transform-load/
|       |   ├─ .env.example
|       |   ├─ DOCKERFILE
|       |   ├─ requirements.txt
|       |   └─ el_pipe.py
|       ├─ data-enrichment/
|       |   ├─ .env.example
|       |   ├─ DOCKERFILE
|       |   ├─ requirements.txt
|       |   └─ data_enrichment.py
|       └─ content-generation
|           ├─ .env.example
|           ├─ DOCKERFILE
|           ├─ requirements.txt
|           └─ fun_facts.py
└─ docs/ 
    ├─ architecture/ 
    |   ├─ images/ 
    |   ├─ data-architecture.md
    |   ├─ software-architecture.md
    |   └─ system-architecture.md
    |   ├─ adr
    |       ├─ ADR-000-template.md
    |       ├─ ...
    |       └─ analysis/
    |           ├─ ADR-000-A-template.md
    |           └─ ...
    ├─ engineering/
    |   ├─ diagrams/ 
    |   ├─ notes/ 
    |   ├─ pseudocode/ 
    |   └─ research/ 
    ├─ planning/
    |   ├─ images/ 
    |   ├─ roadmap.md
    |   └─ scope.md
    └─ ux/
        ├─ diagrams/ 
        └─ wireframes/ 
```
---

### /docs/planning
High-level project definition and execution planning.

- roadmap.md: phased execution plan
- scope.md: project boundaries and constraints

Defines overall project direction and scope.

---

### /docs/architecture
System-level structure and design of the platform.

- system-architecture.md: infrastructure and deployment model
- software-architecture.md: service and application structure
- data-architecture.md: data flow, schema design, and external data sources

Defines how the system is composed and how components interact.

---

### /docs/adr
Research and architectural decision-making artifacts.

- analysis/: exploratory research and evaluation work supporting decisions

Represents reasoning and justification behind system design choices.

---

### /docs/design
User-facing and interaction-level design work.

- wireframes/: UI layout concepts
- behavior/: system and interaction behavior definitions
- diagrams/: UI and UX flow diagrams and conceptual visuals

Defines how the system is experienced and interacted with.

---

### /docs/engineering
Active development workspace and implementation reasoning.

- research/: implementation-specific technical investigation
- notes/: development logs, pseudocode, and working assumptions
- diagrams/: technical flow diagrams and backend or system sketches

Represents in-progress engineering work during feature implementation.

---

## Code Structure (Reference Layer)

### /code/api
Backend API service built with FastAPI
- request handling
- data retrieval layer
- service orchestration

### /code/pipeline
Data ingestion and transformation pipelines */code/pipeline/extract-load* and *transform-load*
- source extraction
- normalization
- deduplication and transformation logic

Data enrichment and generation layer *code/pipeline/data-enrichment* and *content-generation*
- metadata enrichment
- fun fact generation
- auxiliary machine learning processing

### /code/database
Database schema and initialization scripts
- raw schema definitions
- processed schema definitions
- PostgreSQL and PostGIS setup

### /code/frontend
Client-side application
- user interface rendering
- interactive visualization
- API integration layer

### /code/config
Infrastructure and deployment configuration
- Docker Compose setup
- NGINX configuration
- firewall rules
- environment variable templates

---

## Relationship Model

The system follows a traceable documentation and implementation flow:

planning, architecture, decisions, development, code

Where:

- planning defines scope and execution phases
- architecture defines system structure
- decisions define reasoning and justification
- development captures implementation thinking and experimentation
- code is the final executable system

---

## Purpose

This structure ensures:

- separation between planning, reasoning, and implementation
- traceability from intent to production code
- isolated but connected development artifacts
- reproducible understanding of the system from documentation alone
