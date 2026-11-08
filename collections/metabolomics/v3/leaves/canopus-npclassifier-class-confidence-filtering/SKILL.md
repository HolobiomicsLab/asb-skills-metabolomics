---
name: canopus-npclassifier-class-confidence-filtering
description: Use when a canopus_npc_summary table of predicted chemical classes per
  feature/extract must be reduced to high-confidence, recurring classes prior to novelty
  scoring (e.g., computing the CC score against LOTUS/NPClassifier reports).
when_to_use_negative:
- Input is a raw SIRIUS/CANOPUS project space that has not yet been summarized via
  canopus_treemap's npcSummary() — generate the summary table first.
- The analysis does not involve chemical-class novelty assessment against literature
  reports (e.g., only FC or SC components are needed) — class filtering is specific
  to the CC component.
- Class predictions come from a different ontology (e.g., GNPS molecular networking
  annotations rather than NPClassifier) — the CC score compares NPClassifier classes
  to LOTUS's NPClassifyre ontology.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: canopus_treemap
  role: Recomputes the SIRIUS project space and generates the NPClassifier summary
    table (npcSummary) that is subsequently filtered by confidence/recurrence
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: SIRIUS
  role: Produces the molecular formula/structure project space whose CANOPUS class
    predictions are summarized and filtered
- name: CANOPUS
  role: Provides the chemical-class predictions that are filtered by confidence and
    recurrence before literature comparison
- name: Inventa
  role: Applies the min_class_confidence and min_recurrence filters within the Class
    Component (CC) calculation
  repo: https://github.com/luigiquiros/inventa
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:86bbdb5b137b2444be909ab5ec31dda52077f1ea795905f3874fca734365aa5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kaibioinfo/canopus_treemap
  tools:
  - CANOPUS
  - Inventa
  - SIRIUS
  - canopus_treemap
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# canopus-npclassifier-class-confidence-filtering

## Summary

Filters the CANOPUS/NPClassifier chemical-class predictions produced from a SIRIUS project space by class-confidence and recurrence thresholds before they are compared against literature-reported classes in Inventa's Class Component (CC) score.

## When to use

Use when a canopus_npc_summary table of predicted chemical classes per feature/extract must be reduced to high-confidence, recurring classes prior to novelty scoring (e.g., computing the CC score against LOTUS/NPClassifier reports).

## When NOT to use

- Input is a raw SIRIUS/CANOPUS project space that has not yet been summarized via canopus_treemap's npcSummary() — generate the summary table first.
- The analysis does not involve chemical-class novelty assessment against literature reports (e.g., only FC or SC components are needed) — class filtering is specific to the CC component.
- Class predictions come from a different ontology (e.g., GNPS molecular networking annotations rather than NPClassifier) — the CC score compares NPClassifier classes to LOTUS's NPClassifyre ontology.

## Inputs

- canopus_npc_summary.tsv — NPClassifier summary table produced by recomputing the SIRIUS project space with the canopus treemap package
- SIRIUS project space from the feature set (from MZmine2/MZmine3 quantification table via GNPS FBMN)
- Inventa configuration with min_class_confidence (0.8) and min_recurrence (5) parameters

## Outputs

- Per-sample set of filtered, high-confidence NPClassifier chemical classes used for CC-score computation
- CC components (CCs, CCg, CC in range 0–1) per extract written to a TSV results file

## How to apply

First generate the NPClassifier summary table by recomputing the SIRIUS project space with the canopus treemap package, calling C.npcSummary() and exporting it as a TSV/CSV (canopus_npc_summary.tsv). Then filter the predicted chemical classes using the configured min_class_confidence cutoff (0.8 in the Inventa configuration) to keep only classes predicted with high probability, and apply a recurrence threshold (min_recurrence = 5) so that only classes observed in multiple samples/ions are retained. The filtered class set per sample is then used downstream for set subtraction against species-level (CCs) and genus-level (CCg) reported classes derived from LOTUS-based literature data, each contributing 0.5 to the additive CC = CCs + CCg score. The filtering rationale is to prevent low-confidence or singleton class predictions from spuriously flagging an extract as containing a 'new' reported chemical class.

## Related tools

- **canopus_treemap** (Recomputes the SIRIUS project space and generates the NPClassifier summary table (npcSummary) that is subsequently filtered by confidence/recurrence) — https://github.com/kaibioinfo/canopus_treemap
- **SIRIUS** (Produces the molecular formula/structure project space whose CANOPUS class predictions are summarized and filtered)
- **CANOPUS** (Provides the chemical-class predictions that are filtered by confidence and recurrence before literature comparison)
- **Inventa** (Applies the min_class_confidence and min_recurrence filters within the Class Component (CC) calculation) — https://github.com/luigiquiros/inventa

## Examples

```
from canopus import Canopus; C = Canopus(sirius="sirius_projectspace"); C.npcSummary().to_csv("npc_summary.csv")
```

## Evaluation signals

- Validation case: the Pristimera indica roots extract must score CC = 1 for the dihydro-β-agarofuran sesquiterpenoids class (0.5 new at species level + 0.5 new at genus level).
- Filtered class set must be a strict subset of the full canopus_npc_summary predictions — no class below min_class_confidence = 0.8 or with recurrence below 5 survives.
- CC values must lie in the range 0–1 (CC = CCs + CCg, each component at most 0.5).
- The npcSummary output must contain NPClassifier (NPC) classes, not only CANOPUS classypoints, consistent with the NPClassifyre ontology used by LOTUS.

## Limitations

- Recomputing the SIRIUS project space may be unnecessary: 'support for NPC predictions exists since several months in CANOPUS. Thus, it is very likely that you do not have to recompute your projectspace.'
- Thresholds are user-configurable in src/inventa.py, so filtering behavior depends on the configuration passed at run time.
- Binder instances shut down after 15 min of inactivity, so parameters and results must be saved locally when running the pipeline in the cloud.

## Evidence

- [methods] CC score source (methods): [evidence span withheld — anchor status: not_found]
- [readme] npc summary generation (readme): [evidence span withheld — anchor status: external_doc]
- [readme] canopus file input (readme): "`canopus_npc_summary_filename`: Sirius CANOPUS recomputated output format."
- [readme] recompute project space (readme): "Recompute your project space from Sirius using the following code:"
- [abstract] validation finding (abstract): "The Pristimera indica (Willd.) A.C.Sm roots extract was highlighted as a promising source of potentially novel compounds."
