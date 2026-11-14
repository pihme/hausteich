# Hausteich

Working title. A private store for one household's own data, the small counterpart of a data lake: the pond by the house. Part of [Doce Lar](https://github.com/pihme/doce-lar).

Status: just started. Nothing is built yet.

## What it is

Hausteich is passive and has no user interface. It takes in what it is handed, indexes it, and answers search queries. It does not read mailboxes, document archives or folders itself.

- **Sources** are where the data lives: a mailbox, a document archive, a folder on a computer.
- A **Rinnsal** (German for trickle; plural Rinnsale) is a small helper that draws from one source and fills the pond. Rinnsale live in this repository, one directory each.
- **Applications** ask the pond. Search happens in the application, not in Hausteich.

## First goal

Find an old mail or document by describing it. Three Rinnsale, built in this order: one for IMAP mailboxes, one for folders on a computer, one for the paperless-ngx archive. They fill the pond through one intake interface; an existing MCP client is the first application. Details: [SPEC.md](SPEC.md), assumptions: [STACK.md](STACK.md).

## Principles

- Embeddings are computed locally.
- A language model is optional and sits on top of the search. It is reached through an OpenAI-compatible endpoint, so it can be local or a chosen provider, later possibly a different one per privacy level.
- An AI that reads the pond only reads. It has no tool that sends anything.
