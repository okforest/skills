# What OKForest adds to OKF

OKForest reads plain OKF, and adds a few conventions. Each is optional, and a
tree that uses them is still a valid OKF bundle. The full documentation is
at [okforest.ai/docs](https://www.okforest.ai/docs/).

## `.okforest/settings.yaml`

One optional file in the tree's root folder (beside its root `index.md`):

```yaml
# yaml-language-server: $schema=https://www.okforest.ai/schemas/settings.schema.json
name: Sales knowledge
description: Pricing, discounts, and deal desk policies for the sales team.
ignore:
  - README.md
  - drafts/
```

- `name` (up to 100 characters) and `description` (up to 4,000) name the
  tree in OKForest.
- `ignore` lists globs of markdown files that aren't concepts, relative to
  the tree's root: `*` within one folder name, `**` across folders, a
  trailing `/` for a whole folder. Without it, a repository's `README.md`
  is a concept, and fails for having no `type`.
- Spelled `.yaml`, never `.yml`. Unknown keys are ignored, with a warning.

## Type schemas: `.okforest/types/<type>.schema.json`

A JSON Schema for one type's front matter, so its concepts carry the same
fields:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Customer",
  "description": "An organization that buys from us.",
  "type": "object",
  "properties": {
    "relationship": { "enum": ["prospect", "active", "former"] },
    "account_owner": { "type": "string", "format": "email" }
  },
  "required": ["relationship", "account_owner"]
}
```

- The schema's `title` is the type, exactly as concepts write it.
- It describes the type's own fields; OKF's fields (`title`, `tags`,
  `verified`, and the rest) are checked by the OKF rules.
- Every concept of the type must match it: a missing required field or a
  value outside an `enum` is an error.
- Over MCP, `list_types` gives each tree's fields for a type. Read it before
  writing a concept of that type, and fill in what it requires.
- `npx okforest types init .` drafts a schema for each type that has none,
  from what its concepts hold.

## Diagrams as code

Draw diagrams in code blocks, ```` ```d2 ```` or ```` ```mermaid ````, not
as images: they can be reviewed in a diff and edited later. OKForest
renders them, and its checks report a block that doesn't parse.

## Files beside pages

A page's images, PDFs, and other files go beside it, or in a folder below
it, linked from it. OKForest treats the files only that page links to, in
its folder or below, as the page's own: moving the page moves them, and
deleting it can delete them. Material several pages use goes under
`references/`.

## Checks

`npx okforest validate .` (and OKForest itself, for every commit) runs the
OKF rules and OKForest's own. **Errors** must be fixed: OKForest won't apply
a change whose pages have one. **Warnings** and **info** don't stop anything.
`npx okforest rule <rule>` explains a rule and its fix.

Errors:

- `okf/frontmatter`: a concept starts with YAML front matter that parses.
- `okf/type`: front matter has a non-empty string `type`.
- `okf/status`: `status` is `draft`, `stable`, or `deprecated`.
- `okf/datetime`: timestamps are ISO 8601 with an offset.
- `okf/generated`, `okf/verified`, `okf/sources`: those fields are well
  formed (`by` and `at`; each source has a `resource`, unique `id`s).
- `okf/index-frontmatter`: an `index.md` has no front matter, except
  `okf_version` at the root.
- `okf/log-date`: `log.md` dates are `## YYYY-MM-DD`.
- `okf/attested-computation`: an Attested Computation has its contract.
- `okforest/type-schema`: a concept matches its type's schema.
- `okforest/settings`: `.okforest/settings.yaml` is well formed.
- `okforest/d2-syntax`, `okforest/mermaid-syntax`: diagrams parse.

Warnings worth fixing as you write:

- `okf/title`, `okf/description`: a concept has them.
- `okf/broken-link`: a link points at a file the tree has.
- `okf/index-entries`, `okf/index-scope`: an `index.md` lists its own
  folder as `* [Title](path) - description` entries.
- `okf/actor`: actors are `<producer>/<version>`, `human:<id>`, or
  `process:<id>`.
- `okf/footnote-source`: each footnote label matches a source's `id`.
- `okf/log-order`: `log.md` dates are newest first, each once.
- `okforest/slug`: concept file and folder names are lowercase letters,
  digits, and hyphens.
- `okforest/wikilink`: markdown links, not `[[wikilinks]]`.
- `okforest/placeholder`: no leftover template text (`{{name}}`,
  `[INSERT ...]`, `path/to/...`).
