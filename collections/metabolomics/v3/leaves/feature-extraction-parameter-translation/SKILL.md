---
name: feature-extraction-parameter-translation
description: Use when raw LC-MS-based metabolomics data (any sample type, acquisition
  mode, or MS vendor) must be processed for feature extraction but default tool parameters
  would under-extract true positive metabolic features.
when_to_use_negative:
- Input is already a feature table produced by a prior feature extraction run — Paramounter
  measures parameters from raw LC-MS data before extraction, not from processed feature
  tables.
- The downstream feature extraction program is not among the software-specific parameter
  mappings Paramounter supports (article context centers on MS-DIAL-based feature
  extraction and XCMS CentWave mzdiff).
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: Paramounter
  role: R program that automatically measures universal parameters from raw LC-MS-based
    metabolomics data and translates them into software-specific parameters
  repo: https://github.com/HuanLab/Paramounter
- name: R
  role: implementation language of Paramounter; run via R/Rscript to measure and translate
    parameters
- name: MS-DIAL
  role: downstream feature extraction program whose parameters are set from Paramounter's
    translated universal parameters; comparison baseline for the threefold extraction
    gain
- name: XCMS CentWave
  role: feature extractor whose mzdiff dereplication mass tolerance is a translated
    universal parameter (suggested 0.001 or 0.01; negative value disables dereplication)
- name: AutoTuner
  role: alternative parameter-tuning tool used as benchmark; Paramounter yields 3.7-fold
    more true positive features
- name: IPO
  role: alternative parameter-tuning tool used as benchmark; Paramounter yields 1.6-fold
    more true positive features
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:2ef8c91a2fbbe178c780901166909372eb193e347eafc3ae82df661aed21333e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - AutoTuner
  - IPO
  - MS-DIAL
  - Paramounter
  - R
  - XCMS CentWave
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# feature-extraction-parameter-translation

## Summary

Automatically measure universal LC-MS feature-extraction parameters (mass tolerance, peak height, peak width, instrumental shift) directly from raw metabolomics data using the R package Paramounter, then translate them into software-specific parameters (e.g. MS-DIAL) so feature extraction runs in a transparent 'white box' instead of relying on vendor defaults.

## When to use

Use when raw LC-MS-based metabolomics data (any sample type, acquisition mode, or MS vendor) must be processed for feature extraction but default tool parameters would under-extract true positive metabolic features.

## When NOT to use

- Input is already a feature table produced by a prior feature extraction run — Paramounter measures parameters from raw LC-MS data before extraction, not from processed feature tables.
- The downstream feature extraction program is not among the software-specific parameter mappings Paramounter supports (article context centers on MS-DIAL-based feature extraction and XCMS CentWave mzdiff).

## Inputs

- raw LC-MS-based metabolomics data files (pre-feature-extraction)

## Outputs

- universal parameters: mass tolerance, peak height, peak width, instrumental shift
- software-specific parameters for downstream feature extraction (e.g. MS-DIAL)
- extracted metabolic features

## How to apply

Run Paramounter, an R program available from the HuanLab GitHub repository, on the raw LC-MS data files to measure four universal parameters automatically: mass tolerance, peak height, peak width, and instrumental shift. Translate these universal parameters into the software-specific equivalents of the downstream feature extraction program (e.g. MS-DIAL-based extraction); the article reports this yields a threefold increase in extracted metabolites versus MS-DIAL default parameters, and 3.7-fold/1.6-fold more true positive features than AutoTuner/IPO respectively. Two decision points matter: (1) mass tolerance maps to mzdiff, used to dereplicate XCMS CentWave features by similar m/z and retention time — suggested defaults are 0.001 or 0.01, but true positives with smaller mass differences can be removed, so set mzdiff to any negative value to disable dereplication; (2) the optimized peak height maximizes true positives but inflates false positives and can crash software, so a higher threshold (e.g. 2X the optimized value) can be substituted when needed.

## Related tools

- **Paramounter** (R program that automatically measures universal parameters from raw LC-MS-based metabolomics data and translates them into software-specific parameters) — https://github.com/HuanLab/Paramounter
- **R** (implementation language of Paramounter; run via R/Rscript to measure and translate parameters)
- **MS-DIAL** (downstream feature extraction program whose parameters are set from Paramounter's translated universal parameters; comparison baseline for the threefold extraction gain)
- **XCMS CentWave** (feature extractor whose mzdiff dereplication mass tolerance is a translated universal parameter (suggested 0.001 or 0.01; negative value disables dereplication))
- **AutoTuner** (alternative parameter-tuning tool used as benchmark; Paramounter yields 3.7-fold more true positive features)
- **IPO** (alternative parameter-tuning tool used as benchmark; Paramounter yields 1.6-fold more true positive features)

## Examples

```
Rscript Paramounter.R --input raw_lcms_data.mzML --output universal_params.csv
```

## Evaluation signals

- Verify all four universal parameters (mass tolerance, peak height, peak width, instrumental shift) are reported for the input dataset.
- Confirm the translated software-specific parameters are produced for the target feature extraction program (e.g. MS-DIAL).
- Benchmark the extracted feature count against MS-DIAL default parameters — expect roughly a threefold increase in extracted metabolites per the article.
- Count true positive features; expect ~3.7-fold more than AutoTuner and ~1.6-fold more than IPO on comparable data.
- If using mzdiff default (0.001 or 0.01), check whether true positives with mass differences below the tolerance were lost; setting mzdiff negative should retain them.

## Limitations

- The optimized peak height that maximizes true positive features also raises the false positive rate and can cause software crashes; users may need a higher threshold (e.g. 2X the optimized value).
- Dereplication via mzdiff (0.001 or 0.01) can mistakenly remove true positive features with smaller mass differences; mzdiff must be set negative to disable dereplication.
- Validation on 11 datasets covering different sample types, data acquisition modes, and MS vendors demonstrated robustness, but performance outside those covered combinations is not described.

## Evidence
