---
name: univariate-ttest-mz-screening
description: Use when you have a normalized m/z peak table from MS metabolomics data
  with a two-group (or paired) design variable and need to rank m/z values by differential
  intensity.
when_to_use_negative:
- The data are raw, un-normalized peak intensities that have not gone through MetaboShiny's
  filtering/normalization/imputation pipeline
- The design involves more than two groups or multi-factor variables — use ANOVA,
  ASCA, MEBA, or two-factor analysis modes instead
- The goal is metabolite annotation of all m/z values rather than screening for significant
  hits — MetaboShiny performs annotation only on significant hits from the analysis
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MetaboShiny
  role: Interactive GUI that hosts the t-test as one of its 'Per m/z' statistical
    analyses on normalized m/z data
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaboAnalystR
  role: Underlying R package that MetaboShiny delegates the t-test statistical computation
    to
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:63340c2ee0df1f8a12cebbb1a57efa2e37063a8e28a842a72622eb8afa26809c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaboAnalystR
  - MetaboShiny
  - R
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# univariate-ttest-mz-screening

## Summary

Run MetaboAnalystR-backed two-group t-tests directly on m/z peak intensities to rank which m/z values differ significantly between two experimental groups (e.g. lung cancer vs. control urine DI-MS samples).

## When to use

Use when you have a normalized m/z peak table from MS metabolomics data with a two-group (or paired) design variable and need to rank m/z values by differential intensity.

## When NOT to use

- The data are raw, un-normalized peak intensities that have not gone through MetaboShiny's filtering/normalization/imputation pipeline
- The design involves more than two groups or multi-factor variables — use ANOVA, ASCA, MEBA, or two-factor analysis modes instead
- The goal is metabolite annotation of all m/z values rather than screening for significant hits — MetaboShiny performs annotation only on significant hits from the analysis

## Inputs

- Normalized m/z peak intensity table (samples x m/z values) produced by MetaboShiny's normalization workflow
- Metadata table with a two-level experimental group variable (e.g. cancer vs. control) and sample/individual identifiers

## Outputs

- Ranked t-test result table of m/z values with p-values and significance calls
- Input for downstream volcano plot, heatmap, or PLS-DA loading inspection

## How to apply

After completing MetaboShiny's normalization/filtering workflow (filtering, normalization, transformation, scaling, and missing value imputation such as Random Forest), select the t-test option in the Statistics panel under 'Per m/z'. Provide the two-level experimental variable from the metadata (e.g. cancer vs. control), choosing paired or unpaired analysis as appropriate. MetaboShiny delegates the statistical computation to MetaboAnalystR. The result is a ranked table of m/z values by significance (p-value), which can then feed volcano plots, PLS-DA loading inspection, or m/z annotation via database matching (e.g. HMDB) on only the significant hits. In the demonstrated lung cancer case study (MetaboLights MTBLS28, 1005 patients), the t-test was used to rank significantly different m/z values, with known discriminatory m/z values (264, 308, 441, 561) expected among the top hits.

## Related tools

- **MetaboShiny** (Interactive GUI that hosts the t-test as one of its 'Per m/z' statistical analyses on normalized m/z data) — https://github.com/joannawolthuis/MetaboShiny
- **MetaboAnalystR** (Underlying R package that MetaboShiny delegates the t-test statistical computation to)

## Examples

```
In MetaboShiny GUI: Statistics → Per m/z → t-test, select the two-level experimental variable from the metadata (e.g. 'cancer_status') and run on the normalized m/z peak table.
```

## Evaluation signals

- The t-test table lists m/z values with p-values ranked by significance
- In the lung cancer case study, the four known cancer-predictive m/z values (264, 308, 441, 561) should appear among the top-ranked hits
- Results should be consistent with the volcano plot / PLS-DA loading inspection performed on the same normalized dataset
- Only m/z values passing the missing-value threshold set at import (samples missing per m/z) should appear in the output

## Limitations

- Bivariate t-test only captures two-group differences; more complex designs require the multivariate, time-series, or two-factor modes
- Multiple-testing correction across many m/z values is not explicitly described in the provided context
- Annotation of significant m/z values depends on database availability and adduct/isotope scoring, not on the t-test itself

## Evidence

- [methods] Statistical analysis modes: "MetaboShiny offers multiple modes of statistical analysis, with various options for bivariate, multivariate, time series and two-factor experimental variables using paired or unpaired samples"
- [intro] Statistical analysis only on significant hits: "MetaboShiny performs downstream data analysis directly on the m/z ratios and metabolite annotation occurs only on e.g. significant hits that result from the analysis."
- [intro] Workflow steps: "such as data normalization, statistical testing, visualization, clustering or supervised machine learning"
