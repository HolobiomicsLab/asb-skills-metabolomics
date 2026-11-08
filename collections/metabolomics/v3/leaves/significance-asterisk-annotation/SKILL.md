---
name: significance-asterisk-annotation
description: Use when bar-chart group comparisons of lipid features (classes, molecular
  species, or building blocks) need statistical significance encoded as asterisks,
  or when reproducing the barplot_statistics.csv output of LipidOne 2.0.
when_to_use_negative:
- Data require unequal-variance or paired tests — this skill assumes two-tailed Student's
  t-tests with equal variance, as implemented by the LipidOne 2.0 bar chart function
- Analyses beyond bar-chart group significance annotation, e.g., multivariate analyses
  or the lipid biomarker identification function integrating four statistical parameters
- Input p-values are already FDR-corrected — applying Benjamini-Hochberg a second
  time over the family of tests would distort the asterisk thresholds
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3520
tools:
- name: LipidOne 2.0
  role: Web platform whose bar chart function defines the asterisk annotation logic
    being reconstructed; back-end runs R scripts
  repo: https://lipidone.eu
- name: R
  role: Implementation environment for the two-tailed equal-variance t-tests / one-way
    ANOVA, Benjamini-Hochberg FDR, Tukey post hoc comparisons, and barplot_statistics.csv
    generation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:5fc080f079d0a10d3a76e7115811b3a8308a30a35f9481a4dab682fda9e5cbf8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# significance-asterisk-annotation

## Summary

Reconstructs LipidOne 2.0's bar-chart significance logic: per-lipid-feature two-tailed Student's t-tests assuming equal variances (or one-way ANOVA for >2 groups) with Benjamini-Hochberg FDR correction, then maps p-values to one, two, or three asterisk annotations at the .05/.01/.001 thresholds.

## When to use

Use when bar-chart group comparisons of lipid features (classes, molecular species, or building blocks) need statistical significance encoded as asterisks, or when reproducing the barplot_statistics.csv output of LipidOne 2.0.

## When NOT to use

- Data require unequal-variance or paired tests — this skill assumes two-tailed Student's t-tests with equal variance, as implemented by the LipidOne 2.0 bar chart function
- Analyses beyond bar-chart group significance annotation, e.g., multivariate analyses or the lipid biomarker identification function integrating four statistical parameters
- Input p-values are already FDR-corrected — applying Benjamini-Hochberg a second time over the family of tests would distort the asterisk thresholds

## Inputs

- Lipidomic data matrix with experimental group assignments (e.g., the 'ERYTHROCYTES' example dataset downloadable from LipidOne 2.0 via 'START NOW' > 'Load Data Matrix')
- Selection of two or more experimental groups to compare per lipid feature

## Outputs

- barplot_statistics.csv containing t-statistic and FDR (t-test case) or Fisher values, FDR, and Tukey post hoc results (ANOVA case)
- Per-feature significance asterisk annotations (1 asterisk for p < .05, 2 for p < .01, 3 for p < .001) attached to bar-chart group comparisons

## How to apply

Load the lipidomic data matrix with experimental group assignments (e.g., the 'ERYTHROCYTES' example dataset from LipidOne 2.0) into R. For each lipid feature, run a two-tailed Student's t-test assuming equal variances between the selected groups, or a one-way ANOVA when more than two groups are compared; for ANOVA, additionally record the Fisher values and run Tukey post hoc pairwise comparisons. Apply Benjamini-Hochberg FDR correction over the whole family of tests rather than per-feature. Map each FDR-corrected p-value to asterisk annotations: one asterisk for p < .05, two for p < .01, three for p < .001, all two-tailed with equal variance assumed. Finally, write the companion CSV: barplot_statistics.csv with the t-statistic and FDR (t-test case) or the Fisher values, FDR, and Tukey post hoc results (ANOVA case), alongside the asterisk counts used for chart annotation.

## Related tools

- **LipidOne 2.0** (Web platform whose bar chart function defines the asterisk annotation logic being reconstructed; back-end runs R scripts) — https://lipidone.eu
- **R** (Implementation environment for the two-tailed equal-variance t-tests / one-way ANOVA, Benjamini-Hochberg FDR, Tukey post hoc comparisons, and barplot_statistics.csv generation)

## Examples

```
t.test(g1, g2, var.equal=TRUE, alternative="two.sided"); p.adjust(pvals, method="BH")
```

## Evaluation signals

- barplot_statistics.csv contains the expected columns: t-statistic and FDR for the t-test case, or Fisher values, FDR, and Tukey post hoc results for the ANOVA case
- Asterisk assignments follow exact thresholds: 1 asterisk iff p < .05, 2 iff p < .01, 3 iff p < .001
- FDR correction is applied over the family of tests (Benjamini-Hochberg), not per feature independently
- Two-tailed Student's t-test with equal variances (var.equal=TRUE) is used for two-group comparisons; one-way ANOVA plus Tukey post hoc is used for >2 groups
- Every annotated feature has an asterisk count consistent with its reported FDR value in the CSV

## Limitations

- The bar chart function assumes two-tailed tests with equal variance; unequal-variance or paired designs are not covered
- Example datasets ('ERYTHROCYTES,' 'FUNGUS,' 'SCIATIC NERVE') are only accessible through the web platform rather than a persistent repository accession
- The GitHub README documents the older standalone Perl/Windows LipidOne (FreeBSD-licensed, GNUplot/Prima GUI), not the R-backed LipidOne 2.0 web tool whose bar chart statistics this skill reconstructs

## Evidence

- [abstract] LipidOne 2.0 offers a range of mono- and multivariate statistical analyses, specifically tailored to each structural level.: "LipidOne 2.0 offers a range of mono- and multivariate statistical analyses, specifically tailored to each structural level."
- [discussion] Dataset download path via web platform: "freely available for download on the designated page accessible by selecting the “START NOW” button and then “Load Data Matrix.”"
- [readme] README description of the tool the repo hosts: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."
- [intro] Older desktop predecessor built in-house: "The desktop version of Lipid One (Pellegrino et al., 2022), marked our initial attempt to address the gap in lipid analysis tools"
