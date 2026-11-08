---
name: benjamini-hochberg-multiple-testing-correction
description: Use when many metabolite features (e.g. bile-amidate feature rows in
  a relative-quantification table) have each been tested with a Wilcoxon rank-sum
  test, producing a family of raw p-values that must be FDR-adjusted before calling
  disease associations.
when_to_use_negative:
- Only a single hypothesis test is being run (no multiplicity, so no correction is
  needed)
- Input p-values are already adjusted q-values from a previous correction step — re-correcting
  would over-shrink significance
- The goal is hypothesis-wide family-wise error control (e.g. Bonferroni-style) rather
  than FDR control over feature-level metabolomics tests
tools:
- name: statsmodels.stats.multitest.multipletests
  role: Computes Benjamini-Hochberg-adjusted q-values from the raw p-value family
    with method='fdr_bh'
- name: SciPy scipy.stats.mannwhitneyu
  role: Produces the raw two-sided p-values that are fed into the BH correction, one
    per bile-amidate feature per group pair
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:c5c2de69dd84ce0196861059afaa8fc8d116c8ba07b9a66826171e9835a9ae17
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy scipy.stats.mannwhitneyu
  - conda
  - statsmodels.stats.multitest.multipletests
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Applies Benjamini-Hochberg false-discovery-rate correction to raw p-values from pairwise non-parametric tests so that metabolite-feature-level comparisons (e.g. bile-amidate abundances in IBD vs non-IBD) can be called significant at a controlled q-value threshold rather than at unadjusted alpha.

## When to use

Use when many metabolite features (e.g. bile-amidate feature rows in a relative-quantification table) have each been tested with a Wilcoxon rank-sum test, producing a family of raw p-values that must be FDR-adjusted before calling disease associations.

## When NOT to use

- Only a single hypothesis test is being run (no multiplicity, so no correction is needed)
- Input p-values are already adjusted q-values from a previous correction step — re-correcting would over-shrink significance
- The goal is hypothesis-wide family-wise error control (e.g. Bonferroni-style) rather than FDR control over feature-level metabolomics tests

## Inputs

- Vector/list of raw two-sided Wilcoxon rank-sum p-values for one pairwise group comparison (e.g. non-IBD vs CD) across bile-amidate features
- Feature table rows for confirmed bile-amidate features (cholic acid and secondary bile acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp, Tyr and related amino acids) from Metabolomics Workbench studies PR000639 (iHMP2) and PR000677 (PRISM)

## Outputs

- BH-adjusted q-value per bile-amidate feature per comparison (q < 0.05 treated as significant)
- Rows written to bile_amidate_wilcoxon_results.csv containing feature identifier, conjugate assignment, test group pair, raw p-value, BH-adjusted q-value, and group median direction

## How to apply

After running two-sided Wilcoxon rank-sum tests (SciPy scipy.stats.mannwhitneyu, alternative='two-sided') on each bile-amidate feature for a fixed group pair — non-IBD vs CD, non-IBD vs UC, or CD vs UC — collect that family of raw p-values into a vector. Pass the vector to statsmodels.stats.multitest.multipletests with method='fdr_bh' to obtain Benjamini-Hochberg-adjusted q-values; apply the correction within each comparison so each group pair has its own FDR-controlled family. Call a feature significantly changed when its q-value is below 0.05, and record the direction from the group medians (e.g. higher in CD). The validation target from this study is that cholic-acid conjugates to Glu, Ile/Leu, Phe, Thr, Trp and Tyr reach q < 0.05 with positive direction in CD vs non-IBD, while secondary bile acid amidates stay non-significant.

## Related tools

- **statsmodels.stats.multitest.multipletests** (Computes Benjamini-Hochberg-adjusted q-values from the raw p-value family with method='fdr_bh')
- **SciPy scipy.stats.mannwhitneyu** (Produces the raw two-sided p-values that are fed into the BH correction, one per bile-amidate feature per group pair)

## Examples

```
from statsmodels.stats.multitest import multipletests; reject, qvals, _, _ = multipletests(raw_pvalues, method='fdr_bh')
```

## Evaluation signals

- Every output row carries a q-value in [0,1] and q >= p for each feature (BH guarantees adjusted values never below the raw p-values)
- Within each comparison, q-values are monotonically non-decreasing when features are ordered by raw p-value
- Cholic-acid conjugates to Glu, Ile/Leu, Phe, Thr, Trp and Tyr come out q < 0.05 with positive direction (higher in CD) versus non-IBD
- Secondary bile acid amidates remain non-significant (q >= 0.05) after correction, matching the four-cohort validation pattern
- CSV schema check: each line has feature identifier, conjugate assignment, group pair, raw p, BH q, and median direction

## Limitations

- BH correction controls the expected false discovery rate, not family-wise error, so individual significant features may still be false positives
- MS/MS-based identifications do not guarantee against false discovery of closely related isomers or compounds, so significant hits are structural assignments of limited certainty without NMR/X-ray confirmation
- Corrections are applied per comparison; results can differ if the p-value family is pooled across group pairs or cohorts instead

## Evidence

- [abstract] Validation using four distinct human IBD cohorts showed that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn's disease.: [evidence span withheld — anchor status: not_found]
- [discussion] Reverse metabolomics targeted three metabolite classes and found disease associations.: "Here we targeted three metabolite classes of interest and searched for around 2,000 unique compounds in public data and found some relevant disease associations."
- [discussion] MS/MS matching does not guarantee against false discovery of closely related isomers.: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
