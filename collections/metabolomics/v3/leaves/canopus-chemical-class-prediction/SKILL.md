---
name: canopus-chemical-class-prediction
description: Reference for annotating per-feature chemical classes from DDA MS2 spectra
  after MZmine feature detection, when structural annotation (SIRIUS/CSI:FingerID)
  and class taxonomy (CANOPUS, NPClassifier) are needed in an RDF workflow.
when_to_use_negative:
- Input is already a curated structure-to-class mapping (e.g. a LOTUS/ISDB-LOTUS annotation
  table) rather than raw MS2 spectra needing class prediction
- The workflow needs RT-based feature alignment across batches; CANOPUS class prediction
  is per-sample/feature and does not solve alignment
- Features were not processed with MZmine or lack an associated MS2 scan; class prediction
  from spectra alone without the ENPKG export step is not covered
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: CANOPUS
  role: Predicts chemical classes of LC-MS/MS features from MS2 spectra, run on both
    ionization modes without additional parameters
- name: NPClassifier
  role: API that maps the anisomeric SMILES of candidate structures to natural-product
    taxonomy labels
- name: SIRIUS / CSI:FingerID
  role: Upstream step providing formula and candidate 2D structures (with ZODIAC re-ranking)
    whose SMILES feed NPClassifier
- name: ENPKG
  role: Workflow collating per-feature SIRIUS/CSI:FingerID and CANOPUS annotations
    and storing them in the RDF knowledge graph
  repo: https://github.com/enpkg/enpkg_workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:87be96561c0afcf9aadb6c15dba2610742f1710b18b4f83095ebf3dc3a0a80c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - CANOPUS
  - ENPKG
  - NPClassifier
  - SIRIUS / CSI:FingerID
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# CANOPUS chemical class prediction

## Summary

Predicts NPC chemical classes for LC-MS/MS features by running CANOPUS within the ENPKG in-silico structural annotation stage, then mapping the candidate 2D structures' anisomeric SMILES through the NPClassifier API so both class schemes are stored per feature in the knowledge graph.

## When to use

Reference for annotating per-feature chemical classes from DDA MS2 spectra after MZmine feature detection, when structural annotation (SIRIUS/CSI:FingerID) and class taxonomy (CANOPUS, NPClassifier) are needed in an RDF workflow.

## When NOT to use

- Input is already a curated structure-to-class mapping (e.g. a LOTUS/ISDB-LOTUS annotation table) rather than raw MS2 spectra needing class prediction
- The workflow needs RT-based feature alignment across batches; CANOPUS class prediction is per-sample/feature and does not solve alignment
- Features were not processed with MZmine or lack an associated MS2 scan; class prediction from spectra alone without the ENPKG export step is not covered

## Inputs

- Per-sample 'Export for SIRIUS' files (.mgf) with feature MS1 isotopic patterns and MS2 spectra exported by MZmine
- MZmine feature list with associated MS2 scans and RT/mz metadata
- Anisomeric SMILES of candidate 2D structures from CSI:FingerID

## Outputs

- Per-feature CANOPUS chemical class annotations
- Per-feature NPClassifier natural-product taxonomy labels
- Per-feature SIRIUS/CSI:FingerID formula and structure candidates (with ZODIAC re-ranking)
- RDF sub-properties (e.g. enpkg:has_sirius_annotation_*) carrying version and parameter provenance in the knowledge graph

## How to apply

After MZmine exports per-sample 'Export for SIRIUS' files containing MS1 isotopic patterns and MS2 spectra, run SIRIUS with ZODIAC and CSI:FingerID against biological databases (PI mode with SIRIUS v4.9.12 and adducts [M + H]+, [M + NH4]+, [M + K]+, [M + Na]+; NI mode with SIRIUS v5.5.4 and adducts [M – H]−, [M + K-2H]−, [M + Na-2H]−), using 10 ppm tolerance, max m/z 800, and 10 candidates. Then run CANOPUS on the feature spectra of both ionization modes (no additional parameters needed) to predict chemical classes directly from the spectra. In parallel, submit the anisomeric SMILES of the candidate 2D structures returned by CSI:FingerID to the NPClassifier API to retrieve their natural-product taxonomy. Collate the per-feature SIRIUS/CSI:FingerID and CANOPUS annotation records with the ENPKG script at 04_enpkg_sirius_canopus, storing everything in the RDF graph so the class annotations sit alongside ISDB-LOTUS annotations and version/parameter provenance.

## Related tools

- **CANOPUS** (Predicts chemical classes of LC-MS/MS features from MS2 spectra, run on both ionization modes without additional parameters)
- **NPClassifier** (API that maps the anisomeric SMILES of candidate structures to natural-product taxonomy labels)
- **SIRIUS / CSI:FingerID** (Upstream step providing formula and candidate 2D structures (with ZODIAC re-ranking) whose SMILES feed NPClassifier)
- **ENPKG** (Workflow collating per-feature SIRIUS/CSI:FingerID and CANOPUS annotations and storing them in the RDF knowledge graph) — https://github.com/enpkg/enpkg_workflow

## Examples

```
sirius --input <sample>_Export_for_SIRIUS.mgf --adducts '[M+H]+,[M+NH4]+,[M+K]+,[M+Na]+' --zodiac --canopus --csi:fingerid --db biological --ppm 10 --max-mz 800
```

## Evaluation signals

- Every feature with an associated MS2 scan should carry formula and structure candidates within the 10 ppm parent-mass tolerance
- CANOPUS class predictions should be present for features in both PI and NI modes with no additional parameters required
- NPClassifier labels should be retrievable for every candidate 2D structure's anisomeric SMILES
- Each annotation record must keep run versions/parameters as sub-properties (e.g. enpkg:has_sirius_annotation_00c339c3b183cb7fbf466b4d334dbef2) for reproducibility
- Cross-check CANOPUS/NPClassifier classes against ISDB-LOTUS annotations; agreement between the two approaches strengthens confidence

## Limitations

- CANOPUS is parameterized as parameter-free within ENPKG, so no user-tunable thresholds are exposed; quality relies on upstream MZmine export quality
- Class labels depend on CSI:FingerID candidate structures, so annotation confidence inherits the 10-candidate / 10 ppm / max m/z 800 constraints
- Adduct coverage is restricted to the listed PI ([M + H]+, [M + NH4]+, [M + K]+, [M + Na]+) and NI ([M – H]−, [M + K-2H]−, [M + Na-2H]−) sets per SIRIUS version (v4.9.12 for PI, v5.5.4 for NI); other adducts are not covered
- No false-positive/false-negative rate for the structural/class annotations is reported; confidence is qualitative (agreement between ISDB-LOTUS and SIRIUS/CANOPUS)

## Evidence

- [results] CANOPUS run details: [evidence span withheld — anchor status: not_found]
- [results] NPClassifier input: [evidence span withheld — anchor status: not_found]
- [results] provenance: [evidence span withheld — anchor status: not_found]
- [methods] SIRIUS parameters: [evidence span withheld — anchor status: not_found]
- [intro] workflow role: "Each sample is considered individually for taxonomic metadata standardization, feature detection, structural annotation, and FBMN."
- [results] complementary confidence: [evidence span withheld — anchor status: not_found]
- [readme] ENPKG script: [evidence span withheld — anchor status: not_found]
