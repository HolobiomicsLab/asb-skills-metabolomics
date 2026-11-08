---
name: ionization-mode-feature-count-comparison
description: Use when raw untargeted metabolomics LC-MS/MS data must be processed
  with two distinct peak-picking parameter sets (e.g., AutoTuner- vs. IPO-derived
  centWave parameters) and downstream feature-table differences need to be quantified
  per ionization mode.
when_to_use_negative:
- Input is already a finished feature table and the goal is only to annotate or identify
  features — this skill compares tables built from raw data under different parameter
  sets.
- Only one parameter-optimization workflow is available, so no comparative counting
  is possible.
- The analysis goal is robustness of AutoTuner's own parameter estimates (Monte Carlo
  subset testing) rather than AutoTuner-vs-IPO feature-table comparison.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AutoTuner
  role: Estimates centWave peak-picking parameters from raw data in a single inference
    step; these parameters are one of the two compared pipelines for generating feature
    tables.
  repo: https://github.com/crmclean/autotuner
- name: IPO
  role: Iterative isotopologue parameter optimization tool whose derived centWave
    parameters form the second (comparison) pipeline for feature-table generation.
- name: XCMS
  role: Executes centWave peak detection, grouping, and retention time correction
    on the raw data using either parameter set to produce the feature tables being
    compared.
- name: R
  role: Language in which AutoTuner, IPO, and XCMS are distributed and in which the
    comparison and counting are performed.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:79b2bd8007c82f745ab0d437ab56deee60673dde168e22d0942c7d62e77629a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/crmclean/autotuner
  tools:
  - AutoTuner
  - IPO
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ionization-mode-feature-count-comparison

## Summary

Compares feature-table composition between two parameter-optimization workflows (AutoTuner vs. IPO-derived centWave parameters) applied to the same raw LC-MS/MS untargeted metabolomics data split by ionization mode (ESI+ and ESI−), reporting unique and shared feature counts to quantify how parameter choice reshapes downstream feature tables.

## When to use

Use when raw untargeted metabolomics LC-MS/MS data must be processed with two distinct peak-picking parameter sets (e.g., AutoTuner- vs. IPO-derived centWave parameters) and downstream feature-table differences need to be quantified per ionization mode.

## When NOT to use

- Input is already a finished feature table and the goal is only to annotate or identify features — this skill compares tables built from raw data under different parameter sets.
- Only one parameter-optimization workflow is available, so no comparative counting is possible.
- The analysis goal is robustness of AutoTuner's own parameter estimates (Monte Carlo subset testing) rather than AutoTuner-vs-IPO feature-table comparison.

## Inputs

- Raw untargeted metabolomics LC-MS/MS data files in mzML, mzXML, or CDF format, split by positive and negative ionization mode (e.g., MetaboLights dataset MTBLS157 culture samples)
- AutoTuner-derived centWave peak-picking parameters (seven continuous parameters plus one XCMS grouping parameter)
- IPO-derived centWave peak-picking parameters (obtained via iterative optimization)

## Outputs

- Two processed feature tables per ionization mode (one from AutoTuner parameters, one from IPO parameters)
- Counts of unique features per table and shared features between tables, reported per ionization mode (e.g., AutoTuner 203 vs. IPO 2606 unique, 1022 shared in positive mode; 540 vs. 3420, 904 shared in negative mode)
- A CSV (e.g., feature_counts_mtbls157.csv) summarizing unique and shared feature counts per ionization mode and parameter set

## How to apply

Obtain raw data (e.g., from a MetaboLights deposit such as MTBLS157), convert to mzML/mzXML/CDF, and split samples by ionization mode (positive and negative ESI). For each ionization mode, run XCMS centWave peak detection using two independently derived parameter sets: one estimated from raw data by AutoTuner in a single inference step (seven continuous centWave parameters plus a key XCMS grouping parameter), and one obtained by iterative parameter sweeping with IPO. Then run grouping and retention time correction on each parameter set's output, and apply the paper's quality-control filtering to the resulting feature tables. Finally, for each mode, count features unique to each table and features shared between them; this isolates how parameter choice alone affects feature recovery. In the paper's culture dataset this yielded AutoTuner 203 unique / IPO 2606 unique / 1022 shared in positive mode, and 540/3420/904 in negative mode.

## Related tools

- **AutoTuner** (Estimates centWave peak-picking parameters from raw data in a single inference step; these parameters are one of the two compared pipelines for generating feature tables.) — https://github.com/crmclean/autotuner
- **IPO** (Iterative isotopologue parameter optimization tool whose derived centWave parameters form the second (comparison) pipeline for feature-table generation.)
- **XCMS** (Executes centWave peak detection, grouping, and retention time correction on the raw data using either parameter set to produce the feature tables being compared.)
- **R** (Language in which AutoTuner, IPO, and XCMS are distributed and in which the comparison and counting are performed.)

## Examples

```
Rscript -e 'library(Autotuner); Autotuner::createAnnot()' # then feed AutoTuner- vs IPO-derived parameters into XCMS findChromPeaks/groupChromPeaks/adjustRtime per ionization mode
```

## Evaluation signals

- Unique and shared feature counts per ionization mode match the published reference values for the same dataset (culture/MTBLS157: AutoTuner 203 unique, IPO 2606 unique, 1022 shared in positive mode; 540/3420/904 in negative mode).
- Both parameter sets are applied to the identical raw file list and identical ionization-mode split, so any feature-count difference is attributable to parameter choice rather than differing input data.
- XCMS processing stages (centWave peak detection, grouping, retention time correction) are run identically for both parameter sets, with only the centWave/grouping parameters differing.
- Feature tables survive the paper's quality-control filtering before counting, so counts are on filtered tables, not raw feature lists.
- Output CSV contains one row or record per (ionization mode × parameter set) combination with explicit unique/shared counts.

## Limitations

- AutoTuner does not optimize the mzDiff continuous parameter; only a sensitivity analysis was performed, showing distinct mzDiff values had minimal effect on the returned feature table.
- The Group difference parameter cannot be validated for robustness because it requires a non-automatable cross-sample comparison during AutoTuner's TIC peak detection step.
- AutoTuner's divide-and-conquer speed advantage comes at a loss of comparing EIC peaks across samples to estimate retention time correction algorithms.
- Paucity of size-validated metabolome datasets precludes further evaluation of the feature-number comparison, so absolute counts may not generalize to other datasets.
- Feature identification was beyond the scope of the source study; counts alone do not assess whether retained features are correct annotations.
- AutoTuner currently requires R version 3.6 or greater and at least 3 samples of raw data converted from proprietary instrument formats, plus a spreadsheet mapping sample names to experimental factors.

## Evidence

- [results] AutoTuner generates smaller feature tables with more features retained after quality control than IPO-derived parameters: "For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1."
- [readme] AutoTuner requires raw data in mzML/mzXML/CDF and a sample metadata sheet: [evidence span withheld — anchor status: external_doc]
- [intro] AutoTuner parameter estimation is a single-step inference on raw data rather than iterative parameter testing: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
- [discussion] Feature-count comparison is limited by lack of size-validated datasets: "The paucity of size-validated metabolome datasets precludes further evaluation of the feature number comparison."
