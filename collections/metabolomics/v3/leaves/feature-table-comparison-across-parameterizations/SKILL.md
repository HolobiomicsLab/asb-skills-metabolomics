---
name: feature-table-comparison-across-parameterizations
description: Use when two feature tables were produced from identical raw LC-MS data
  with different parameter-optimization tools (e.g., AutoTuner vs IPO centWave settings)
  and you must quantify overlap, subset-specific features, or per-subset MS2 spectra
  depletion/enrichment.
when_to_use_negative:
- Input is raw instrument data (.mzML/.mzXML/.CDF) rather than already-generated feature
  tables — parameter estimation (e.g., AutoTuner) must run first
- The two feature tables were generated with the same parameter set, so the tool-specific/intersect
  partitioning has no contrast to test
- The goal is peak-picking parameter estimation itself (e.g., determining centWave
  parameters) rather than evaluating downstream tables — use the AutoTuner parameter-estimation
  skill instead
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_2269
tools:
- name: AutoTuner
  role: 'Source of one feature table: provides dataset-specific centWave parameters
    used to generate the AutoTuner-derived feature set being compared'
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: IPO
  role: 'Source of the comparison feature table: iterative parameter optimization
    supplying the IPO-derived centWave parameters for the reference feature set'
- name: R
  role: Analysis environment for partitioning feature sets, counting MS2 spectra,
    and running stats::phyper hypergeometric tests
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:6d50e4b4c5db1d03075b6b23e7b2524cdc32c93275475bfc8a5f63c4e9bde422
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - IPO
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Feature-table comparison across parameterizations

## Summary

Quantitatively compares feature tables generated from the same untargeted metabolomics raw data under different peak-picking parameterizations (e.g., AutoTuner-derived vs IPO-derived XCMS/centWave parameters), partitioning features into tool-specific and intersect sets and testing for differences such as MS2 spectral enrichment.

## When to use

Use when two feature tables were produced from identical raw LC-MS data with different parameter-optimization tools (e.g., AutoTuner vs IPO centWave settings) and you must quantify overlap, subset-specific features, or per-subset MS2 spectra depletion/enrichment.

## When NOT to use

- Input is raw instrument data (.mzML/.mzXML/.CDF) rather than already-generated feature tables — parameter estimation (e.g., AutoTuner) must run first
- The two feature tables were generated with the same parameter set, so the tool-specific/intersect partitioning has no contrast to test
- The goal is peak-picking parameter estimation itself (e.g., determining centWave parameters) rather than evaluating downstream tables — use the AutoTuner parameter-estimation skill instead

## Inputs

- Feature tables from XCMS/centWave processing of qTOF/orbitrap/FT-ICR raw data (.mzML, .mzXML, or .CDF inputs) using AutoTuner-derived parameters
- Feature tables from the same raw data processed with IPO-derived centWave parameters
- Per-feature MS2 spectrum counts for each ionization mode (positive and negative)

## Outputs

- Partitioned feature sets (AutoTuner-specific, IPO-specific, intersect) per ionization mode
- MS2-spectrum counts normalized by total features per subset
- Enrichment ratios (IPO-specific vs intersect)
- Hypergeometric p-values per ionization mode
- Compiled results table, e.g., ms2_enrichment_results.csv

## How to apply

Load the culture-dataset feature tables produced with AutoTuner-derived and IPO-derived XCMS/centWave parameters for both positive and negative ionization modes. Partition features into AutoTuner-specific, IPO-specific, and intersect (shared) sets per ionization mode, and count associated MS2 spectra in each set. Normalize MS2-spectrum counts by the total number of features in each subset to control for subset size. Run a hypergeometric test in R (stats::phyper) assessing whether IPO-specific features are depleted for MS2 spectra relative to the intersect, separately per mode; the reference result was hypergeometric p < 10^-10 in both modes. Compile normalized counts, enrichment ratios, and p-values into a results table (e.g., ms2_enrichment_results.csv).

## Related tools

- **AutoTuner** (Source of one feature table: provides dataset-specific centWave parameters used to generate the AutoTuner-derived feature set being compared) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **IPO** (Source of the comparison feature table: iterative parameter optimization supplying the IPO-derived centWave parameters for the reference feature set)
- **R** (Analysis environment for partitioning feature sets, counting MS2 spectra, and running stats::phyper hypergeometric tests)

## Examples

```
p <- stats::phyper(q = n_ms2_ipo - 1, m = n_ipo_features, n = n_intersect_features, k = n_ms2_total, lower.tail = FALSE)  # R
```

## Evaluation signals

- Hypergeometric p-values for IPO-specific MS2 depletion should be < 1e-10 in both positive and negative ionization modes (reference result)
- Partition completeness: AutoTuner-specific + IPO-specific + intersect feature counts must sum to the total feature count in each ionization mode
- Normalized MS2-spectrum counts (spectra per feature) should show IPO-specific subsets lower than the intersect subset (enrichment ratio < 1)
- Identical raw-data input files must be used for both parameterizations; any feature-count difference must be attributable to parameters, not input divergence
- Results table (e.g., ms2_enrichment_results.csv) should contain normalized counts, enrichment ratios, and p-values for both ionization modes

## Limitations

- Identifying each distinct feature was beyond the scope of the source study — the comparison quantifies feature counts and MS2 enrichment, not feature identity/annotation
- The paucity of size-validated metabolome datasets precludes further evaluation of the feature number comparison
- IPO-derived results may reflect IPO-specific parameter choices, e.g., the peak-width of missing standards falling below the Minimum Peak-width parameter selected by IPO, so differences are parameterization-dependent

## Evidence

- [discussion] these results suggest that AutoTuner-derived parameters generate smaller datasets with more information per feature: "While identifying each distinct feature was beyond the scope of this study, these results suggest that AutoTuner-derived parameters generate smaller datasets with more information per feature."
- [discussion] centWave requires tuning of 11 distinct parameters: "the centWave algorithm, used by both XCMS and MZmine2 data processing tools, requires tuning of 11 distinct parameters"
- [discussion] IPO Minimum Peak-width explanation: "One possible explanation for the lower accuracy of IPO is that the peak-width of the missing standards was below the Minimum Peak-width parameter selected by IPO"
- [readme] AutoTuner identifies dataset specific parameters to process untargeted metabolomics data: [evidence span withheld — anchor status: external_doc]
