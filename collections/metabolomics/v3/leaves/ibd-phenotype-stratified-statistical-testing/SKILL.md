---
name: ibd-phenotype-stratified-statistical-testing
description: Use when relative-quantification feature tables (e.g. from Metabolomics
  Workbench studies PR000639/iHMP2 and PR000677/PRISM) must be tested for abundance
  differences across IBD phenotypes to validate newly discovered compounds such as
  conjugated bile acids.
when_to_use_negative:
- The input is already a summarized or pre-tested result set rather than raw per-sample
  relative-abundance feature tables — re-running pairwise tests would double-count
  or be undefined.
- The dataset lacks sample-level IBD phenotype metadata, so samples cannot be assigned
  to the non-IBD / CD / UC stratification required for the pairwise tests.
- The analysis targets discovery of new spectral features rather than validating known
  candidate compounds — reverse metabolomics / MASST search is the upstream step,
  not this statistical testing skill.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0157
tools:
- name: conda
  role: Environment manager used to install dependencies (with mamba and nextflow)
    for running the local workflow tooling associated with this paper's reproducible
    analyses.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:dff08ee650c71d09bc6f0aa47bbbcb0311cc5f5cf621c7275a4b597d4d3a8d02
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - conda
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# ibd-phenotype-stratified-statistical-testing

## Summary

Statistical validation workflow that stratifies metabolomics feature-abundance tables by IBD phenotype (non-IBD control, Crohn's disease, ulcerative colitis) and applies pairwise two-sided Wilcoxon rank-sum tests with Benjamini-Hochberg FDR correction to confirm disease associations of candidate metabolites such as bile amidates.

## When to use

Use when relative-quantification feature tables (e.g. from Metabolomics Workbench studies PR000639/iHMP2 and PR000677/PRISM) must be tested for abundance differences across IBD phenotypes to validate newly discovered compounds such as conjugated bile acids.

## When NOT to use

- The input is already a summarized or pre-tested result set rather than raw per-sample relative-abundance feature tables — re-running pairwise tests would double-count or be undefined.
- The dataset lacks sample-level IBD phenotype metadata, so samples cannot be assigned to the non-IBD / CD / UC stratification required for the pairwise tests.
- The analysis targets discovery of new spectral features rather than validating known candidate compounds — reverse metabolomics / MASST search is the upstream step, not this statistical testing skill.

## Inputs

- Public relative-quantification metabolomics feature table from Metabolomics Workbench study PR000639 (iHMP2 stool cohort)
- Public relative-quantification metabolomics feature table from Metabolomics Workbench study PR000677 (PRISM IBD fecal cohort)
- Sample-level IBD phenotype metadata mapping each sample to non-IBD control, Crohn's disease, or ulcerative colitis

## Outputs

- CSV of per-feature test results with feature identifier, conjugate assignment, comparison group pair, raw p-value, BH-adjusted q-value, and direction of group median difference
- List of metabolite features significantly enriched or depleted in a given IBD phenotype (q < 0.05) with directionality

## How to apply

Download the public relative-quantification feature tables for the iHMP2 stool cohort (PR000639) and the PRISM IBD fecal cohort (PR000677) via their Metabolomics Workbench study accessions and parse each table with pandas. Subset the rows to the metabolite features of interest — for bile amidates, cholic acid and secondary bile acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp and Tyr — then stratify samples by IBD phenotype metadata into non-IBD control, CD and UC groups. For each feature, run two-sided Wilcoxon rank-sum tests (scipy.stats.mannwhitneyu, alternative='two-sided') for each pairwise group comparison: non-IBD vs CD, non-IBD vs UC, and CD vs UC. Apply Benjamini-Hochberg false-discovery-rate correction (statsmodels.stats.multitest.multipletests, method='fdr_bh') to the p-values within each comparison to obtain adjusted q-values. Write the feature identifier, conjugate assignment, test group pair, raw p-value, BH-adjusted q-value and group median directions to an output CSV (e.g. bile_amidate_wilcoxon_results.csv). Validation follows the paper's four-cohort finding: cholic-acid conjugates to Glu, Ile/Leu, Phe, Thr, Trp and Tyr are significantly increased in CD versus non-IBD (q < 0.05, positive direction), while secondary bile acid amidates show no significant difference.

## Related tools

- **conda** (Environment manager used to install dependencies (with mamba and nextflow) for running the local workflow tooling associated with this paper's reproducible analyses.)

## Examples

```
from scipy.stats import mannwhitneyu; u, p = mannwhitneyu(feature_abundance[phenotype=='CD'], feature_abundance[phenotype=='non-IBD'], alternative='two-sided')
```

## Evaluation signals

- Cholic-acid conjugates to Glu, Ile/Leu, Phe, Thr, Trp and Tyr show a positive group-median direction and q < 0.05 for the non-IBD vs CD comparison, matching the paper's validated result across four IBD cohorts.
- Secondary bile acid amidates show no significant difference (q ≥ 0.05) between non-IBD and CD, confirming the reported class-specific pattern.
- Every feature-comparison row in the output CSV carries both a raw p-value and a BH-adjusted q-value from the same multipletests call, and q-values are monotonically ordered with p-values within each comparison group.
- Per-feature sample counts before and after phenotype stratification match the totals in the source feature table, with no samples dropped due to missing phenotype assignment.
- Output CSV contains all three pairwise group comparisons (non-IBD vs CD, non-IBD vs UC, CD vs UC) for each tested feature.

## Limitations

- Without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data, so feature-to-structure assignments carry isomer ambiguity.
- The MS/MS matching protocol used upstream does not guarantee against false discovery of closely related isomers or compounds, which propagates into downstream statistical testing.
- Public metabolomics data in MASST (~1.2 billion spectra) do not encompass all molecules, limiting re-analyses of public data in the discovery phase that precedes this validation step.

## Evidence

- [abstract] Validation using four distinct human IBD cohorts showed that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn's disease.: "Validation using four distinct human IBD cohorts showed that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn’s disease."
- [full_text] Validation workflow detail: [evidence span withheld — anchor status: not_found]
- [full_text] FDR correction detail: [evidence span withheld — anchor status: not_found]
- [discussion] Isomer ambiguity caveat: "without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data"
- [discussion] False discovery caveat: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
- [readme] Tooling dependency: "You will need to have conda, mamba, and nextflow installed to run things locally."
