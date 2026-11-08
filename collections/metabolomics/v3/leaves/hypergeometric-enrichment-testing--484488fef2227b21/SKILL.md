---
name: hypergeometric-enrichment-testing
description: Use when comparing two feature tables generated with different XCMS/centWave
  parameters (AutoTuner vs IPO) and you need to test whether a parameter-specific
  subset is enriched or depleted for MS2 spectra.
when_to_use_negative:
- When the input is already a single feature table without a comparison subset (e.g.
  no AutoTuner vs IPO contrast) — there is no hypergeometric population/success structure
  to test
- When MS2 spectrum counts have not been generated or cannot be linked to individual
  features — the test requires per-feature MS2 association
- When features cannot be partitioned into specific vs intersect sets (e.g. no shared
  identifiers between the two tables) — normalization and the hypergeometric denominator
  would be undefined
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AutoTuner
  role: Source of the parameter-optimized feature tables whose MS2-spectral content
    is compared against IPO-derived tables
  repo: https://github.com/crmclean/Autotuner
- name: IPO
  role: Reference parameter-optimization tool whose feature tables provide the comparison
    subset in the enrichment test
- name: R
  role: Environment for partitioning features, counting MS2 spectra, and running stats::phyper
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:35ff1304a82181e0473f579cf42b8bcb711ba826882cf4dbff3603d4791a5af0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/crmclean/Autotuner
  tools:
  - AutoTuner
  - IPO
  - R
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hypergeometric-enrichment-testing

## Summary

Assess whether IPO-specific features in a culture dataset are depleted of MS2 spectra relative to the intersect of AutoTuner- and IPO-derived feature tables, using normalized counts and a hypergeometric test (stats::phyper) per ionization mode.

## When to use

Use when comparing two feature tables generated with different XCMS/centWave parameters (AutoTuner vs IPO) and you need to test whether a parameter-specific subset is enriched or depleted for MS2 spectra.

## When NOT to use

- When the input is already a single feature table without a comparison subset (e.g. no AutoTuner vs IPO contrast) — there is no hypergeometric population/success structure to test
- When MS2 spectrum counts have not been generated or cannot be linked to individual features — the test requires per-feature MS2 association
- When features cannot be partitioned into specific vs intersect sets (e.g. no shared identifiers between the two tables) — normalization and the hypergeometric denominator would be undefined

## Inputs

- Feature tables from untargeted metabolomics data processed with AutoTuner-derived and IPO-derived XCMS/centWave parameters (culture dataset)
- MS2 spectrum counts associated with each feature, per ionization mode (positive and negative)

## Outputs

- ms2_enrichment_results.csv containing normalized MS2 counts, enrichment ratios, and hypergeometric p-values for AutoTuner-specific, IPO-specific, and intersect feature subsets per ionization mode

## How to apply

Load the culture-dataset feature tables produced with AutoTuner-derived and IPO-derived XCMS/centWave parameters for both negative and positive ionization modes. Partition features into AutoTuner-specific, IPO-specific, and intersect (shared) sets, and count the MS2 spectra associated with each set per ionization mode. Normalize MS2-spectrum counts by the total number of features in each set so that subset size does not confound the comparison. Then run a hypergeometric test in R using stats::phyper to assess whether IPO-specific features are depleted for MS2 spectra relative to the intersect, separately in each ionization mode. The expected result is significant depletion (p < 10⁻¹⁰) in both negative and positive ionization modes, which supports the interpretation that AutoTuner-derived parameters yield fewer, more informative features. Compile normalized counts, enrichment ratios, and p-values into ms2_enrichment_results.csv.

## Related tools

- **AutoTuner** (Source of the parameter-optimized feature tables whose MS2-spectral content is compared against IPO-derived tables) — https://github.com/crmclean/Autotuner
- **IPO** (Reference parameter-optimization tool whose feature tables provide the comparison subset in the enrichment test)
- **R** (Environment for partitioning features, counting MS2 spectra, and running stats::phyper) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html

## Examples

```
phyper(q = n_IPO_specific_with_MS2 - 1, m = n_intersect_with_MS2, n = n_total_features - n_intersect, k = n_IPO_specific)
```

## Evaluation signals

- ms2_enrichment_results.csv exists and contains normalized MS2 counts, enrichment ratios, and hypergeometric p-values for each of AutoTuner-specific, IPO-specific, and intersect subsets in both ionization modes
- Hypergeometric p-value for IPO-specific MS2 depletion is < 1e-10 in both negative and positive ionization modes
- MS2 spectrum counts are normalized by the total number of features in each subset (i.e. the ratio, not raw counts, drives the comparison)
- Feature subsets (AutoTuner-specific, IPO-specific, intersect) partition the combined feature space without overlap

## Limitations

- The test is grounded in the culture dataset; the article notes a paucity of size-validated metabolome datasets precludes broader evaluation of the feature-number comparison
- Identifying each distinct feature was beyond the scope of this study, so enrichment is assessed at the feature-count level rather than at the level of annotated metabolites
- AutoTuner-derived parameters are expected to yield smaller feature tables with more information per feature; the enrichment test does not itself identify which features are correct

## Evidence

- [results] IPO-specific features were significantly depleted for MS2 spectra relative to the intersect region, with hypergeometric test p < 10−10 in both negative and positive ionization modes.: [evidence span withheld — anchor status: not_found]
- [methods] Normalize MS2-spectrum counts by the total number of features in each set: [evidence span withheld — anchor status: not_found]
- [discussion] AutoTuner-derived parameters generate smaller datasets with more information per feature: "these results suggest that AutoTuner-derived parameters generate smaller datasets with more information per feature."
