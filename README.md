# Reference URL → QuickStatements

A static page that finds Wikidata items citing a given URL (including any URL under it) and
generates QuickStatements to add fixed statements to them.

Typical use: you have an nb.no book that describes many people. Those people already cite
pages from the book (`https://www.nb.no/items/URN:NBN:...`) as references. The page then adds
`described by source` (P1343) → the book's Wikidata item to all of them.

## How it works

1. Queries the Wikidata API `list=exturlusage` for the URL prefix. This uses the external-links
   index, so it is fast and covers URLs in statement values and references.
2. Loads the matched items with `wbgetentities` (in batches of 50) to:
   - keep only humans (P31 = Q5), if that option is on
   - skip items that already have the statement
3. Writes QuickStatements v1 lines, optionally with the matched URL as a reference (`S854`).

Everything runs in the browser. It needs no backend and no login.

## Statement syntax

Put one statement per line: property, value, then optional qualifiers or sources, in QuickStatements
form. Separate them with `|`, a tab or spaces:

```
P1343|Q123456
P1343|Q123456|P304|"12-14"
```

## Shareable links

`index.html?url=<prefix>&pairs=P1343|Q123456` fills in the form and runs straight away.

## Deploy

Push `index.html` to a repo, then turn on **Settings → Pages → Deploy from branch (main, / root)**.
