---
name: ms2-spectrum-count-normalization
description: Use when comparing untargeted metabolomics feature tables produced with
  different centWave parameter sets (AutoTuner- vs IPO-derived) and you need to test
  whether a feature subset is depleted for MS2 spectra after controlling for subset
  size.
when_to_use_negative:
- Input is a single feature table (no comparison of parameter-optimization-derived
  feature sets) — the partition into AutoTuner-specific/IPO-specific/intersect sets
  is required for the depletion test
- Feature identity/annotation, not MS2 spectral coverage, is the analysis goal — the
  source study states 'identifying each distinct feature was beyond the scope of this
  study'
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: R
  role: Runs the hypergeometric depletion test via stats::phyper and compiles the
    enrichment results table
- name: AutoTuner
  role: Provides the AutoTuner-derived centWave parameters that generate the reference
    feature table against which IPO-specific features are compared
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: IPO
  role: Provides the comparison (isotopologue parameter optimization) feature table
    whose unique features are tested for MS2-spectrum depletion
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:c4c19131915b998dc751e988726df881db49f09ce3363d1583053f87170b6b07
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - IPO
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ms2-spectrum-count-normalization

## Summary

Assesses whether features unique to one parameter-optimization route (e.g. IPO-specific vs AutoTuner/intersect features in XCMS/centWave feature tables) are enriched or depleted for MS2 spectra, using size-normalized spectrum counts and a hypergeometric test run separately per ionization mode.

## When to use

Use when comparing untargeted metabolomics feature tables produced with different centWave parameter sets (AutoTuner- vs IPO-derived) and you need to test whether a feature subset is depleted for MS2 spectra after controlling for subset size.

## When NOT to use

- Input is a single feature table (no comparison of parameter-optimization-derived feature sets) — the partition into AutoTuner-specific/IPO-specific/intersect sets is required for the depletion test
- Feature identity/annotation, not MS2 spectral coverage, is the analysis goal — the source study states 'identifying each distinct feature was beyond the scope of this study'

## Inputs

- XCMS/centWave feature table generated with AutoTuner-derived parameters (per ionization mode)
- XCMS/centWave feature table generated with IPO-derived parameters (per ionization mode)
- MS2 spectrum counts associated with each feature, per ionization mode

## Outputs

- ms2_enrichment_results.csv containing normalized MS2-spectrum counts, enrichment ratios, and hypergeometric p-values for AutoTuner-specific, IPO-specific, and intersect feature sets in both positive and negative ionization modes

## How to apply

Load the dataset feature tables generated with AutoTuner-derived and IPO-derived XCMS/centWave parameters, separately for positive and negative ionization modes. Partition features into AutoTuner-specific, IPO-specific, and intersect (shared) sets, and count the MS2 spectra associated with each set. Because subset sizes differ, normalize MS2-spectrum counts by the total number of features in each set before comparing enrichment ratios. Then run a hypergeometric test with R's stats::phyper, treating the intersect as the background population, to assess whether IPO-specific features are depleted for MS2 spectra relative to the intersect. Compile normalized counts, enrichment ratios, and p-values per ionization mode into a results table (ms2_enrichment_results.csv). In the AutoTuner culture-dataset validation, this test yielded p < 1e-10 for IPO-specific depletion in both modes, consistent with AutoTuner-derived parameters producing smaller feature tables with more information per feature.

## Related tools

- **R** (Runs the hypergeometric depletion test via stats::phyper and compiles the enrichment results table)
- **AutoTuner** (Provides the AutoTuner-derived centWave parameters that generate the reference feature table against which IPO-specific features are compared) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **IPO** (Provides the comparison (isotopologue parameter optimization) feature table whose unique features are tested for MS2-spectrum depletion)

## Examples

```
p_ipo_depletion <- phyper(q = ms2_ipo_specific - 1, m = n_intersect, n = n_ipo_specific, k = ms2_total, lower.tail = FALSE)
```

## Evaluation signals

- Feature sets partition exhaustively into AutoTuner-specific, IPO-specific, and intersect (shared) with no features lost in the partition
- Normalized MS2-spectrum counts (spectra divided by total features per set) are computed for each set in each ionization mode before ratio comparison
- Hypergeometric p-values are reported separately for positive and negative ionization modes, not pooled
- Reproducing the culture dataset should yield IPO-specific depletion with hypergeometric p < 1e-10 in both ionization modes

## Limitations

- The test assumes the intersect feature set is the appropriate background population; if neither parameter set is a reference standard, depletion may reflect parameter differences rather than feature quality
- Identifying each distinct feature was beyond the scope of the source study, so MS2-spectrum depletion cannot be attributed to specific compound classes without additional annotation
- The paucity of size-validated metabolome datasets precludes further evaluation of the feature-number comparison underlying the enrichment test

## Evidence

- [abstract] in comparison to isotopologue parameter optimization (IPO), the most commonly-used parameter selection tool: "in comparison to isotopologue parameter optimization (IPO), the most commonly-used parameter selection tool"
- [discussion] AutoTuner-derived parameters generate smaller datasets with more information per feature: "these results suggest that AutoTuner-derived parameters generate smaller datasets with more information per feature."
- [readme] AutoTuner is used to identify dataset specific parameters to process untargeted metabolomics data.: [evidence span withheld — anchor status: external_doc]
