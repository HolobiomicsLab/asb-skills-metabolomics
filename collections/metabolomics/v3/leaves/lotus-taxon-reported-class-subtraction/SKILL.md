---
name: lotus-taxon-reported-class-subtraction
description: Use when prioritizing natural extracts for chemical novelty and you need
  to flag CANOPUS/NPClassifier-predicted chemical classes that are absent from literature
  reports for the extract's species and genus.
when_to_use_negative:
- No CANOPUS/SIRIUS output is available for the feature table — Inventa falls back
  to the Feature Component (FC), which needs only GNPS/MZmine inputs, so class subtraction
  cannot run
- The goal is compound-level literature novelty (Literature Component LC) rather than
  chemical-class novelty; LC uses reported compound counts, not class-set subtraction
- Predicted classes fall below min_class_confidence = 0.8 or recurrence below 5 —
  they are filtered out before subtraction and must not drive the CC score
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: Inventa
  role: 'Hosts the CC computation: loads the CANOPUS NPC summary, filters classes
    by min_class_confidence/min_recurrence, performs species/genus reported-class
    subtraction, and writes the CC column to the output TSV'
  repo: https://github.com/luigiquiros/inventa
- name: SIRIUS
  role: Produces the project space whose CANOPUS class predictions are recomputed
    and summarized for comparison against literature reports
- name: CANOPUS
  role: Source of the predicted chemical classes (NPClassifier/NPC classes) that are
    compared with the reported classes
- name: canopus_treemap
  role: Recomputes the SIRIUS project space and generates the NPC summary file (npc_summary.csv
    / canopus_npc_summary.tsv) consumed by the CC step
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: LOTUS
  role: Provides taxon-level reported compound/class information from which species-level
    and genus-level reported class sets are derived
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:a74990840c9fca27fd9dffbc04c5df241ffb6d2b79b020f1320fbd26da8df621
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - CANOPUS
  - Inventa
  - LOTUS
  - SIRIUS
  - canopus_treemap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# LOTUS taxon-reported class subtraction for the Class Component (CC) score

## Summary

Computes Inventa's Class Component (CC) by subtracting LOTUS-derived chemical classes reported for a taxon's species and genus from CANOPUS/NPClassifier classes predicted for the extract, scoring 0.5 per level where a predicted class is unreported.

## When to use

Use when prioritizing natural extracts for chemical novelty and you need to flag CANOPUS/NPClassifier-predicted chemical classes that are absent from literature reports for the extract's species and genus.

## When NOT to use

- No CANOPUS/SIRIUS output is available for the feature table — Inventa falls back to the Feature Component (FC), which needs only GNPS/MZmine inputs, so class subtraction cannot run
- The goal is compound-level literature novelty (Literature Component LC) rather than chemical-class novelty; LC uses reported compound counts, not class-set subtraction
- Predicted classes fall below min_class_confidence = 0.8 or recurrence below 5 — they are filtered out before subtraction and must not drive the CC score

## Inputs

- canopus_npc_summary.tsv — SIRIUS CANOPUS recomputed NPClassifier summary table
- Metadata table in GNPS FBMN format with mandatory ATTRIBUTE_Species (and organ) columns
- Taxon-level reported compound/class information from the LOTUS database (NPClassifyre ontology)
- Inventa configuration with min_class_confidence = 0.8 and min_recurrence = 5

## Outputs

- Per-sample CC values (CC = CCs + CCg, range 0–1) in a TSV file
- List of specific new known chemical classes at species level (CCs) and genus level (CCg) per sample

## How to apply

Load the CANOPUS NPClassifier summary table (canopus_npc_summary.tsv) produced by recomputing the SIRIUS project space with the canopus_treemap package, then filter predicted chemical classes with the configured min_class_confidence = 0.8 cutoff and min_recurrence = 5 recurrence threshold. Separately, load taxon-level reported compound/class information from LOTUS and derive species-level and genus-level reported class sets per extract using the metadata species and genus columns (e.g., GNPS-format ATTRIBUTE_Species). Perform a string set subtraction of reported classes from the filtered predicted class set per sample, assigning CCs = 0.5 when a class is new to the species and CCg = 0.5 when new to the genus, then sum CC = CCs + CCg (range 0–1). Assemble per-sample CC values alongside the specific new known chemical classes at species and genus levels and write the results as a TSV file. Validate by checking that the Pristimera indica roots extract scores CC = 1 for the dihydro-β-agarofuran sesquiterpenoids class (0.5 new at species + 0.5 new at genus).

## Related tools

- **Inventa** (Hosts the CC computation: loads the CANOPUS NPC summary, filters classes by min_class_confidence/min_recurrence, performs species/genus reported-class subtraction, and writes the CC column to the output TSV) — https://github.com/luigiquiros/inventa
- **SIRIUS** (Produces the project space whose CANOPUS class predictions are recomputed and summarized for comparison against literature reports)
- **CANOPUS** (Source of the predicted chemical classes (NPClassifier/NPC classes) that are compared with the reported classes)
- **canopus_treemap** (Recomputes the SIRIUS project space and generates the NPC summary file (npc_summary.csv / canopus_npc_summary.tsv) consumed by the CC step) — https://github.com/kaibioinfo/canopus_treemap
- **LOTUS** (Provides taxon-level reported compound/class information from which species-level and genus-level reported class sets are derived)

## Examples

```
from canopus import Canopus
C = Canopus(sirius="sirius_projectspace")
C.npcSummary().to_csv("npc_summary.csv")
```

## Evaluation signals

- Pristimera indica roots extract must score CC = 1 for the dihydro-β-agarofuran sesquiterpenoids class (0.5 new at species level + 0.5 new at genus level)
- CC values are bounded in [0, 1] since CC = CCs + CCg with each level contributing at most 0.5
- Filtered predicted class set must contain only classes with CANOPUS/NPClassifier confidence >= 0.8 and recurrence >= 5 before subtraction
- Output TSV must pair each sample's CC value with the explicit new known chemical classes at species and genus levels
- Species/genus reported class sets must be derived from the same metadata species/genus columns used for the GNPS FBMN metadata table

## Limitations

- CC is binary per level (0 or 0.5); it signals presence of an unreported class rather than grading novelty extent
- String set subtraction requires consistent class nomenclature; LOTUS uses the NPClassifyre ontology while predictions come from NPClassifier, so mismatched labels can create false 'new' classes
- Depends on recomputing the SIRIUS project space with canopus_treemap, although the README notes NPC prediction support has existed in CANOPUS for months and recompute may not be needed
- Threshold-dependent: classes below min_class_confidence = 0.8 or with recurrence below 5 are excluded from the novelty call

## Evidence

- [methods] Class Component CC definition: "The **Class Component (CC)** indicates if an unreported [**CANOPUS**](https://www.nature.com/articles/s41587-020-0740-8) chemical class is detected"
- [readme] CC scoring scheme: "A CC value of 1 implies that the chemical class is new to both the species (CCs 0.5) and the genus (CCg 0.5)."
- [methods] LOTUS ontology note: "given that the [Lotus Dabase](https://lotus.naturalproducts.net/) uses the NPClassifyre ontology"
- [readme] NPC summary generation via canopus_treemap: "you can create a summary file with all NPC classes by the following code:
```python
from canopus import Canopus
C = Canopus(sirius="sirius_projectspace")
C.npcSummary().to_csv("npc_summary.csv")"
- [abstract] Pristimera indica validation finding: "The Pristimera indica (Willd.) A.C.Sm roots extract was highlighted as a promising source of potentially novel compounds."
