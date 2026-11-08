---
name: lcms-universal-parameter-measurement
description: Use when raw LC-MS metabolomics data must be processed with feature-extraction
  software (MS-DIAL, XCMS CentWave, AutoTuner, IPO) and default parameter sets would
  discard true-positive metabolic features.
when_to_use_negative:
- Input is already a processed feature table — Paramounter measures parameters from
  raw LC-MS data prior to feature extraction, so pre-extracted features cannot benefit.
- The user explicitly wants default software parameters and is not concerned about
  the default MS-DIAL settings dropping metabolite features — applying Paramounter
  would change the parameter set unnecessarily.
- Dereplication of co-eluting isobaric features is required and cannot be disabled
  — note that Paramounter's recommended peak-height optimization can inflate false
  positives.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: Paramounter
  role: R program that automatically measures universal parameters from raw LC-MS-based
    metabolomics data and translates them to software-specific settings
  repo: https://github.com/HuanLab/Paramounter
- name: MS-DIAL
  role: feature-extraction program whose default parameters are compared against Paramounter-derived
    settings; extracted metabolite counts are compared
- name: XCMS CentWave
  role: feature extractor whose mzdiff mass tolerance is used to dereplicate features;
    Paramounter-derived parameters feed its peak-picking settings
- name: AutoTuner
  role: baseline auto-tuning program for head-to-head true-positive-feature comparison
    against Paramounter
- name: IPO
  role: baseline XCMS parameter optimizer for head-to-head true-positive-feature comparison
    against Paramounter
- name: R
  role: language in which Paramounter is implemented and executed
  repo: https://github.com/HuanLab/Paramounter
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:9a8ba18b26994783eb8d5ac696b2ec2f2cb4cc13466d40c0c1a0b4324116fa52
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lcms-universal-parameter-measurement

## Summary

Measure universal LC-MS feature-extraction parameters (mass tolerance, peak height, peak width, instrumental shift) directly from raw metabolomics data using Paramounter, then translate them to software-specific settings for MS-DIAL, XCMS CentWave, or other processors.

## When to use

Use when raw LC-MS metabolomics data must be processed with feature-extraction software (MS-DIAL, XCMS CentWave, AutoTuner, IPO) and default parameter sets would discard true-positive metabolic features.

## When NOT to use

- Input is already a processed feature table — Paramounter measures parameters from raw LC-MS data prior to feature extraction, so pre-extracted features cannot benefit.
- The user explicitly wants default software parameters and is not concerned about the default MS-DIAL settings dropping metabolite features — applying Paramounter would change the parameter set unnecessarily.
- Dereplication of co-eluting isobaric features is required and cannot be disabled — note that Paramounter's recommended peak-height optimization can inflate false positives.

## Inputs

- raw LC-MS-based metabolomics data files (e.g. mzML/raw vendor files) from any of the 11 validated sample types, acquisition modes, and MS vendors
- target feature-extraction program name (MS-DIAL, XCMS CentWave, AutoTuner, IPO)

## Outputs

- measured universal parameters: mass tolerance, peak height, peak width, instrumental shift
- software-specific parameter set translated for the target program (e.g. cond_paramounter_msdial for MS-DIAL or CentWave settings for XCMS)

## How to apply

Run Paramounter (R) on the raw LC-MS files to automatically measure the four universal parameters — mass tolerance, peak height, peak width, and instrumental shift — from the data itself before any feature extraction. Translate these measurements into software-specific settings for the chosen processor (e.g. MS-DIAL or XCMS CentWave) instead of using the program's defaults. For XCMS CentWave, remember that mzdiff acts as the mass tolerance for dereplicating features with similar m/z values and retention times (suggested defaults 0.001 or 0.01, negative value disables dereplication); because small-mass-difference true positives can be removed by mistake, disabling dereplication or using a higher peak-height threshold (e.g. 2× the optimized value) are explicit decision points. Paramounter optimizes peak height to maximize true-positive features, which trades higher false-positive rates and possible software crashes; validate by counting extracted features against defaults and by comparing true-positive yields to AutoTuner (expect ~3.7× more) and IPO (expect ~1.6× more).

## Related tools

- **Paramounter** (R program that automatically measures universal parameters from raw LC-MS-based metabolomics data and translates them to software-specific settings) — https://github.com/HuanLab/Paramounter
- **MS-DIAL** (feature-extraction program whose default parameters are compared against Paramounter-derived settings; extracted metabolite counts are compared)
- **XCMS CentWave** (feature extractor whose mzdiff mass tolerance is used to dereplicate features; Paramounter-derived parameters feed its peak-picking settings)
- **AutoTuner** (baseline auto-tuning program for head-to-head true-positive-feature comparison against Paramounter)
- **IPO** (baseline XCMS parameter optimizer for head-to-head true-positive-feature comparison against Paramounter)
- **R** (language in which Paramounter is implemented and executed) — https://github.com/HuanLab/Paramounter

## Examples

```
Rscript Paramounter.R --input raw_lcms_files/ --output cond_paramounter_msdial
```

## Evaluation signals

- Running MS-DIAL with Paramounter-derived parameters (cond_paramounter_msdial) versus default parameters (cond_default_msdial) on the same raw data should show approximately a threefold increase in extracted metabolites.
- In the AutoTuner/IPO benchmark, Paramounter-generated parameters should yield ~3.7× more true-positive features than AutoTuner and ~1.6× more than IPO, with features classified against a ground-truth annotation standard and dereplicated by the mzdiff mass tolerance.
- The output parameter set must contain all four universal parameters (mass tolerance, peak height, peak width, instrumental shift) measured from the raw data rather than hardcoded defaults.
- If the optimized peak height yields excessive false positives or crashes the downstream software, raising the threshold to 2× the optimized value should reduce false-positive count while retaining true positives.
- Validation across multiple raw LC-MS datasets (sample types, acquisition modes, MS vendors) should show Paramounter's measurements remain robust, consistent with the 11-dataset validation reported in the article.

## Limitations

- Paramounter's optimized peak height maximizes true-positive features but also raises the false-positive rate and increases the likelihood of a software crash; users may need a higher peak-height threshold (e.g. 2×) to mitigate this.
- mzdiff dereplication with default values (0.001 or 0.01) can mistakenly remove true-positive features whose mass differences fall below the tolerance; disabling dereplication via a negative mzdiff value trades this off against retaining duplicate features.
- No discussion-section limitations were extracted from the article text provided, so additional caveats beyond the README notes are not documented here.

## Evidence
