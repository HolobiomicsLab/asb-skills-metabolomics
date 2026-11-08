---
name: sirius-commandline-workflow
description: Use when an MZmine3 feature table with representative MS/MS spectra must
  receive molecular formula and adduct annotations before downstream candidate retrieval,
  especially for positive-ion-mode metabolomics data where the original study provides
  no processed spectra.
when_to_use_negative:
- The goal is ranking molecular candidate structures against spectra — that is the
  downstream MVP retrieval step, not SIRIUS formula/adduct assignment.
- The feature table already carries formula/adduct annotations from a prior SIRIUS
  run and only needs candidate retrieval or classification (e.g. NPClassifier).
- Data are negative ion mode or otherwise outside the positive-ion common-adduct assumptions
  used in this workflow's default settings.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS v6.1.0
  role: Performs the formula and adduct assignment on MZmine3-extracted features using
    default positive-ion adduct settings
- name: MZmine3
  role: Upstream feature extraction and representative spectra selection that produces
    the feature table consumed by SIRIUS
- name: MVP
  role: Downstream multiview contrastive model that consumes the annotated features
    for molecular candidate ranking
  repo: github.com/HassounLab/MVP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:5fb09187ca6190327055adddec6afef1faa8a11ca84e3d73c33082fadd2f0b66
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MVP
  - MZmine3
  - SIRIUS v6.1.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sirius-commandline-workflow

## Summary

Runs SIRIUS v6.1.0 formula and adduct assignment on an MZmine3-derived LC–MS/MS feature table (representative spectra, positive ion mode) with default settings, producing per-feature formula/adduct records and an assignment hit rate (e.g. 866 of 900 features in the cultured-vs-conventional meat replication).

## When to use

Use when an MZmine3 feature table with representative MS/MS spectra must receive molecular formula and adduct annotations before downstream candidate retrieval, especially for positive-ion-mode metabolomics data where the original study provides no processed spectra.

## When NOT to use

- The goal is ranking molecular candidate structures against spectra — that is the downstream MVP retrieval step, not SIRIUS formula/adduct assignment.
- The feature table already carries formula/adduct annotations from a prior SIRIUS run and only needs candidate retrieval or classification (e.g. NPClassifier).
- Data are negative ion mode or otherwise outside the positive-ion common-adduct assumptions used in this workflow's default settings.

## Inputs

- MZmine3 feature table with representative MS/MS spectra per feature (positive ion mode)
- Positive-ion-mode LC–MS/MS data in mzML format (upstream of MZmine3)

## Outputs

- Per-feature table of molecular formula and adduct assignments keyed by feature id
- Assignment hit-rate count (features with both formula and adduct assigned, e.g. 866/900)

## How to apply

Start from the MZmine3 feature table (feature list with representative spectra) exported by the upstream feature-extraction step, restricted to positive ion mode data if following the meat metabolomics evaluation. Launch SIRIUS v6.1.0 with default formula and adduct assignment settings, allowing SIRIUS to consider common positive-ion adducts for the positive mode spectra. Parse the SIRIUS output and compile per-feature formula and adduct assignment records into a single table keyed by feature id. Count the features for which SIRIUS returned both a molecular formula and an adduct assignment; in the replicated meat workflow this was 866 of the 900 MZmine3-detected features. This step precedes candidate structure retrieval (within 0.01 Da tolerance from FoodDB, HMDB, MassSpecGym, NIST23, GNPS Multiplex) in the overall annotation pipeline.

## Related tools

- **SIRIUS v6.1.0** (Performs the formula and adduct assignment on MZmine3-extracted features using default positive-ion adduct settings)
- **MZmine3** (Upstream feature extraction and representative spectra selection that produces the feature table consumed by SIRIUS)
- **MVP** (Downstream multiview contrastive model that consumes the annotated features for molecular candidate ranking) — github.com/HassounLab/MVP

## Evaluation signals

- The count of features with both a molecular formula and an adduct assignment matches the expected 866 out of 900 MZmine3-detected features.
- Every processed feature row in the compiled table has a formula/adduct record keyed by feature id (no dangling or missing ids).
- Assigned adducts are limited to common positive-ion adducts consistent with positive mode acquisition.
- The feature id space of the SIRIUS output matches the MZmine3 feature list row-for-row, allowing 1:1 joining.

## Limitations

- The original meat study provided neither processed spectra nor a mapping to identified metabolites, so the 866/900 result is a replication-level validation rather than a comparison against published annotations.
- 34 of 900 features received no formula/adduct assignment, so downstream candidate retrieval covers only the assigned subset.
- Assignment quality depends on the upstream MZmine3 feature extraction and representative spectra selection; SIRIUS default settings were not tuned in this replication.

## Evidence

- [results] Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0: "Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0"
- [results] For this evaluation, we focus on the positive ion mode data: "For this evaluation, we focus on the positive ion mode data"
- [results] Since neither the processed spectra nor the mapping to the identified metabolites was provided, we replicated the study: "Since neither the processed spectra nor the mapping to the identified metabolites was provided, we replicated the study"
