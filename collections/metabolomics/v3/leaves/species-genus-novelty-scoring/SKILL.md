---
name: species-genus-novelty-scoring
description: 'Use when prioritizing natural extracts by chemical novelty: you have
  CANOPUS NPC class predictions for a feature table and literature-reported compounds/classes
  for the taxa, and need to flag classes new to species or genus.'
when_to_use_negative:
- You only have GNPS FBMN annotations and no CANOPUS/SIRIUS NPC class predictions
  — use the Literature Component (LC) or Feature Component (FC) instead
- The extract's species or genus is absent from LOTUS reports AND class predictions
  are below the 0.8 confidence cutoff, since CC would be driven by unverified classes
- You want extract-set spectral similarity (outlier) ranking — that is the Similarity
  Component (SC) via MEMO, not CC
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3332
tools:
- name: Inventa
  role: Computes the CC score and assembles per-sample novelty ranks from filtered
    CANOPUS classes vs LOTUS reports
  repo: https://github.com/luigiquiros/inventa
- name: SIRIUS
  role: Produces the molecular formula/structure project space whose recomputation
    yields NPC class predictions
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: CANOPUS
  role: Supplies predicted chemical classes (via NPC ontology) that are compared against
    reported classes
- name: canopus_treemap
  role: Recomputes the Sirius project space and writes the canopus_npc_summary table
    consumed by the CC calculation
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: LOTUS
  role: Source of literature-reported compounds/classes per taxon used for the species/genus
    comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:e5770ce7c65179e49b1a0e5df599cfe63f0929ea27bf9963b54f4e998a8d78ab
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# species-genus-novelty-scoring

## Summary

Computes the Class Component (CC) of Inventa's novelty rank by string-matching CANOPUS/NPClassifier-predicted chemical classes against literature-reported classes (LOTUS) at the species and genus levels, awarding 0.5 per unreported level so CC ranges 0–1.

## When to use

Use when prioritizing natural extracts by chemical novelty: you have CANOPUS NPC class predictions for a feature table and literature-reported compounds/classes for the taxa, and need to flag classes new to species or genus.

## When NOT to use

- You only have GNPS FBMN annotations and no CANOPUS/SIRIUS NPC class predictions — use the Literature Component (LC) or Feature Component (FC) instead
- The extract's species or genus is absent from LOTUS reports AND class predictions are below the 0.8 confidence cutoff, since CC would be driven by unverified classes
- You want extract-set spectral similarity (outlier) ranking — that is the Similarity Component (SC) via MEMO, not CC

## Inputs

- canopus_npc_summary.tsv — Sirius CANOPUS recomputed output from canopus_treemap npcSummary()
- GNPS-format metadata table with ATTRIBUTE_Species and ATTRIBUTE_Organe (and genus derived) columns
- Taxon-level reported compound/class information from LOTUS (NPClassifyre ontology)
- User-configured min_class_confidence (0.8) and min_recurrence (5) thresholds

## Outputs

- Per-sample CC values (CCs + CCg, 0–1) as a TSV file
- List of new known chemical classes at species and genus levels per extract
- CC component contributing to the combined Inventa priority rank

## How to apply

Recompute the SIRIUS project space with the canopus_treemap package to produce the CANOPUS NPClassifier summary (canopus_npc_summary.tsv), then filter predicted classes with min_class_confidence = 0.8 and min_recurrence = 5. Build species- and genus-level reported class sets from the metadata ATTRIBUTE_Species/genus columns using LOTUS taxon-level reports. For each sample, subtract the reported class set from the filtered predicted class set: assign CCs = 0.5 when a class is new to the species and CCg = 0.5 when new to the genus, and sum CC = CCs + CCg (range 0–1). Write per-sample CC values with the specific new chemical classes at each level as a TSV for ranking alongside FC, LC, and SC.

## Related tools

- **Inventa** (Computes the CC score and assembles per-sample novelty ranks from filtered CANOPUS classes vs LOTUS reports) — https://github.com/luigiquiros/inventa
- **SIRIUS** (Produces the molecular formula/structure project space whose recomputation yields NPC class predictions) — https://bio.informatik.uni-jena.de/software/sirius/
- **CANOPUS** (Supplies predicted chemical classes (via NPC ontology) that are compared against reported classes)
- **canopus_treemap** (Recomputes the Sirius project space and writes the canopus_npc_summary table consumed by the CC calculation) — https://github.com/kaibioinfo/canopus_treemap
- **LOTUS** (Source of literature-reported compounds/classes per taxon used for the species/genus comparison)

## Examples

```
from canopus import Canopus; C = Canopus(sirius="sirius_projectspace"); C.npcSummary().to_csv("npc_summary.csv")
```

## Evaluation signals

- CC values fall in [0, 1]; CC = 1 exactly when the class is new to both species (CCs = 0.5) and genus (CCg = 0.5)
- Pristimera indica roots extract scores CC = 1 for the dihydro-β-agarofuran sesquiterpenoids class (0.5 new at species + 0.5 new at genus), matching the isolated thirteen new dihydro-β-agarofuran sesquiterpenes
- Only classes passing min_class_confidence = 0.8 and min_recurrence = 5 enter the subtraction step
- Output is a TSV containing per-sample CC plus the specific new chemical classes at species and genus levels

## Limitations

- CC depends on CANOPUS class prediction quality and the confidence/recurrence cutoffs; low-confidence classes are excluded and cannot contribute novelty
- Literature coverage in LOTUS is incomplete — a 'new' class may simply be unreported rather than truly novel
- As a proof of concept CC was applied to Celastraceae plants; other taxonomic scopes may need recalibrated thresholds

## Evidence

- [readme] CC additive rule: "A CC value of 1 implies that the chemical class is new to both the species (CCs 0.5) and the genus (CCg 0.5)."
- [methods] CC definition: "indicates if an unreported [**CANOPUS**](https://www.nature.com/articles/s41587-020-0740-8) chemical class is detected"
- [readme] NPC summary generation: "you can create a summary file with all NPC classes by the following code: 
```python
from canopus import Canopus
C = Canopus(sirius="sirius_projectspace")
C.npcSummary().to_csv("npc_summary.csv")"
- [abstract] Validation finding: "The Pristimera indica (Willd.) A.C.Sm roots extract was highlighted as a promising source of potentially novel compounds"
