---
name: chemical-class-novelty-scoring
description: Decision support for prioritizing natural extracts whose MS/MS-derived
  chemical taxonomy (CANOPUS/NPC classes) contains classes absent from literature
  reports for the same species or genus, as one of four components in Inventa's Priority
  Score ranking.
when_to_use_negative:
- No CANOPUS/SIRIUS chemical-taxonomy annotations are available for the feature set
  — CC cannot be computed and the run degrades to the Feature Component (FC) only
- The taxon already has extensive literature-reported chemistry and the goal is literature-gap
  assessment rather than unreported-class detection — the Literature Component (LC)
  is the appropriate score
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Inventa
  role: Computes the Class Component from CANOPUS annotations and combines it into
    the Priority Score
  repo: https://github.com/luigiquiros/inventa
- name: SIRIUS
  role: Provides molecular formula and structure annotations feeding CANOPUS class
    predictions
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: CANOPUS
  role: Predicts chemical classes for unannotated MS/MS features; an unreported CANOPUS
    class triggers the CC score
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: canopus_treemap
  role: Recomputes the Sirius project space to produce the npc_summary.csv/canopus_npc_summary.tsv
    that Inventa reads
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: GNPS
  role: Supplies the feature-based molecular networking annotations cleaned before
    class assessment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:4fd1b26bb65d6990da331f0a11fe823ff97bd489f89dc7dc405f2b21bd5669e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - CANOPUS
  - GNPS
  - Inventa
  - MZmine3
  - SIRIUS
  - canopus_treemap
  - scikit-bio
  - timaR
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# chemical-class-novelty-scoring

## Summary

Computes Inventa's Class Component (CC), which flags extracts containing a CANOPUS-predicted chemical class not yet reported in the literature for that species or genus, contributing up to 1.0 (0.5 species + 0.5 genus novelty) to the Priority Score.

## When to use

Decision support for prioritizing natural extracts whose MS/MS-derived chemical taxonomy (CANOPUS/NPC classes) contains classes absent from literature reports for the same species or genus, as one of four components in Inventa's Priority Score ranking.

## When NOT to use

- No CANOPUS/SIRIUS chemical-taxonomy annotations are available for the feature set — CC cannot be computed and the run degrades to the Feature Component (FC) only
- The taxon already has extensive literature-reported chemistry and the goal is literature-gap assessment rather than unreported-class detection — the Literature Component (LC) is the appropriate score

## Inputs

- Sirius CANOPUS recomputed output in canopus_npc_summary.tsv format (via kaibioinfo/canopus_treemap)
- GNPS-format metadata table with ATTRIBUTE_Species and ATTRIBUTE_Organe headers
- MZmine 2/3 feature quantitative table (Peak area columns)
- Annotation files (ISDB/timaR, SIRIUS) to distinguish annotated features

## Outputs

- Per-extract CC value (and CCs/CCg sub-scores) in the results TSV
- Extract ranking contribution to the final Priority Score

## How to apply

Run SIRIUS with CANOPUS on the GNPS/FBMN feature set, then recompute the project space with the canopus_treemap package to produce the canopus_npc_summary.tsv file that Inventa ingests via `canopus_npc_summary_filename`. Inventa maps each feature's predicted chemical class against reported classes for the extract's taxon, requiring min_class_confidence = 0.8 (e.g. NPC classification confidence) and min_recurrence = 5 (a class must recur in at least 5 features) before a novelty claim is made. A CC value of 1 implies the chemical class is new to both the species (CCs = 0.5) and the genus (CCg = 0.5); reported classes reduce the score accordingly. The CC is then combined additively with FC, LC, and SC, modulated by weight w4, to form the extract's Priority Score.

## Related tools

- **Inventa** (Computes the Class Component from CANOPUS annotations and combines it into the Priority Score) — https://github.com/luigiquiros/inventa
- **SIRIUS** (Provides molecular formula and structure annotations feeding CANOPUS class predictions) — https://bio.informatik.uni-jena.de/software/sirius/
- **CANOPUS** (Predicts chemical classes for unannotated MS/MS features; an unreported CANOPUS class triggers the CC score) — https://github.com/kaibioinfo/canopus_treemap
- **canopus_treemap** (Recomputes the Sirius project space to produce the npc_summary.csv/canopus_npc_summary.tsv that Inventa reads) — https://github.com/kaibioinfo/canopus_treemap
- **GNPS** (Supplies the feature-based molecular networking annotations cleaned before class assessment)

## Examples

```
from canopus import Canopus; C = Canopus(sirius="sirius_projectspace"); C.npcSummary().to_csv("npc_summary.csv")
```

## Evaluation signals

- CC values fall in [0, 1], with CC = 1 exactly when the class is new to both species (CCs = 0.5) and genus (CCg = 0.5)
- canopus_npc_summary.tsv loads without errors in the Inventa notebook (recomputed via canopus_treemap after SIRIUS run)
- Only classes passing min_class_confidence = 0.8 and recurring in at least min_recurrence = 5 features contribute to CC
- In the Celastraceae validation run, the top-ranked extract (Pristimera indica roots, PS 3.23) carries CC = 1, matching Table 1

## Limitations

- CC depends entirely on CANOPUS prediction quality for features lacking spectral-library annotations; mispredicted classes translate directly into false novelty
- Literature-derived class reports for species/genus come from the LOTUS database (NPClassifyre ontology), so incompleteness of LOTUS biases novelty upward
- The canopus_npc_summary requires an extra recomputation step after SIRIUS; without it the CC is skipped and only FC-based ranking is possible

## Evidence

- [methods] Class Component definition: "The **Class Component (CC)** indicates if an unreported [**CANOPUS**](https://www.nature.com/articles/s41587-020-0740-8) chemical class is detected"
- [readme] CC scoring semantics: [evidence span withheld — anchor status: external_doc]
- [readme] Recomputation step: [evidence span withheld — anchor status: external_doc]
- [methods] Lotus ontology dependency: "given that the [Lotus Dabase](https://lotus.naturalproducts.net/) uses the NPClassifyre ontology"
- [abstract] Proof of concept result: "Inventa was applied to a set of plants from the Celastraceae family as a proof of concept."
