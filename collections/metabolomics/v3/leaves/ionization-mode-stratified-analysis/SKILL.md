---
name: ionization-mode-stratified-analysis
description: Use when a feature table is split by ionization mode (ESI+ vs ESI−) and
  you must compare feature subsets — e.g. tool-specific vs shared features — for MS2
  spectral enrichment.
when_to_use_negative:
- The input is already a single merged feature table without positive/negative ionization
  mode labels — the mode-stratified partitioning cannot be performed.
- The goal is simply to pick peak-picking parameters (AutoTuner's core task); this
  skill compares downstream feature-table subsets, not parameters.
- The dataset lacks MS2 spectra for the features of interest, since enrichment/depletion
  cannot be assessed without spectral counts.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: AutoTuner
  role: Generates the dataset-specific centWave parameters whose derived feature tables
    are compared between subsets and ionization modes
  repo: https://github.com/crmclean/Autotuner
- name: IPO
  role: Alternative parameter-selection tool whose specific (non-intersect) features
    form the depletion test group
- name: XCMS
  role: centWave peak-picking/alignment engine that produces the feature tables being
    partitioned per ionization mode
- name: R
  role: Hosts the hypergeometric test (stats::phyper) and CSV export of normalized
    counts and p-values
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:aa65df9294ef8e8f116375f40d63ec644e3532f7a7b733881179800ffa161aba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/crmclean/Autotuner
  tools:
  - AutoTuner
  - IPO
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ionization-mode-stratified-analysis

## Summary

Analyze untargeted metabolomics feature tables and MS2 spectral counts separately for positive and negative ionization modes, comparing parameter-selection subsets (AutoTuner-specific, IPO-specific, intersect) with size-normalized spectral enrichment and hypergeometric tests.

## When to use

Use when a feature table is split by ionization mode (ESI+ vs ESI−) and you must compare feature subsets — e.g. tool-specific vs shared features — for MS2 spectral enrichment.

## When NOT to use

- The input is already a single merged feature table without positive/negative ionization mode labels — the mode-stratified partitioning cannot be performed.
- The goal is simply to pick peak-picking parameters (AutoTuner's core task); this skill compares downstream feature-table subsets, not parameters.
- The dataset lacks MS2 spectra for the features of interest, since enrichment/depletion cannot be assessed without spectral counts.

## Inputs

- Raw LC-MS/MS data converted from proprietary instrument formats (.mzML, .mzXML, or .CDF)
- Feature tables generated with AutoTuner-derived XCMS/centWave parameters, per ionization mode
- Feature tables generated with IPO-derived XCMS/centWave parameters, per ionization mode
- MS2 spectra associated with each feature in each ionization mode

## Outputs

- Per-mode (positive/negative) partition of features into AutoTuner-specific, IPO-specific, and intersect subsets
- MS2-spectrum counts normalized by feature counts for each subset and ionization mode
- Hypergeometric depletion p-values per ionization mode (e.g. p < 10−10 in both negative and positive modes)
- Compiled results table such as ms2_enrichment_results.csv

## How to apply

Process raw untargeted metabolomics data (qTOF, orbitrap, FT-ICR; .mzML/.mzXML/.CDF inputs) into feature tables independently per ionization mode, partitioning features into AutoTuner-specific, IPO-specific, and intersect sets. In each mode, count MS2 spectra associated with features in each subset and normalize those counts by the total number of features in the subset so subset size does not drive the comparison. Run a hypergeometric test in R (stats::phyper) asking whether the IPO-specific features are depleted for MS2 spectra relative to the intersect, reporting results per ionization mode. Compile normalized counts, enrichment ratios, and p-values into a results table (e.g. ms2_enrichment_results.csv) and treat the mode-stratified depletion as a single analysis only when both modes agree (e.g. both reaching p < 10−10).

## Related tools

- **AutoTuner** (Generates the dataset-specific centWave parameters whose derived feature tables are compared between subsets and ionization modes) — https://github.com/crmclean/Autotuner
- **IPO** (Alternative parameter-selection tool whose specific (non-intersect) features form the depletion test group)
- **XCMS** (centWave peak-picking/alignment engine that produces the feature tables being partitioned per ionization mode)
- **R** (Hosts the hypergeometric test (stats::phyper) and CSV export of normalized counts and p-values) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html

## Examples

```
Rscript -e 'p <- phyper(q, m, n - m, k, lower.tail = TRUE); write.csv(data.frame(mode, subset, n_features, n_ms2, ratio, p), "ms2_enrichment_results.csv")'
```

## Evaluation signals

- Feature counts for AutoTuner-specific, IPO-specific, and intersect subsets sum to the total feature count within each ionization mode (no feature double-counted or dropped).
- Normalized MS2-spectrum counts (counts / total features per subset) are computed for every subset × mode cell, not just raw counts.
- stats::phyper returns a p-value per ionization mode, and the depletion is significant (p < 1e-10) in both negative and positive modes for the IPO-specific group.
- ms2_enrichment_results.csv contains per-mode rows with normalized counts, enrichment ratios, and hypergeometric p-values for all three subsets.
- Parameter sets used to generate the compared tables came from AutoTuner and IPO on the same raw samples, so subset differences reflect parameter choice, not input data.

## Limitations

- AutoTuner estimates seven continuous centWave parameters but not mzDiff, whose sensitivity was checked separately; differences in mzDiff could contribute to mode-specific feature sets.
- The Group difference parameter is estimated via a non-automatable cross-sample comparison during TIC peak detection, so its robustness could not be fully checked.
- AutoTuner's divide-and-conquer approach does not compare EIC peaks across samples for retention-time correction, which could affect downstream feature alignment per mode.
- The sliding window analysis needs user input in the first portion of AutoTuner; mis-set inputs could shift subset boundaries before the per-mode enrichment analysis.
- The study did not identify individual features, so enrichment reflects spectral support rather than compound-level annotation confidence.

## Evidence

- [intro] AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS, and it determines a key parameter: "AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS, and it determines a key parameter"
- [readme] AutoTuner requires at least 3 samples of raw data converted from proprietary instrument formats (eg .mzML, .mzXML, or .CDF): [evidence span withheld — anchor status: external_doc]
- [intro] AutoTuner is freely distributed through BioConductor as an R package: "AutoTuner is freely distributed through BioConductor as an R package."
- [abstract] We performed a Monte Carlo experiment to test the robustness of AutoTuner parameter selection, and found that AutoTuner generated similar parameter estimates from random subsets of samples: "We performed a Monte Carlo experiment to test the robustness of AutoTuner parameter selection, and found that AutoTuner generated similar parameter estimates from random subsets of samples."
- [discussion] may eliminate the need for user input during the first portion of AutoTuner: "may eliminate the need for user input during the first portion of AutoTuner"
