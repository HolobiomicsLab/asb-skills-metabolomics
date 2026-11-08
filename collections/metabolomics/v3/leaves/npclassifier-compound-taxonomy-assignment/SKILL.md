---
name: npclassifier-compound-taxonomy-assignment
description: Use when per-feature SIRIUS/CSI:FingerID candidate structures (2D SMILES)
  from LC-MS/MS DDA metabolomics data must be classified into natural-product chemical
  classes to enrich the knowledge graph for downstream SPARQL querying.
when_to_use_negative:
- When the goal is formula/structure prediction itself (e.g. assigning molecular formulas
  or candidate structures to MS2 spectra) — NPClassifier only classifies structures
  that have already been assigned; use SIRIUS/CSI:FingerID first.
- When samples have not been processed by MZmine feature detection and SIRIUS annotation,
  so no candidate SMILES exist yet.
- When the desired classification is CANOPUS chemical class prediction from spectra
  rather than structure-based NP taxonomy; CANOPUS is a separate step that does not
  require NPClassifier.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: NPClassifier
  role: Web API that assigns natural-product taxonomy (pathway/superclass/class) to
    each candidate structure's anisomeric SMILES produced by CSI:FingerID.
- name: SIRIUS/CSI:FingerID
  role: Upstream structural annotation tool producing the candidate 2D structures
    (SMILES) that NPClassifier then classifies.
- name: ENPKG (04_enpkg_sirius_canopus)
  role: Automated script that orchestrates NPClassifier API calls, collates its outputs
    with SIRIUS/CANOPUS annotations, and writes them into the sample RDF graph.
  repo: https://github.com/enpkg/enpkg_workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:38c7bf40faae0df8ea83216ea792d7a1c773fe72c2704553cf427cbca5536f9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - ENPKG
  - ENPKG (04_enpkg_sirius_canopus)
  - NPClassifier
  - SIRIUS/CSI:FingerID
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# npclassifier-compound-taxonomy-assignment

## Summary

Assigns natural-product chemical taxonomy to each candidate compound structure produced by the ENPKG SIRIUS/CSI:FingerID stage by submitting its anisomeric SMILES to the NPClassifier API, so that structural annotations in the knowledge graph carry a chemical-class dimension.

## When to use

Use when per-feature SIRIUS/CSI:FingerID candidate structures (2D SMILES) from LC-MS/MS DDA metabolomics data must be classified into natural-product chemical classes to enrich the knowledge graph for downstream SPARQL querying.

## When NOT to use

- When the goal is formula/structure prediction itself (e.g. assigning molecular formulas or candidate structures to MS2 spectra) — NPClassifier only classifies structures that have already been assigned; use SIRIUS/CSI:FingerID first.
- When samples have not been processed by MZmine feature detection and SIRIUS annotation, so no candidate SMILES exist yet.
- When the desired classification is CANOPUS chemical class prediction from spectra rather than structure-based NP taxonomy; CANOPUS is a separate step that does not require NPClassifier.

## Inputs

- Anisomeric SMILES strings of candidate 2D structures returned by SIRIUS/CSI:FingerID per feature (from the 'Export for SIRIUS' .mgf feature files of each sample)
- Per-sample feature annotation records from ENPKG script 04_enpkg_sirius_canopus (SIRIUS/CSI:FingerID and CANOPUS outputs)

## Outputs

- Per-feature NPClassifier natural-product taxonomy records (chemical class, superclass, pathway) linked to each candidate structure
- RDF triples in the sample's knowledge graph linking annotations via enpkg:has_sirius_annotation sub-properties, alongside the saved NPClassifier API parameters for reproducibility

## How to apply

Run this skill after the SIRIUS/CSI:FingerID structure annotation step (ENPKG script 04_enpkg_sirius_canopus) and after CANOPUS class predictions, on both PI and NI mode feature lists. For every feature that received candidate 2D structures from CSI:FingerID (10 ppm parent-mass tolerance, 10 candidates considered, adducts e.g. [M + H]+ and [M - H]-), extract the anisomeric SMILES of each candidate and submit them to the NPClassifier API to retrieve its natural-product taxonomy (e.g. chemical class and pathway-level categories). Collate the returned NPClassifier records with the per-feature SIRIUS/CSI:FingerID and CANOPUS annotations in the same processing script, saving all parameters and versions for reproducibility, and write them into the sample's RDF graph as sub-properties of enpkg:has_sirius_annotation (e.g. enpkg:has_sirius_annotation_00c339c3b183cb7fbf466b4d334dbef2). The rationale is that chemical-class labels let the knowledge graph connect structural annotations to chemotaxonomic and bioactivity patterns via SPARQL, complementing the ISDB-LOTUS annotations so that agreement across approaches strengthens confidence in spectral annotations.

## Related tools

- **NPClassifier** (Web API that assigns natural-product taxonomy (pathway/superclass/class) to each candidate structure's anisomeric SMILES produced by CSI:FingerID.)
- **SIRIUS/CSI:FingerID** (Upstream structural annotation tool producing the candidate 2D structures (SMILES) that NPClassifier then classifies.)
- **ENPKG (04_enpkg_sirius_canopus)** (Automated script that orchestrates NPClassifier API calls, collates its outputs with SIRIUS/CANOPUS annotations, and writes them into the sample RDF graph.) — https://github.com/enpkg/enpkg_workflow

## Examples

```
python 04_enpkg_sirius_canopus.py --input_dir <sample_dir> --sirius <path_to_sirius>  # includes NPClassifier API calls for each candidate SMILES
```

## Evaluation signals

- Every feature with an associated MS2 scan that received a CSI:FingerID candidate structure also carries a NPClassifier class label in the graph (no missing annotations for annotated structures).
- NPClassifier records appear in the RDF graph as sub-properties of enpkg:has_sirius_annotation (e.g. enpkg:has_sirius_annotation_00c339c3b183cb7fbf466b4d334dbef2) with retrievable tool versions/parameters.
- Candidate SMILES submitted to the API are anisomeric (stereochemistry-aware) and match the structures stored from the SIRIUS annotation step.
- Chemical-class annotations agree with CANOPUS and ISDB-LOTUS results where both exist, strengthening confidence in spectral annotations.

## Limitations

- NPClassifier classifies structures, not spectra, so it inherits any errors from the upstream SIRIUS/CSI:FingerID structure assignments.
- It relies on an external web API — network access and API availability are required, and the workflow saves parameters/versions at each step to keep runs reproducible.
- Coverage is limited to compounds recognizable within natural-product chemical space; structures outside this space may receive weak or uninformative class labels.

## Evidence

- [methods] structural annotation step feeding NPClassifier: [evidence span withheld — anchor status: not_found]
- [methods] ENPKG graph integration: [evidence span withheld — anchor status: not_found]
- [results] confidence via cross-approach agreement: [evidence span withheld — anchor status: not_found]
- [methods] upstream SIRIUS parameters: [evidence span withheld — anchor status: not_found]
- [readme] NPClassifier role in ENPKG workflow: [evidence span withheld — anchor status: not_found]
