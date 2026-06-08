# Roadmap

## ➡️ Phase 1 — Project Foundation & External Dependencies
- Acquire domain name, email service, and hosting infrastructure
- Provision primary server environment
- Initialize repository structure (primary + mirror)
- Define  and implement documentation system structure

### Documentation Scope
- README
- Architecture documentation (system, backend, frontend)
- Development logs (ETL, enrichment, data research)

---

## Phase 2 — Infrastructure Setup (Server + Core Services)
- Configure operating system environment
- Install and configure:
  - Docker
  - Firewall rules
  - NGINX
  - Python runtime
  - PostGIS database
  - FastAPI framework
- Establish secure environment configuration system (.env handling)
- Define baseline security and service architecture

---

## Phase 3 — Static Frontend & Web Serving
- Build static frontend application (HTML, CSS, JavaScript)
- Configure NGINX for local HTTPS serving
- Implement responsive layout (desktop, tablet, mobile)
- Validate cross-device rendering and usability

---

## Phase 4 — Data Source Schema (Raw Ingestion Layer)
- Design initial `source_data` PostGIS schema
- Integrate data models from:
  - IUCN
  - GBIF
  - Encyclopedia of Life (EoL)
  - Catalogue of Life (CoL)
- Implement schema creation and permission scripts
- Define secure API key management via environment configuration

---

## Phase 5 — ETL Pipeline (Data Ingestion & Normalization)
- Implement Python-based ETL ingestion system
- Retrieve and normalize external biodiversity datasets
- Perform deduplication across sources
- Support incremental updates for changed records
- Standardize transformation into structured database objects

---

## Phase 6 — Core Application Schema (v2 Data Model)
- Design enriched application schema:
  - species metadata
  - taxonomy and conservation status
  - temporal attributes and updates
  - geographic distribution fields
- Implement REST API layer using FastAPI
- Enable species retrieval and randomized selection endpoints
- Support frontend-driven data queries

---

## Phase 7 — Geospatial Processing Layer
- Generate GeoJSON outputs for species distributions
- Implement interpolation methods for incomplete spatial data
- Develop distribution approximation logic
- Standardize spatial outputs for frontend rendering

---

## Phase 8 — API Integration & Frontend Connectivity
- Connect frontend to backend REST API
- Implement robust handling for:
  - missing or incomplete data
  - API latency or failure states
- Define fallback UI behavior for partial datasets
- Validate full data flow from database to frontend

---

## Phase 9 — Metadata Enrichment (ML Layer)
- Implement metadata tagging system:
  - common name enrichment
  - related species inference
- Develop similarity-based enrichment logic
- Integrate ML outputs into application schema

---

## Phase 10 — Summarization & LLM Integration
- Integrate LLM pipeline for species summarization
- Generate automated “fun facts” and descriptions
- Implement prompt engineering and output validation layer
- Ensure consistency and reproducibility of generated outputs

---

## Phase 11 — Testing, Automation & Deployment Preparation
- Conduct full system testing across stack (DB, API, frontend)
- Implement scheduled data update jobs (cron-based pipeline execution)
- Define recurring data refresh cycle and maintenance windows
- Finalize production deployment configuration

---

## Final Step — Production Deployment
- Deploy system to public domain
- Enable HTTPS and production server configuration
- Publish application for public access
