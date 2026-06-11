# Giraffe — Concept Doc (Draft)

## Core Concept

Giraffe is a platform for maintaining structured datasets consisting of items with shared fields.

The platform is fundamentally data-centric.

Giraffe aims to be the go-to solution whenever you find yourself collecting table-like data and want to record it. You may start by just manually entering data, then selectively automate the acquisition of individual fields (enrichment), and optionally use discovery to expand the dataset. Both the data and its acquisition logic may start simple and evolve into something fairly complex over time.

The central abstraction is:

```text
Dataset
  -> Items
    -> Fields
      -> Optional enrichment strategy
```

The system should remain fully usable without any AI functionality enabled.

---

# The Three Core Layers

Giraffe consists of three primary layers:

1. Structured Data Layer
2. Enrichment Layer
3. Discovery Layer

These layers are additive and optional.

---

# 1. Structured Data Layer

The foundation of the platform is structured dataset management.

Users can:

* create datasets
* define fields
* create items
* manually edit data
* organise and maintain curated collections of structured information

Examples of datasets:

* software tools
* books
* companies
* products
* research sources
* APIs
* datasets
* media collections

This layer functions independently of AI.

The platform should support:

* table-based editing
* CRUD operations
* generic APIs
* import/export
* arbitrary schemas

The structured data layer is the core product.

---

# 2. Enrichment Layer

Users can attach optional acquisition strategies to individual fields, describing how the value of that field is obtained.

Strategies span a spectrum of determinism:

* Deterministic: e.g. a specific API call or the extraction of a known page element — no AI involved
* Non-deterministic: natural-language LLM instructions

Example of a deterministic strategy:

```text
Field: githubStars
Strategy:
  GET https://api.github.com/repos/{owner}/{repo} -> stargazers_count
```

Examples of LLM-based strategies:

```text
Field: githubStars
Strategy:
  "Find the current GitHub star count"
```

Or:

```text
Field: summary
Strategy:
  "Read the homepage and summarise the product"
```

Open question: whether a strategy is strictly either deterministic or LLM-based, or whether the two can be intertwined within a single strategy (e.g. a deterministic fetch followed by LLM post-processing).

Characteristics:

* Strategies are optional — AI even more so
* Strategies are attached to fields
* Manual editing always remains possible
* Users can selectively automate only parts of a dataset

The system may support:

* periodic update runs
* stale data refreshes
* re-researching fields
* validation and correction workflows

The purpose of this layer is AI-assisted structured data maintenance.

---

# 3. Discovery Layer

The platform may optionally support dataset expansion and discovery.

Examples:

* discover similar tools
* discover related books
* discover companies within a category
* detect duplicates
* detect obsolete entries
* discover missing items

This layer operates at the dataset level rather than the field level.

The goal is to help users expand and maintain curated datasets over time.

Even within discovery workflows, the system remains fundamentally data-centric rather than agent-centric.

---

# Flexible Usage Model

Users can choose any level of automation.

Examples:

## Fully Manual

Users manually:

* create items
* define fields
* enter and edit data

## Partially Automated

Users selectively attach AI strategies to individual fields.

## Highly Automated

Users enable discovery and periodic update workflows.

The platform should support gradual adoption of automation without requiring users to commit to fully autonomous workflows.

---

# Reusable Acquisition Logic

Field strategies and discovery instructions should be saveable and reusable, so that acquisition logic can be applied to new contexts with minimal duplication.

Example: the logic for researching coworking spaces in Bali should be reusable for researching coworking spaces in Georgia, with only the context-specific parts adapted.

This applies to:

* field-level enrichment strategies
* dataset-level discovery instructions

---

# Schema and Data Evolution

Schemas are not static: fields can be added, removed, or redefined after a dataset has been created, and existing data could in principle be re-fetched accordingly (e.g. populating a newly added field across all existing items).

Open question: how much of an actual feature this is under the current focus — for fields with acquisition strategies, a schema change probably means updating these strategies rather than the system automatically re-extracting data.

---

# Data Exposition and Openness

The data maintained within Giraffe is intended to be exposed in a generic and reusable way rather than being tightly coupled to a specific frontend or application model.

---

# Related Solutions and Positioning

## Adjacent product categories

* Databases and spreadsheets: store structured data, but all acquisition and upkeep is manual
* ETL tools and data pipelines: automate data flows, but assume already structured sources
* Search engines and research tools: find information, but return documents rather than maintained structured datasets

Giraffe sits in the gap between these: a data-centric platform for curated structured datasets that is fully usable manually, with optional and gradual automation of data acquisition.

## Related products

* **Firecrawl Agent** — https://www.firecrawl.dev/app/agent
  Firecrawl Agent focuses on automated web data extraction. Users can define targets (e.g. websites or pages), and the system crawls and extracts structured data from them. It is particularly useful for scraping and structuring information from the web, with an emphasis on automation and scalability.

* **rtrvr.ai** — https://www.rtrvr.ai/
  rtrvr.ai provides tools for retrieving and extracting information from online sources. It enables users to query and gather data from across the web and transform it into usable structured outputs. The focus is on flexible retrieval and transformation of external data into formats suitable for further use.

* **Airtable AI Agents** — https://www.airtable.com/platform/ai-agents
  Airtable AI Agents extend Airtable's database-like environment with AI-driven automation. They allow users to enrich records, automate workflows, and extract information within Airtable bases. The system combines structured data management with AI-assisted operations directly inside a familiar spreadsheet-like interface.

Unlike these automation-first tools, Giraffe is manual-first: automation and AI are optional layers on top of an independently useful structured data product.
