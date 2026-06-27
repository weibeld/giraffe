# Giraffe — Concept Doc

## Core Idea

Giraffe aims to be the go-to solution whenever you find yourself collecting table-like data and want to record it. You may start by just manually entering data, then selectively automate the determination of individual field values, and optionally use discovery to expand the dataset. Both the data and its value determination and discovery rules may start simple and evolve into something fairly complex over time.

---

## Use Cases

TODO: the content of this sectio is outdated with respect to the updated sections below.

1. **Manual Data Curation:** freely define table-like data sets.
2. **Automated Value Determination:** define value determination rules for specific data fields and have Giraffe determine the values for you. This spares you from repetitive, time-consuming data lookups. Determination rules may range from static functions to API calls or web fetches as well as free-form LLM instructions. Value determinations can be rerun at any time to keep the values up to date.
3. **Data Discovery:** define discovery rules for new data items and have Giraffe discover new items for you. This may be used to either get an initial overview of data in a specific area (for example, "find all coworking spaces in City X" or "find all books about Topic X"), or to extend or validate an existing data set.
4. **Rule Reuse and Adaptation:** reuse value determination and discovery rules of existing data sets for new data sets to accelerate the acquisition of related data (for example, reuse publication year or page count determination rules of an existing book data set for a new book data set, or reuse discovery rules for coworking spaces in City X to discover coworking spaces in City Y).

---

## Core Concepts

### Data Model

Giraffe uses standard table terminology: **table**, **row**, **column**, **cell**, and **value** (the content of a cell).

A table can have any number of columns and rows. A column has a data type and an optional automation configuration (see [Automation Layer](#automation-layer)). For columns with automation configuration, the values are determined automatically. The values of all cells (with or without automation configuration) can be set manually (see [Manual Layer](#manual-layer)). In the case of cells that belong to columns with automation configuration, the manually set value shadows the automatically determined value. When a manually set value is deleted from such a cell, the previous automatically determined value takes effect again.

#### Column Data Types

TODO: to be determined, but probably just the usual database data types

#### Schema Modifications

TODO: how to handle table schema modifications like changing the type of a column, merging or splitting fields, etc.? Support this at all? It probably complicates a lot especially regarding the automation layer.

#### Primary Key

TODO: does each table a specifc column as the primary key?

#### Value Metadata

Every value carries a **value_metadata** object, currently containing:

- `last_updated`: the timestamp of when this value was last changed

### Automation Layer

TODO: terminology: use computation layer and computed value instead?

- Each column may have an automation config
- The automation config defines the "recipe" for how to compute the value in this column for each row
- The computation may be based on other values in the same row and may include API calls, web fetches, etc., and also LLM operations
  - TODO: should computations be assembled like chains or workflows or pipelines, i.e. actions that can be stitched together, e.g. like "first make this API request, then to do this data processing, etc."?
- The automation config may optionally include periodic re-execution in which case the computation is re-run at a freely definable interval. For columns without periodic re-execution in the automation config, the computation is run only once. 
- Columns without an automation config stay empty in the automation layer, but they can assume values from the manual layer

- TODO: how to track the dependencies of automation config if they depend on values of other columns in the same row? For example, if an automation config depends on the value of a manual column, then the computation can only run as soon as this value has been entered by the user? And must rerun whenever this value changes (e.g. by an update by the user)? And what if the computation depends on another value that is computed itself? Would there be some order and chain reaction then in what order computations can run (if the dependencies span multiple levels)? And is this a similar pattern as in reactive programming, and could this problem be addressed with reactive programming?
  - TODO: also need to make sure to prevent dependency deadlocks, probably
  - TODO: another consequence of this is that the computations of each cell are probably independent of each other even from those in the same row. Because on some row, all depenencies for a computation might be met, on another not yet. So that means that not all similar computations (e.g. all fetches of the Google Maps coordinates) are executed together, but for each row separately at possibly different times.
- TODO: is a primary key column (if we have this) the only column that cannot have an automation config? But maybe actually not, imagine we use the Google Maps ID as the primary key and the user starts a new row by inserting the name (of e.g. a coworking space), then the automation config of the Google Maps ID field (the primary key) could determine the Google Maps ID of this place by querying Google Maps (with a specific high probability).

### Manual Layer

- The manual layer is an overlay of user-definend values over a table
- The user may define a value for every cell in a table
- The values in the manual layer shadow the values in the automation layer for query operations
  - That means, if a user defines a manual value for a cell that also has a computed value, then any query for this cell's value returns the manual value instead of the computed value
- The shadowing of the manual layer over the automation layer for query operations also applies to the computation of values within the automation layer 
  - That means, if the user overwrites a previously computed value with a manual value in Cell A, and Cell A is a dependency for the cmoputation of the value of Cell B, then the value of Cell B is recomputed taking the new manual value of Cell A into account
- TODO: whether to keep computing computed values that are shadowed by manual values in the background: as defined above, these comptued values would be completely hidden as even computations for other values take the overlaying manual value into account, not the computed value, so there is a question whether this continuation of computation makes sense (in particular for periodically re-computed values). so an alternative could be to just save a snapshot of the old computed value (including its metadata) when the override happens, and then restore this snapshot when the manual value is deleted. but then there is the question if this not actually introduces additional complexity and special handling (saving snapshots, etc) versus just keeping the normal course of action as if the shadowing didn't happen (as mentioned, only the query operation really needs to differ)
- If a a manual value that shadows a computed value is deleted, then the previous computed value becomes thew new outward-facing value of the cell again

TODO: given the fact that computatinos of values themselves (which are part of the automation layer) also need to take into account the manual value from the manual layer, check if the post-it analogy still holds.

### Item Discovery

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

### TBD: Storage Backend and Version Control

TODO: this is still a big question mark. As mentioned, it would be ideal to have all state in a Git repository (like e.g. [Owl](https://github.com/weibeld/owl)), however, i don't know if this is efficient and even possible. Then probably every edit of a value would need to results in a commit. And especially for computed values with periodic re-executions, this might result in large numbers of commits, especially considering that there must be an individual operation for every cell, not for a column as a whole (see Automation Layer above). And i don't know if reading from text files and writing to text files as a source of truth is efficient for an application that is supposed to show some real-time behaviour (e.g. user enters a manual value which enables the computation of a computed value, and this should ideally happen in real-time so that the causality of the operations becomes clear to the user). Alternatives would maybe be something like SQLite but then were would the data live? The ideal situation is that all data and all crucial state is in a Git repo and everything else is just runtime and stateless, i.e. so that whatever runtime deployment there is (see "TBD: Deployment Model") this deployment could just vanish and nothing would be lost (you can reinstall the CLI tool or whatever and fully reinitialise the app). In other words, avoid storing data on some storage infrastructure that we have to maintain. That would be the end goal, and we should evaluate if there is a way to achieve this. Btw, this would also solve the version control concern basically for free. The verbatim text of the former "6. Version Control" section is below.

```
- All versions of the data are saved and can be recovered

**Open Questions:**

- What format to use for version control? I have a strong preference for Git (but not sure if it's efficient or possible)
- Do versions need to be "committed" explicitly or is it implicitly (in the style of auto-save)?
- Does version control only apply to data or also to the rules (value determination rules and discovery rules)?
```

### TBD: User Interface

TODO: there are probably 2 or 3 main inerfaces of the app. First, a read-only API, which is already a given as we certainly need an API (ideally REST returning JSON). Next, the question is whether the interactive user interface should be a GUI (e.g. web-based) or command-line based (e.g. a TUI) or both. In an ideal world there are probably both with identical functionality. However, we shoudl probably just do one first and thus first evaluate which one is more straightforward to build first. This feeds then in also to the deployment model (and is related to the storage backend): where does the API run (and what does it use as its data backend)? For a CLI, what does the CLI connect to for functionality (i.e. client-server or everything executed locally) and data? And for a web-based GUI what tech is used (e.g. client-side, client-server, etc) and what is the data backend? The verbatim text of the former "5. Interface" section is below.

```
- Data sets are exposed through an API (see open questions)
- Allows Giraffe to be used as a backend for client applications

**Open Questions:**

- In what form to expose the data sets (API type, data format)?
- Is the interface read-only or does it also allow writing? For example, could a client application also create or edit data instead of just reading it? In that case, what about rules, could they also be created or edited through the interface? And what about writing values to fields that are governed by a value determination rule?
```

### TBD: Deployment Model

#### Problem 

TODO: this is probably one of the biggest and most central question marks and probably depends on the storage backend approach we choose (see "TBD: Storage Backend and Version Control") and also influences the user interface concerns (see "TBD: User Interface"). In other words, where does stuff run? Is there a central deployed runtime that implements all functionality (maybe including the API) and that a potential web-based GUI interface and a command-line interface connect to as clients? Or is the runtime logic implemented in each user interface application (i.e. in the GUI web app and in the CLI and the read aspects in the API)? Also this does not yet address the storage backend decision, i.e. where the data lives (see "TBD: Storage Backend and Version Control") but assuming that the storage backend could indeed be a Git repo, at least this aspect could be factored out from the deployment model.

#### Vision (draft)

- Storage backend with source-of-truth data on GitHub (to be verified)
- Central backend containing all business logic and exposing an internal API
- User API, web app, and CLI all talk to the central backend (the user API and the internal API are separate layers)
- Central backend, user API, and web app bundled as a single deployment unit (still logically separated)
- CLI locally installed on demand
- TODO: what deployment platform and how does the deployment bundle look like?

Discarded options:

- No backend, but business logic in individual client applications (user API, web app, CLI): code duplication, feature drift, etc.
- Serverless solutios for backend, user API, and web app (e.g. AWS API Gateway, Kong, Nginx, AWS Lambda, Cloudflare Workers, Vercel Functions, etc.): tight coupling to provider, not necessarily easier deployment and maintenance, might require Terraform
- Web app not co-located with backend but deployed as a regular web app or static web app independently (e.g. on GitHub Pages): additional deployment complexity for no apparent benefit.

---

## Details

### Automation Configs

TODO: detail how the automation configs of columns will look like on a conceptual level (i.e. chain or pipeline of tasks, etc.). I think this would subsume what we previously called "rules" and also what we called "rule reuse" (which is probably rather a UI-level concern). The content of the former  "4. Rule Reuse and Adaptation" section is verbatim below.

```
- All rules (value determination rules and discovery rules) are saved implicitly (see open questions)
- When editing new rules, the user can browse existing rules and decide to reuse them
- Existing rules can be either applied as-is or copied and customised (see open questions)

**Open Questions:**

- Should rules be saved and made available for reuse implicitly? Or does the user need to explicitly save rules to make them reusable?
- Should rules be handled "by reference" or "by value", i.e. are they independent objects that can be edited outside any of their usages ("by reference") or is each usage an independent copy ("by value")? In other words, if a rule is edited, should the edits apply to all data sets that use this rule (e.g. a general way to obtain the page count of *any* book and this can be edited to e.g. use a different source and then it automatically applies to all data sets about books) or only to the data set in whose context it is edited?
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
