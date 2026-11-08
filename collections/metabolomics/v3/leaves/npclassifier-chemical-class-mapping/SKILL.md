---
name: npclassifier-chemical-class-mapping
description: Use when compound–organism records need to be assigned to NPClassifier
  chemical classes and class-level censuses (distinct class counts, class coverage
  fractions per family) must be computed or validated.
when_to_use_negative:
- When you only need compound counts or taxonomy counts without chemical-class aggregation
  — deduplicate on SMILES/InChI and skip the class column mapping
- When input is already a class-level summary (e.g. an existing npc_summary.csv) —
  consume it directly rather than re-running SIRIUS/CANOPUS
- When the database does not carry the NPClassifier ontology — LOTUS does, but generic
  structure databases would require separate class prediction
edam_operation: http://edamontology.org/operation_2990
tools:
- name: LOTUS
  role: Source database whose dump carries compound–organism pairs pre-annotated with
    the NPClassifier chemical ontology for class counting
  repo: https://lotus.naturalproducts.net/
- name: SIRIUS/CANOPUS
  role: Predicts chemical classes for unreported MS features; CANOPUS output feeds
    the Class Component of extract prioritization
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: canopus_treemap
  role: Recomputes the SIRIUS project space and produces the NPC class summary (npcSummary().to_csv)
    used as the chemical taxonomy table
  repo: https://github.com/kaibioinfo/canopus_treemap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:0d8b9631e1d345a610ac8363ec891d3cebaa2e6c3ced9f542a6cc5b085013f2f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LOTUS
  - SIRIUS/CANOPUS
  - canopus_treemap
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# npclassifier-chemical-class-mapping

## Summary

Map reported natural products (from LOTUS compound–organism pairs or predicted MS annotations) to the NPClassifier chemical ontology and count distinct classes per taxon, enabling chemotaxonomic censuses such as the fraction of Archaeplastida-wide classes covered by a family like Celastraceae.

## When to use

Use when compound–organism records need to be assigned to NPClassifier chemical classes and class-level censuses (distinct class counts, class coverage fractions per family) must be computed or validated.

## When NOT to use

- When you only need compound counts or taxonomy counts without chemical-class aggregation — deduplicate on SMILES/InChI and skip the class column mapping
- When input is already a class-level summary (e.g. an existing npc_summary.csv) — consume it directly rather than re-running SIRIUS/CANOPUS
- When the database does not carry the NPClassifier ontology — LOTUS does, but generic structure databases would require separate class prediction

## Inputs

- LOTUS database dump (compound–organism pairs with taxonomy and NPClassifier ontology columns) from lotus.naturalproducts.net
- Target taxon name at family level (e.g. Celastraceae) and reference set (Archaeplastida)
- SIRIUS CANOPUS project space (for MS-derived class predictions), processed via canopus_treemap

## Outputs

- Count of distinct NPClassifier chemical classes per taxon (e.g. 130 for Celastraceae)
- Fraction of Archaeplastida-wide classes covered by the taxon (~20%)
- Aggregated census CSV (lotus_celastraceae_census.csv) with unique NPs, genera, species, classes, and fraction
- NPC class summary table (npc_summary.csv / canopus_npc_summary_filename) for annotation-based workflows

## How to apply

Download the LOTUS dump (lotus.naturalproducts.net), which already reports natural products together with organism taxonomy and the NPClassifier chemical ontology, and load it with pandas. Filter the reported compound–organism pairs to the target taxon (e.g. family-level taxonomy equals Celastraceae), deduplicate compounds on canonical structure identifiers (SMILES/InChI), then map each compound to its NPClassifier chemical class column and count distinct classes. Compute the taxon's class count as a fraction of the Archaeplastida-wide class set (expected ~20% for Celastraceae, 130 classes). For untargeted MS data, obtain class predictions by running SIRIUS/CANOPUS and recomputing the project space with the canopus_treemap Canopus wrapper, which emits an NPC summary CSV (canopus_npc_summary_filename) used as the chemical taxonomy input to downstream ranking. Validate by checking that class counts reproduce the reported census values and that every filtered compound has a resolvable class annotation.

## Related tools

- **LOTUS** (Source database whose dump carries compound–organism pairs pre-annotated with the NPClassifier chemical ontology for class counting) — https://lotus.naturalproducts.net/
- **SIRIUS/CANOPUS** (Predicts chemical classes for unreported MS features; CANOPUS output feeds the Class Component of extract prioritization) — https://bio.informatik.uni-jena.de/software/sirius/
- **canopus_treemap** (Recomputes the SIRIUS project space and produces the NPC class summary (npcSummary().to_csv) used as the chemical taxonomy table) — https://github.com/kaibioinfo/canopus_treemap

## Examples

```
from canopus import Canopus; C = Canopus(sirius="sirius_projectspace"); C.npcSummary().to_csv("npc_summary.csv")
```

## Evaluation signals

- Class census reproduces reported values: 130 NPClassifier classes for Celastraceae, ~20% of Archaeplastida-wide classes
- Every filtered compound–organism pair resolves to a non-null NPClassifier class before counting distinct classes
- Deduplicated structure identifiers (SMILES/InCHI) yield 4,800 unique NPs, confirming the filter/dedup step is correct before class mapping
- canopus_treemap output npc_summary.csv contains an NPC class column for all features submitted to SIRIUS
- Class count differs from genus count (~38) and species count (168), catching accidental counts against the wrong column

## Limitations

- Class coverage is 'approximately' bounded — the ~38 genera count and ~20% Archaeplastida class fraction carry approximate qualifiers; small deviations can reflect taxonomy-name drift, so clean taxon names against a reference such as Open Tree of Life
- The genus count value is qualified as approximate, so any deviation beyond that qualifier indicates an incorrect filter or identifier-deduplication step rather than a class-mapping error
- CANOPUS predictions cover only features with assignable MS2 spectra; unreported features without class predictions are excluded from class censuses in MS-based workflows

## Evidence

- [methods] LOTUS uses NPClassifier ontology: "given that the [Lotus Dabase](https://lotus.naturalproducts.net/) uses the NPClassifyre ontology"
- [methods] Class Component detects unreported CANOPUS chemical class: "The **Class Component (CC)** indicates if an unreported [**CANOPUS**](https://www.nature.com/articles/s41587-020-0740-8) chemical class is detected"
- [readme] canopus_treemap generates NPC class summary CSV: [evidence span withheld — anchor status: external_doc]
- [other] Workflow step: map compounds to NPClassifier class column: [evidence span withheld — anchor status: not_found]
