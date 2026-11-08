---
name: taxon-compound-census-counting
description: Use when a prioritization or novelty score needs the number of compounds
  already reported for a given species, genus, or family, or when the chemotaxonomic
  breadth of a family must be quantified from a database like LOTUS.
when_to_use_negative:
- The input is already a prioritized extract list or MZmine quantification table —
  this skill enumerates literature-reported compounds from a database dump, not spectral
  features.
- The analysis requires scoring novel spectral annotations (Feature, Class, or Similarity
  components); the census only supplies the Literature Component baseline.
- The target organism lacks reliable family-level taxonomy in the database; filtering
  on family would silently drop entries — resolve names via Open Tree of Life first.
tools:
- name: LOTUS
  role: Source database whose dump links reported natural products to organism taxonomy
    and NPClassifier chemical ontology; filtered to the target family for the census
  repo: https://lotus.naturalproducts.net/
- name: Open Tree of Life
  role: Used to clean species and genera names to currently recognized names before
    counting distinct taxa
  repo: https://opentree.readthedocs.io/en/latest/readme.html
- name: Inventa
  role: Consumer of the per-taxon reported-compound counts, which feed the Literature
    Component of its novelty rank
  repo: https://github.com/luigiquiros/inventa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:e07103a8c231a325013da3a659eb892424ea1bf053241b67a2d0cbd5a1787cb5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Inventa
  - LOTUS
  - Open Tree of Life
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# taxon-compound-census-counting

## Summary

Enumerate the chemotaxonomic scope of a plant family — unique natural products, genera, species, and chemical classes — by filtering reported compound–organism pairs in a natural-products database dump (e.g. LOTUS) to entries whose family-level taxonomy matches the target family, then counting distinct identifiers per axis. Provides the reported-compound baseline that Inventa's Literature Component compares against.

## When to use

Use when a prioritization or novelty score needs the number of compounds already reported for a given species, genus, or family, or when the chemotaxonomic breadth of a family must be quantified from a database like LOTUS.

## When NOT to use

- The input is already a prioritized extract list or MZmine quantification table — this skill enumerates literature-reported compounds from a database dump, not spectral features.
- The analysis requires scoring novel spectral annotations (Feature, Class, or Similarity components); the census only supplies the Literature Component baseline.
- The target organism lacks reliable family-level taxonomy in the database; filtering on family would silently drop entries — resolve names via Open Tree of Life first.

## Inputs

- LOTUS database dump from lotus.naturalproducts.net (compound–organism–NPClassifier linked table)
- pandas-readable flat file of reported compound–organism pairs with family-level taxonomy columns
- Open Tree of Life name-resolution service or API for cleaning species/genera names

## Outputs

- Aggregated census CSV (e.g. lotus_celastraceae_census.csv) with unique NP count, distinct genera, distinct species, distinct NPClassifier classes, and Archaeplastida class-coverage fraction
- Per-taxon reported-compound counts usable as the Literature Component (LC) input for Inventa

## How to apply

Download the LOTUS database dump from lotus.naturalproducts.net, which links reported compounds to organism taxonomy and the NPClassifier chemical ontology. Load the dump with pandas and filter reported compound–organism pairs to rows whose family-level taxonomy equals the target family (e.g. Celastraceae). Count unique natural products by canonical structure identifiers (SMILES/INCHI) rather than by compound name to avoid double-counting, and count distinct genera and species within the family, cleaning taxon names against currently recognized names via the Open Tree of Life as the Inventa README prescribes. Map each reported compound to its NPClassifier class column and count distinct classes, then compute the fraction of Archaeplastida-wide classes covered by the family. Write the aggregated census (unique NPs, genera, species, classes, class-coverage fraction) to a file such as lotus_celastraceae_census.csv; validate the counts against reported values — ~4,800 unique NPs, ~38 genera, 168 species, and 130 NPClassifier classes (~20% of Archaeplastida classes) for Celastraceae — where any deviation beyond the 'approximately' qualifier on the genus count signals an incorrect filter or identifier-deduplication step.

## Related tools

- **LOTUS** (Source database whose dump links reported natural products to organism taxonomy and NPClassifier chemical ontology; filtered to the target family for the census) — https://lotus.naturalproducts.net/
- **Open Tree of Life** (Used to clean species and genera names to currently recognized names before counting distinct taxa) — https://opentree.readthedocs.io/en/latest/readme.html
- **Inventa** (Consumer of the per-taxon reported-compound counts, which feed the Literature Component of its novelty rank) — https://github.com/luigiquiros/inventa

## Examples

```
python census.py --lotus-dump lotus.tsv --family Celastraceae --out lotus_celastraceae_census.csv
```

## Evaluation signals

- Unique NP count must be computed on canonical structure identifiers (SMILES/INCHI); count by name instead and the number inflates due to synonymy.
- For Celastraceae the census must reproduce ~4,800 unique NPs, ~38 genera, 168 species, and 130 NPClassifier classes (~20% of Archaeplastida classes); deviation beyond the 'approximately' qualifier on the genus count indicates an incorrect filter or deduplication step.
- The family filter must be applied to reported compound–organism pairs, not to all compounds in the dump; compounds reported for other families must not enter the count.
- Distinct-class count should be compared as a fraction of Archaeplastida-wide classes to confirm the ~20% coverage expectation.
- Taxa whose names were cleaned via Open Tree of Life should not be double-counted under synonyms.

## Limitations

- Counts are bounded by what the source dump (LOTUS) reports; compounds not yet in the database are invisible to the census.
- The genus count is only approximate, so an exact match is not a valid pass criterion.
- Species and genera names must be cleaned against currently recognized names (Open Tree of Life) before counting; unresolved synonyms distort the per-taxon totals.

## Evidence

- [methods] LOTUS family-level census of reported Celastraceae natural products: "given that the [Lotus Dabase](https://lotus.naturalproducts.net/) uses the NPClassifyre ontology"
- [readme] Taxon names must be cleaned before counting distinct species/genera: [evidence span withheld — anchor status: external_doc]
- [readme] Reported-compound counts feed the Literature Component score: "The **Literature Component (LC)** is a score based on the number of compounds reported in the literature for the taxon of a given extract."
- [abstract] Celastraceae was the proof-of-concept family for the census-driven prioritization: "Inventa was applied to a set of plants from the Celastraceae family as a proof of concept."
