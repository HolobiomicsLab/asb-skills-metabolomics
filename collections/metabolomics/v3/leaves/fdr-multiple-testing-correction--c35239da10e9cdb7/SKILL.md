---
name: fdr-multiple-testing-correction
description: Use when a lipidomic dataset is compared across two or more experimental
  groups with a test per lipid feature (t-test/ANOVA), and corrected p-values plus
  significance asterisks are needed.
when_to_use_negative:
- Only a single pre-specified hypothesis test is planned, so family-wise/FDR multiplicity
  correction is unnecessary
- The analysis requires multivariate statistics or biomarker identification rather
  than per-feature univariate group comparisons (use LipidOne 2.0's dedicated multivariate/biomarker
  functions instead)
- The data violate the equal-variance Student's t-test assumption and a Welch-type
  or non-parametric test is required instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3365
tools:
- name: LipidOne 2.0
  role: Web platform whose bar chart function implements the t-test/ANOVA with FDR
    correction and asterisk annotation; example datasets downloadable from https://lipidone.eu
  repo: https://lipidone.eu
- name: R
  role: Back-end language running the Student's t-tests / one-way ANOVA, Benjamini-Hochberg
    FDR correction, and Tukey post hoc comparisons
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:68479abad2ea8fbe79a5d9e0bec28ea0d04f362a00d057794dba3c91bda97c7b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# fdr-multiple-testing-correction

## Summary

Applies Benjamini-Hochberg FDR correction over the family of per-lipid significance tests (Student's t-test or one-way ANOVA) that underlies LipidOne 2.0's bar chart function, mapping corrected p-values to one, two, or three asterisk annotations at < .05, < .01, and < .001 thresholds.

## When to use

Use when a lipidomic dataset is compared across two or more experimental groups with a test per lipid feature (t-test/ANOVA), and corrected p-values plus significance asterisks are needed.

## When NOT to use

- Only a single pre-specified hypothesis test is planned, so family-wise/FDR multiplicity correction is unnecessary
- The analysis requires multivariate statistics or biomarker identification rather than per-feature univariate group comparisons (use LipidOne 2.0's dedicated multivariate/biomarker functions instead)
- The data violate the equal-variance Student's t-test assumption and a Welch-type or non-parametric test is required instead

## Inputs

- Lipidomic data matrix (features x samples) with experimental group assignments, e.g. the 'ERYTHROCYTES', 'FUNGUS', or 'SCIATIC NERVE' example datasets from LipidOne 2.0
- Group labels defining the two-group comparison (t-test) or multi-group design (ANOVA)

## Outputs

- barplot_statistics.csv containing the t-statistic and FDR (t-test case) or the Fisher values, FDR, and Tukey post hoc results (ANOVA case)
- Per-feature significance asterisk annotations (1 asterisk p < .05, 2 for p < .01, 3 for p < .001)

## How to apply

Load the lipidomic data matrix with experimental group assignments (e.g. the 'ERYTHROCYTES' example dataset from LipidOne 2.0) into R. For each lipid feature, run a two-tailed Student's t-test assuming equal variances between two selected groups, or a one-way ANOVA when more than two groups are compared. Compute the FDR correction (Benjamini-Hochberg) over the family of tests, and in the ANOVA case additionally record the Fisher values and run Tukey post hoc pairwise comparisons. Map each result's corrected p-value to asterisks: one asterisk for p < .05, two for p < .01, three for p < .001. Write the companion CSV (barplot_statistics.csv) containing the t-statistic and FDR (t-test case) or the Fisher values, FDR, and Tukey post hoc results (ANOVA case).

## Related tools

- **LipidOne 2.0** (Web platform whose bar chart function implements the t-test/ANOVA with FDR correction and asterisk annotation; example datasets downloadable from https://lipidone.eu) — https://lipidone.eu
- **R** (Back-end language running the Student's t-tests / one-way ANOVA, Benjamini-Hochberg FDR correction, and Tukey post hoc comparisons)

## Examples

```
t.test(x, y, var.equal=TRUE, alternative="two.sided"); p.adj <- p.adjust(pvals, method="BH")
```

## Evaluation signals

- barplot_statistics.csv contains exactly the expected columns: t-statistic + FDR for t-tests, or Fisher values + FDR + Tukey post hoc results for ANOVA
- Asterisk counts correspond to the specified cutoffs: 1 for p < .05, 2 for p < .01, 3 for p < .001, with two-tailed equal-variance tests
- FDR values are monotonically consistent with raw p-values under Benjamini-Hochberg (FDR-adjusted p >= raw p)
- Number of tests equals the number of lipid features compared across the selected groups; re-running the same group selection reproduces identical statistics

## Limitations

- Tests are calculated two-tailed assuming equal variances, which may not suit all lipidomic intensity distributions
- The public GitHub repository (matteogiulietti/LipidOne) documents the earlier standalone desktop Lipid One written in Perl for Microsoft Windows using Perl Prima and Gnuplot, not the R back-end of the LipidOne 2.0 web tool described in this workflow

## Evidence

- [other] The bar chart function assigns one, two, or three asterisks for statistical significance based on p-value thresholds (< .05, < .01, < .001), calculated two-tailed with equal variance.: [evidence span withheld — anchor status: not_found]
- [other] Compute FDR correction over the family of tests (Benjamini-Hochberg), and for ANOVA additionally record the Fisher values and run Tukey post hoc pairwise comparisons.: [evidence span withheld — anchor status: not_found]
- [abstract] LipidOne 2.0 (https://lipidone.eu) is a new web bioinformatic tool for the analysis of lipidomic data.: "LipidOne 2.0 (https://lipidone.eu) is a new web bioinformatic tool for the analysis of lipidomic data."
- [readme] LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset.: "LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system."
