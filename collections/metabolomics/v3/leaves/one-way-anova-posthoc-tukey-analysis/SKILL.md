---
name: one-way-anova-posthoc-tukey-analysis
description: Use when a lipidomic data matrix (lipid features × samples with group
  assignments) must be compared across MORE than two experimental groups and per-lipid
  significance with FDR-corrected pairwise calls is required for bar-chart annotation.
when_to_use_negative:
- Exactly two experimental groups are compared — the LipidOne 2.0 bar chart function
  uses a two-tailed Student's t-test (equal variance) with t-statistic and FDR in
  barplot_statistics.csv instead of ANOVA/Tukey
- No group structure exists in the data (a single experimental group), so no between-group
  ANOVA can be computed
- The goal is multivariate analysis (e.g. biomarker identification or lipid pathway
  analysis) rather than per-feature univariate group comparison
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LipidOne 2.0
  role: Web platform whose bar chart function performs the ANOVA, FDR correction,
    and Tukey post hoc analysis and renders the asterisk annotations
  repo: https://lipidone.eu
- name: R
  role: Back-end language in which LipidOne 2.0's statistical scripts (t-tests, ANOVA,
    FDR, Tukey) are implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:dfdbe0058b40205640808d6c94148e9f94706ac8adaa742900a64499adf03603
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# one-way-anova-posthoc-tukey-analysis

## Summary

One-way ANOVA with Benjamini-Hochberg FDR correction and Tukey post hoc pairwise comparisons, as implemented in the LipidOne 2.0 bar chart function to assign significance asterisks when comparing more than two experimental groups at the class, molecular species, or building-block level.

## When to use

Use when a lipidomic data matrix (lipid features × samples with group assignments) must be compared across MORE than two experimental groups and per-lipid significance with FDR-corrected pairwise calls is required for bar-chart annotation.

## When NOT to use

- Exactly two experimental groups are compared — the LipidOne 2.0 bar chart function uses a two-tailed Student's t-test (equal variance) with t-statistic and FDR in barplot_statistics.csv instead of ANOVA/Tukey
- No group structure exists in the data (a single experimental group), so no between-group ANOVA can be computed
- The goal is multivariate analysis (e.g. biomarker identification or lipid pathway analysis) rather than per-feature univariate group comparison

## Inputs

- Lipidomic data matrix (lipid features × samples) with experimental group assignments, e.g. the 'ERYTHROCYTES', 'FUNGUS', or 'SCIATIC NERVE' example datasets from https://lipidone.eu
- Selection of ≥3 experimental groups to compare (lipid classes, molecular species, or building blocks)

## Outputs

- barplot_statistics.csv containing the Fisher values, FDR (Benjamini-Hochberg), and Tukey post hoc pairwise comparison results for each lipid feature
- Bar chart with significance asterisk annotations: 1 asterisk for p < .05, 2 for p < .01, 3 for p < .001

## How to apply

Load the lipidomic data matrix with experimental group assignments (e.g. the 'ERYTHROCYTES' example dataset downloadable from LipidOne 2.0) and, for each lipid feature, run a one-way ANOVA across the selected groups — use a two-tailed Student's t-test with equal variance instead if exactly two groups are compared. Apply Benjamini-Hochberg FDR correction over the family of tests across lipid features, and record the Fisher values from each ANOVA. Run Tukey post hoc pairwise comparisons between group pairs to identify which groups differ. Map each result's p-value to asterisk annotation — one asterisk for p < .05, two for p < .01, three for p < .001 (all calculated two-tailed with equal variance). Write the companion CSV barplot_statistics.csv containing the Fisher values, the FDR, and the Tukey post hoc results. This mirrors LipidOne 2.0's bar chart function, whose back-end is coded in R scripts.

## Related tools

- **LipidOne 2.0** (Web platform whose bar chart function performs the ANOVA, FDR correction, and Tukey post hoc analysis and renders the asterisk annotations) — https://lipidone.eu
- **R** (Back-end language in which LipidOne 2.0's statistical scripts (t-tests, ANOVA, FDR, Tukey) are implemented)

## Evaluation signals

- barplot_statistics.csv exists and contains, per lipid feature, the Fisher values, the FDR, and the Tukey post hoc results (ANOVA case) — not the t-statistic column reserved for the two-group t-test case
- Asterisk annotation follows the exact threshold mapping: 1 asterisk for p < .05, 2 for p < .01, 3 for p < .001
- All tests are computed two-tailed assuming equal variances, matching the bar chart function's specification
- Benjamini-Hochberg FDR is applied over the family of per-lipid tests, not per-test uncorrected p-values
- The analysis was run only on features with ≥3 groups; two-group features were routed to the t-test path

## Limitations

- The bar chart statistics assume equal variances and two-tailed testing; unequal-variance or one-tailed scenarios are outside this function's specification
- Example datasets (ERYTHROCYTES, FUNGUS, SCIATIC NERVE) are only downloadable through the web platform's 'START NOW' page rather than a persistent repository accession
- The public GitHub README documents the earlier desktop LipidOne written in Perl for Windows, not the R-backed LipidOne 2.0 web service implementing these bar chart statistics
- Pairwise interpretation depends on Tukey post hoc output being retained; without it, significant ANOVA Fisher values cannot be attributed to specific group pairs

## Evidence

- [abstract] LipidOne 2.0 offers a range of mono- and multivariate statistical analyses, specifically tailored to each structural level.: "LipidOne 2.0 offers a range of mono- and multivariate statistical analyses, specifically tailored to each structural level."
- [abstract] It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains).: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [discussion] The three examples of lipidomic datasets are freely available for download: [evidence span withheld — anchor status: not_found]
- [readme] LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset.: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."
