# Giraffe — Concept Doc

## Core Idea

Giraffe aims to be the go-to solution whenever you find yourself collecting table-like data and want to record it. You may start by just manually entering data, then selectively automate the determination of individual field values, and optionally use discovery to expand the dataset. Both the data and its value determination and discovery rules may start simple and evolve into something fairly complex over time.

---

## Use Cases

1. **Manual Data Curation:** freely define table-like data sets.
2. **Automated Value Determination:** define value determination rules for specific data fields and have Giraffe determine the values for you. This spares you from repetitive, time-consuming data lookups. Determination rules may range from static functions to API calls or web fetches as well as free-form LLM instructions. Value determinations can be rerun at any time to keep the values up to date.
3. **Data Discovery:** define discovery rules for new data items and have Giraffe discover new items for you. This may be used to either get an initial overview of data in a specific area (for example, "find all coworking spaces in City X" or "find all books about Topic X"), or to extend or validate an existing data set.
4. **Rule Reuse and Adaptation:** reuse value determination and discovery rules of existing data sets for new data sets to accelerate the acquisition of related data (for example, reuse publication year or page count determination rules of an existing book data set for a new book data set, or reuse discovery rules for coworking spaces in City X to discover coworking spaces in City Y).

---

## Features

### 1. Data Recording and Editing

- Data recording and editing in a spreadsheet-style manner
- A data set is a collection of records (lines) consisting of one or more fields (columns)
- Each field has a type (integer, string, date, etc.)
- Data structure can be edited (change field types, merge or split fields, etc.)
- Currently out of scope: no joins or other operations between data sets are planned - each data set stands on its own and is isolated from other data sets

**Open Questions:**

- Does each record require a specific field which acts as a key or unique identifier?

### 2. Automated Value Determination

- Determination of field values by the application based on user-defined value determination rules
- Value determination rules may be:
  1. Static rules: for example, static functions of other values in the data
  2. Deterministic operations: for example, API calls or web fetches
  3. LLM instructions: free-form instructions for how to obtain the value
- Value determination operations can be re-run to refresh values
- Data restructuring (e.g. changing the type of a field) may invalidate existing value determination rules, which is indicated to the user and handled appropriately by the application (see open questions)

**Open Questions:**

- Should the value determination rules be in the form of a chain of sub-rules (for example, first make a deterministic API call, then use LLM instructions to analyse the response of the call and extract the value)?
- Should the user be able to overwrite values that have been determined automatically? If yes, then how would reruns of value determination rules handle manually edited values, i.e. would they overwrite a value that has been explicitly overwritten by the user again, or would they leave it out?
- How to handle rule invalidation by data restructurings?

### 3. Data Discovery

- Discovery of new records by the application based on user-defined discovery rules
- Discovery rules are either pure LLM instructions or may also contain deterministic operations (see open questions)
- Discovery only detects new records represented by a specific field (see open questions) and doesn't create any other data (remaining fields are filled in either with automated value determination or manually)
- Discovery can be run before any data exists (e.g. for bootstrapping a data set) or when there already exists any number of records in the data set (e.g. for completing or validating the completion of a data set)

**Open Questions:**

- Should discovery be fully LLM-based or also allow for deterministic operations (e.g. an API call followed by a filter operation)? If also allowing for deterministic operations, then how to implement this (syntax, execution, etc.)?
- Do discovery rules need to be tied to a specific field and their result is only the population of this field? Does it need to be the key field (if the data set has a key field)?
- Are discovery operations manual one-off operations or are they somehow automatically run (e.g. to keep the data set comprehensive as, for example, new entities emerge)?
- Does a successful discovery run leave a data set in an incomplete state, in particular, only the field that the discovery returns is filled in and the other fields are empty? In other words, is this incomplete or inconsistent state by design?
- Can there be multiple discovery rules, possibly tied to different fields and be run independently of each other? Or can there be only a single discovery rule at a time, and to apply a different discovery rule, the existing one needs to be edited?

### 4. Rule Reuse and Adaptation

- All rules (value determination rules and discovery rules) are saved implicitly (see open questions)
- When editing new rules, the user can browse existing rules and decide to reuse them
- Existing rules can be either applied as-is or copied and customised (see open questions)

**Open Questions:**

- Should rules be saved and made available for reuse implicitly? Or does the user need to explicitly save rules to make them reusable?
- Should rules be handled "by reference" or "by value", i.e. are they independent objects that can be edited outside any of their usages ("by reference") or is each usage an independent copy ("by value")? In other words, if a rule is edited, should the edits apply to all data sets that use this rule (e.g. a general way to obtain the page count of *any* book and this can be edited to e.g. use a different source and then it automatically applies to all data sets about books) or only to the data set in whose context it is edited?

### 5. Interface

- Data sets are exposed through an API (see open questions)
- Allows Giraffe to be used as a backend for client applications

**Open Questions:**

- In what form to expose the data sets (API type, data format)?
- Is the interface read-only or does it also allow writing? For example, could a client application also create or edit data instead of just reading it? In that case, what about rules, could they also be created or edited through the interface? And what about writing values to fields that are governed by a value determination rule?

### 6. Version Control

- All versions of the data are saved and can be recovered

**Open Questions:**

- What format to use for version control? I have a strong preference for Git (but not sure if it's efficient or possible)
- Do versions need to be "committed" explicitly or is it implicitly (in the style of auto-save)?
- Does version control only apply to data or also to the rules (value determination rules and discovery rules)?

---

## Positioning and Related Solutions

### Related Solution Categories

Some categories of tools that address similar goals as Giraffe include:

- Text files: no type or consistency enforcement, no automation
- Databases: type and consistency enforcements, but no automation
- Spreadsheets: limited type and consistency enforcements, limited value determination rules, but no automation
- ETL tools: automation for moving and transforming data, but assume structured data input sources

None of these seamlessly combine manual curation with automated data enrichment and discovery like Giraffe.

### Related Concrete Solutions

Some concrete tools focussing on LLM-assisted automation of data creation include:

- [rtrvr.ai](https://www.rtrvr.ai/)
- [Firecrawl Agent](https://www.firecrawl.dev/app/agent)
- [Airtable AI Agents](https://www.airtable.com/platform/ai-agents)

By experience, these tools focus on the discovery of new data and are fully LLM-driven. They have no or very little support for deterministic steps and they don't allow fine-grained control for how to obtain the data for specific fields. The ability for this latter point is the key differentiator for Giraffe.
