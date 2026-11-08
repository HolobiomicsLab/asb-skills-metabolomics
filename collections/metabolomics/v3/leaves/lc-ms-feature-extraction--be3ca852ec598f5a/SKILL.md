---
name: lc-ms-feature-extraction
description: Use when starting from publicly deposited LC–MS/MS mzML raw files (e.g.
  Metabolomics Workbench) and you need chromatographic features with linked representative
  MS2 spectra as a feature table for downstream annotation.
when_to_use_negative:
- You already have a processed feature table — this skill operates on raw mzML files,
  not on exported feature lists
- The data are negative ion mode or non-metabolomics MS runs — the article's evaluation
  explicitly focuses on positive ion mode data
- You are benchmarking annotation algorithms on MassSpecGym spectra rather than extracting
  features from deposited study raw files
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine3
  role: Performs mass detection, chromatogram building, feature detection/deconvolution,
    and representative spectra selection on the mzML files
- name: SIRIUS v6.1.0
  role: Downstream formula and adduct assignment applied to the MZmine3 feature list
- name: MVP
  role: Consumes the resulting features/representative spectra for molecular candidate
    ranking in the metabolite annotation pipeline
  repo: https://github.com/HassounLab/MVP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:b06914ddb99022e40d77fbc16e75b7f8d9e0cd97f083b1756c7a2d06421ead5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/MVP
  tools:
  - MVP
  - MZmine3
  - SIRIUS v6.1.0
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lc-ms-feature-extraction

## Summary

Extracts chromatographic features (m/z + retention time) with representative MS2 spectra from deposited LC–MS/MS mzML files using MZmine3, replicating the cultured-vs-conventional meat metabolomics study (~900 features).

## When to use

Use when starting from publicly deposited LC–MS/MS mzML raw files (e.g. Metabolomics Workbench) and you need chromatographic features with linked representative MS2 spectra as a feature table for downstream annotation.

## When NOT to use

- You already have a processed feature table — this skill operates on raw mzML files, not on exported feature lists
- The data are negative ion mode or non-metabolomics MS runs — the article's evaluation explicitly focuses on positive ion mode data
- You are benchmarking annotation algorithms on MassSpecGym spectra rather than extracting features from deposited study raw files

## Inputs

- LC–MS/MS raw files in mzML format from the Metabolomics Workbench public deposit (cultured vs conventional meat study)
- Positive ion mode mzML files for import into MZmine3

## Outputs

- feature_table.csv containing m/z, retention time, and linked representative MS2 spectrum per feature
- Detected feature count (~900 features for the meat study data)
- Optional: SIRIUS v6.1.0 formula and adduct assignments per feature (866 of 900 features)

## How to apply

Download the LC–MS/MS mzML raw files from the Metabolomics Workbench deposit and import the positive ion mode files into MZmine3 (this replication explicitly focuses on positive ion mode data). Run mass detection, then chromatogram building to construct extracted ion chromatograms, and execute MZmine3 feature detection/deconvolution to obtain features characterized by m/z and retention time. Apply MZmine3's representative spectra selection module to assign a single representative MS2 spectrum to each feature, then export the feature list (m/z, retention time, and linked representative MS2 spectra) to feature_table.csv. Downstream, formula and adduct assignment can follow with SIRIUS v6.1.0; in the meat study replication ~900 features were detected, of which 866 received formula/adduct assignments from SIRIUS.

## Related tools

- **MZmine3** (Performs mass detection, chromatogram building, feature detection/deconvolution, and representative spectra selection on the mzML files)
- **SIRIUS v6.1.0** (Downstream formula and adduct assignment applied to the MZmine3 feature list)
- **MVP** (Consumes the resulting features/representative spectra for molecular candidate ranking in the metabolite annotation pipeline) — https://github.com/HassounLab/MVP

## Evaluation signals

- feature_table.csv exists and parses as valid CSV
- Detected feature count matches the ~900 features reported for the meat study (866 of which received SIRIUS formula/adduct assignments)
- Every feature in the table has m/z, retention time, and exactly one linked representative MS2 spectrum
- Input mzML files were confirmed to come from the Metabolomics Workbench deposit and only positive ion mode files were imported
- The exported table supports downstream candidate retrieval within the 0.01 Da tolerance used by the annotation pipeline

## Limitations

- Neither the processed spectra nor the mapping to the identified metabolites was provided by the original meat study, so the feature count is a replication result rather than a shared reference
- Annotations attached downstream (e.g. via MVP/SIRIUS) are putative and require experimental validation
- The reported evaluation focuses on positive ion mode data only

## Evidence

- [results] Feature extraction and representative spectra selection are performed using MZmine3: "Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0"
- [results] Applying the MZmine3-based workflow to the meat study data detects 900 features: [evidence span withheld — anchor status: not_found]
- [results] LC–MS/MS data in mzML format are publicly available on the Metabolomics Workbench: "LC–MS/MS data in mzML format are publicly available on the Metabolomics Workbench"
- [results] For this evaluation, we focus on the positive ion mode data: "For this evaluation, we focus on the positive ion mode data"
- [results] Since neither the processed spectra nor the mapping to the identified metabolites was provided, we replicated the study: "Since neither the processed spectra nor the mapping to the identified metabolites was provided, we replicated the study"
