# Wikidata tools

Small single-page tools for batch work on Wikidata. They all run in the browser and need no server.

**Live:** https://andjar.github.io/wikidata-tools/

| Tool | File | Output |
|---|---|---|
| Reference URL → QuickStatements | `url-to-quickstatements.html` | QuickStatements |
| Reference fixer | `reference-fixer.html` | Direct edits (OAuth 2.0 token) |
| Chain builder | `chain-builder.html` | QuickStatements |

`index.html` is the landing page.

---

## Reference URL → QuickStatements (`url-to-quickstatements.html`)

A static page that finds Wikidata items citing a given URL (including any URL under it) and
generates QuickStatements to add fixed statements to them.

Typical use: you have an nb.no book that describes many people. Those people already cite
pages from the book (`https://www.nb.no/items/URN:NBN:...`) as references. The page then adds
`described by source` (P1343) → the book's Wikidata item to all of them.

### How it works

1. Queries the Wikidata API `list=exturlusage` for the URL prefix. This uses the external-links
   index, so it is fast and covers URLs in statement values and references.
2. Loads the matched items with `wbgetentities` (in batches of 50) to:
   - keep only humans (P31 = Q5), if that option is on
   - skip items that already have the statement
3. Writes QuickStatements v1 lines, optionally with the matched URL as a reference (`S854`).

Everything runs in the browser. It needs no backend and no login.

### Statement syntax

Put one statement per line: property, value, then optional qualifiers or sources, in QuickStatements
form. Separate them with `|`, a tab or spaces:

```
P1343|Q123456
P1343|Q123456|P304|"12-14"
```

### Shareable links

`url-to-quickstatements.html?url=<prefix>&pairs=P1343|Q123456` fills in the form and runs straight away.

---

## Reference fixer (`reference-fixer.html`)

Adds properties to **existing references** whose reference URL (P854) starts with a prefix.
Example: every reference pointing to `https://www.nrk.no/` gets `stated in` (P248) = the NRK item.

QuickStatements can't change an existing reference. It would add a second one instead. So this page
calls `wbsetreference` directly. That call replaces only the matched reference, identified by its
hash, and keeps the statement, its qualifiers, its rank and its other references.

1. Finds items through `exturlusage`, then walks every statement → reference to find references with a matching P854.
2. Shows a preview with the new snaks highlighted. Each reference has its own checkbox.
3. Applies the edits one by one, roughly one per second. It sends `baserevid` and respects `maxlag`, and it shows a diff link for each edit.

Values: `P248|Q123` for an item, `P1476|"text"` for a string, `P577|+2024-05-01T00:00:00Z/11` for a date (precision 11 = day).
With the default option on, a property the reference already has is left alone, so a P248 that is already there is never changed.

**Authentication.** Editing uses an OAuth 2.0 bearer token. MediaWiki ≥ 1.44 accepts the
`crossorigin` parameter, so a static page can make authenticated calls. Each user registers their own
*owner-only* OAuth 2.0 consumer on meta (grants: Basic rights + Edit existing pages). Owner-only
consumers are approved automatically and give you an access token straight away. The token is sent
only to wikidata.org, and it is stored in localStorage only if you tick that option.

Prefill: `reference-fixer.html?url=https://www.nrk.no/&adds=P248|Q123&summary=...`

---

## Chain builder (`chain-builder.html`)

Builds follows/followed by chains. Load items with SPARQL, drag them into order, and get QuickStatements.

- **Item mode** puts P155/P156 on the items themselves. **Qualifier mode** adds them as qualifiers to a
  statement, e.g. P179 = series. Both properties can be changed, so office holders work too:
  P39 = office, with P1365/P1366.
- Items that are already linked on Wikidata are grouped into locked blocks with a green edge, and each block moves as one unit.
  If a link exists on only one side, the missing side is added.
- A **break** splits the chain. A *gap* break adds nothing. An *unknown item* break adds “unknown value”
  (`somevalue`) to the neighbouring items.
- **Leave out** moves an item into a separate list so it is not part of the chain.
- Every link is labelled ✓ exists, + new or ⚠ conflict. A conflict is an item already linked to a different item, and nothing is added for that link.

Limitation: an item can appear only once in a chain, so someone who held the same office twice doesn't work yet.
Uses SortableJS from jsDelivr.

## Deploy

The repo is served with GitHub Pages: turn on **Settings → Pages → Deploy from branch (main, / root)**.
