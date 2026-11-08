---
name: feature-count-comparison-analysis
description: Reference for quantifying how a parameter tuning approach improves metabolic
  feature extraction yield, by counting extracted features under tuned vs. default
  settings and computing the fold increase.
when_to_use_negative:
- Input is already a processed feature table — re-extraction comparison is not applicable
- The goal is parameter tuning itself rather than benchmarking the yield improvement
- No baseline/default parameter condition is available for comparison
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Paramounter
  role: Measures universal parameters from raw LC-MS data and translates them into
    software-specific settings for the tuned condition
  repo: https://github.com/HuanLab/Paramounter
- name: MS-DIAL
  role: Feature extraction engine run under both tuned and default parameter sets
    to produce the two feature counts being compared
- name: AutoTuner
  role: Alternative parameter-tuning tool used as a baseline in the true-positive
    feature count comparison
- name: IPO
  role: Alternative parameter-tuning tool used as a baseline in the true-positive
    feature count comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:fb2a9e7821aadf103504a7b9d198d9f72dfaebb8e2bc87b3bef5144cf1621615
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# feature-count-comparison-analysis

## Summary

Benchmark analysis that compares the number of extracted metabolite features between a parameter-tuned condition (e.g., Paramounter-measured settings) and a baseline condition (e.g., software defaults) on the same raw LC-MS data, reporting the fold increase in features.

## When to use

Reference for quantifying how a parameter tuning approach improves metabolic feature extraction yield, by counting extracted features under tuned vs. default settings and computing the fold increase.

## When NOT to use

- Input is already a processed feature table — re-extraction comparison is not applicable
- The goal is parameter tuning itself rather than benchmarking the yield improvement
- No baseline/default parameter condition is available for comparison

## Inputs

- Raw LC-MS metabolomics data files (e.g., centroided mzML or similar)
- Feature extraction program (MS-DIAL) with both tuned and default parameter sets
- Paramounter R package from HuanLab/Paramounter

## Outputs

- CSV table listing extracted-feature counts side by side (tuned vs. default)
- Fold-increase value (e.g., ~3x for Paramounter vs. MS-DIAL defaults)

## How to apply

Run the feature extraction program (e.g., MS-DIAL) twice on the identical raw LC-MS files: once with the tuned parameters derived from Paramounter's measurement of universal parameters (mass tolerance, peak height, peak width, instrumental shift), and once with the program's default parameters. Count the number of extracted metabolite features under each condition and assemble a side-by-side comparison table. Compute the fold increase as the ratio of tuned-condition feature count to default-condition feature count. The article reports a threefold increase when Paramounter-derived parameters are used in MS-DIAL, so a successful run should reproduce that order-of-magnitude gain. Optionally extend the benchmark by counting true positive features against AutoTuner and IPO baselines, where Paramounter reportedly yields 3.7- and 1.6-fold more true positive features respectively.

## Related tools

- **Paramounter** (Measures universal parameters from raw LC-MS data and translates them into software-specific settings for the tuned condition) — https://github.com/HuanLab/Paramounter
- **MS-DIAL** (Feature extraction engine run under both tuned and default parameter sets to produce the two feature counts being compared)
- **AutoTuner** (Alternative parameter-tuning tool used as a baseline in the true-positive feature count comparison)
- **IPO** (Alternative parameter-tuning tool used as a baseline in the true-positive feature count comparison)

## Examples

```
R: library(Paramounter); params <- measure(raw_data); msdial_run(raw_data, cond = "paramounter_msdial"); msdial_run(raw_data, cond = "default_msdial"); fold_increase <- count(params) / count(defaults)
```

## Evaluation signals

- The tuned condition yields approximately a threefold increase in extracted metabolites compared to MS-DIAL defaults
- Feature counts are computed on the same raw data files under both conditions (only the parameter set differs)
- A side-by-side CSV comparison table is produced with counts for both conditions
- If extended to true-positive benchmarking, Paramounter produces 3.7-fold more true positive features than AutoTuner and 1.6-fold more than IPO

## Limitations

- The optimized peak height tuned by Paramounter carries a higher false-positive feature rate and increases the likelihood of software crash; users can raise the peak height threshold (e.g., 2X the optimized value) to reduce false positives
- Default mzdiff dereplication values (0.001 or 0.01) can remove true positive features whose mass differences fall below the tolerance; setting mzdiff to a negative value disables dereplication
- Validated on 11 datasets covering different sample types, data acquisition modes, and MS vendors — performance on datasets outside this coverage is not guaranteed

## Evidence

- [abstract] Paramounter-measured parameters yield ~3x more metabolites than MS-DIAL defaults: "Applying Paramounter is demonstrated to provide a threefold increase in the extracted metabolites compared to using default parameters in MS-DIAL-based feature extraction."
- [abstract] Paramounter outperforms AutoTuner and IPO in true-positive feature counts: "Paramounter generates 3.7- and 1.6-fold more true positive features than AutoTuner and IPO, respectively"
- [readme] Optimized peak height raises false positive rate and crash risk; 2X threshold mitigates: "Paramounter tunes an optimized peak height to maximize the number of true positive features. A drawback of that optimized value is the higher rate of false positive features and the likelihood of"
