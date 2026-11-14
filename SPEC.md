# Hausteich specification

Draft for the first cut. Stack and assumptions: [STACK.md](STACK.md).

## 1. Goal of the first cut

The owner finds an old mail by describing it, faster than in his mail program.

Three parts, and one interface between each pair:

```
source (mailbox) ──IMAP──> Rinnsal ──intake──> Hausteich <──query── application
```

- **Rinnsal for IMAP** reads the mailboxes and delivers every message to the pond.
- **Hausteich** stores what it is handed, cuts it into chunks, embeds and indexes them, and answers queries.
- **Application:** for the first cut an existing MCP client, through a thin MCP adapter that calls the query interface.

## 2. Vocabulary

| Term | Meaning |
| --- | --- |
| Source | One place where data lives, registered in Hausteich under a name: one mailbox, one folder tree, one archive. |
| Item | One thing from a source: a message, a file, a document. Identified by the source and an id the Rinnsal chooses. |
| Chunk | A passage of an item, the unit that is embedded, indexed and returned as a hit. |
| Rinnsal | A helper that draws from one source and fills the pond. |
| Privacy level | A label on a source. Stored and returned; nothing acts on it yet. |

## 3. Intake: Rinnsal to Hausteich

REST over HTTP, JSON bodies, paths under `/v1`.

### 3.1 Sources and tokens

- A source is created on the server by the owner, with a name, a kind (`mail`, `file`, `document`) and a privacy level.
- Each source has one token. A Rinnsal sends it as `Authorization: Bearer …` and can write only to its own source. Tokens live in files on the server.

### 3.2 Deliver an item

`PUT /v1/sources/{source}/items/{id}`

```json
{
  "version": "an opaque string that changes when the item changes",
  "format": "text/plain",
  "text": "the body",
  "title": "the subject, or the file name",
  "author": "who wrote it",
  "recipients": ["…"],
  "date": "2026-03-14T09:26:53Z",
  "container": "the folder or mailbox path",
  "thread": "an id shared by items that belong together",
  "locator": "how an application opens the original, for example a message id",
  "attachments": [{"name": "offer.pdf", "media_type": "application/pdf", "size": 48211}]
}
```

- `id` is stable for the life of the item and chosen by the Rinnsal. For mail it has to survive a move between folders.
- `version`, `format` and `text` are required. Everything else is optional.
- `format` is one Hausteich can cut: `text/plain`, `text/markdown`, `text/html`.
- The call is idempotent. Same `version` as stored: nothing happens, answer `200`. New item: `201`. Changed item: the old chunks are replaced, `200`.
- The call returns when the item is searchable. There is no queue in the first cut.
- Knowledge about the source stays in the Rinnsal: it decodes MIME, picks the body part, and decides what the text is. Hausteich cuts and indexes what it gets.
- Attachments are listed, not stored or indexed, in the first cut.

### 3.3 Remove an item

`DELETE /v1/sources/{source}/items/{id}` removes the item and its chunks. Removing an unknown id answers `204` as well.

### 3.4 What does the pond have?

`GET /v1/sources/{source}/items?after={id}&limit={n}` lists ids and versions in id order, page by page. A Rinnsal compares this with its source to find what is new, changed or gone, so it needs no state of its own.

## 4. Cutting and indexing

- Hausteich cuts an item into chunks by its format.
- The title is context for every chunk of the item: a passage of a mail is found together with its subject.
- Author, recipients, date, container, thread and source are stored per item and usable as filters.
- How this maps onto RAGite, and what RAGite has to change, is in STACK.md.

## 5. Query: application to Hausteich

### 5.1 Search

`POST /v1/search`

```json
{
  "query": "the offer for the heat pump from last spring",
  "filters": {"sources": ["…"], "kinds": ["mail"], "author": "…", "after": "2026-01-01", "before": "2026-07-01", "container": "…"},
  "limit": 10
}
```

Answer: a list of hits, best first. Each hit has the chunk text, a score, and the item it belongs to: source, id, title, author, date, container, thread, locator and privacy level. Several chunks of one item are returned as one hit with its best passages.

- A search needs no language model.
- An application has its own token, for reading only.

### 5.2 Fetch an item

`GET /v1/sources/{source}/items/{id}` returns the stored text and all fields of one item.

### 5.3 MCP adapter

A small separate process offers two tools, `search` and `get_item`, and calls 5.1 and 5.2. It adds nothing of its own.

## 6. Rules

- Hausteich never opens a connection to a source.
- Nothing in Hausteich or the MCP adapter sends data anywhere. An AI that uses the pond can read and has no tool that sends.
- Embeddings are computed on the server.

## 7. Done when

- All mailboxes are in the pond, and a second run of the Rinnsal delivers only what changed.
- A message deleted or moved in the mailbox is gone or moved in the pond after the next run.
- The owner finds five mails he names in advance by describing them, each within the first ten hits.

Measured with the real mailboxes and written down here:

- how long the first fill takes, and how large the SQLite file gets;
- how long a search takes, with and without filters;
- how well search works for German text with the chosen embedding model;
- whether a local language model is good enough to turn a question into queries and sum up hits.

## 8. Open questions

- Transport inside the home network: plain HTTP with tokens, or TLS.
- Whether the Rinnsal for IMAP sends HTML bodies as HTML or as extracted text, and what it does with quoted replies and signatures.
- Whether items are grouped by thread in results.
- Attachments: when their text is indexed, and who extracts it.
- The order of work against the RAGite changes (STACK.md).
