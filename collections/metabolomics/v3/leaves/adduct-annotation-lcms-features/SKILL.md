---
name: adduct-annotation-lcms-features
description: Use when you have an MZmine3 feature table (or equivalent feature list
  with representative MS/MS spectra) from positive ion mode LC–MS/MS data and need
  per-feature molecular formula and adduct assignments before candidate structure
  retrieval.
when_to_use_negative:
- Input is already a SIRIUS-annotated feature table — re-running adduct annotation
  would duplicate the 866/900 assignment step
- Data is negative ion mode — the article explicitly focuses adduct assignment on
  positive ion mode data and common positive-ion adducts
- Goal is candidate structure retrieval or metabolite classification — use the downstream
  0.01 Da candidate retrieval and NPClassifier steps instead of re-annotating adducts
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS v6.1.0
  role: Assigns molecular formulae and adducts to each MZmine3 feature using default
    settings for positive ion mode data
- name: MZmine3
  role: Upstream feature extraction tool producing the feature table and representative
    spectra that are input to this annotation step
- name: MVP
  role: Downstream multiview contrastive annotation framework that consumes the formula/adduct-annotated
    features to rank molecular candidates
  repo: https://github.com/HassounLab/MVP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:675a8c17744cd041eb95a400c6cc323a29bbcb37791cc111889d8f8a28965459
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/MVP
  tools:
  - MVP
  - MZmine3
  - SIRIUS v6.1.0
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# adduct-annotation-lcms-features

## Summary

Assign molecular formulae and ion adducts to LC–MS/MS features after feature extraction, using SIRIUS v6.1.0 on positive ion mode MZmine3 output; this step converts raw feature lists into formula/adduct-annotated rows that feed candidate structure retrieval.

## When to use

Use when you have an MZmine3 feature table (or equivalent feature list with representative MS/MS spectra) from positive ion mode LC–MS/MS data and need per-feature molecular formula and adduct assignments before candidate structure retrieval.

## When NOT to use

- Input is already a SIRIUS-annotated feature table — re-running adduct annotation would duplicate the 866/900 assignment step
- Data is negative ion mode — the article explicitly focuses adduct assignment on positive ion mode data and common positive-ion adducts
- Goal is candidate structure retrieval or metabolite classification — use the downstream 0.01 Da candidate retrieval and NPClassifier steps instead of re-annotating adducts

## Inputs

- MZmine3 feature table / feature list with representative MS/MS spectra exported from upstream feature extraction (positive ion mode LC–MS/MS, mzML-derived)
- SIRIUS v6.1.0 tool configuration (default formula and adduct assignment settings)

## Outputs

- Per-feature table keyed by feature id with assigned molecular formula and adduct
- Count of features that received both a formula and an adduct assignment (e.g. 866 of 900 in the paper's meat study)

## How to apply

Start from the MZmine3 feature list containing representative spectra exported from upstream feature extraction (in the paper's meat metabolomics workflow, MZmine3 detected 900 features). Launch SIRIUS v6.1.0 with default formula and adduct assignment settings, allowing it to consider common positive-ion adducts appropriate for positive ion mode data. Parse the SIRIUS output and compile per-feature formula/adduct assignment records into a single table keyed by feature id. Rationale: formula and adduct assignment is the prerequisite step before candidate structure retrieval, which is performed within a 0.01 Da tolerance from FoodDB, HMDB, MassSpecGym, NIST23, and the GNPS Multiplex library, so features lacking a formula/adduct cannot proceed to candidate ranking. The count of features with both a molecular formula and an adduct assignment is a direct quality metric for the step.

## Related tools

- **SIRIUS v6.1.0** (Assigns molecular formulae and adducts to each MZmine3 feature using default settings for positive ion mode data)
- **MZmine3** (Upstream feature extraction tool producing the feature table and representative spectra that are input to this annotation step)
- **MVP** (Downstream multiview contrastive annotation framework that consumes the formula/adduct-annotated features to rank molecular candidates) — https://github.com/HassounLab/MVP

## Examples

```
sirius --input feature_table.mgf --output sirius_results --mode pos --formula --adduct
```

## Evaluation signals

- Number of features receiving both a formula and an adduct equals the expected value (866 out of 900 in the paper's meat metabolomics replication)
- Every feature row in the compiled table has a non-empty molecular formula and a non-empty adduct assignment
- The compiled per-feature table is keyed consistently by feature id and joins cleanly to the upstream MZmine3 feature list
- Adduct assignments are consistent with positive ion mode (only common positive-ion adducts appear)
- Downstream candidate retrieval (0.01 Da tolerance) can be executed for all annotated features without missing-formula errors

## Limitations

- Formula and adduct assignments are putative and require experimental validation
- In the paper's meat study, neither the processed spectra nor the mapping to identified metabolites was available, so the formula/adduct assignment had to be replicated from raw data rather than validated against ground truth
- Peak labels (and by extension formula/adduct interpretation) can be inaccurate; the article notes representation quality could benefit from more accurate peak labeling

## Evidence

- [results] Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0: "Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0"
- [results] Since neither the processed spectra nor the mapping to the identified metabolites was provided, we replicated the study: "Since neither the processed spectra nor the mapping to the identified metabolites was provided, we replicated the study"
- [results] The reported annotations are putative and require experimental validation.: "The reported annotations are putative and require experimental validation."
- [results] For this evaluation, we focus on the positive ion mode data: "For this evaluation, we focus on the positive ion mode data"
- [results] Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library: "Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library"
