---
name: autotuner-parameter-optimization
description: Use when raw LC-MS-based metabolomics data (e.g., .mzML/.mzXML) must
  be processed for feature extraction but peak-picking parameters are unknown or default
  values underperform, and an automated, data-driven parameter measurement is needed.
when_to_use_negative:
- The input is already an extracted feature table rather than raw LC-MS data — parameter
  measurement must run on raw profiles.
- A manually validated, sample-specific parameter set already exists for the instrument
  and acquisition mode, making auto-tuning unnecessary.
- The target feature-extraction program is not among those to which Paramounter translates
  universal parameters (e.g., XCMS CentWave, MS-DIAL).
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Paramounter
  role: R program that automatically measures universal parameters (mass tolerance,
    peak height, peak width, instrumental shift) from raw LC-MS data and translates
    them to software-specific parameters; tuned an optimized peak height that yielded
    3.7-fold more true positive features than AutoTuner and 1.6-fold more than IPO
  repo: https://github.com/HuanLab/Paramounter
- name: AutoTuner
  role: Comparison baseline that auto-tunes its peak-picking parameters from the same
    raw LC-MS data; produced fewer true positive features than Paramounter in the
    head-to-head benchmark
- name: IPO
  role: Comparison baseline that auto-optimizes XCMS CentWave parameters; produced
    1.6-fold fewer true positive features than Paramounter
- name: XCMS CentWave
  role: Feature-extraction engine that consumes the tuned parameter set and produces
    metabolic features subsequently dereplicated by mzdiff mass tolerance
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:8ff0d616bfe33e0a0f14ad75851d46231e5ffe23c1729ac3fac8de2c2ad7c4c6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - AutoTuner
  - IPO
  - Paramounter
  - R
  - XCMS CentWave
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# autotuner-parameter-optimization

## Summary

Automatic optimization of LC-MS peak-picking parameters for metabolic feature extraction: universal parameters (mass tolerance, peak height, peak width, instrumental shift) are measured directly from raw data by Paramounter, or tuned by AutoTuner/IPO, then translated to software-specific parameters (e.g., XCMS CentWave) before feature extraction.

## When to use

Use when raw LC-MS-based metabolomics data (e.g., .mzML/.mzXML) must be processed for feature extraction but peak-picking parameters are unknown or default values underperform, and an automated, data-driven parameter measurement is needed.

## When NOT to use

- The input is already an extracted feature table rather than raw LC-MS data — parameter measurement must run on raw profiles.
- A manually validated, sample-specific parameter set already exists for the instrument and acquisition mode, making auto-tuning unnecessary.
- The target feature-extraction program is not among those to which Paramounter translates universal parameters (e.g., XCMS CentWave, MS-DIAL).

## Inputs

- Raw LC-MS-based metabolomics data files (e.g., .mzML/.mzXML profiles)
- Ground-truth annotation standard for classifying features as true positives (for benchmarking)
- Choice of target feature-extraction program (XCMS CentWave, MS-DIAL)

## Outputs

- Measured universal parameter set (mass tolerance, peak height, peak width, instrumental shift)
- Translated software-specific peak-picking parameters (XCMS CentWave ppmin/ppmmax, peakwidth, mzdiff, etc.)
- Feature table of extracted metabolic features after mzdiff dereplication
- Benchmark table of true-positive feature counts per tuning method (benchmark_tp_features.csv)

## How to apply

Run Paramounter (R, HuanLab/Paramounter) on the raw LC-MS data to automatically measure the universal parameters — mass tolerance, peak height, peak width, and instrumental shift — prior to feature extraction, then translate these to software-specific parameters for XCMS CentWave or MS-DIAL. Set the CentWave mzdiff dereplication tolerance to the suggested default 0.001 or 0.01; note that some true positive features with mass differences smaller than mzdiff may be removed, and any negative mzdiff disables dereplication. Paramounter's optimized peak height maximizes true positive features but raises false positives and software-crash risk, so users can raise the threshold (e.g., 2X the optimized peak height) to reduce false positives. After extraction with XCMS CentWave, apply the mzdiff-based dereplication filter to remove features with similar m/z values and retention times. In the reported head-to-head benchmark, Paramounter's parameter set yielded 3.7-fold more true positive features than AutoTuner and 1.6-fold more than IPO under the same ground-truth annotation standard.

## Related tools

- **Paramounter** (R program that automatically measures universal parameters (mass tolerance, peak height, peak width, instrumental shift) from raw LC-MS data and translates them to software-specific parameters; tuned an optimized peak height that yielded 3.7-fold more true positive features than AutoTuner and 1.6-fold more than IPO) — https://github.com/HuanLab/Paramounter
- **AutoTuner** (Comparison baseline that auto-tunes its peak-picking parameters from the same raw LC-MS data; produced fewer true positive features than Paramounter in the head-to-head benchmark)
- **IPO** (Comparison baseline that auto-optimizes XCMS CentWave parameters; produced 1.6-fold fewer true positive features than Paramounter)
- **XCMS CentWave** (Feature-extraction engine that consumes the tuned parameter set and produces metabolic features subsequently dereplicated by mzdiff mass tolerance)

## Evaluation signals

- The measured parameter set contains all four universal parameters: mass tolerance, peak height, peak width, and instrumental shift.
- mzdiff used for dereplication falls within the suggested defaults (0.001 or 0.01), or is explicitly negative if dereplication was disabled.
- After CentWave extraction and mzdiff dereplication, Paramounter's parameter set should yield ~3.7x more true positive features than AutoTuner's and ~1.6x more than IPO's under the same ground-truth standard.
- benchmark_tp_features.csv tabulates true-positive counts per method and fold advantage values consistent with 3.7 (vs AutoTuner) and 1.6 (vs IPO).
- If the optimized peak height was raised (e.g., 2X) to curb false positives, the extracted feature count should decrease relative to the fully optimized run.

## Limitations

- Paramounter's optimized peak height maximizes true positive features but causes a higher false positive rate and a likelihood of software crash; a higher peak height threshold (e.g., 2X the optimized value) reduces false positives.
- Default mzdiff dereplication (0.001 or 0.01) can mistakenly remove true positive metabolic features whose mass differences are smaller than the tolerance; negative mzdiff disables dereplication.
- Validation covered 11 datasets (different sample types, data acquisition modes, and MS vendors); performance outside that scope is not asserted by the paper.

## Evidence

- [abstract] Paramounter generates 3.7- and 1.6-fold more true positive features than AutoTuner and IPO, respectively: "Paramounter generates 3.7- and 1.6-fold more true positive features than AutoTuner and IPO, respectively"
- [abstract] automatically measures these universal parameters from raw LC-MS-based metabolomics data prior to metabolic feature extraction: "automatically measures these universal parameters from raw LC-MS-based metabolomics data prior to metabolic feature extraction"
- [intro] mzdiff is used as the mass tolerance to dereplicate the features (similar m/z values and retention times) extracted by XCMS CentWave: "mzdiff is used as the mass tolerance to dereplicate the features (similar m/z values and retention times) extracted by XCMS CentWave"
- [readme] users can try a higher peak height threshold to reduce the number of false positive features (e.g., 2X the optimized peak height threshold): "users can try a higher peak height threshold to reduce the number of false positive features (e.g., 2X the optimized peak height threshold)"
- [readme] if a user wants to disable the dereplication function, set the mzdiff to be any negative value: "if a user wants to disable the dereplication function, set the mzdiff to be any negative value"
