# Giraffe — Concept Doc (Draft)

## Core Concept

Giraffe is a platform for maintaining structured datasets consisting of items with shared fields.

The platform is fundamentally data-centric.

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
2. AI Enrichment Layer
3. AI Discovery Layer

These layers are additive and optional.

---

# 1. Structured Data Layer

The foundation of the platform is structured dataset management.

Users can:

* create datasets
* define fields
* create items
* manually edit data
* organize and maintain curated collections of structured information

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

# 2. AI Enrichment Layer

AI functionality exists primarily as optional field-level augmentation.

Users can attach optional LLM-based strategies to individual fields.

Example:

```text
Field: githubStars
Strategy:
  "Find the current GitHub star count"
```

Or:

```text
Field: summary
Strategy:
  "Read the homepage and summarize the product"
```

Characteristics:

* AI is optional
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

# 3. AI Discovery Layer

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

# Data Exposition and Openness

The data maintained within Giraffe is intended to be exposed in a generic and reusable way rather than being tightly coupled to a specific frontend or application model.
