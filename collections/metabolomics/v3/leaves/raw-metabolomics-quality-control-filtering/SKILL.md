---
name: raw-metabolomics-quality-control-filtering
description: Use when raw mzML/mzXML/CDF/qTOF, Orbitrap or FT-ICR data must be processed
  into a features table and QC filtering is needed to remove noise-inflating features
  before or after XCMS/MZmine2 centWave processing.
when_to_use_negative:
- The input is already a processed, noise-filtered features table rather than raw
  spectrometer output needing peak-picking and QC.
- The dataset is too small to estimate parameters robustly — AutoTuner requires at
  least 3 samples of raw data, and ~9–12 samples are recommended for stable CV < 0.1
  estimates; below that the QC-parameter optimization is unreliable.
- The goal is feature/compound identification (e.g. annotation or isotope-label tracing)
  — the paper states identifying each distinct feature was beyond the scope of this
  study.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: AutoTuner
  role: Selects the seven continuous centWave peak-picking parameters plus the XCMS
    grouping parameter from raw data in a single statistical-inference step, preventing
    noise inflation before QC filtering.
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: XCMS
  role: Runs centWave peak detection, grouping, and retention-time correction on the
    raw files to produce the features table that is subsequently QC-filtered.
- name: MZmine2
  role: Alternative processing tool that consumes the AutoTuner-optimized centWave
    parameters for peak picking and feature table generation.
- name: IPO
  role: Baseline iterative parameter-optimization alternative to AutoTuner for comparison
    when validating parameter-selection QC outcomes.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:d7ebeb7e78faf0a19d1e0d9f676420aab7f57d9e882e20744d253a5411a41073
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - IPO
  - MZmine2
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8260869565217391
n_claims: 23
---

# raw-metabolomics-quality-control-filtering

## Summary

QC filtering of untargeted metabolomics raw-data processing: parameter optimization to avoid noise inflation in the feature table, followed by quality-control filtering of features (as done in prior E. coli isotope-labeling validation work) to yield a clean, information-dense features table.

## When to use

Use when raw mzML/mzXML/CDF/qTOF, Orbitrap or FT-ICR data must be processed into a features table and QC filtering is needed to remove noise-inflating features before or after XCMS/MZmine2 centWave processing.

## When NOT to use

- The input is already a processed, noise-filtered features table rather than raw spectrometer output needing peak-picking and QC.
- The dataset is too small to estimate parameters robustly — AutoTuner requires at least 3 samples of raw data, and ~9–12 samples are recommended for stable CV < 0.1 estimates; below that the QC-parameter optimization is unreliable.
- The goal is feature/compound identification (e.g. annotation or isotope-label tracing) — the paper states identifying each distinct feature was beyond the scope of this study.

## Inputs

- Raw untargeted metabolomics files converted from proprietary formats (.mzML, .mzXML, or .CDF) from at least 3 samples
- Sample metadata spreadsheet with at least two columns: sample name and experimental factor
- Raw untargeted metabolomics experiment requiring a features table (peak-picking + retention time correction)

## Outputs

- Cleaned, QC-filtered features table (feature rows across samples) with reduced noise inflation
- Dataset-specific optimized centWave parameter set (7 continuous parameters + XCMS grouping parameter) for XCMS/MZmine2

## How to apply

Before feature detection, optimize the dataset-specific centWave parameters (AutoTuner estimates seven continuous centWave parameters plus the XCMS grouping parameter from raw data; mzDiff is the one continuous parameter not optimized, and a sensitivity analysis showed distinct values had minimal effect on the returned feature table). Parameter optimization must occur prior to analysis to avoid noise inflation within the feature table; AutoTuner reduces the total combinations of available centWave parameters from at least 24*5^8 choices to 40. Run peak-picking, grouping, and retention-time correction to generate the features table, then apply the stringent quality-control filtering validated in prior work on the E. coli metabolome after isotope-labeling experiments. Since no universal parameter set exists, do not skip optimization; instead recommend 9 to 12 samples (culture/community) to obtain parameter estimates with CV values less than 0.1, and support manual inspection of raw data as a quality-control check for the data generation steps leading up to the analysis.

## Related tools

- **AutoTuner** (Selects the seven continuous centWave peak-picking parameters plus the XCMS grouping parameter from raw data in a single statistical-inference step, preventing noise inflation before QC filtering.) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **XCMS** (Runs centWave peak detection, grouping, and retention-time correction on the raw files to produce the features table that is subsequently QC-filtered.)
- **MZmine2** (Alternative processing tool that consumes the AutoTuner-optimized centWave parameters for peak picking and feature table generation.)
- **IPO** (Baseline iterative parameter-optimization alternative to AutoTuner for comparison when validating parameter-selection QC outcomes.)

## Examples

```
BiocManager::install("Autotuner")
```

## Evaluation signals

- For most of the estimated parameters, check that coefficient of variation (CV) across the parameter-estimation samples is less than 0.1 (9–12 samples recommended).
- Confirm the optimized parameter set covers 7 of the 8 continuous centWave parameters (mzDiff excluded) plus the XCMS grouping parameter, reducing the combinatorial space from ≥24*5^8 to 40 choices.
- Compare the QC-filtered feature-table size/feature counts between AutoTuner- and IPO-derived parameter runs (AutoTuner-derived parameters generate smaller datasets with more information per feature).
- Manually inspect raw data as a quality-control check for the data generation steps preceding the analysis, per the paper's recommendation.

## Limitations

- AutoTuner is not parallelized; parallelization is a possible future improvement, so very large datasets may be slow to process.
- The divide-and-conquer speed gain comes at a loss of comparing EIC peaks across samples, which limits retention-time-correction estimation.
- The sliding window analysis requires user input during the first portion of AutoTuner.
- The Group difference parameter is estimated through a non-automatable cross-sample comparison during the TIC peak detection step, so its robustness could not be checked.
- No universal set of parameters exists for all datasets — optimization must be re-run per dataset before analysis to avoid noise inflation.

## Evidence

- [intro] No universal set of parameters exists for all datasets, hence parameter optimization must occur prior to analysis to avoid noise inflation within the feature table.: "No universal set of parameters exists for all datasets, hence parameter optimization must occur prior to analysis to avoid noise inflation within the feature table."
- [intro] AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS, and it determines a key parameter: "AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS, and it determines a key parameter"
- [discussion] For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1.: "For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1."
- [discussion] reduces the total combinations of available centWave parameters from at least 24* 58 possible choices of parameters to 40.: "reduces the total combinations of available centWave parameters from at least 24* 58 possible choices of parameters to 40."
- [discussion] We support manual inspection of the raw data, because it provides a quality control check for the data generation steps leading up to the analysis.: "We support manual inspection of the raw data, because it provides a quality control check for the data generation steps leading up to the analysis."
- [discussion] previous work validating the metabolome of Escherichia coli after performing stringent isotope labeling experiments and quality control filtering.: "previous work validating the metabolome of Escherichia coli after performing stringent isotope labeling experiments and quality control filtering."
- [readme] For input, AutoTuner requires at least 3 samples of raw data converted from proprietary instrument formats (eg .mzML, .mzXML, or .CDF).: [evidence span withheld — anchor status: external_doc]
- [discussion] In regards to the last continuous parameter not optimized by AutoTuner, mzDiff, we performed a sensitivity analysis to show that distinct values had minimal effect on the returned feature table: "In regards to the last continuous parameter not optimized by AutoTuner, mzDiff, we performed a sensitivity analysis to show that distinct values had minimal effect on the returned feature table"
