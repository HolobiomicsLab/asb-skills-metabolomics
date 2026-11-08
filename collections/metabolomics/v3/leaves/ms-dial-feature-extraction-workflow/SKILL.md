---
name: ms-dial-feature-extraction-workflow
description: Use when raw LC-MS-based metabolomics data must be converted into a metabolic
  feature table and software-specific extraction parameters need to be set automatically
  rather than guessed with MS-DIAL defaults.
when_to_use_negative:
- The input is already a processed feature table rather than raw LC-MS data — Paramounter
  measures parameters from raw data prior to feature extraction.
- The user wants only default MS-DIAL settings and is not concerned about the default
  pipeline's lower metabolite yield — the whole point of this workflow is to replace
  defaults with measured parameters.
- Feature extraction is being performed in XCMS CentWave without a need to translate
  universal parameters into MS-DIAL-specific settings.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: Paramounter
  role: R program that automatically measures universal parameters (mass tolerance,
    peak height, peak width, instrumental shift) from raw LC-MS data and translates
    them into MS-DIAL-specific parameters for feature extraction
  repo: https://github.com/HuanLab/Paramounter
- name: MS-DIAL
  role: Feature extraction program whose data processing is driven by the Paramounter-translated
    software-specific parameters, replacing default parameters
- name: R
  role: Language in which Paramounter is implemented and executed to measure universal
    parameters prior to MS-DIAL feature extraction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:55da153b3fc5719f93bf8402e6b3d4ab86c601a26a8d3972907fa6404152c247
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - MS-DIAL
  - Paramounter
  - R
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# ms-dial-feature-extraction-workflow

## Summary

Feature extraction of metabolites from raw LC-MS data driven by universal parameters (mass tolerance, peak height, peak width, instrumental shift) measured by Paramounter and translated into MS-DIAL-specific settings, yielding roughly threefold more extracted metabolites than MS-DIAL defaults.

## When to use

Use when raw LC-MS-based metabolomics data must be converted into a metabolic feature table and software-specific extraction parameters need to be set automatically rather than guessed with MS-DIAL defaults.

## When NOT to use

- The input is already a processed feature table rather than raw LC-MS data — Paramounter measures parameters from raw data prior to feature extraction.
- The user wants only default MS-DIAL settings and is not concerned about the default pipeline's lower metabolite yield — the whole point of this workflow is to replace defaults with measured parameters.
- Feature extraction is being performed in XCMS CentWave without a need to translate universal parameters into MS-DIAL-specific settings.

## Inputs

- Raw LC-MS-based metabolomics data files (e.g., vendor RAW/mzML inputs consumed by MS-DIAL)
- Raw LC-MS-based metabolomics data across diverse sample types, data acquisition modes, and MS vendors (11 validation datasets)

## Outputs

- Metabolic feature table (extracted metabolites/features with m/z and retention time) produced by MS-DIAL using Paramounter-derived parameters
- Per-dataset counts of extracted metabolites/features, suitable for assembling an 11-dataset validation summary

## How to apply

Run Paramounter (an R program) on the raw LC-MS-based metabolomics data to automatically measure the universal parameters — mass tolerance, peak height, peak width, and instrumental shift — prior to feature extraction. Let Paramounter translate those universal parameters into software-specific parameters for MS-DIAL-based data processing, then execute the MS-DIAL feature extraction with those derived settings instead of default parameters. Expect roughly a threefold increase in extracted metabolites relative to default MS-DIAL settings; benchmarked comparisons against AutoTuner and IPO showed Paramounter-driven extraction yields 3.7-fold and 1.6-fold more true positive features, respectively. If the optimized peak height causes excessive false positives or software crashes, raise the peak height threshold (e.g., 2X the optimized value) as the article recommends. Record the extracted metabolite/feature counts per dataset (covering sample type, acquisition mode, and MS vendor) to confirm robust behavior across datasets.

## Related tools

- **Paramounter** (R program that automatically measures universal parameters (mass tolerance, peak height, peak width, instrumental shift) from raw LC-MS data and translates them into MS-DIAL-specific parameters for feature extraction) — https://github.com/HuanLab/Paramounter
- **MS-DIAL** (Feature extraction program whose data processing is driven by the Paramounter-translated software-specific parameters, replacing default parameters)
- **R** (Language in which Paramounter is implemented and executed to measure universal parameters prior to MS-DIAL feature extraction)

## Examples

```
Rscript Paramounter.R --input raw.mzML --extractor MS-DIAL  # then run MS-DIAL feature extraction with the Paramounter-translated parameters
```

## Evaluation signals

- Extracted metabolite count is approximately threefold higher than a default-parameter MS-DIAL-based feature extraction on the same data.
- In benchmark comparisons, the extracted features show 3.7-fold more true positive features than AutoTuner and 1.6-fold more than IPO.
- The workflow succeeds on the full 11-dataset validation set spanning different sample types, data acquisition modes, and MS vendors.
- Feature table contains m/z values and retention times consistent with MS-DIAL feature extraction output.
- Per-dataset metadata (sample type, acquisition mode, MS vendor) is recorded alongside each feature count for the summary table.

## Limitations

- Paramounter's optimized peak height maximizes true positive features but causes a higher rate of false positive features and the likelihood of software crash; users can use a higher peak height threshold (e.g., 2X the optimized value) to reduce false positives.
- When translating mzdiff as mass tolerance to dereplicate XCMS CentWave features (suggested default 0.001 or 0.01), true positive features with mass differences smaller than that value may be removed by mistake; setting mzdiff to any negative value disables dereplication.
- No discussion-section limitations were available in the provided context.

## Evidence
