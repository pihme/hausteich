# Hausteich

Working title. A private store for one household's own data, the small counterpart of a data lake: the pond by the house.

Status: just started. Nothing is built yet.

## What it is

Hausteich is passive and has no user interface. It takes in what it is handed, indexes it, and answers search queries. It does not read mailboxes, document archives or folders itself.

- **Sources** are where the data lives: a mailbox, a document archive, a folder on a computer.
- **Wells** are small helpers that draw from one source and fill the pond. They live in this repository, one directory per well. (A German name for them is still open.)
- **Applications** ask the pond. Search happens in the application, not in Hausteich.

## First goal

Find an old mail by describing it: one well for IMAP, the pond with intake and query, and an existing MCP client as the first application.

## Principles

- Embeddings are computed locally.
- A language model is optional and sits on top of the search. It is reached through an OpenAI-compatible endpoint, so it can be local or a chosen provider, later possibly a different one per privacy level.
- An AI that reads the pond only reads. It has no tool that sends anything.
