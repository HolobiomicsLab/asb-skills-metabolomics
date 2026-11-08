---
name: lcms-raw-data-autotuning-benchmark
description: Use when raw LC-MS metabolomics data must be processed and you need to
  decide which auto-tuning method (Paramounter, AutoTuner, or IPO) yields the most
  true positive features before committing to a peak-picking parameter set.
when_to_use_negative:
- Input is already a processed feature table — this skill benchmarks parameter tuning
  from raw LC-MS data, not post-extraction feature lists.
- You only need MS-DIAL default-vs-optimized parameter comparison without a ground-truth
  standard for true-positive classification.
- The raw data are not LC-MS metabolomics acquisitions (e.g., proteomics or non-MS
  data), where the universal parameters (mass tolerance, peak height, peak width,
  instrumental shift) and CentWave/IPO/AutoTuner tooling do not apply.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3391
tools:
- name: Paramounter
  role: Measures universal parameters (mass tolerance, peak height, peak width, instrumental
    shift) from raw LC-MS data and translates them to XCMS CentWave parameters; benchmarked
    as the reference method generating 3.7-fold more true positive features than AutoTuner.
  repo: https://github.com/HuanLab/Paramounter
- name: AutoTuner
  role: Competing auto-tuning method run on the same raw LC-MS data to auto-tune peak-picking
    parameters for the true-positive comparison.
- name: IPO
  role: Competing auto-optimization method run on the same raw LC-MS data to optimize
    XCMS CentWave parameters for the true-positive comparison.
- name: XCMS CentWave
  role: Feature extraction engine applied identically to each of the three parameter
    sets, with mzdiff-based dereplication applied to the extracted features.
- name: R
  role: Implementation language; Paramounter is an R program and the benchmark table
    is produced as an R-based workflow output.
  repo: https://github.com/HuanLab/Paramounter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:2c262d33a5330cbf7c7c7baaebec9f60b47ccbb2ea6c0e67bb43ba5c23bb46e2
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
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lcms-raw-data-autotuning-benchmark

## Summary

A head-to-head benchmarking procedure that compares automated LC-MS parameter-tuning methods (Paramounter, AutoTuner, IPO) by running each on the same raw metabolomics data, extracting features with XCMS CentWave, and counting true positive metabolic features against a ground-truth annotation standard.

## When to use

Use when raw LC-MS metabolomics data must be processed and you need to decide which auto-tuning method (Paramounter, AutoTuner, or IPO) yields the most true positive features before committing to a peak-picking parameter set.

## When NOT to use

- Input is already a processed feature table — this skill benchmarks parameter tuning from raw LC-MS data, not post-extraction feature lists.
- You only need MS-DIAL default-vs-optimized parameter comparison without a ground-truth standard for true-positive classification.
- The raw data are not LC-MS metabolomics acquisitions (e.g., proteomics or non-MS data), where the universal parameters (mass tolerance, peak height, peak width, instrumental shift) and CentWave/IPO/AutoTuner tooling do not apply.

## Inputs

- Raw LC-MS-based metabolomics data (unprocessed instrument files) shared identically across all compared autotuning methods
- Ground-truth annotation standard for classifying extracted features as true positives

## Outputs

- Per-method true-positive feature counts for Paramounter, AutoTuner, and IPO
- Fold-advantage metrics of Paramounter over AutoTuner and IPO (expected ~3.7-fold and ~1.6-fold)
- benchmark_tp_features.csv tabulating true-positive feature counts per autotuning method
- XCMS CentWave parameter sets derived from each autotuning method

## How to apply

Run Paramounter on the raw LC-MS data to measure the universal parameters (mass tolerance, peak height, peak width, instrumental shift) and translate them into XCMS CentWave parameters; in parallel, run AutoTuner and IPO on the identical raw data to auto-tune/optimize their respective parameter sets. Extract features for each of the three parameter sets using XCMS CentWave, then apply the mzdiff-based dereplication filter (suggested default 0.001 or 0.01; set mzdiff to any negative value to disable dereplication) to remove features with similar m/z values and retention times. Classify the extracted features as true positives using the same ground-truth annotation standard across all methods, tabulate true-positive counts per method, and compute Paramounter's fold advantage (reported as 3.7-fold over AutoTuner and 1.6-fold over IPO). Decision point: because Paramounter's optimized peak height maximizes true positives at the cost of higher false positives, also consider re-running with a raised peak height threshold (e.g., 2X the optimized value) to reduce false positives before finalizing the benchmark.

## Related tools

- **Paramounter** (Measures universal parameters (mass tolerance, peak height, peak width, instrumental shift) from raw LC-MS data and translates them to XCMS CentWave parameters; benchmarked as the reference method generating 3.7-fold more true positive features than AutoTuner.) — https://github.com/HuanLab/Paramounter
- **AutoTuner** (Competing auto-tuning method run on the same raw LC-MS data to auto-tune peak-picking parameters for the true-positive comparison.)
- **IPO** (Competing auto-optimization method run on the same raw LC-MS data to optimize XCMS CentWave parameters for the true-positive comparison.)
- **XCMS CentWave** (Feature extraction engine applied identically to each of the three parameter sets, with mzdiff-based dereplication applied to the extracted features.)
- **R** (Implementation language; Paramounter is an R program and the benchmark table is produced as an R-based workflow output.) — https://github.com/HuanLab/Paramounter

## Examples

```
Rscript run_tp_benchmark.R --raw raw.mzML --methods paramounter,autotuner,ipo --output benchmark_tp_features.csv
```

## Evaluation signals

- Paramounter's true-positive count exceeds AutoTuner's by approximately 3.7-fold and IPO's by approximately 1.6-fold when the identical raw data and ground-truth standard are used.
- Each method is scored on the identical raw LC-MS file(s) with feature extraction performed by the same XCMS CentWave configuration, so any true-positive differences are attributable to the parameter sets, not the extractor.
- The dereplication step is consistently configured across all three arms (same mzdiff value, e.g., 0.001 or 0.01, or a negative value if dereplication is disabled) so the filter does not bias one method.
- Output file benchmark_tp_features.csv exists and contains one true-positive count per method (Paramounter, AutoTuner, IPO) plus fold-advantage columns.
- True-positive classification applies the same ground-truth annotation standard to all three methods' feature lists before counting.

## Limitations

- Paramounter's optimized peak height maximizes true positive features but causes a higher rate of false positive features and a likelihood of software crash; users can try a higher peak height threshold (e.g., 2X the optimized value) to reduce false positives.
- The mzdiff dereplication default (0.001 or 0.01) may remove true positive metabolic features with mass differences smaller than that value by mistake; setting mzdiff to any negative value disables dereplication.
- The published fold-advantage (3.7x vs AutoTuner, 1.6x vs IPO) was validated on 11 datasets covering different sample types, data acquisition modes, and MS vendors — results on data outside this validation scope may differ.

## Evidence
