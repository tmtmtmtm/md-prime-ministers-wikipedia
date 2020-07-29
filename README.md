Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item — https://www.wikidata.org/wiki/Q1769526 —
contains all the data expected already.

Step 2: Tracking page
=====================

PositionHolderHistory already exists — starting version is
https://www.wikidata.org/w/index.php?title=Talk:Q1769526&oldid=1241536021
with 14 dated memberships, none undated; and 12 warnigs.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit [add_P39.js script](add_P39.js)
to configure the Item ID and source URL.

Step 4: Get local copy of Wikidata information
==============================================

```sh
wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
  xargs wd sparql existing-p39s.js > wikidata.json
```

Step 5: Scrape
==============

Comparison/source = https://en.wikipedia.org/wiki/Prime_Minister_of_Moldova

```sh
wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
  xargs bundle exec ruby scraper.rb | tee wikipedia.csv
```

Fairly easy, other than dual end dates for Ion Ciubuc. I took the
earlier one, as there's an acting PM for the remaining dates.

Step 6: Create missing P39s
===========================

```sh
bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
  wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"
```

5 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/7054bcfd53143

Step 7: Add missing qualifiers
==============================

```sh
bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
  wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"
```

5 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/7947c762a62af

Step 8: Refresh the Tracking Page
==================================

After a few other manual changes, new version at
https://www.wikidata.org/w/index.php?title=Talk:Q1769526&oldid=1241565091
