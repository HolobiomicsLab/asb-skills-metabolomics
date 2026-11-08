---
name: positive-ion-mode-spectra-handling
description: Use when re-analyzing positive ion mode LC–MS/MS metabolomics data and
  you need MZmine3 feature tables plus SIRIUS formula/adduct assignments (e.g., replicating
  the cultured-vs-conventional meat study).
when_to_use_negative:
- The acquisition or evaluation concerns negative ion mode data — the article's meat-study
  evaluation explicitly focuses on the positive ion mode data only.
- The feature table already carries SIRIUS formula/adduct assignments; re-running
  assignment would be redundant and could shift the expected 866/900 count.
- The goal is ranking candidate structures for individual or consensus spectra (MVP/MassSpecGym
  retrieval) rather than producing feature-level formula/adduct assignments.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MZmine3
  role: Feature extraction and representative spectra selection on positive ion mode
    LC–MS/MS data; exports the feature table consumed by SIRIUS
- name: SIRIUS v6.1.0
  role: Assigns a molecular formula and adduct to each MZmine3 feature using common
    positive-ion adducts; its parsed output forms the per-feature assignment table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:0d5c325d6fddcc9c37c62fab3ec04b481057a1da3b1527d33765ca0f13ca9288
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MVP
  - MZmine3
  - SIRIUS v6.1.0
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# positive-ion-mode-spectra-handling

## Summary

Skill for the article's positive ion mode LC–MS/MS handling chain: MZmine3 feature extraction and representative spectra selection, followed by SIRIUS v6.1.0 formula and adduct assignment, turning raw mzML into a per-feature assignment table (866 of 900 meat-study features assigned).

## When to use

Use when re-analyzing positive ion mode LC–MS/MS metabolomics data and you need MZmine3 feature tables plus SIRIUS formula/adduct assignments (e.g., replicating the cultured-vs-conventional meat study).

## When NOT to use

- The acquisition or evaluation concerns negative ion mode data — the article's meat-study evaluation explicitly focuses on the positive ion mode data only.
- The feature table already carries SIRIUS formula/adduct assignments; re-running assignment would be redundant and could shift the expected 866/900 count.
- The goal is ranking candidate structures for individual or consensus spectra (MVP/MassSpecGym retrieval) rather than producing feature-level formula/adduct assignments.

## Inputs

- Positive ion mode LC–MS/MS data in mzML format (the meat-study data are on the Metabolomics Workbench)
- MZmine3 feature list / feature table with representative MS/MS spectra per feature, exported from the upstream MZmine3 feature-extraction step

## Outputs

- Single per-feature assignment table keyed by MZmine3 feature id, with a molecular formula and an adduct for each processed feature
- Hit-rate count of features with both formula and adduct assigned (866 of 900 in the meat replication)

## How to apply

Obtain the LC–MS/MS data in mzML format (for the meat study, from the Metabolomics Workbench) and, following the article, restrict the evaluation to the positive ion mode data. Run MZmine3 for feature extraction and representative spectra selection, exporting a feature table in which each feature carries a representative MS/MS spectrum. Launch SIRIUS v6.1.0 with default formula and adduct assignment settings so common positive-ion adducts (e.g., [M+H]+) are considered; keep the version pinned, since these assignments feed downstream candidate retrieval within a 0.01 Da tolerance against FoodDB, HMDB, MassSpecGym, NIST23, and the GNPS Multiplex library. Parse the SIRIUS output into a single table keyed by MZmine3 feature id, recording both the molecular formula and the adduct per feature. Validate by counting the features for which SIRIUS returned both assignments — in the meat replication this must equal 866 of the 900 detected features, the hit rate reported for this stage.

## Related tools

- **MZmine3** (Feature extraction and representative spectra selection on positive ion mode LC–MS/MS data; exports the feature table consumed by SIRIUS)
- **SIRIUS v6.1.0** (Assigns a molecular formula and adduct to each MZmine3 feature using common positive-ion adducts; its parsed output forms the per-feature assignment table)

## Evaluation signals

- The compiled assignment table contains exactly one record per MZmine3 feature id, with no duplicate or orphan feature entries
- The count of features for which SIRIUS returned both a molecular formula and an adduct equals 866 out of the 900 detected features (meat replication validation)
- Only positive ion mode mzML files/features are included in the run; no negative mode data leaks in
- SIRIUS version is pinned to v6.1.0 with default formula/adduct settings, so reruns reproduce the same assignment set
- Each assigned feature's formula+adduct neutral mass is consistent with the feature mass within the 0.01 Da tolerance used for downstream candidate retrieval

## Limitations

- The original meat study provided neither processed spectra nor a mapping to identified metabolites, so the 866/900 result is a replication hit rate rather than a direct comparison
- Formula/adduct assignments (and downstream MVP annotations) are putative and require experimental validation
- Scope is positive ion mode only; negative-mode features from the same samples are not handled by this skill

## Evidence

- [results] Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0: "Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0"
- [results] For this evaluation, we focus on the positive ion mode data: "For this evaluation, we focus on the positive ion mode data"
- [results] LC–MS/MS data in mzML format are publicly available on the Metabolomics Workbench: "LC–MS/MS data in mzML format are publicly available on the Metabolomics Workbench"
- [discussion] a study that compares metabolic profiles between cultured and conventional meat: "a study that compares metabolic profiles between cultured and conventional meat"
