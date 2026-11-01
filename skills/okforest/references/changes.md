# Changing a tree with OKForest's MCP tools

OKForest's MCP server (`https://mcp.okforest.ai/mcp`) reads an account's
trees, and makes changes that wait for a tree's editors: nothing reaches a
tree until an editor applies the change in OKForest, as one commit. The
editors are emailed about each new request.

Trees are named by **slug** (`list_trees`); pages by their **path** in the
tree (`customers/acme-corp.md`, as `get_concept` gives it); concepts by a
Forest-wide **ID**, the tree's slug first (`company:customers/acme-corp`).

## Before changing anything

1. `search_concepts`, then `get_concept` for each page you'll touch. An edit
   must match the page's body exactly as `get_concept` gives it.
2. `list_types` for the fields a type's schema expects, before writing a
   concept of that type.
3. `list_changes` for an open change that already does the same; point the
   person to it rather than asking twice.

## `propose_change`: the edits, written

```json
{
  "tree": "company",
  "title": "Add Globex, and say Acme Corp buys the loyalty plan",
  "edits": [
    {
      "kind": "create",
      "path": "customers/globex.md",
      "text": "---\ntype: Customer\ntitle: Globex\ndescription: A manufacturing customer since 2026.\n---\n\nGlobex buys our [loyalty plan](/plans/loyalty.md).\n"
    },
    {
      "kind": "edit",
      "path": "customers/acme-corp.md",
      "fields": { "tags": ["customer", "retail", "loyalty"], "stale_after": null },
      "replacements": [
        { "old": "our retail analytics plan.", "new": "our retail analytics and loyalty plans." }
      ]
    }
  ]
}
```

Up to 20 edits, applied in order:

- **`create`** (`path`, `text`): a new page, written whole: front matter
  with a `type` and a `title`, then its body. Its folder's `index.md` lists
  it, if the folder has one.
- **`edit`** (`path`, `fields`, `replacements`):
  - `fields` sets front-matter keys to new values; `null` removes one. The
    rest of the front matter stays as written.
  - `replacements` change the body, in order. Each `old` must appear exactly
    once: include enough around it. An empty `new` removes the text.
- **`move`** (`from`, `to`, `files`): a new folder, a new file name, or
  both. Links to it and `index.md` entries follow it; its own files move
  with it unless `files` is false.
- **`deprecate`** (`path`, `successor`): marks it `status: deprecated`, kept
  for its links and history; with a `successor` path, its body starts by
  naming what replaces it.
- **`delete`** (`path`, `files`): its `index.md` entries go, and its own
  files too unless `files` is false. Links to it from other pages break, so
  prefer `deprecate` for a page others link to.

Each page can be in one edit of a change. Keep a change to one purpose,
small enough to review at a glance; propose several changes for unrelated
things.

**What comes back:**

- Saved: the change, with its `state` (`in-review` for an editor's own,
  `requested` for anyone else's), any `findings` (warnings worth fixing in a
  follow-up), and its `url`. Give the person the `url`.
- Not saved, "Nothing was saved. Edit 2 (edit): ...": an edit that couldn't
  apply (its text isn't there, or is there more than once; a path that's
  taken; a page the tree doesn't have). Fix that edit and call again.
- Not saved, "the checks found errors to fix first": a page would break a
  rule (see [okforest.md](okforest.md)), listed by file, line, and rule. Fix
  them and call again.
- Refused, "OKForest can't change tree ...": the tree is read-only (a public
  repository, editing turned off, or no write access), with the reason to
  tell the person.

## `request_change`: asked in words

```json
{
  "about": "company:customers/acme-corp",
  "title": "Add Acme Corp's renewal date",
  "instruction": "Their contract renews on 1 March 2027, says their account manager."
}
```

For when the person wants to ask rather than write the edits, or when the
edits need an editor's judgment. `title` says what should change; the
`instruction` gives the details in the person's words. An editor's own
request starts as `drafting`, for them to write; anyone else's is
`requested`.

## Following a change

- `list_changes` (`tree`, `states`): a tree's changes, newest first, by
  state: `requested`, `drafting`, `in-review`, or done (`applied`,
  `declined`, `withdrawn`).
- `get_change` (`tree`, `id`): one change's words, each file's edit (front
  matter field by field, then its lines), its checks, who decided or
  applied it, and its comments. A declined change's reason is among its
  comments.

Applying, accepting, and declining happen in OKForest, on the change's page.
