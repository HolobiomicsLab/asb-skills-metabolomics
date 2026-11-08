---
name: natural-products-database-querying
description: Use when a research question requires counts or inventories of reported
  natural products per taxon (family/genus/species) from a structured NP database
  dump, or when chemical-class coverage of a clade must be benchmarked against a larger
  kingdom-wide set.
when_to_use_negative:
- The goal is prioritizing extracts for novelty from untargeted MS data (use Inventa's
  FC/LC/CC/SC scoring instead of a database census)
- The database does not provide organism taxonomy or chemical ontology columns, so
  family-level filtering and NPClassifier class mapping cannot be performed
- Only literature-reported compounds per single species are needed for Inventa's Literature
  Component, not a database-wide clade census
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: LOTUS
  role: Source database dump queried for compound–organism pairs with NPClassifyR
    ontology chemical classes
  repo: https://lotus.naturalproducts.net/
- name: Open Tree of Life
  role: Cleaning taxon names (species/genus) against currently recognized names before
    counting distinct taxa
  repo: https://opentree.readthedocs.io/en/latest/readme.html
- name: pandas
  role: Loading the LOTUS dump and filtering/counting unique NPs, genera, species,
    and classes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:aeda6426e54a4ba5be43159628fe9543e59f303aaae49c98b6c7cf28e570bae5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LOTUS
  - Open Tree of Life
  - pandas
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# natural-products-database-querying

## Summary

Queries curated natural-product databases such as LOTUS to extract compound–organism pairs, then filters them by taxonomic rank (e.g. family=Celastraceae), deduplicates compounds by canonical structure identifiers (SMILES/InChI), and aggregates chemotaxonomic censuses of unique NPs, genera, species, and NPClassifier chemical classes.

## When to use

Use when a research question requires counts or inventories of reported natural products per taxon (family/genus/species) from a structured NP database dump, or when chemical-class coverage of a clade must be benchmarked against a larger kingdom-wide set.

## When NOT to use

- The goal is prioritizing extracts for novelty from untargeted MS data (use Inventa's FC/LC/CC/SC scoring instead of a database census)
- The database does not provide organism taxonomy or chemical ontology columns, so family-level filtering and NPClassifier class mapping cannot be performed
- Only literature-reported compounds per single species are needed for Inventa's Literature Component, not a database-wide clade census

## Inputs

- LOTUS database dump (TSV/CSV) with compound–organism pairs, organism taxonomy, and NPClassifier chemical class columns
- Canonical structure identifiers (SMILES or InChI) per reported natural product
- Optional: Open Tree of Life access for validating/cleaning taxon names

## Outputs

- Aggregated chemotaxonomic census table (e.g. lotus_celastraceae_census.csv) with unique NP count, genus count, species count, distinct NPClassifier class count, and fraction of kingdom-wide classes
- Filtered set of unique compound–organism pairs for the target family

## How to apply

Download the LOTUS database dump from lotus.naturalproducts.net, which reports natural products together with organism taxonomy and NPClassifier chemical ontology, and load it with pandas. Filter reported compound–organism pairs to entries whose family-level taxonomy equals the target taxon (e.g. Celastraceae). Count unique natural products using canonical structure identifiers (SMILES/InChI) to deduplicate repeated reports of the same molecule, and count distinct genera and species, cleaning taxon names against currently recognized names via Open Tree of Life. Map each compound to its NPClassifier chemical class column (LOTUS uses the NPClassifyR ontology), count distinct classes, and compute the fraction of kingdom-wide (e.g. Archaeplastida) classes covered by the family. Write the aggregated census (unique NPs, genera, species, classes, fraction) to a CSV such as lotus_celastraceae_census.csv; 'approximately' qualified counts (e.g. ~38 genera) tolerate small deviations from the reference values, but exact counts like 4,800 NPs must match.

## Related tools

- **LOTUS** (Source database dump queried for compound–organism pairs with NPClassifyR ontology chemical classes) — https://lotus.naturalproducts.net/
- **Open Tree of Life** (Cleaning taxon names (species/genus) against currently recognized names before counting distinct taxa) — https://opentree.readthedocs.io/en/latest/readme.html
- **pandas** (Loading the LOTUS dump and filtering/counting unique NPs, genera, species, and classes)

## Examples

```
import pandas as pd; df = pd.read_csv('lotus_dump.tsv', sep='\t'); fam = df[df['organism_family']=='Celastraceae']; fam.drop_duplicates('inchi')['inchi'].count()
```

## Evaluation signals

- Reproduced census matches the reference values: 4,800 unique NPs (0.98% of Archaeplastida entries), ~38 genera, 168 species for Celastraceae
- Class census matches 130 NPClassifier classes, ~20% of Archaeplastida classes
- Unique-compound counts are computed on deduplicated canonical identifiers (SMILES/InChI), not raw report rows
- Any deviation beyond the 'approximately' qualifier on the genus count indicates an incorrect taxonomic filter or identifier-deduplication step

## Limitations

- Counts depend on the state of the LOTUS dump at download time and on dictionary-style reporting (LOTUS and the Dictionary of Natural Products agree at 4,800 unique NPs, but the genus count is only 'approximately' 38)
- Taxon names in the dump may not be currently recognized and must be cleaned (e.g. via Open Tree of Life) or genus/species counts will be wrong

## Evidence

- [methods] LOTUS database usage with chemical ontology: "given that the [Lotus Dabase](https://lotus.naturalproducts.net/) uses the NPClassifyre ontology"
- [methods] Open Tree of Life for taxon cleaning: "you can use the [Open Tree of Life](https://opentree.readthedocs.io/en/latest/readme.html)"
- [readme] README instruction to clean species names with Open Tree of Life: [evidence span withheld — anchor status: not_found]
- [abstract] Celastraceae proof of concept grounding the taxon choice: "Inventa was applied to a set of plants from the Celastraceae family as a proof of concept."
