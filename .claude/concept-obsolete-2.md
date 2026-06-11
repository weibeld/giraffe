# Product Characterization: Self-Maintaining Structured Datasets

## 1. Core Problem

Maintaining structured datasets is tedious and error-prone.

Given:
- A dataset with entities (e.g. movies, books, places)
- A defined schema (fields A, B, C, D)

Typical workflow today:
- Adding a new item requires manual research for each field
- Ensuring consistency across items is difficult
- Updating volatile fields (ratings, prices, availability, etc.) requires ongoing manual effort

This leads to:
- incomplete datasets  
- inconsistent data quality  
- outdated information  
- high maintenance cost  

---

## 2. Key Insight

The fundamental problem is not data storage or querying, but:

**Maintaining a complete, consistent, and up-to-date dataset over time.**

Existing tools:
- Databases → store structured data
- Spreadsheets → flexible but manual
- ETL tools → operate on existing structured sources
- Search engines → discover information but don’t structure or maintain it

None solve:
**Automatic completion and continuous upkeep of datasets derived from messy, external sources.**

---

## 3. Proposed Solution

A system that:

**Automatically builds and maintains structured datasets of entities from messy, unstructured sources.**

---

## 4. Core Abstraction

The system operates on:

### Inputs
- A set of entities (e.g. URLs, ISBNs, names), OR
- A concept defining a set of entities (e.g. “all Kubernetes books”)

### Schema
- A user-defined set of fields (A, B, C, D)

### Output
- A structured dataset where:
  - every entity has consistent fields  
  - missing data is automatically filled  
  - data is kept up to date over time  

---

## 5. Core Capabilities

### 5.1 Extraction (Enrichment)

For each entity:
- Automatically research and extract required fields
- Normalize data into a consistent schema

Example:
Input: ISBN  
Output: title, author, publication year, rating, etc.

---

### 5.2 Completion

Given a partially filled dataset:
- Fill missing fields for existing items
- Ensure all items conform to the schema

---

### 5.3 Continuous Updates

For volatile fields:
- Periodically re-run extraction
- Keep data fresh (e.g. ratings, prices, availability)

---

### 5.4 Discovery (Optional)

Given a concept:
- Identify instances of that concept from open data sources
- Populate dataset automatically

Examples:
- “all beaches in Bali”
- “all tools in category X”
- “all Kubernetes books”

Discovery can be:
- one-time (initial dataset creation)
- recurring (keep dataset comprehensive)

---

### 5.5 Dataset Evolution

The system supports ongoing changes:

- Add/remove fields  
- Modify field definitions  
- Refine dataset scope  
- Split/merge datasets  
- Manually curate items  

The system adapts and re-processes data accordingly.

---

## 6. System Behavior

The system maintains a continuous loop:

entity set → extract + normalize → store structured dataset → monitor → update → (optional) expand entity set

---

## 7. Role of AI (LLMs / Agents)

The system relies on generative AI to:

- interpret concepts (e.g. “Kubernetes books”)  
- discover relevant entities  
- extract structured data from unstructured sources  
- infer missing values  
- normalize heterogeneous information  

This enables:
**semantic understanding + flexible extraction + automation**

---

## 8. Key Differentiators

### From Search Engines
- Search → returns documents  
- This system → returns structured datasets  

### From ETL / Data Pipelines
- ETL → assumes structured input  
- This system → works with messy, external data  

### From Databases / Spreadsheets
- Static storage  
- Manual updates  

This system:
- automatically completes data  
- continuously maintains it  

---

## 9. Conceptual Model

The system can be understood as:

**A database whose data is automatically populated and maintained from the open world.**

Or:

**A living dataset that evolves and maintains itself over time.**

---

## 10. Primary Use Cases

### Dataset Maintenance
- Maintain collections (movies, tools, places, etc.)
- Keep all fields complete and up to date

### Dataset Creation
- Generate datasets from concepts
- Achieve broad coverage with minimal manual effort

### Comparison / Analysis
- Structured data enables comparison, filtering, and decision-making  

### Application Input
- Use datasets as backend data sources, API inputs, or content for apps

---

## 11. Summary

The system addresses:

**The gap between unstructured world knowledge and structured, maintainable datasets.**

It transforms:

messy, incomplete, evolving information → structured, consistent, living datasets

---

## 12. Core Value Proposition

**Build and maintain structured datasets with minimal manual effort, even when data comes from messy, dynamic sources.**
