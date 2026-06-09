# Software Architecture

## 1. Introduction

### 1.1 Purpose

This document describes the software architecture of VanishingRange: the application components, API design, data access patterns, client interaction model, and quality attributes of the system from a software perspective. It is intended to support development, contribution, and technical review of the platform.

### 1.2 Scope

This document covers the frontend application, REST API, data access layer, client interaction model, and published data contracts. Infrastructure and deployment architecture is addressed in system-architecture.md. Data pipeline and data model architecture is addressed in data-architecture.md.

### 1.3 References

- [ADR-001: VPS Hosting Provider Selection](../decisions/ADR-001-VPS.md)
- [ADR-002: Operating System Selection](../decisions/ADR-002-OS.md)
- [ADR-003: Containerization and Deployment Orchestration Strategy](../decisions/ADR-003-Containerization.md)
- [system-architecture.md](system-architecture.md)
- [data-architecture.md](data-architecture.md)
- [Leaflet.js Documentation](https://leafletjs.com)
- [FastAPI Documentation](https://fastapi.tiangolo.com)

---

## 2. Functional Overview

### 2.1 User Capabilities

VanishingRange provides the following user-facing capabilities:

- Select a species from a searchable dropdown list bounded to IUCN threatened and extinct taxa
- Select a time range using start and end year controls
- View an interactive map displaying the selected species' geographic range as a layered polygon series, color coded by year
- View a species information panel displaying taxonomy, conservation status, common names, habitat summary, and AI-generated fun facts
- Access source attribution and provenance information for all displayed content
- Copy pre-formatted APA and MLA citations for the displayed species record and underlying source datasets
- Navigate the map freely (pan, zoom, and interact with polygon layers)

### 2.2 System Responsibilities

The system is responsible for:

- Serving the static frontend application via NGINX
- Responding to frontend data requests via the FastAPI REST API
- Delivering precomputed GeoJSON polygon series for requested species and year ranges
- Delivering species metadata, conservation status, common names, and enriched content
- Delivering provenance metadata for all displayed content
- Handling null and missing data states gracefully without breaking the user interface
- Returning meaningful error states for invalid or out-of-range requests

---

## 3. Application Architecture

### 3.1 High-Level Architecture

The application follows a static frontend / REST API / database pattern with no server-side rendering. All UI logic runs in the browser. The API layer is read-only at request time. All content is precomputed and stored in the site schema prior to serving.

```
[Browser]
    |
    | HTTPS
    v
[NGINX] -- serves static HTML, CSS, JS assets
    |
    | Internal Docker network
    v
[FastAPI] -- REST API, reads from site schema
    |
    | Internal Docker network
    v
[PostGIS] -- site schema (precomputed GeoJSON + species content)
```

### 3.2 Component Diagram

*[To be added -- component diagram showing frontend modules, API endpoints, and database schemas]*

---

## 4. Component Architecture

### 4.1 Front-End Components

| Component | Technology | Responsibility |
|-----------|------------|----------------|
| Map Layer | Leaflet.js + OpenStreetMap | Interactive map rendering, polygon layer management, year-coded color display |
| Species Selector | Custom JS dropdown | Species search and selection, triggers API request on selection |
| Time Range Selector | Custom JS controls | Start and end year selection, triggers API request on change |
| Species Panel | Custom JS + HTML | Displays species information, fun facts, provenance attribution, and citation tools |
| API Client | Fetch API | HTTP requests to FastAPI endpoints, response handling, error state management |
| Citation Generator | Custom JS | Formats APA and MLA citations from species metadata for clipboard copy |

### 4.2 API Components

| Component | Technology | Responsibility |
|-----------|------------|----------------|
| Species List Endpoint | FastAPI | Returns paginated list of available species with basic metadata |
| Species Detail Endpoint | FastAPI | Returns full species metadata, content, and provenance for a given species |
| Polygon Series Endpoint | FastAPI | Returns GeoJSON polygon series for a given species and year range |
| Related Species Endpoint | FastAPI | Returns related species associations for a given species |

### 4.3 Component Responsibilities

**Map Layer** manages the Leaflet map instance, renders GeoJSON polygon layers received from the API, applies year-coded color scaling across the polygon series, and handles layer transparency stacking. It does not perform any data fetching or business logic.

**Species Selector** presents the available species list to the user, supports text filtering, and triggers downstream data requests when a selection is made. It is responsible for preventing invalid selections from reaching the API.

**Time Range Selector** presents year range controls and validates that the selected range is within the available data window for the selected species. It prevents out-of-range requests from reaching the API.

**Species Panel** renders all non-map content for the selected species: taxonomy, status, common names, summary, fun facts, and data provenance. It handles null content states gracefully, displaying appropriate messaging when data is unavailable rather than rendering empty fields.

**API Client** manages all HTTP communication between the frontend and FastAPI. It handles request construction, response parsing, loading state management, and error state surfacing to the UI.

**FastAPI** handles all incoming data requests, queries the PostGIS site schema, and returns structured JSON responses. It performs no data transformation at request time, all content is precomputed. It enforces input validation and returns standardized error responses for invalid requests.

---

## 5. API Architecture

### 5.1 Endpoints

*[To be completed during implementation -- endpoint paths, methods, and parameter definitions]*

Planned endpoints:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/species` | GET | Returns list of available species |
| `/species/{id}` | GET | Returns full species detail and metadata |
| `/species/{id}/polygons` | GET | Returns GeoJSON polygon series for year range |
| `/species/{id}/related` | GET | Returns related species list |

### 5.2 Request Model

*[To be completed during implementation -- request parameter definitions and validation rules]*

### 5.3 Response Model

*[To be completed during implementation -- response schema definitions per endpoint]*

### 5.4 Error Handling

The API returns standardized error responses for the following conditions:

- Invalid species identifier — 404 Not Found
- Requested year range outside available data window — 400 Bad Request with available range in response body
- Missing polygon data for requested year range — 200 OK with null polygon entries and a data availability indicator
- Internal processing error — 500 Internal Server Error with error reference

All error responses follow a consistent JSON structure to allow the frontend to handle them uniformly.

---

## 6. Data Access Architecture

### 6.1 Database Interaction

FastAPI interacts exclusively with the site schema in PostGIS. It has no access to the source schema at request time. All queries are read-only. No writes occur through the API layer.

Database interaction uses an async Python database library to avoid blocking the API event loop during query execution.

### 6.2 Stored Procedures

*[To be completed during implementation -- stored procedure definitions if applicable]*

### 6.3 Read-Only Access Model

The FastAPI database user is granted SELECT permissions on the site schema only. It has no access to the source schema and no INSERT, UPDATE, or DELETE permissions on any schema. This is enforced at the database role level, not only at the application level.

---

## 7. Client Interaction Model

### 7.1 Request Lifecycle

1. User selects a species from the species selector dropdown
2. Frontend API client sends a request to `/species/{id}` to retrieve species metadata
3. Species panel renders with available metadata while polygon request is in flight
4. Frontend API client sends a request to `/species/{id}/polygons`
5. On response, map layer clears existing polygons and renders all returned GeoJSON series with year-coded color scaling
6. User may adjust year range: selector triggers map layer updates
7. User may select a different species: process repeats from step 2

### 7.2 UI Update Flow

All UI updates are driven by API responses. The frontend maintains local state for the current species selection and year range. State changes trigger API requests. API responses trigger UI updates. There is no client-side data caching between sessions.

### 7.3 Error State Presentation

The frontend handles the following error states without breaking the user experience:

- **Species not found:** Species selector prevents selection of invalid identifiers. If an invalid identifier reaches the API, the species panel displays a not found message.
- **No polygon data for selected year range:** Map layer displays a message indicating no range data is available for the selected period. Adjacent years with available data are suggested where possible.
- **Partial polygon data:** Years within the requested range that have no polygon data are skipped in the layer rendering. A legend indicator marks which years in the series have data and which do not.
- **API unavailable:** A connection error state is displayed in both the map layer and species panel with a retry prompt.
- **Invalid user input:** Year range selectors enforce valid input bounds client-side. Out-of-range values are corrected to the nearest valid value rather than submitted to the API.

---

## 8. Quality Attribute Realization

### 8.1 Performance

Frontend performance is optimized through precomputed GeoJSON delivery. The API performs only database reads at request time; no transformation, no model inference, no external API calls. GeoJSON polygon files are stored in the site schema ready for direct delivery.

For species with large polygon series, the polygon endpoint can accept a year range parameter so the frontend requests only the polygons needed for the current view rather than the full historical record.

### 8.2 Integrity

The API serves only content that has passed through the full pipeline, results from extraction, transformation, enrichment, and audience suitability screening. Incomplete or unvalidated records are not published to the site schema and cannot be served to users.

### 8.3 Transparency

All API responses include provenance metadata alongside content. The frontend is responsible for rendering provenance attribution in context, inline with the content it describes rather than in a separate documentation section.

AI-generated content fields in the API response carry explicit type labels (`generated`, `derived`, or `authoritative`) so the frontend can apply appropriate display treatment without hard-coding content type assumptions.

### 8.4 Maintainability

The frontend components are primarily built with vanilla JavaScript, HTML, and CSS - no build toolchain, no framework dependencies, no npm ecosystem. This minimizes maintenance overhead and reduces the risk of dependency rot over time. Any contributor with basic web development knowledge can read and modify the frontend without toolchain setup.

*[To be determined -- frontend framwork selection for map rendering beterrn Leaflet, OpenLayers, MapBox]*

The API is built with FastAPI, which auto-generates OpenAPI documentation from route definitions. This documentation serves as a living API reference without requiring separate maintenance.

---

## 9. Published Data Contract

### 9.1 Subject Data

Species detail responses include the following fields:

| Field | Type | Source | Description |
|-------|------|--------|-------------|
| species_id | string | Internal | Unique species identifier |
| scientific_name | string | IUCN / CoL | Binomial nomenclature |
| common_names | array | IUCN / EoL | List of common names with language tags |
| iucn_status | string | IUCN | Current Red List status category |
| iucn_status_year | integer | IUCN | Year of most recent IUCN assessment |
| taxonomy | object | IUCN / CoL | Kingdom, phylum, class, order, family, genus |
| description | string | EoL | Species description text |
| available_years | array | Internal | Years for which polygon data exists |
| data_type | string | Internal | authoritative / derived / interpolated |

### 9.2 GeoJSON Data

Polygon series responses include an ordered array of annual polygon objects:

| Field | Type | Description |
|-------|------|-------------|
| year | integer | Year the polygon represents |
| geojson | object | GeoJSON Feature object with polygon geometry |
| data_type | string | authoritative / interpolated |
| source | string | Source system for this year's polygon |
| observation_count | integer | Number of observations contributing to this polygon (null for IUCN-sourced polygons) |

### 9.3 Related Subjects

Related species responses include an ordered list of semantically similar species:

| Field | Type | Description |
|-------|------|-------------|
| species_id | string | Related species identifier |
| scientific_name | string | Binomial nomenclature |
| common_name | string | Primary common name |
| iucn_status | string | Red List status |
| similarity_basis | string | Brief description of the similarity relationship |

### 9.4 Generated Facts

Fun fact fields in species detail responses include:

| Field | Type | Description |
|-------|------|-------------|
| fact_text | string | AI-generated fun fact text |
| source_text_attribution | string | Attribution for the source description used as model input |
| source_system | string | Source system for the input description |
| generation_model | string | Model identifier used for generation |
| content_type | string | Always `generated` for AI-produced content |

### 9.5 Provenance Metadata

All responses include a provenance block:

| Field | Type | Description |
|-------|------|-------------|
| source_system | string | Primary originating source system |
| source_record_id | string | Record identifier in source system |
| source_url | string | Direct URL to source record where available |
| ingestion_date | string | ISO 8601 date of last ingestion |
| last_updated | string | ISO 8601 date of last record update |

---

## 10. Generated Content Presentation

### 10.1 Display Rules

AI-generated content is displayed in the species panel under a clearly labeled section. The label explicitly identifies the content as AI-generated. Generated content is visually distinguished from authoritative source content through consistent UI treatment — a distinct section header, a content type badge, and reduced visual prominence relative to authoritative fields.

Fun facts are presented as supplementary educational content, not as primary species information. The species description sourced from EoL or IUCN is presented first and with greater visual weight.

### 10.2 Provenance Requirements

Every generated content item displayed to users includes:

- An explicit AI-generated label
- Attribution to the source description used as model input
- A link to the originating source record
- The name of the model used for generation

These elements are rendered inline with the content, not in a footnote or separate documentation section.

### 10.3 AI Content Disclaimer

The species panel includes a brief disclaimer in proximity to generated content:

> Fun facts are generated by an AI model from species descriptions provided by [source attribution]. AI-generated content may contain errors and should not be treated as an authoritative source. Always verify important facts with the original source.

The disclaimer is concise enough not to overwhelm a K12 student while being sufficient for teachers and administrators evaluating the platform for classroom use.

---

## 11. Related ADRs

- [ADR-001: VPS Hosting Provider Selection](adr/ADR-001-VPS.md)
- [ADR-002: Operating System Selection](adr/ADR-002-OS.md)
- [ADR-003: Containerization and Deployment Orchestration Strategy](adr/ADR-003-Containerization.md)
