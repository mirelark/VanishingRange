# Data Architecture

## 1. Introduction

### 1.1 Purpose

This document describes the data architecture of VanishingRange: the structure, flow, classification, and governance of data from external authoritative sources through internal processing pipelines to public-facing presentation. It is intended to support development, maintenance, and transparency review of the system's data handling practices.

### 1.2 Scope

This document covers all data ingested, processed, generated, and published by VanishingRange, including species range data, observation records, taxonomy, conservation status, AI-generated content, and provenance metadata. It does not cover application infrastructure or frontend architecture, which are addressed in system-architecture.md and software-architecture.md respectively.

### 1.3 Audience

This document is intended for:

- Developers contributing to or maintaining VanishingRange
- Educators and administrators evaluating the platform for classroom use
- Conservation professionals or researchers reviewing data handling practices
- General users seeking to understand how species information is sourced, processed, and presented

### 1.4 References

- [ADR-001: VPS Hosting Provider Selection](../decisions/ADR-001-VPS.md)
- [ADR-002: Operating System Selection](../decisions/ADR-002-OS.md)
- [ADR-003: Containerization and Deployment Orchestration Strategy](../decisions/ADR-003-Containerization.md)
- [system-architecture.md](system-architecture.md)
- [software-architecture.md](software-architecture.md)
- [IUCN Red List API Documentation](https://api.iucnredlist.org)
- [GBIF API Documentation](https://www.gbif.org/developer/summary)

---

## 2. Data Architecture Overview

### 2.1 Data Architecture Goals

The data architecture is designed to:

- Present authoritative species range and conservation data accurately and transparently
- Clearly distinguish between authoritative source data, derived processed data, and AI-generated content
- Ensure all published content is attributable to a verifiable source
- Minimize compute load at request time through precomputed outputs
- Support incremental data updates without full pipeline re-execution where possible
- Protect sensitive species location data to reduce poaching and trafficking risk
- Serve a K12 educational audience with age-appropriate, plainly communicated content

### 2.2 Data Architecture Principles

**Authoritative sources first.** IUCN and GBIF data is treated as the ground truth for species range and conservation status. Derived and generated content supplements but does not override authoritative records.

**Transparency by design.** Every piece of content presented to users is attributed to its source. AI-generated content is explicitly labeled and distinguished from authoritative data.

**Precompute for performance.** GeoJSON polygon outputs and enriched species content are computed during ETL processing and stored in the site schema, not generated at request time. This minimizes backend compute load and improves frontend responsiveness.

**Minimal footprint.** The dataset scope is bounded to IUCN Red List threatened and extinct species. Least concern and near-threatened species are excluded to keep storage and processing requirements within single-node constraints.

**Data minimization.** No user data is collected or stored. No data beyond what is necessary for species visualization and education is ingested or retained.

**Protect sensitive location data.** Precise species sighting locations from the most recent 90 days are withheld from the frontend to reduce poaching and trafficking risk. Historical range data is displayed in full.

### 2.3 High-Level Data Flow

```
External Sources (IUCN, GBIF, EoL, CoL, Protected Planet)
        |
        v
[EL Pipeline] --> Source Schema (raw ingested data + provenance)
        |
        v
[Transform Pipeline] --> Site Schema (precomputed GeoJSON polygons + species content)
        |
        v
[ML/NLP Pipeline] --> Site Schema (enriched metadata + AI-generated fun facts)
        |
        v
[FastAPI] --> Frontend (species data, polygon series, provenance metadata)
```

---

## 3. Data Sources

### 3.1 Source System Inventory

| Source | Type | Primary Use |
|--------|------|-------------|
| IUCN Red List | Authoritative conservation authority | Species range polygons, conservation status, threat assessments |
| GBIF | Observation aggregator | Georeferenced species occurrence records |
| Encyclopedia of Life (EoL) | Taxonomy and species descriptions | Supplementary species descriptions and common names |
| Catalogue of Life (CoL) | Taxonomy authority | Supplementary taxonomy normalization |
| Protected Planet | Habitat and protected area data | Supplementary habitat and range context |

### 3.2 Source Data Characteristics

**IUCN Red List:** Provides authoritative range polygons and conservation status for assessed species. Coverage is strongest for threatened and extinct classifications. Data is structured and well-documented. Polygons represent expert-reviewed range assessments rather than raw observation density.

**GBIF:** Provides georeferenced occurrence records aggregated from institutional collections, citizen science, and research datasets. Data quality varies by species and region. Temporal coverage spans from historical museum records to present-day observations. Structural inconsistencies exist across contributing datasets.

**EoL and CoL:** Provide supplementary taxonomy and species description content. Used primarily to supplement IUCN descriptions for AI enrichment processing.

**Protected Planet:** Provides habitat and protected area geometry. Used as supplementary range context where IUCN polygon coverage is limited.

### 3.3 Source Data Authority

IUCN Red List is the primary authoritative source for conservation status and species range. All conservation status classifications presented to users are sourced directly from IUCN assessments. GBIF observation data is treated as supplementary evidence for range visualization, not as an override to IUCN range polygons.

Where IUCN range polygons are unavailable for a given species-year combination, GBIF observation density is used as input to species distribution modeling and interpolation workflows. The provenance of all range data is tracked and surfaced to users.

### 3.4 Source Data Refresh Cadence

The EL pipeline checks for updates from source APIs on a scheduled basis. The following logic applies:

- If the target database schema is empty, a full load of all in-scope species is performed
- If the target schema is populated, only records updated since the last successful run are retrieved and processed
- IUCN reassessment cycles and GBIF occurrence updates trigger incremental updates to affected species records
- The ML enrichment pipeline runs independently of the EL pipeline and processes only records flagged as updated since the last enrichment run

---

## 4. Data Classification

### 4.1 Public Data

Data sourced directly from external authoritative systems and presented to users with attribution. Includes:

- IUCN species range polygons
- IUCN conservation status and threat assessments
- GBIF georeferenced occurrence records (subject to 90-day recency exclusion)
- EoL and CoL taxonomy and species descriptions
- Protected Planet habitat geometry

All public data is attributed to its source in the database and surfaced to users via the provenance display model.

### 4.2 Derived Data

Data produced by VanishingRange processing pipelines from authoritative source inputs. Derived data does not exist in source systems and is the product of transformation logic applied to ingested records. Includes:

- Precomputed annual GeoJSON polygon series per species
- Normalized and deduplicated taxonomy records
- Interpolated range polygons for years with insufficient observation density
- Related species associations produced by semantic similarity modeling

Derived data is labeled as derived in the provenance model and distinguished from authoritative source data in user-facing displays.

### 4.3 Generated Data

Content produced by AI/NLP models operating on source descriptions and taxonomy records. Generated content is not sourced from any authoritative external system and is produced entirely by VanishingRange processing pipelines. Includes:

- AI-extracted species fun facts
- Plain language species summaries adapted for middle school reading level

Generated content is explicitly labeled as AI-generated in user-facing displays. Source descriptions used as model inputs are attributed to their originating source. Generated content is subject to audience suitability screening prior to publication.

### 4.4 Excluded Data

Data that is deliberately withheld from the frontend for ethical or scope reasons:

- **Recent sighting locations:** Precise species occurrence locations from the most recent 90 days are withheld to reduce poaching and trafficking risk. Historical range data is displayed in full.
- **Least concern and near-threatened species:** Excluded from scope to manage dataset size within single-node infrastructure constraints. These species are generally well-represented in existing public resources.
- **User data:** No user data is collected, stored, or processed. VanishingRange has no user accounts, no cookies, and no tracking mechanisms.

### 4.5 Internal Processing Data

Data used internally by processing pipelines that is not published to the frontend. Includes:

- Raw ingested source records in the source schema prior to transformation
- Provenance tracking metadata for pipeline runs
- Intermediate transformation artifacts
- Model input and output logs

---

# 5. Data Model

### 5.1 Source Schema

### 5.2 Site Schema

### 5.3 Key Entities

### 5.4 Relationships

---

# 6. Data Ingestion Architecture

### 6.1 Extraction Process

### 6.2 Incremental Updates

### 6.3 Deduplication

### 6.4 Missing Data Handling

### 6.5 Data Freshness Controls

---

# 7. Transformation Architecture

### 7.1 Transformation Pipeline

### 7.2 Data Normalization

### 7.3 GIS Processing

### 7.4 Publication Preparation

---

# 8. Enrichment Architecture

### 8.1 Related Subject Discovery

### 8.2 NLP Model Overview

### 8.3 Similarity Determination

### 8.4 Enrichment Outputs

---

# 9. Generated Content Architecture

### 9.1 Generated Content Principles

### 9.2 Fact Candidate Identification

### 9.3 Audience Suitability Screening

### 9.4 Plain Language Transformation

### 9.5 Validation Workflow

### 9.6 Publication Requirements

---

## 10. Provenance Architecture

### 10.1 Provenance Principles

All data presented to users is attributable to a verifiable source. The following principles govern provenance tracking:

- Every species record in the site schema carries a reference to its authoritative source
- Derived data carries a reference to the source records from which it was produced and the processing logic that produced it
- Generated content carries a reference to the source text used as model input and identifies the model and configuration used for generation
- Provenance is surfaced to users in context, not buried in documentation

### 10.2 Provenance Data Model

The provenance model tracks the following attributes per record:

| Attribute | Description |
|-----------|-------------|
| source_system | Originating external system (IUCN, GBIF, EoL, CoL, Protected Planet) |
| source_record_id | Identifier of the record in the originating system |
| source_url | Direct URL to the originating record where available |
| ingestion_date | Date the record was ingested into the source schema |
| data_type | Classification: authoritative, derived, or generated |
| generation_model | Model identifier for generated content (null for authoritative/derived) |
| generation_input_source | Source text reference for generated content (null for authoritative/derived) |
| last_updated | Date of most recent update to the record |

### 10.3 Lineage Tracking

Processing pipelines record lineage at each transformation stage:

- EL pipeline records source system, retrieval date, and raw record identifier for each ingested record
- Transform pipeline records which source records contributed to each derived output
- ML pipeline records which source descriptions were used as model input for each generated output

Lineage data is stored in the source schema and referenced by the site schema via foreign key relationships.

### 10.4 Provenance Presentation

Provenance is surfaced to users in the following ways:

- Species information panel displays the source system and a link to the authoritative record for all displayed data
- AI-generated fun facts display an explicit label identifying the content as AI-generated and attribute the source description used as model input
- Range polygon series display the data type (authoritative, derived, or interpolated) for each year in the visualization
- Citation guide in the site footer provides pre-formatted APA and MLA citations for both VanishingRange and the underlying source datasets

---

## 11. Trust and Transparency Model

### 11.1 Authoritative Data

Species range polygons, conservation status, and threat assessments sourced directly from IUCN are presented as authoritative. These records are produced by subject matter experts through peer-reviewed assessment processes. VanishingRange does not modify or reinterpret authoritative source content.

Users are shown the IUCN source attribution and a direct link to the originating assessment record wherever authoritative data is displayed.

### 11.2 Derived Data

Derived data (including interpolated range polygons, normalized taxonomy, and related species associations) is clearly distinguished from authoritative source data. Where interpolation has been applied to fill gaps in historical range data, users are shown a label indicating that the polygon is an estimate rather than a directly sourced record, along with the methodology used to produce it.

The goal of derivation is to produce the most accurate and useful visualization possible from available data, not to fabricate precision that does not exist in source records.

### 11.3 Generated Data

AI-generated content (fun facts and plain language summaries) is explicitly labeled in the user interface. Users are shown:

- A clear indicator that the content was produced by an AI model
- The source text from which the content was derived, with attribution to the originating source system
- A brief explanation that AI-generated content may contain errors and should not be treated as authoritative

The intended audience for generated content is K12 students. Audience suitability screening is applied before publication to ensure content is appropriate for the target age range.

### 11.4 User Visibility

The trust and transparency model is designed to be visible without being intrusive. Labels and attributions are present in context rather than requiring users to navigate to documentation. The goal is to build appropriate trust in the data, not to overwhelm students with caveats, but to ensure teachers and administrators can evaluate the platform's data quality practices without needing to request documentation separately.

### 11.5 Verification Mechanisms

Users can verify the source of any displayed data by:

- Following source attribution links to the originating authoritative record
- Reviewing the citation guide for dataset-level attribution
- Accessing the data architecture documentation for a full description of processing methodology

---

# 12. Data Quality Architecture

### 12.1 Quality Objectives

### 12.2 Validation Rules

### 12.3 Interpolation Rules

### 12.4 Quality Indicators

### 12.5 Error Handling

---

# 13. Data Publication Architecture

### 13.1 Published Dataset Structure

### 13.2 Publication Workflow

### 13.3 Data Availability

### 13.4 Data Consumption Model

---

## 14. Privacy and Data Governance

### 14.1 Data Exclusion Policy

Precise species occurrence locations from the most recent 90 days are excluded from the frontend. This policy exists to reduce the risk that real-time location data could be used to facilitate poaching or trafficking of threatened and endangered species.

The 90-day window was selected as a balance between data freshness and protection. Historical range data predating the exclusion window is displayed in full. The exclusion is applied at the data retrieval stage: excluded records are not retrieved from the source data sets to prevent mishandling or access.

### 14.2 Sensitive Data Handling

VanishingRange handles no personally identifiable information. No user accounts exist. No user behavior is tracked. No cookies are set. No analytics services are integrated.

The only sensitive data category present in the system is recent species occurrence location data, which is handled under the exclusion policy described in 14.1.

### 14.3 Retention Policy

Source data is retained in the source schema for pipeline processing purposes. There is no defined retention limit for source data; records are updated incrementally and superseded records are replaced rather than archived. The source schema does not store historical pipeline run artifacts beyond what is needed for lineage tracking.

No user data is retained because no user data is collected.

### 14.4 Data Minimization

The dataset scope is bounded to IUCN Red List threatened and extinct species. This boundary was chosen for the following reasons:

- Least concern and near-threatened species would significantly expand dataset size beyond single-node infrastructure constraints
- The conservation narrative is most urgent and educationally compelling for threatened and extinct species
- These species are the primary focus of IUCN assessment efforts and have the strongest data coverage
- Existing public awareness and cultural conservation practices are generally stronger for common species, reducing the marginal educational value of including them

The minimization boundary is documented as a deliberate scope decision rather than a technical limitation.

---

# 15. Data Lifecycle Management

### 15.1 Acquisition

### 15.2 Storage

### 15.3 Processing

### 15.4 Publication

### 15.5 Retirement

---

# 16. Operational Data Processes

### 16.1 Processing Schedule

### 16.2 Pipeline Dependencies

### 16.3 Recovery Procedures

### 16.4 Monitoring

---

## 17. Risks and Assumptions

### 17.4 Assumptions

The following assumptions underlie the data architecture:

- IUCN and GBIF APIs remain publicly accessible and maintain backward-compatible data structures
- IUCN Red List polygon coverage for threatened and extinct species is sufficient to serve as the primary range data source for the majority of in-scope species
- GBIF observation density is sufficient to support meaningful range visualization for species where IUCN polygons are unavailable or incomplete
- The 90-day recency exclusion provides meaningful protection against poaching and trafficking risk while preserving the educational value of the visualization
- Ollama with Mistral 7B running on available CPU resources produces output of sufficient quality for K12 educational use after prompt engineering and audience suitability screening
- Middle school reading level is an appropriate baseline for the target audience and serves both younger and older K12 students adequately
- Source API structures and data formats will not change significantly without advance notice from data providers
