---
name: msdial-feature-extraction
description: Use when raw LC-MS metabolomics data need to be converted into a metabolite
  feature table via MS-DIAL and you want data-derived parameters rather than MS-DIAL
  defaults to maximize the number of true positive features.
when_to_use_negative:
- Input is already a processed feature table rather than raw LC-MS data — Paramounter
  measures parameters from raw LC-MS-based metabolomics data prior to feature extraction,
  so there is nothing to tune.
- The goal is feature extraction with XCMS CentWave rather than MS-DIAL — the mzdiff
  dereplication and peak-height caveats described here apply to XCMS CentWave features;
  for MS-DIAL the skill specifically targets the threefold increase over default parameters.
- Data were not acquired with MS vendors/ acquisition modes covered by Paramounter's
  validation set of 11 datasets — robustness across sample types, acquisition modes,
  and vendors was demonstrated on that set, so applying it outside this scope is untested.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Paramounter
  role: R program that automatically measures the universal parameters from the raw
    LC-MS data and translates them into MS-DIAL-specific settings for feature extraction
  repo: https://github.com/HuanLab/Paramounter
- name: MS-DIAL
  role: performs the feature extraction on the raw LC-MS data, run once with Paramounter-derived
    parameters and once with default parameters to quantify the improvement
- name: R
  role: language in which Paramounter is implemented; used to run the parameter measurement
    step
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:ce32eda4c38b02309baaf35a545b86350f50bf54b9cc98b882afec20f9c609d3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - MS-DIAL
  - Paramounter
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# msdial-feature-extraction

## Summary

Running MS-DIAL feature extraction on raw LC-MS metabolomics data with parameters derived by Paramounter, which measures universal parameters (mass tolerance, peak height, peak width, instrumental shift) from the raw data and translates them into MS-DIAL-specific settings; applying these tuned parameters gives a threefold increase in extracted metabolites compared to MS-DIAL defaults.

## When to use

Use when raw LC-MS metabolomics data need to be converted into a metabolite feature table via MS-DIAL and you want data-derived parameters rather than MS-DIAL defaults to maximize the number of true positive features.

## When NOT to use

- Input is already a processed feature table rather than raw LC-MS data — Paramounter measures parameters from raw LC-MS-based metabolomics data prior to feature extraction, so there is nothing to tune.
- The goal is feature extraction with XCMS CentWave rather than MS-DIAL — the mzdiff dereplication and peak-height caveats described here apply to XCMS CentWave features; for MS-DIAL the skill specifically targets the threefold increase over default parameters.
- Data were not acquired with MS vendors/ acquisition modes covered by Paramounter's validation set of 11 datasets — robustness across sample types, acquisition modes, and vendors was demonstrated on that set, so applying it outside this scope is untested.

## Inputs

- raw LC-MS-based metabolomics data files (comparison dataset referenced in the article's MS-DIAL comparison)
- Paramounter R package from https://github.com/HuanLab/Paramounter

## Outputs

- extracted metabolite feature counts for MS-DIAL run with Paramounter-derived parameters (cond_paramounter_msdial)
- extracted metabolite feature counts for MS-DIAL run with default parameters (cond_default_msdial)
- CSV table listing extracted-feature counts side by side with the computed fold increase (~3x)

## How to apply

Obtain the raw LC-MS files and the Paramounter R package from the HuanLab/Paramounter GitHub deposit. First run Paramounter on the raw data so it automatically measures the universal parameters (mass tolerance, peak height, peak width, and instrumental shift) and translates them into MS-DIAL-specific settings. Then run MS-DIAL feature extraction twice on the same raw data: once with the Paramounter-derived parameters and once with MS-DIAL default parameters. Count the extracted metabolite features under each condition, assemble a CSV with the counts side by side, and compute the fold increase; accept the result when the Paramounter-derived condition shows approximately a threefold increase over defaults, matching the figure reported in the article. If the optimized peak height produces excessive false positives or software crashes, raise the peak height threshold (e.g. 2X the optimized value) as the README advises, and if dereplication removes true positives with mass differences smaller than the mzdiff tolerance (suggested default 0.001 or 0.01), set mzdiff to any negative value to disable dereplication.

## Related tools

- **Paramounter** (R program that automatically measures the universal parameters from the raw LC-MS data and translates them into MS-DIAL-specific settings for feature extraction) — https://github.com/HuanLab/Paramounter
- **MS-DIAL** (performs the feature extraction on the raw LC-MS data, run once with Paramounter-derived parameters and once with default parameters to quantify the improvement)
- **R** (language in which Paramounter is implemented; used to run the parameter measurement step)

## Examples

```
Rscript Paramounter.R --input raw_lcms_data --target MS-DIAL && msdial -i raw_lcms_data --params cond_paramounter_msdial
```

## Evaluation signals

- The Paramounter-derived MS-DIAL condition yields approximately a threefold increase in the number of extracted metabolites compared to the default-parameters condition on the same raw data.
- The extracted-feature counts under both conditions (cond_paramounter_msdial vs cond_default_msdial) are tabulated side by side in a CSV and the fold increase is computed.
- Paramounter succeeds in automatically measuring the four universal parameters (mass tolerance, peak height, peak width, instrumental shift) from the raw LC-MS data without manual intervention.
- If false positives or software crashes occur at the optimized peak height, applying a higher threshold (e.g. 2X the optimized value) reduces the false positive feature count.

## Limitations

- Paramounter's optimized peak height maximizes true positive features but comes with a higher rate of false positive features and the likelihood of software crash; a higher peak height threshold (e.g. 2X) trades sensitivity for precision.
- The mzdiff dereplication tolerance (suggested default 0.001 or 0.01) can mistakenly remove true positive features with smaller mass differences; setting mzdiff to a negative value disables dereplication entirely.
- Robustness was validated on 11 datasets covering different sample types, data acquisition modes, and MS vendors; behavior outside that scope is not established.

## Evidence
