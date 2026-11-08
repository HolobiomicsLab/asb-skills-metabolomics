---
name: lipidomic-feature-group-statistics
description: Use when a lipidomic data matrix with experimental group assignments
  must be tested per lipid feature (class, molecular species, or building-block level)
  to flag significant group differences with significance asterisks.
when_to_use_negative:
- When the goal is multivariate analysis (e.g., heatmaps via pheatmap or lipid pathway/network
  analysis via igraph) rather than per-feature univariate group comparisons
- When the study design violates the tool's assumptions — the bar-chart statistics
  use two-tailed Student's t-tests assuming equal variances, so paired designs or
  unequal-variance comparisons need a different approach
- When the data matrix has not been prepared and nomenclature-standardized per LipidOne
  2.0's dataset-preparation and lipid-nomenclature protocols (Basic Protocol 1 / Support
  Protocol)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: LipidOne 2.0
  role: Web platform whose bar-chart function computes the t-test/ANOVA statistics,
    FDR correction, Tukey post hoc, and assigns the 1–3 asterisk significance annotations
  repo: https://lipidone.eu
- name: R
  role: Back-end environment in which the statistical scripts (t-tests, one-way ANOVA,
    Benjamini-Hochberg FDR, Tukey post hoc) are implemented and can be re-run offline
    on the data matrix
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:158bf1cd04992fe1315912f9dc764000bf2c17b26b142b8b52bfacba700b29fe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lipidomic-feature-group-statistics

## Summary

Reproduce the LipidOne 2.0 bar-chart group-comparison statistics: per-lipid-feature two-tailed Student's t-tests (equal variance) or one-way ANOVA with Benjamini-Hochberg FDR, Tukey post hoc, and p-value-to-asterisk annotation (< .05, < .01, < .001).

## When to use

Use when a lipidomic data matrix with experimental group assignments must be tested per lipid feature (class, molecular species, or building-block level) to flag significant group differences with significance asterisks.

## When NOT to use

- When the goal is multivariate analysis (e.g., heatmaps via pheatmap or lipid pathway/network analysis via igraph) rather than per-feature univariate group comparisons
- When the study design violates the tool's assumptions — the bar-chart statistics use two-tailed Student's t-tests assuming equal variances, so paired designs or unequal-variance comparisons need a different approach
- When the data matrix has not been prepared and nomenclature-standardized per LipidOne 2.0's dataset-preparation and lipid-nomenclature protocols (Basic Protocol 1 / Support Protocol)

## Inputs

- Lipidomic data matrix with experimental group assignments (e.g., the ERYTHROCYTES example dataset downloadable from LipidOne 2.0 via 'START NOW' → 'Load Data Matrix')
- Mapping of lipid features to the structural level being analyzed (lipid classes, molecular species, or building blocks/acyl chains)
- Group-factor specification for two-group t-tests or multi-group one-way ANOVA

## Outputs

- barplot_statistics.csv containing the t-statistic and FDR (t-test case) or Fisher values, FDR, and Tukey post hoc results (ANOVA case)
- Per-lipid-feature significance asterisk annotations (1 asterisk for p < .05, 2 for p < .01, 3 for p < .001) used by the bar chart

## How to apply

Load the lipidomic data matrix with its group assignments (e.g., the ERYTHROCYTES example dataset from LipidOne 2.0) into R, since LipidOne 2.0's back-end is coded in R scripts. For each lipid feature, run a two-tailed Student's t-test assuming equal variances when comparing two experimental groups, or a one-way ANOVA when more than two groups are compared. Apply Benjamini-Hochberg FDR correction over the family of tests; for ANOVA also record the Fisher values and run Tukey post hoc pairwise comparisons. Map each corrected result to asterisk annotations — one asterisk for p < .05, two for p < .01, three for p < .001 — matching the bar-chart display logic. Finally, write the companion CSV (barplot_statistics.csv) holding the t-statistic and FDR (t-test case) or the Fisher values, FDR, and Tukey post hoc results (ANOVA case), so every plotted asterisk is traceable to a numeric test result.

## Related tools

- **LipidOne 2.0** (Web platform whose bar-chart function computes the t-test/ANOVA statistics, FDR correction, Tukey post hoc, and assigns the 1–3 asterisk significance annotations) — https://lipidone.eu
- **R** (Back-end environment in which the statistical scripts (t-tests, one-way ANOVA, Benjamini-Hochberg FDR, Tukey post hoc) are implemented and can be re-run offline on the data matrix)

## Evaluation signals

- barplot_statistics.csv exists and contains the correct column set for the test type: t-statistic + FDR for two-group t-tests, or Fisher values + FDR + Tukey post hoc results for ANOVA
- Asterisk counts in the bar chart exactly match the p-value thresholds: one asterisk for p < .05, two for p < .01, three for p < .001
- FDR correction is applied over the whole family of per-feature tests (Benjamini-Hochberg), not per-feature independently
- ANOVA runs additionally report Fisher values and Tukey post hoc pairwise comparisons for every group pair
- Re-running the pipeline on the ERYTHROCYTES example dataset reproduces the same significance calls as the web bar chart

## Limitations

- The bar-chart statistics assume two-tailed Student's t-tests with equal variances; paired or heteroscedastic designs are not supported by this path
- Asterisk thresholds (< .05, < .01, < .001) discard exact p-values from the plot annotation, so the companion CSV is required to recover precise significance levels
- Example datasets (ERYTHROCYTES, FUNGUS, SCIATIC NERVE) are only accessible through the web platform rather than a persistent repository accession
- The public GitHub README (matteogiulietti/LipidOne) documents the standalone Perl/Windows desktop Lipid One tool, not the R-based back-end of the LipidOne 2.0 web statistics

## Evidence

- [abstract] Mono/multivariate statistics tailored to each lipid level: "LipidOne 2.0 offers a range of mono- and multivariate statistical analyses, specifically tailored to each structural level."
- [discussion] Example datasets available for download via the web platform: [evidence span withheld — anchor status: not_found]
- [intro] Three-level analysis previously required intensive submatrix calculations: [evidence span withheld — anchor status: not_found]
- [readme] README describes the earlier standalone Lipid One tool: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."
