---
name: wilcoxon-rank-sum-group-comparison
description: Use when comparing relative-abundance metabolomics features (e.g. bile-amidate
  intensities) across three unordered phenotype groups (non-IBD control, CD, UC) without
  assuming normality, and per-feature adjusted p-values are needed.
when_to_use_negative:
- Input is raw MS/MS spectra or mzML files rather than a processed relative-quantification
  feature table
- The metabolites of interest lack prior structural confirmation (feature-to-conjugate
  assignment), e.g. candidates only found via MASST spectral matching without synthesis
  validation
- Groups have paired or matched designs requiring a signed-rank test rather than the
  independent-sample rank-sum test
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: conda
  role: Environment manager used to install and run the Python statistical stack (pandas,
    SciPy, statsmodels) locally, per the paper's workflow tooling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:627795f7264a6b9a9b145365bbe76562d4b25b62a29c53fdb87ccb89c9eabacd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# wilcoxon-rank-sum-group-comparison

## Summary

Non-parametric two-sided Wilcoxon rank-sum (Mann–Whitney) testing of metabolite feature abundances across phenotype groups, with Benjamini–Hochberg FDR correction, used here to validate that microbial bile acid amidates are elevated in Crohn's disease in the iHMP2 and PRISM cohorts.

## When to use

Use when comparing relative-abundance metabolomics features (e.g. bile-amidate intensities) across three unordered phenotype groups (non-IBD control, CD, UC) without assuming normality, and per-feature adjusted p-values are needed.

## When NOT to use

- Input is raw MS/MS spectra or mzML files rather than a processed relative-quantification feature table
- The metabolites of interest lack prior structural confirmation (feature-to-conjugate assignment), e.g. candidates only found via MASST spectral matching without synthesis validation
- Groups have paired or matched designs requiring a signed-rank test rather than the independent-sample rank-sum test

## Inputs

- Relative-quantification feature table from Metabolomics Workbench study PR000639 (iHMP2 stool cohort)
- Relative-quantification feature table from Metabolomics Workbench study PR000677 (PRISM IBD fecal cohort)
- Sample-level IBD phenotype metadata (non-IBD control, Crohn's disease, ulcerative colitis)
- List of confirmed bile-amidate feature identifiers with amino-acid conjugate assignments

## Outputs

- bile_amidate_wilcoxon_results.csv with columns: feature identifier, conjugate assignment, test group pair, raw p-value, BH-adjusted q-value, and group median directions
- Per-comparison q-value vectors from Benjamini–Hochberg FDR correction

## How to apply

Download the public relative-quantification feature tables for Metabolomics Workbench studies PR000639 (iHMP2 stool) and PR000677 (PRISM fecal), parse them with pandas, and subset rows to the confirmed bile-amidate features (cholic acid and secondary bile acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp, Tyr and related amino acids). Stratify samples by the IBD phenotype metadata into non-IBD control, Crohn's disease (CD), and ulcerative colitis (UC). For each feature, run two-sided tests with scipy.stats.mannwhitneyu (alternative='two-sided') for each pairwise comparison: non-IBD vs CD, non-IBD vs UC, and CD vs UC. Apply Benjamini–Hochberg correction within each comparison using statsmodels.stats.multitest.multipletests(method='fdr_bh') to obtain q-values, then record feature ID, conjugate assignment, group pair, raw p, adjusted q, and group median direction. Significance is judged at q < 0.05 with the expected positive direction (higher abundance in CD vs non-IBD).

## Related tools

- **conda** (Environment manager used to install and run the Python statistical stack (pandas, SciPy, statsmodels) locally, per the paper's workflow tooling)

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(cd_abund, ctrl_abund, alternative='two-sided')
```

## Evaluation signals

- Cholic-acid conjugates to Glu, Ile/Leu, Phe, Thr, Trp and Tyr show significantly increased abundance in CD versus non-IBD (q < 0.05, positive median direction), matching the four-cohort validation
- Conjugates of secondary bile acids show no significant difference between CD and non-IBD controls
- Every reported q-value equals the BH-adjusted value from multipletests(method='fdr_bh') applied within, not across, each comparison pair
- Output CSV contains all three pairwise comparisons (non-IBD vs CD, non-IBD vs UC, CD vs UC) for every bile-amidate feature
- p-values derive from scipy.stats.mannwhitneyu with alternative='two-sided' on relative-abundance values, not on log-transformed or imputed data

## Limitations

- MS/MS matching does not guarantee against false discovery of closely related isomers or compounds, so feature-to-conjugate assignments should be treated as provisional without NMR or X-ray structural confirmation
- MASST public spectra (~1.2 billion) do not encompass all molecules, limiting which public datasets can be re-analyzed in the discovery phase
- Metabolites must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition, so absent features may reflect sensitivity rather than biology

## Evidence

- [abstract] Validation using four distinct human IBD cohorts showed that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn’s disease.: "Validation using four distinct human IBD cohorts showed that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn’s disease."
- [full_text] Stratify by phenotype and run pairwise two-sided rank-sum tests with BH correction.: [evidence span withheld — anchor status: not_found]
- [full_text] FDR correction uses statsmodels multipletests with fdr_bh method.: [evidence span withheld — anchor status: not_found]
- [readme] README notes conda is required to run the workflow locally.: "You will need to have conda, mamba, and nextflow installed to run things locally."
- [discussion] MS/MS matching does not preclude closely related isomer false discoveries.: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
