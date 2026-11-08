---
name: fixed-effects-anova-feature-testing
description: Use when you have a wide-format feature table (samples as columns, features
  as rows) plus a design file mapping samples to treatment groups, and need per-feature
  tests of group differences.
when_to_use_negative:
- Input is raw spectral data (mzML/NetCDF) rather than a preprocessed feature table
  with a design file — this skill operates on the 'data analysis' stage of the workflow,
  after peak picking and preprocessing.
- Samples have no grouping annotation in the design file, so the single-factor treatment-group
  model has no factor to fit.
- Group sizes are too small or errors are grossly non-normal for the tool's stated
  assumption of normal, identically distributed errors — the Kruskal-Wallis non-parametric
  test in the same suite is the alternative.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: SECIMTools
  role: Provides the ANOVA tool that fits the single-factor fixed-effects model per
    feature, computes pairwise contrasts, and generates volcano plots and diagnostics;
    runnable standalone or wrapped for Galaxy.
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: 'Alternate execution environment: the SECIMTools ANOVA tool can be run as
    a Galaxy-wrapped tool within metabolomics workflow suites.'
- name: Python
  role: Standalone execution of the ANOVA tool from the secimtools pypi package.
  repo: https://pypi.python.org/pypi/secimtools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:f2109539e6e6675faff91a3d11cc20314b971927f63bbe240d8f9a8abb0fc1a5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# fixed-effects-anova-feature-testing

## Summary

Per-feature single-factor fixed-effects ANOVA for metabolomics feature tables: fits the SECIMTools ANOVA model across treatment groups, extracts pairwise-contrast p-values, group means, F-statistics, and significance flags per feature.

## When to use

Use when you have a wide-format feature table (samples as columns, features as rows) plus a design file mapping samples to treatment groups, and need per-feature tests of group differences.

## When NOT to use

- Input is raw spectral data (mzML/NetCDF) rather than a preprocessed feature table with a design file — this skill operates on the 'data analysis' stage of the workflow, after peak picking and preprocessing.
- Samples have no grouping annotation in the design file, so the single-factor treatment-group model has no factor to fit.
- Group sizes are too small or errors are grossly non-normal for the tool's stated assumption of normal, identically distributed errors — the Kruskal-Wallis non-parametric test in the same suite is the alternative.

## Inputs

- Wide-format feature table: samples as columns, metabolomics features as rows with unique feature identifiers (SECIMTools example dataset format)
- Design file mapping each sample to its treatment group annotation

## Outputs

- anova_results.csv — per-feature table of group means, ANOVA F-statistics, pairwise contrast p-values, and binary significance flags
- Volcano plots (log base 10 p-value vs difference between group means) — optional
- Model diagnostics plots — optional

## How to apply

Load the SECIMTools-style wide dataset — rows are unique metabolomics feature identifiers, columns are samples — together with its design file that maps each sample to its treatment group annotation. For each feature row, fit the fully fixed single-factor ANOVA model across the treatment groups as implemented in the SECIMTools ANOVA tool, operating under the tool's assumptions of normal, identically distributed errors and calculating all pairwise contrasts. Extract the raw p-value for each pairwise group contrast, per-feature group means, and the ANOVA F-statistic. Derive binary significance flag columns directly from the contrast p-values. Write the per-feature results table (group means, F-statistics, p-values, flags) to anova_results.csv; optionally render the volcano plots (log base 10 p-value versus difference between group means) and model diagnostics the tool produces. Rationale: the article positions ANOVA among SECIMTools' basic statistical analysis methods for metabolomics, complementing PCA, PLS-DA, t-test, and Kruskal-Wallis, and standard data formats let this step link downstream to other SECIMTools tools.

## Related tools

- **SECIMTools** (Provides the ANOVA tool that fits the single-factor fixed-effects model per feature, computes pairwise contrasts, and generates volcano plots and diagnostics; runnable standalone or wrapped for Galaxy.) — https://github.com/SECIMTools
- **Galaxy** (Alternate execution environment: the SECIMTools ANOVA tool can be run as a Galaxy-wrapped tool within metabolomics workflow suites.)
- **Python** (Standalone execution of the ANOVA tool from the secimtools pypi package.) — https://pypi.python.org/pypi/secimtools

## Examples

```
python anova_analysis.py --wide_table secim_example.csv --design_file design.csv --output anova_results.csv
```

## Evaluation signals

- anova_results.csv exists with one row per feature identifier from the input wide table and columns for group means, F-statistic, pairwise p-values, and flags.
- Every pairwise group contrast present in the design file has a corresponding p-value column; no contrast is silently dropped.
- Binary flag columns agree exactly with the sign of the corresponding p-value (non-trivial p-values map to 1, NA/absent to 0).
- Per-feature group means recomputed directly from the input table match the values reported in anova_results.csv.
- If volcano plots are rendered, each x-axis value equals the difference of the two group means and each y-axis value equals -log10(p) for that feature/contrast pair.

## Limitations

- The model assumes normal and identically distributed errors; heavily skewed or heteroscedastic metabolomics features violate this — the article lists the Kruskal-Wallis non-parametric test as the suite's alternative.
- The tool tests a single fixed factor across treatment groups; multi-factor or mixed-effects designs are not described in the article.
- Feature intensity must be comparable across run order — the article's Run Order Regression (ROR) tool exists precisely to evaluate run-order effects, so uncorrected batch/run-order drift can confound the ANOVA contrasts.

## Evidence

- [abstract] basic statistical analysis methods (partial least squares - discriminant analysis, analysis of variance, t-test, Kruskal-Wallis non-parametric test): "basic statistical analysis methods (partial least squares - discriminant analysis, analysis of variance, t-test, Kruskal-Wallis non-parametric test)"
- [readme] ANOVA across treatment groups with volcano plots and diagnostics: "basic statistical analysis methods (partial least squares - discriminant analysis, analysis of variance)"
- [other] fixed-effects ANOVA workflow details from task card: [evidence span withheld — anchor status: not_found]
- [methods] Run Order Regression caveat: "The ROR tool uses linear regression to evaluate the relationship between feature intensity and the run order."
- [readme] SECIMTools availability as pypi package: "SECIMTools are available as a secimtools pypi package. Project has also been packaged for bioconda and Galaxy Genomics Framework."
