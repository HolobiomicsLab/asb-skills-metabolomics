---
name: equal-variance-t-test-group-comparison
description: Use when comparing lipid abundances of two experimental groups across
  lipid features (classes, molecular species, or building blocks) and you need p-values,
  FDR correction, and asterisk-annotated significance per feature.
when_to_use_negative:
- Comparing more than two groups where only pairwise information is needed without
  ANOVA-level Fisher values and Tukey post hoc context.
- Groups with clearly unequal variances or non-normal abundance distributions — the
  t-test assumes equal variance, so Welch's t-test or non-parametric tests would be
  more appropriate.
- Analyses outside lipidomics group comparison (e.g. gene-level differential expression),
  since this skill reproduces the specific bar-chart statistics of the LipidOne 2.0
  lipidomic platform.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3318
tools:
- name: LipidOne 2.0
  role: Web platform whose bar-chart function performs the two-group equal-variance
    t-test / ANOVA comparison with FDR correction and asterisk annotation; used as
    reference implementation.
  repo: https://lipidone.eu
- name: R
  role: Runtime for scripting the per-feature t-tests/ANOVA, Benjamini-Hochberg FDR
    correction, Tukey post hoc comparisons, and barplot_statistics.csv output, mirroring
    LipidOne 2.0's R-script back-end.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:b36b55649393b3ee36e3ae0f2da4b2a750e50a11bd7c839f37545954d8bb5959
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# equal-variance-t-test-group-comparison

## Summary

Reconstructs the two-group significance testing used by the LipidOne 2.0 bar-chart function: two-tailed Student's t-tests assuming equal variances (or one-way ANOVA with Tukey post hoc for >2 groups), FDR-corrected across lipid features, with p-values mapped to significance asterisks and written to barplot_statistics.csv.

## When to use

Use when comparing lipid abundances of two experimental groups across lipid features (classes, molecular species, or building blocks) and you need p-values, FDR correction, and asterisk-annotated significance per feature.

## When NOT to use

- Comparing more than two groups where only pairwise information is needed without ANOVA-level Fisher values and Tukey post hoc context.
- Groups with clearly unequal variances or non-normal abundance distributions — the t-test assumes equal variance, so Welch's t-test or non-parametric tests would be more appropriate.
- Analyses outside lipidomics group comparison (e.g. gene-level differential expression), since this skill reproduces the specific bar-chart statistics of the LipidOne 2.0 lipidomic platform.

## Inputs

- Lipidomic data matrix with experimental group assignments (e.g. the 'ERYTHROCYTES' example dataset from LipidOne 2.0)
- Selected experimental group labels to compare (two groups for the t-test case, >2 for ANOVA)
- Lipid feature list at one structural level: lipid classes, molecular species, or building blocks (acyl/alkyl/alkenyl chains)

## Outputs

- Per-lipid t-statistic and FDR-corrected p-value (t-test case)
- Fisher (F) values, FDR, and Tukey post hoc pairwise comparison results (ANOVA case)
- Companion CSV file barplot_statistics.csv pairing each bar chart with its test statistics
- Significance asterisk annotations (1 for p < .05, 2 for p < .01, 3 for p < .001)

## How to apply

Load the lipidomic data matrix with group assignments (e.g. the 'ERYTHROCYTES' example dataset from LipidOne 2.0) into R. For each lipid feature, run a two-tailed Student's t-test assuming equal variances between the two groups; switch to one-way ANOVA (recording Fisher values and running Tukey post hoc pairwise comparisons) when more than two groups are compared. Apply Benjamini-Hochberg FDR correction over the whole family of per-lipid tests to control the false discovery rate. Map corrected results to asterisks: one asterisk for p < .05, two for p < .01, three for p < .001, all computed two-tailed. Write the companion barplot_statistics.csv containing the t-statistic and FDR (t-test case) or Fisher values, FDR, and Tukey post hoc results (ANOVA case), mirroring LipidOne 2.0's R-based back-end behavior.

## Related tools

- **LipidOne 2.0** (Web platform whose bar-chart function performs the two-group equal-variance t-test / ANOVA comparison with FDR correction and asterisk annotation; used as reference implementation.) — https://lipidone.eu
- **R** (Runtime for scripting the per-feature t-tests/ANOVA, Benjamini-Hochberg FDR correction, Tukey post hoc comparisons, and barplot_statistics.csv output, mirroring LipidOne 2.0's R-script back-end.)

## Evaluation signals

- Asterisk annotation matches thresholds: exactly one asterisk for p < .05, two for p < .01, three for p < .001, two-tailed.
- barplot_statistics.csv exists and contains t-statistic + FDR columns for the t-test case, and Fisher values + FDR + Tukey post hoc columns for the ANOVA case.
- Number of tests performed equals the number of lipid features in the selected data matrix, and FDR is applied across that full family (Benjamini-Hochberg).
- Per-feature t-statistic recomputed independently (e.g. R t.test with var.equal=TRUE, two-tailed) matches the recorded values.
- Group membership matrix matches the experimental group assignments used by the platform (e.g. ERYTHROCYTES groups).

## Limitations

- The desktop version of Lipid One is a standalone Perl/Windows application (Prima GUI + Gnuplot) — distinct from the LipidOne 2.0 web tool whose statistics are reproduced here; reproducing them requires reimplementing the R-based back-end behavior.
- Example datasets such as 'ERYTHROCYTES', 'FUNGUS', and 'SCIATIC NERVE' are only accessible through the web platform rather than a persistent repository accession.
- Equal-variance assumption means unequal-variance group designs are not covered by this skill.

## Evidence

- [other] bar-chart two-group statistics finding: [evidence span withheld — anchor status: not_found]
- [abstract] LipidOne 2.0 platform: "LipidOne 2.0 (https://lipidone.eu) is a new web bioinformatic tool for the analysis of lipidomic data."
- [discussion] R back-end: "coding its back-end interac- tions with the R scripts"
- [discussion] example datasets: [evidence span withheld — anchor status: not_found]
- [readme] desktop LipidOne README: "LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system."
