# Giraffe — Concept Doc

TODO: adapt intro paragraph

Giraffe aims to be the go-to solution whenever you find yourself collecting table-like data and want to record it. You may start by just manually entering data, then selectively automate the determination of individual field values, and optionally use discovery to expand the dataset. Both the data and its value determination and discovery rules may start simple and evolve into something fairly complex over time.

## The Problem

TODO: describe the problem, no application that combines LLM capabilities, conventional automation, and manual edits in an optimal way, either everything manual and complex automation, or fully LLM-based with no manual control.

---

## The Solution

TODO: how Giraffe addresses this

---

## Features

TODO: the content of this sectio is outdated with respect to the updated sections below.

1. **Manual Data Curation:** freely define table-like data sets.
2. **Automated Value Determination:** define value determination rules for specific data fields and have Giraffe determine the values for you. This spares you from repetitive, time-consuming data lookups. Determination rules may range from static functions to API calls or web fetches as well as free-form LLM instructions. Value determinations can be rerun at any time to keep the values up to date.
3. **Data Discovery:** define discovery rules for new data items and have Giraffe discover new items for you. This may be used to either get an initial overview of data in a specific area (for example, "find all coworking spaces in City X" or "find all books about Topic X"), or to extend or validate an existing data set.
4. **Rule Reuse and Adaptation:** reuse value determination and discovery rules of existing data sets for new data sets to accelerate the acquisition of related data (for example, reuse publication year or page count determination rules of an existing book data set for a new book data set, or reuse discovery rules for coworking spaces in City X to discover coworking spaces in City Y).

New outline

- Creating and editing datasets manually
- Defining automation configs for fields and having values computed automatically
- Setting manual values that override computed ones (the overlay model)
- Persisting datasets to a Giraffe repository and restoring from one (mention sharing with other users)
- Discovering new records (if implemented)
- (Reusing automation configs across datasets -> maybe don't stress this too much)
- Serving data through the public API

---

## Core Concepts

A dataset can have any number of fields and records. A field has a data type and an optional automation configuration (see [Automation Layer](#automation-layer)). For fields with automation configuration, the values are determined automatically. The values of all cells (with or without automation configuration) can be set manually (see [Manual Layer](#manual-layer)). In the case of cells that belong to fields with automation configuration, the manually set value shadows the automatically determined value. When a manually set value is deleted from such a cell, the previous automatically determined value takes effect again.

### Field Data Types

TODO: to be determined, but probably just the usual database data types

### Schema Modifications

TODO: how to handle dataset schema modifications like changing the type of a field, merging or splitting fields, etc.? Support this at all? It probably complicates a lot especially regarding the automation layer.

### Primary Key

TODO: does each dataset have a specific field as the primary key?

### Value Metadata

Every value carries a **value_metadata** object, currently containing:

- `last_updated`: the timestamp of when this value was last changed

### Automation Layer

TODO: terminology: use computation layer and computed value instead?

- Each field may have an automation config
- The automation config defines the "recipe" for how to compute the value in this field for each record
- The computation may be based on other values in the same record and may include API calls, web fetches, etc., and also LLM operations
  - TODO: should computations be assembled like chains or workflows or pipelines, i.e. actions that can be stitched together, e.g. like "first make this API request, then to do this data processing, etc."?
- The automation config may optionally include periodic re-execution in which case the computation is re-run at a freely definable interval. For fields without periodic re-execution in the automation config, the computation is run only once.
- Fields without an automation config stay empty in the automation layer, but they can assume values from the manual layer

- TODO: how to track the dependencies of automation config if they depend on values of other fields in the same record? For example, if an automation config depends on the value of a manual field, then the computation can only run as soon as this value has been entered by the user? And must rerun whenever this value changes (e.g. by an update by the user)? And what if the computation depends on another value that is computed itself? Would there be some order and chain reaction then in what order computations can run (if the dependencies span multiple levels)? And is this a similar pattern as in reactive programming, and could this problem be addressed with reactive programming?
  - TODO: also need to make sure to prevent dependency deadlocks, probably
  - TODO: another consequence of this is that the computations of each value are probably independent of each other even from those in the same record. Because on some record, all dependencies for a computation might be met, on another not yet. So that means that not all similar computations (e.g. all fetches of the Google Maps coordinates) are executed together, but for each record separately at possibly different times.
- TODO: is a primary key field (if we have this) the only field that cannot have an automation config? But maybe actually not, imagine we use the Google Maps ID as the primary key and the user starts a new record by inserting the name (of e.g. a coworking space), then the automation config of the Google Maps ID field (the primary key) could determine the Google Maps ID of this place by querying Google Maps (with a specific high probability).

#### Automation Configs

TODO: detail how the automation configs of columns will look like on a conceptual level (i.e. chain or pipeline of tasks, etc.). I think this would subsume what we previously called "rules" and also what we called "rule reuse" (which is probably rather a UI-level concern). The content of the former  "4. Rule Reuse and Adaptation" section is verbatim below.

```
- All rules (value determination rules and discovery rules) are saved implicitly (see open questions)
- When editing new rules, the user can browse existing rules and decide to reuse them
- Existing rules can be either applied as-is or copied and customised (see open questions)

**Open Questions:**

- Should rules be saved and made available for reuse implicitly? Or does the user need to explicitly save rules to make them reusable?
- Should rules be handled "by reference" or "by value", i.e. are they independent objects that can be edited outside any of their usages ("by reference") or is each usage an independent copy ("by value")? In other words, if a rule is edited, should the edits apply to all data sets that use this rule (e.g. a general way to obtain the page count of *any* book and this can be edited to e.g. use a different source and then it automatically applies to all data sets about books) or only to the data set in whose context it is edited?
```

### Manual Layer

- The manual layer is an overlay of user-defined values over a dataset
- The user may define a value for every cell in a dataset
- The values in the manual layer shadow the values in the automation layer for query operations
  - That means, if a user defines a manual value for a cell that also has a computed value, then any query for this cell's value returns the manual value instead of the computed value
- The shadowing of the manual layer over the automation layer for query operations also applies to the computation of values within the automation layer 
  - That means, if the user overwrites a previously computed value with a manual value in Cell A, and Cell A is a dependency for the cmoputation of the value of Cell B, then the value of Cell B is recomputed taking the new manual value of Cell A into account
- TODO: whether to keep computing computed values that are shadowed by manual values in the background: as defined above, these comptued values would be completely hidden as even computations for other values take the overlaying manual value into account, not the computed value, so there is a question whether this continuation of computation makes sense (in particular for periodically re-computed values). so an alternative could be to just save a snapshot of the old computed value (including its metadata) when the override happens, and then restore this snapshot when the manual value is deleted. but then there is the question if this not actually introduces additional complexity and special handling (saving snapshots, etc) versus just keeping the normal course of action as if the shadowing didn't happen (as mentioned, only the query operation really needs to differ)
- If a a manual value that shadows a computed value is deleted, then the previous computed value becomes thew new outward-facing value of the cell again

TODO: given the fact that computatinos of values themselves (which are part of the automation layer) also need to take into account the manual value from the manual layer, check if the post-it analogy still holds.

### Record Discovery

TODO: adapt from legacy "3. Data Discovery" (verbatim copy below). The question here is also if we want to implement this feature at all. In particular, can we be sure that this provides any useful value at all in general situations and doesn't just degenerate to a prompt to an agent like "Find me all coworking spaces in Bali" which may produce far from satsifactory results (e.g. hallucinations, incomplete and outdated data, etc.)? In other words, is it realistic that we can shoehorn the behaviour of LLMs into sufficiently narrow guidelines to reliably produce useful results? I explored this a bit in the /Users/dw/repos/weibeld/bali-coworking-spaces repository (we can analyse what's there when we make a final decision here), and i came up with something like a confidence index that the agent determines evaluating whether something is really a coworking space or just some other listing on google maps. The results were nevertheless not really promising, as far as i can remember, and it also looked like it may become pretty complex quickly (both for the implementation and for the users who will finally use it). One of the problems are the hallucinations (in some of these early discovery runs, some website URLs were completely hallucinated). But i think this could be bypassed by the discovery only producing the primary key values of new items and not any other data (i think that's the only viable approach which also answers some of the open question of the former content below). We should evaluate which way to go here. Either come up with a large amount of complexity and hope that we can "tame" LLMs, or just bypass it and do not much more than an agent chat interface (like "Find me all of X..."), or then just let it be altogether and refer the user to off-the-loop exploration. In other words, take the effort and try something or just focus on the other aspects of the app.

```
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
```

---

## System Architecture

### Storage and Persistence Model

- Each dataset is backed by its own data store (operational storage) and optionally its own Giraffe repo (persistent storage)
- A data store is a database (e.g. SQLite) local to the backend and managed by the backend
- A Giraffe repo is a Git repository at a remote location (e.g. on GitHub) that the backend has read and write access to
- The data store is the source of truth of a dataset (including the data itself, metadata, configuration, etc.)
- UI clients read and write data exclusively from and to the data store through the backend
- The Giraffe repo is the persistent representation of a dataset, enabling both version control and sharing of the dataset (see below)
- Every dataset must have a data store but the Giraffe repo is optional
- The lifecycles of datasets and data stores are coupled to each other
- The lifecycles of datasets and Giraffe repos are independent of each other (for example, a dataset can be created without a Giraffe repo, a Giraffe repo can be assigned to an existing dataset, a Giraffe repo can be deleted without deleting the dataset, a dataset can be deleted without deleting the Giraffe repo, etc.)
- The content of a dataset's Giraffe repo is a serialisation of the complete content of the dataset's data store (including the data itself, metadata, configuration, etc.)
- The Giraffe repo serialisation format balances human readability and compactness
- The backend contains functionality to validate the format of a Giraffe repo
- The backend can fully and losslessly construct or reconstruct a data store from any version (corresponding to a Git commit) of a Giraffe repo
- The backend can persist the serialisation of the current content of a data store to its Giraffe repo (as a Git commit)
- The decision which state of the data store to persist to the Giraffe repo and which version of the Giraffe repo to restore the data store from is done by the user, enabling full version control at the dataset level
- Persistence to the Giraffe repo can also be configured to be periodical at a regular schedule by the user
- The backend can import a Giraffe repo for which no dataset exists yet, in which case a corresponding dataset is created and a corresponding data store is constructed from the Giraffe repo
- Giraffe repos can be shared between users (e.g. through Git forks), enabling flexible sharing of datasets
- A Giraffe repo is not intended to be written to by users (except in extraordinary cases such as troubleshooting) nor by any other process except the backend

```
TODO: open questions:
1. Do Giraffe repos support branching and merging (corresponding to Git branching and merging) and is this branching and merging propagated to the user control level? That is, can the user create branches of a dataset and use them in a similar way as Git branches? Or is the history of a dataset restricted to be linear?
```

Discarded options:

- Giraffe repo as the source of of a dataset: difficult or impossible to implement cleanly as every change would need to be recorded in the Giraffe repo first, which is a relatively slow process
- Single data store and Git repo for the entire application (i.e. containing all datasets): this would forego a number of advantages that having a dedicated data store and Giraffe repo per dataset has:
  1. Isolated version history: change history of a dataset visible at a glance in the corresponding Giraffe repo's commit history
  2. Granular version control: any specific version of a dataset can be restored from its Giraffe repo without affecting the state of any other datasets
  3. Shareability: Giraffe repos can be shared between users which essentially enables the sharing and independent evolution of datasets across users and Giraffe installations
  4. Modularity: datasets can be modularly added and deleted from a Giraffe installation without having side-effects on any other datasets
  5. Optional persistence: datasets can be created without a Giraffe repo (e.g. for temporary experimentation) without leaving any traces anywhere in the persistence layer

### Deployment Model

- The deployment consists of a central backend and three UI clients
- The backend is the sole owner of business logic, operational storage, and persistent state.
- The three UI clients are 1) the API (for reading datasets), 2) the web app (for reading and writing datasets), and 3) the CLI (for reading and writing datasets)
- The backend provides an operational interface for the UI clients
- The UI clients interact only with the backend and only through the backend interface
- The backend, API, and web app are deployable as a single deployment bundle for ease of deployment
- The CLI is installed on the user's machine separately

```
TODO: open questions:
1. On what platform is the deployment bundle (backend, API, and web app) deployed (compute, serverless, etc.)?
2. Is the deployment and architecture single-user or multi-user? That is, do users deploy and use their own Giraffe installation, or is there one centrally managed installation that users sign up to? Implementing the application single-user first but already abstracting user as an entity (especially in the data model, such as a user as the owner of each dataset, and as the owner for integration configuration such as LLM keys, which would allow bring-your-own LLM) that can later be generalised to multiple users (e.g. add authentication, session management, user registration, etc. later)?
```

Discarded options:

- Distributed business logic: no central backend that contains all business logic, but implementing business logic in each UI client. This would result in massive duplication and code and logic and effort to keep functionality and features in sync.
- Separate deployments for backend, API, and web app: this would increase deployment complexity for no apparent benefit
- Cloud provider specific deployment: making heavy use of cloud services (compute, serverless, API gateways, etc.). This increases deployment complexity due to lock-in to a specific cloud provider and possible an Infrastructure as Code (IaC) tool such as Terraform.

### User Interaction Model

- The user interacts with Giraffe through three separate UI clients: an API, a web app, and a CLI
- The API is read-only and allows reading existing Giraffe datasets in an appropriate format (e.g. JSON or YAML)
- The API is intended to be mainly used by independent applications that use the Giraffe datasets as a data source
- The web app and CLI both have similar functionality and scope
- The web app and CLI are intended to be interactively used by a human user
- The web app and CLI allow both reading and editing data, metadata and configuration, as well as executing persistence operations (e.g. persisting the current state of a dataset to the Giraffe repo, or restoring a dataset from a Giraffe repo)
- The web app and CLI are both designed for real-time updates on the source of truth of the datasets from the backend (e.g. by using WebSockets of Server-Sent Events)
- The CLI is implemented as a TUI (e.g. with Go Bubbletea or Rust Ratatui)
- All UI clients interact with (and only with) the backend through the backend interface

```
TODO: open questions:
1. Using reactive programming techniques for the web app and CLI (reacting on dependency changes of computed values)?
2. If there is a TUI (strong preference for it), is the web app still necessary (both providing the same functionality)?
```

---

## Related Solutions

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

---

## Appendix: Terminology

TODO: check if still necessary, move to appendix, etc.

Giraffe uses the following terminology:

- For its data model: "dataset", "record", "field", and "value" corresponding to "table", "row", "column", and "value" in a conventional table-based data model.
- For its storage and persistence model: "data store" for the operational storage, and "Giraffe repo" for the persistent Git repository layer
- For its deployment model: "backend" for the central component containing most business logic, and "UI client" for the peripheral user-facing applications. The three UI clients are called "API", "web app", and "CLI" are used.
