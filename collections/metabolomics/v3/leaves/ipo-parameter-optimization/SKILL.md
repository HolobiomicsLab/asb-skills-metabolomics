---
name: ipo-parameter-optimization
description: Use when raw LC-MS metabolomics data must be peak-picked with XCMS CentWave
  but no manual parameter tuning is available, and an objective, auto-optimized parameter
  set (e.g., for benchmarking against Paramounter or AutoTuner) is needed.
when_to_use_negative:
- When a direct measurement of universal parameters is preferred over auto-optimization
  — use Paramounter instead, which measured mass tolerance, peak height, peak width,
  and instrumental shift directly and produced 1.6-fold more true positive features
  than IPO
- When the input is already a feature table — IPO optimizes parameters for peak picking
  from raw LC-MS data, not re-processing of extracted features
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IPO
  role: Auto-optimizes XCMS CentWave peak-picking parameters from the same raw LC-MS
    data used for the other methods
- name: XCMS CentWave
  role: Performs feature extraction using the IPO-optimized parameter set; mzdiff
    dereplicates features with similar m/z values and retention times
- name: Paramounter
  role: Benchmarked comparison target that generated 1.6-fold more true positive features
    than IPO
  repo: https://github.com/HuanLab/Paramounter
- name: AutoTuner
  role: Auto-tuning method benchmarked alongside IPO and Paramounter
- name: R
  role: Programming environment in which IPO, XCMS CentWave, and Paramounter are run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:4994b2c4f575046a466f667ec34a5338ef8eed9538ddb0c2cc57d3b1a6079079
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# ipo-parameter-optimization

## Summary

Auto-optimize XCMS CentWave peak-picking parameters with IPO on raw LC-MS metabolomics data, then perform feature extraction with the optimized parameter set as a benchmark baseline against Paramounter and AutoTuner.

## When to use

Use when raw LC-MS metabolomics data must be peak-picked with XCMS CentWave but no manual parameter tuning is available, and an objective, auto-optimized parameter set (e.g., for benchmarking against Paramounter or AutoTuner) is needed.

## When NOT to use

- When a direct measurement of universal parameters is preferred over auto-optimization — use Paramounter instead, which measured mass tolerance, peak height, peak width, and instrumental shift directly and produced 1.6-fold more true positive features than IPO
- When the input is already a feature table — IPO optimizes parameters for peak picking from raw LC-MS data, not re-processing of extracted features

## Inputs

- Raw LC-MS-based metabolomics data files
- XCMS CentWave parameter ranges to be optimized by IPO
- Ground-truth annotation standard for true-positive classification (for benchmarking)

## Outputs

- IPO-optimized XCMS CentWave parameter set
- Feature table extracted with XCMS CentWave using IPO-optimized parameters
- True-positive feature counts per autotuning method (benchmark_tp_features.csv with fold-advantage of Paramounter over IPO)

## How to apply

Run IPO on the same raw LC-MS data used for all competing methods so the parameter sets are comparable. IPO auto-optimizes the XCMS CentWave parameters (e.g., mass tolerance, peak height, peak width, mzdiff) from the raw data. After optimization, perform feature extraction with XCMS CentWave using the IPO-derived parameters, then apply the mzdiff-based dereplication filter that removes features with similar m/z values and retention times within the mzdiff mass tolerance (suggested defaults 0.001 or 0.01; a negative mzdiff disables dereplication). Classify extracted features as true positives against the ground-truth annotation standard used in the reported comparison and tabulate true-positive counts per method. In the published head-to-head comparison, IPO's optimized parameters yielded 1.6-fold fewer true positive features than Paramounter, so IPO serves as the auto-optimization baseline.

## Related tools

- **IPO** (Auto-optimizes XCMS CentWave peak-picking parameters from the same raw LC-MS data used for the other methods)
- **XCMS CentWave** (Performs feature extraction using the IPO-optimized parameter set; mzdiff dereplicates features with similar m/z values and retention times)
- **Paramounter** (Benchmarked comparison target that generated 1.6-fold more true positive features than IPO) — https://github.com/HuanLab/Paramounter
- **AutoTuner** (Auto-tuning method benchmarked alongside IPO and Paramounter)
- **R** (Programming environment in which IPO, XCMS CentWave, and Paramounter are run)

## Evaluation signals

- IPO completes parameter optimization on the raw LC-MS data and emits a concrete XCMS CentWave parameter set (no default parameters left in place)
- Feature extraction applies the mzdiff-based dereplication filter with mzdiff set to 0.001 or 0.01 (or a negative value if dereplication is intentionally disabled)
- True-positive classification uses the same ground-truth annotation standard across Paramounter, AutoTuner, and IPO feature tables
- Benchmark table (benchmark_tp_features.csv) contains true-positive counts per method and computed fold advantages; expected ordering per the article is Paramounter > IPO (1.6-fold) > AutoTuner (3.7-fold vs Paramounter)
- All three methods are run on identical raw LC-MS input data, ensuring parameter sets are comparable

## Limitations

- IPO generated 1.6-fold fewer true positive features than Paramounter in the head-to-head comparison, so it should not be expected to match Paramounter's sensitivity
- The mzdiff dereplication step (default 0.001 or 0.01) can remove true positive metabolic features whose mass differences are smaller than the tolerance by mistake
- No discussion-section content was available, so additional IPO-specific limitations (e.g., runtime, robustness) could not be extracted

## Evidence
