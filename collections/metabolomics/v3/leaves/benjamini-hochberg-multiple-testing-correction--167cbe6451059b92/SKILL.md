---
name: benjamini-hochberg-multiple-testing-correction
description: Use when a differential analysis of grouped NMR peak features produces
  thousands of per-feature p-values and you need FDR-controlled significance calls
  comparable across frameworks such as speaq 2.0 and MetaboAnalyst.
when_to_use_negative:
- Input is already a table of FDR-adjusted p-values from MetaboAnalyst — re-correcting
  would be inappropriate
- The analysis involves a single pre-specified hypothesis rather than a family of
  hundreds of grouped-peak tests, where no multiplicity correction is needed
- The spectra are still raw and unbinned; BH correction presupposes the peak-picking
  and grouping steps (avoiding binning) have already produced the feature matrix
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3301
tools:
- name: speaq 2.0
  role: Produces the 677-feature peak/sample/value matrix whose differential analysis
    p-values are BH-corrected
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R implementation in which the FDR-corrected differential analysis of grouped
    peaks is performed
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Environment for running the BH correction (e.g. p.adjust with method='BH')
    on the p-value vectors
- name: MetaboAnalyst
  role: Alternative framework generating the 245-feature matrix whose differential
    p-values are BH-corrected for the comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:b92630841c4b2361a985cfdba1c8d935f988a301b2a72aae315cd30cfbb34e6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - MetaboAnalyst
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Apply Benjamini-Hochberg FDR correction to per-feature p-values from differential NMR peak analyses (e.g. speaq 2.0's 677-feature matrix vs MetaboAnalyst's 245-feature matrix on the onion intake in mice data) so highly significant features can be flagged and cross-mapped without inflating false discoveries across the thousands of grouped peaks.

## When to use

Use when a differential analysis of grouped NMR peak features produces thousands of per-feature p-values and you need FDR-controlled significance calls comparable across frameworks such as speaq 2.0 and MetaboAnalyst.

## When NOT to use

- Input is already a table of FDR-adjusted p-values from MetaboAnalyst — re-correcting would be inappropriate
- The analysis involves a single pre-specified hypothesis rather than a family of hundreds of grouped-peak tests, where no multiplicity correction is needed
- The spectra are still raw and unbinned; BH correction presupposes the peak-picking and grouping steps (avoiding binning) have already produced the feature matrix

## Inputs

- Vector of raw per-feature p-values from the speaq 2.0 differential analysis on the 677-feature grouped peak matrix
- Vector of raw per-feature p-values from the MetaboAnalyst differential analysis on the 245-feature grouped peak matrix
- Group/condition factor labeling the dietary groups of the onion intake in mice NMR samples

## Outputs

- BH-adjusted (FDR-corrected) p-value vector per feature matrix
- List of highly significant features per framework (speaq 2.0 vs MetaboAnalyst)
- Cross-map table (e.g. Table 2) recording which MetaboAnalyst features are covered by speaq 2.0 counterparts

## How to apply

After obtaining the peak/sample/value feature matrices — 677 features from the speaq 2.0 moving-window grouping (0.03 ppm window, 0.015 ppm steps) and 245 features from MetaboAnalyst's identical grouping parameters on the onion intake in mice peak data — run the per-feature differential abundance test between the dietary groups. Then apply Benjamini-Hochberg correction over the full vector of p-values for each matrix separately to derive FDR-adjusted p-values. Flag 'highly significant' features from each corrected list and cross-map the significant MetaboAnalyst features onto the significant speaq 2.0 features (e.g. Table 2 of the comparison study). The rationale is that peak grouping yields hundreds of simultaneously tested features, so raw p-values would overstate significance; BH correction keeps the false discovery rate controlled while remaining compatible with the article's finding that every highly significant MetaboAnalyst feature has at least one highly significant speaq 2.0 counterpart.

## Related tools

- **speaq 2.0** (Produces the 677-feature peak/sample/value matrix whose differential analysis p-values are BH-corrected) — https://github.com/beirnaert/speaq
- **speaq R package** (R implementation in which the FDR-corrected differential analysis of grouped peaks is performed) — https://cran.r-project.org/package=speaq
- **R** (Environment for running the BH correction (e.g. p.adjust with method='BH') on the p-value vectors)
- **MetaboAnalyst** (Alternative framework generating the 245-feature matrix whose differential p-values are BH-corrected for the comparison)

## Examples

```
adj.p <- p.adjust(feature_pvals, method = "BH")
```

## Evaluation signals

- Every highly significant MetaboAnalyst feature matches at least one highly significant speaq 2.0 feature in the cross-map (Table 2 coverage check)
- Feature counts reproduce the expected resolution difference: 677 features in the speaq 2.0 matrix vs 245 in the MetaboAnalyst matrix
- Adjusted p-values are monotonically >= raw p-values for each feature, as guaranteed by the BH procedure
- The same grouping parameters (0.03 ppm window, 0.015 ppm steps) were used for both frameworks before differential testing, so p-value families are comparable
- Flagging of 'highly significant' features uses a consistent FDR threshold across both the speaq 2.0 and MetaboAnalyst result sets

## Limitations

- MetaboAnalyst's lower resolution (245 vs 677 features) arises because it sums close peaks together, removing multiplet information — cross-mapped significance calls are therefore coarser than speaq 2.0's
- The correction was applied to the onion intake in mice comparison with two dietary groups; behavior for more than two groups is supported by the package's differential analysis but not exercised by this comparison
- BH correction controls FDR across the whole feature family, so significance thresholds for the 245-feature and 677-feature matrices are not directly interchangeable in stringency

## Evidence

- [abstract] starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [intro] complement MetaboAnalyst: "to construct a method that can complement established tools for NMR data analysis like MetaboAnalyst [12], by improving performance"
- [supplementary] onion dataset: "speaq 2.0 workflow applied to the onion intake in mice data [37]."
- [readme] differential analysis capability: [evidence span withheld — anchor status: external_doc]
