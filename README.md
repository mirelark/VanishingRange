# VanishingRange.org
A free, open educational tool visualizing the historical and current geographical range of IUCN threatened and extinct species, built to help k-12 students understand biodiversity loss through explorable data.

## About
VanishingRange aggregates species range and observation from IUCN, GBIF, and allied sources to produce interactive geographic visualizations of threatened and extinct species across recorded history. The project scope is intentionally bounded to IUCN Red List threatened and extinct classifications: species where the conservation story is most urgent and the data most complete. Threatened, extinct-in-the-wild, and fully extinct species are included, as the distinction between the three carry educational weight.

The tool is designed with a middle-school reading level as its baseline, making it accessible to younger students while remaining substantive enough for secondary coursework and general public use.

## How it works
Users select a species and a time range from a simple interface. VanishingRange updates the map visualization showing the species' geographic range as an annual polygon series, color coded by year, built from IUCN range data and GBIF observation records. A species information panel provides taxonomy, current conservation status, and AI extracted species "fun facts" drawn from various source descriptions and attributed to their original records.

## Simple Architecture Overview
VanishingRange runs on a containerized stack hosted on a dedicated cloud server.
```
nginx (static frontend) > FastAPI (REST API) > PostGIS (site data schema) < PostGIS (source data schema) < ETL Python (source ingestion)
                                             ^ Transform/ML Python (data enrichment)
```
The source schema holds raw ingested data with full provenance tracking. The site schema holds precomputed geoJSON polygons and enriched species content served directly to the frontend. 

Full architecture documentation including decisioning and supporting analysis is available in the [architecture docs](docs/Architecture).

## Data Sources
+ **IUCN Red List** as the authoritative species range polygons and conservation status
+ **GBIF** for georeferenced species observation records
+ **Encyclopedia of Life** and **Catalogue of Life** for supplementary taxonomy and species descriptions
+ **Protected Planet** for supplementary habitat and range context 

All source data is attributed and provenance-tracked within the database, and displayed on the front-end where relevant. Precise species sighting locations are witheld from the front-end for the most recent 90 days to reduce poaching and trafficking risk. Historical range data is displayed in full.

In the scenario there is missing historical data, ranges are interpolated, supplemented by sighting data and species distribution modeling, to provide a more natural distribution.

## Project Status
VanishingRange is currently in active development. See the [project roadmap](docs/planning/roadmap.md) and [project board](https://github.com/users/mirelark/projects/1) for current progress and upcoming milestones.

## Data and Privacy Policy
VanishingRange collects no user data, uses no cookies, requires no accounts, and uses no tracking of any kind. 

## License
MIT License, see [LICENSE](docs/license.txt) for details

## Citation guide
Citations for this site and source data from IUCN and GBIF are available in the footer of each page, available in both APA and MLA format for historical range data.

For species information and fun facts, pre-built citations for the information source are available by hovering over the reference superscript next to the related information.
