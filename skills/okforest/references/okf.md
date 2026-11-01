# The Open Knowledge Format (OKF) v0.2

A summary for writing concepts. The specification itself is
[SPEC.md](https://github.com/GoogleCloudPlatform/open-knowledge-format/blob/main/SPEC.md);
where this summary and the specification differ, the specification wins.

## Files

- A **bundle** (an OKForest tree) is a folder of markdown files, organized in
  folders however suits the knowledge.
- `index.md` and `log.md` are reserved at every level: never concepts.
- Every other `.md` file is a **concept**. Its **ID** is its path without
  `.md`: `customers/acme-corp`.
- Other files (images, PDFs, code) may sit anywhere. The `references/`
  folder conventionally holds external material, run instructions, and code
  that concepts point to.

## Front matter

A YAML block between `---` lines at the very top of the file.

| Field         | Required?   | What it holds                                                    |
| ------------- | ----------- | ---------------------------------------------------------------- |
| `type`        | Required    | A short, descriptive kind: `Customer`, `Metric`, `BigQuery Table` |
| `title`       | Recommended | The display name                                                 |
| `description` | Recommended | One sentence summarizing the concept                             |
| `resource`    | Optional    | A URI for the asset the concept describes, if it describes one   |
| `tags`        | Optional    | A list of short strings                                          |

Any other key is allowed, and tools keep keys they don't know. OKForest's
type schemas can require more fields per type (see
[okforest.md](okforest.md)).

### Provenance: `sources`

```yaml
sources:
  - id: ga4-schema
    resource: https://developers.google.com/analytics/bigquery/export-schema
    title: GA4 BigQuery Export schema
```

- Each entry needs a `resource`: a URL, a path in the bundle, or a
  description of a scope (`all queries in project X`).
- `id` keys per-claim attribution: a footnote `[^ga4-schema]` in the body
  attributes the claim to that source.
- Optional signals per source: `author` (an actor), `usage_count`, and
  `last_modified`; `usage_window: { from, to }` beside `sources` frames the
  counts.

### Trust: `generated` and `verified`

```yaml
generated: { by: reference_agent/gemini-2.5-pro, at: 2026-06-20T22:53:05Z }
verified:
  - { by: human:jane, at: 2026-06-25T09:00:00Z }
```

- `generated` says who or what wrote the current content (`by` is required).
- `verified` lists who or what confirmed it; one `{ by, at }` mapping is also
  allowed.
- The **trust tier** follows from `verified`: none is *unverified*; only
  non-`human:` actors is *machine-confirmed*; any `human:` actor is
  *human-reviewed*.
- Never add `verified` for a check that didn't happen.

### Actors

Who did something, written one of three ways: `<producer>/<version>` for an
agent or tool (`reference_agent/gemini-2.5-pro`), `human:<id>` for a person,
`process:<id>` for an automated process. Trust tiers key off `human:`.

### Lifecycle

- `status`: `draft`, `stable` (the default when absent), or `deprecated`
  (kept for its links and history, no longer current).
- `stale_after`: an instant after which the content is stale.

### Timestamps

ISO 8601 datetimes with an explicit offset: `2026-06-30T14:00:00Z`.

## Body

Standard markdown. Prefer structure (headings, lists, tables, fenced code)
to long prose. Conventional headings: `# Schema` (an asset's columns or
fields), `# Examples`, and `# Computation` (for an Attested Computation).

## Links

- Standard markdown links between concepts, never `[[wikilinks]]`.
- **Absolute** (from the bundle root, starting with `/`) is recommended:
  `[orders](/tables/orders.md)`. **Relative** works too: `[orders](../tables/orders.md)`.
- A link says that two concepts are related; the sentence around it says how.
- A link to a page that doesn't exist yet isn't an error in OKF: it may be
  knowledge not yet written. OKForest warns about it.
- The same path forms work in path-valued fields: `resource`,
  `sources[].resource`, and an Attested Computation's `computation`,
  `executor.resource`, and `attester.resource`.

## `index.md`

Lists its own folder's contents, so a reader or an agent sees what's there
before opening anything:

```markdown
# Customers

* [Acme Corp](acme-corp.md) - Our largest retail customer.
* [Regions](regions/) - Customers by region.
```

- No front matter, except `okf_version: "0.2"` in the bundle's root `index.md`.
- Each entry should carry the description from the concept's front matter.
- It lists only its own folder: entries point within it.

## `log.md`

```markdown
# Update log

## 2026-05-22
* **Update**: Added [Customer Metrics](/tables/customer-metrics.md).
* **Creation**: Started the [Dataplex playbook](/playbooks/dataplex.md).
```

Date headings are `## YYYY-MM-DD`, newest first, each once. The bold word
(`**Update**`, `**Creation**`, `**Deprecation**`) is a convention.

## Attested Computations

A concept with `type: Attested Computation` carries a sanctioned way to
compute a value, so a consumer can confirm the value came from running it:
a `runtime`, `parameters`, an `executor` that runs it and returns a receipt,
an `attester` that checks the receipt without a model, and the computation
itself (a `computation` path, or a code block under `# Computation`). See
the specification's §10 before writing one.
