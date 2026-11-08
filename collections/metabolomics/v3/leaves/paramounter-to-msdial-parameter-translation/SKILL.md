---
name: paramounter-to-msdial-parameter-translation
description: Use when raw LC-MS metabolomics files must be processed in MS-DIAL but
  software-specific extraction parameters are unknown, and you want to measure them
  from the data itself rather than relying on default parameters.
when_to_use_negative:
- You already have a validated feature table and only need downstream statistical
  analysis — this skill is exclusively about parameter derivation prior to feature
  extraction.
- You are tuning XCMS or IPO rather than MS-DIAL — Paramounter's comparison to AutoTuner
  and IPO suggests those pipelines are alternatives, not targets of this translation
  step.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Paramounter
  role: R program that automatically measures universal parameters from raw LC-MS
    data and translates them into software-specific parameters for MS-DIAL feature
    extraction
  repo: https://github.com/HuanLab/Paramounter
- name: MS-DIAL
  role: Feature extraction software that consumes the Paramounter-derived parameters;
    compared against its default-parameter run to quantify the improvement
- name: R
  role: Language in which Paramounter is implemented and executed
  repo: https://github.com/HuanLab/Paramounter
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:08495c753665213c359993efd7990b9daa94189ae16bb11257d6a635218b182c
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# paramounter-to-msdial-parameter-translation

## Summary

Use the Paramounter R package to automatically measure universal parameters (mass tolerance, peak height, peak width, instrumental shift) directly from raw LC-MS metabolomics data and translate them into MS-DIAL-specific settings for feature extraction, replacing manual default-parameter configuration.

## When to use

Use when raw LC-MS metabolomics files must be processed in MS-DIAL but software-specific extraction parameters are unknown, and you want to measure them from the data itself rather than relying on default parameters.

## When NOT to use

- You already have a validated feature table and only need downstream statistical analysis — this skill is exclusively about parameter derivation prior to feature extraction.
- You are tuning XCMS or IPO rather than MS-DIAL — Paramounter's comparison to AutoTuner and IPO suggests those pipelines are alternatives, not targets of this translation step.

## Inputs

- Raw LC-MS-based metabolomics data files (the comparison dataset referenced in the article)
- MS-DIAL software environment

## Outputs

- MS-DIAL-specific parameter settings derived from measured universal parameters
- Extracted metabolite feature list from MS-DIAL feature extraction

## How to apply

Obtain the raw LC-MS files and install the Paramounter R package from the HuanLab/Paramounter GitHub repository. Run Paramounter on the raw LC-MS data to automatically measure the universal parameters — mass tolerance, peak height, peak width, and instrumental shift — that are shared among feature extraction programs. Paramounter then translates these universal parameters into MS-DIAL-specific settings (e.g. the mzdiff mass tolerance used to dereplicate features with similar m/z values and retention times). Run MS-DIAL feature extraction using the Paramounter-derived parameters; in the article's demonstration this yielded a threefold increase in extracted metabolites compared with MS-DIAL default parameters. Note that Paramounter's optimized peak height maximizes true positive features but increases false positives and crash risk, so users can raise the threshold (e.g. 2X the optimized value) to suppress false positives.

## Related tools

- **Paramounter** (R program that automatically measures universal parameters from raw LC-MS data and translates them into software-specific parameters for MS-DIAL feature extraction) — https://github.com/HuanLab/Paramounter
- **MS-DIAL** (Feature extraction software that consumes the Paramounter-derived parameters; compared against its default-parameter run to quantify the improvement)
- **R** (Language in which Paramounter is implemented and executed) — https://github.com/HuanLab/Paramounter

## Evaluation signals

- The number of extracted metabolite features under the Paramounter-derived condition should show approximately a threefold increase over the MS-DIAL default-parameter condition on the same raw data.
- Paramounter's measured parameters must cover the four universal parameters named in the article: mass tolerance, peak height, peak width, and instrumental shift.
- Compare true positive feature counts against AutoTuner and IPO results — Paramounter should produce 3.7-fold and 1.6-fold more true positive features respectively.
- Verify that the mzdiff-derived mass tolerance is applied as the dereplication threshold in the resulting MS-DIAL parameter set.
- Validate robustness by running the workflow across multiple datasets with different sample types, acquisition modes, and MS vendors — the article reported success on 11 such datasets.

## Limitations

- Paramounter's optimized peak height yields a higher rate of false positive features and increases the likelihood of software crash; raising the threshold (e.g. 2X the optimized value) reduces false positives.
- Using the suggested default mzdiff of 0.001 or 0.01 may remove true positive features whose mass differences fall below that value; setting mzdiff to a negative value disables dereplication.
- The threefold improvement claim is specific to the article's LC-MS comparison dataset and MS-DIAL pipeline; the absolute gain may differ for other instruments or acquisition modes.

## Evidence
