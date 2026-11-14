# Stack

Assumptions about how Hausteich is built. Each line says where it comes from: **decided** by the owner, **assumed** as a starting point, or **open**. Change this file when one of them changes.

## Shape

| Assumption | Status |
| --- | --- |
| Hausteich is headless. It has no user interface. | decided |
| It is passive. It never reaches out to a mailbox, an archive or a folder; a Rinnsal brings the data. | decided |
| It talks REST over HTTP, for intake and for queries (SPEC.md). | decided |
| An MCP server is a thin adapter in front of the query interface, a separate small process in this repository. Hausteich itself stays REST only. | assumed |
| It runs on the owner's home server and is reached inside the home network only. | decided |

## Search core: RAGite

| Assumption | Status |
| --- | --- |
| [RAGite](https://github.com/pihme/ragite) is the basis: SQLite as the store, embeddings on the CPU, hybrid search (vector plus full text), a link from every hit back to its document and position. | decided |
| One SQLite file holds everything. No other database. | assumed |
| Embeddings are computed locally, by the model built into the RAGite build. | decided |
| Chunking moves into Hausteich. RAGite keeps embedding, indexing and search. | decided |

### What RAGite has to change for this

RAGite 1.1.5 cuts documents itself, with four built-in cutters chosen by the document's format, and a caller cannot supply one. Two open RAGite issues already describe the changes Hausteich needs:

- [ragite#117](https://github.com/pihme/ragite/issues/117): a `Chunker` trait in `ragite-core` and a second Rust layer (working name `ragite-custom`) for a Rust program that embeds SQLite and registers its own chunkers. This is the way chunking can move into Hausteich while a document keeps its chunk list, offsets and heading path. The issue leaves four decisions open, among them whether this waits for the 2.0 crate that works without SQLite.
- [ragite#108](https://github.com/pihme/ragite/issues/108): embed and index each chunk together with its heading path. For mail this is what lets every passage of a message be found together with its subject.

Both touch requirements that RAGite's spec marks as closed, so they need the owner's decision there.

### Consequences and risks

| Point | Status |
| --- | --- |
| Hausteich's server is written in Rust. ragite#117 only works for a Rust host that compiles against RAGite. | decided |
| The owner's mail is in several languages. RAGite's built-in model, potion-retrieval-32M, is English. RAGite takes another model at build time (its REQ-23), not at run time, so Hausteich uses its own build. Candidate: potion-multilingual-128M (101 languages, MIT), which RAGite's spec lists but never benchmarked. Tracked in [ragite#154](https://github.com/pihme/ragite/issues/154). | open, important |
| Vector search in RAGite goes through sqlite-vec, which as far as known compares against every stored vector. With 10 GB of mail that may be one to three million chunks. Whether queries stay fast enough is not known. Tracked in [ragite#155](https://github.com/pihme/ragite/issues/155). | open, to be measured (SPEC.md) |
| Hausteich is RAGite's reference user. What it needs is filed there and collected in [ragite#156](https://github.com/pihme/ragite/issues/156); numbers from real use go back onto those issues. | decided |
| Until ragite#117 lands, Hausteich uses RAGite's built-in cutters through the formats it already sends, and moves chunking when the change is there. | assumed |

## Language model

| Assumption | Status |
| --- | --- |
| Search works without a language model. A model is an optional layer in the application, for turning a question into queries and for summing up hits. | assumed |
| A model is reached through an OpenAI-compatible endpoint: local if the hardware allows, otherwise an own key limited to chosen models (zero retention, EU). | decided |
| Every source carries a privacy level. Nothing acts on it yet; later it can select the endpoint. | assumed |

## Rinnsale

| Assumption | Status |
| --- | --- |
| A Rinnsal is its own small process, one directory each in this repository, with its own release artifact where needed. | decided |
| There are three to start with: IMAP mailboxes, folders on a computer, and the paperless-ngx archive. paperless-ngx stays as the archive and is fed from, not replaced. | decided |
| A Rinnsal knows its source and nothing about the index. It speaks only the intake interface, and it holds the credentials for its source. | assumed |
| A Rinnsal turns its source into text. Hausteich never sees a PDF or an office file. | assumed |
| Language per Rinnsal. | open; one toolchain for everything would be simplest |

## Extraction service

| Assumption | Status |
| --- | --- |
| Text from PDFs and office files comes from a service: a Rinnsal sends a document there and gets a Docling document back, which it delivers to the pond in the Docling format RAGite already cuts. | decided |
| The service is taken off the shelf: [docling-serve](https://github.com/docling-project/docling-serve), the Docling project's own API server, with a CPU container image. | assumed, not tried |
| It runs on the home server, next to Hausteich, and is not part of this repository. | assumed |

To check before relying on it: that its JSON output is the serialized Docling document RAGite accepts (RAGite checks `schema_name` and ignores the version), which endpoints and which API key header the current release has, and how long a conversion takes on four CPU cores, with and without text recognition for scans.

## Not set up yet

License, AGENTS.md, team rules and labels from presets, CI, releases.
