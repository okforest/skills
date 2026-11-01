---
name: okforest
description: Write, review, and change knowledge in OKForest knowledge trees, which are Open Knowledge Format (OKF) bundles of markdown concepts with YAML front matter, plus OKForest's settings, type schemas, and diagrams. Use when writing or reviewing OKF concepts, index.md or log.md files, or .okforest/ settings and type schemas, and when asking for or proposing changes to a tree through OKForest's MCP tools (request_change, propose_change, list_changes, get_change).
license: Apache-2.0
---

# OKForest

OKForest serves an account's **Forest**: its **knowledge trees**, each an
[Open Knowledge Format](https://github.com/GoogleCloudPlatform/open-knowledge-format)
(OKF) bundle in a Git repository. A tree is a folder of markdown files; each
file other than `index.md` and `log.md` is a **concept**: YAML front matter,
then a markdown body. OKForest adds a few conventions on top of OKF (a
settings file, type schemas, diagrams), all optional, all still valid OKF.

There are two ways to change a tree:

- **Through OKForest** (its MCP server's tools, or its app): a change waits
  for one of the tree's editors, who reviews and applies it as one commit.
- **In the repository**, editing files directly, checked with
  `npx okforest validate .`.

## Writing a concept

```markdown
---
type: Customer
title: Acme Corp
description: Our largest retail customer.
tags: [customer, retail]
---

# Acme Corp

Acme Corp buys our [retail analytics plan](/plans/retail-analytics.md).
```

- **`type` is required**, a short descriptive string (`Customer`, `Metric`,
  `BigQuery Table`). Reuse the types the tree already has: OKForest's
  `list_types` tool lists them, with the fields each type's schema expects.
- **Give a `title` and a one-sentence `description`.** Index entries, search,
  and previews use them.
- **Name files as slugs:** lowercase letters, digits, and hyphens
  (`customers/acme-corp.md`). The path without `.md` is the concept's ID.
- **Keep what you don't know out.** Don't invent `verified`, `generated`, or
  `sources` entries; OKForest stamps who wrote and who verified a page when
  an editor applies a change.
- **Structure the body** with headings, lists, and tables. `# Schema`,
  `# Examples`, and `# Computation` have conventional meanings.
- **Diagrams are code blocks**, ```` ```d2 ```` or ```` ```mermaid ````, not
  images, so they can be diffed and edited.
- **Timestamps** are ISO 8601 with an offset: `2026-06-30T14:00:00Z`.

## Linking

- Use standard markdown links, never `[[wikilinks]]`.
- Prefer **absolute** links, from the tree's root: `[orders](/tables/orders.md)`.
  They survive moving the linking page. Relative links (`../tables/orders.md`)
  work too.
- A link's surrounding words say what the relationship is.
- Put a page's own files (images, PDFs, spreadsheets) beside it, or in a
  folder below it, and link them. Material many pages share goes under
  `references/`.

## `index.md` and `log.md`

- `index.md` lists **its own folder's** contents, one entry per line, under
  headings: `* [Title](relative-path.md) - description`. No front matter,
  except `okf_version: "0.2"` in the tree's root `index.md`.
- `log.md` records changes, newest first, under `## YYYY-MM-DD` headings:
  `* **Update**: what changed, with [links](/path.md).`

When OKForest makes a change, it updates these for you: a new page joins its
folder's index; a moved page's entries and the links to it follow it.

## Changing a tree through OKForest

With OKForest's MCP server connected:

1. **Find and read first.** `search_concepts`, then `get_concept` for each
   page you'll touch: edits must match its text exactly.
2. **Check `list_changes`** for an open change that already asks for the same
   thing.
3. **Propose or request:**
   - `propose_change` when you can write the edits: pages created, edited,
     moved, deprecated, or deleted.
   - `request_change` when the person wants to ask in their own words, or the
     edits need an editor's judgment.
4. **Keep each change small**, with one purpose, and a title that says what
   it does ("Add Globex as a customer"). Each page can be in one edit of a
   change.
5. **Fix what comes back.** If an edit doesn't apply, or a page would have
   errors, nothing is saved, and the message says what to fix: correct it and
   call again.
6. **Give the person the change's `url`.** Nothing reaches the tree until an
   editor applies it there; never say a change is made before then.

`get_change` shows a change's edits, checks, and comments, and where it is:
requested, drafting, in review, or done.

For each tool's input, with examples, see [references/changes.md](references/changes.md).

## Changing a tree in its repository

- Run `npx okforest validate .` from the tree's root before committing, and
  fix every error (warnings don't fail it). `npx okforest rule <rule>` explains
  a finding's rule and its fix.
- Add new pages to their folder's `index.md`, and fix the links to a page you
  move.
- `npx okforest init <folder> --template company` starts a new tree;
  `npx okforest types init .` drafts a schema for each type that has none.

## Reference

- [references/okf.md](references/okf.md): the OKF format in detail: every
  front-matter field, provenance and trust, links, index and log files.
- [references/okforest.md](references/okforest.md): what OKForest adds:
  `.okforest/settings.yaml`, type schemas, diagrams, and the checks it runs.
- [references/changes.md](references/changes.md): OKForest's MCP tools for
  changes, with examples.
