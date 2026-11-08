---
name: anova-tukey-kramer-posthoc-analysis
description: Use when PCA sample scores derived from a prepared DSO (X data matrix
  with biological/blank class labels) must be tested for class separation across biological
  groups, especially when more than two classes require post hoc comparisons.
when_to_use_negative:
- The input is raw mass spectral data that has not yet been processed (peak picking,
  alignment, replicate/blank/sample filtering, missing value imputation) and combined
  into a DSO.
- The goal is visual inspection of PCA score plots or loadings — this tool produces
  no graphical output.
- A two-class comparison is needed and a standalone univariate t-test would suffice;
  note Galaxy-M's current implementation includes no separate univariate statistics
  tools, so the scores-test route is tied to PCA output.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Galaxy-M
  role: Provides the PCA with Scores Test tool wrapper and MATLAB-based scripts that
    run the per-PC t-test / ANOVA with Tukey-Kramer post hoc analysis and write the
    summary statistic file
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: PLS-Toolbox
  role: Underlying MATLAB multivariate toolbox used for the PCA computation whose
    retained component scores feed the univariate tests
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Matlab
  role: Runtime/development environment (MATLAB Compiler Runtime 8.3, Matlab 2014a)
    for the compiled PCA with Scores Test executables in Galaxy-M
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:b21d737520369ffd50af5be5f727e775506020ad7b606fdcf6841557bbce743c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy-M
  - Matlab
  - PLS-Toolbox
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# anova-tukey-kramer-posthoc-analysis

## Summary

Statistical evaluation of PCA sample scores in the Galaxy-M 'PCA with Scores Test' tool: a per-principal-component univariate test of class separation (t-test for two classes; one-way ANOVA with Tukey-Kramer post hoc comparisons for more than two), replacing visual interpretation of PCA plots with formal significance testing.

## When to use

Use when PCA sample scores derived from a prepared DSO (X data matrix with biological/blank class labels) must be tested for class separation across biological groups, especially when more than two classes require post hoc comparisons.

## When NOT to use

- The input is raw mass spectral data that has not yet been processed (peak picking, alignment, replicate/blank/sample filtering, missing value imputation) and combined into a DSO.
- The goal is visual inspection of PCA score plots or loadings — this tool produces no graphical output.
- A two-class comparison is needed and a standalone univariate t-test would suffice; note Galaxy-M's current implementation includes no separate univariate statistics tools, so the scores-test route is tied to PCA output.

## Inputs

- Prepared DSO (X data matrix with row and column labels plus biological/blank class labels)
- Class label definitions (biological vs blank groups) embedded in the DSO
- Optional user-specified number of principal components to retain (otherwise auto-selected from cumulative explained variance)

## Outputs

- Summary statistic text file containing per-PC test statistics and significance results for class separation

## How to apply

Load the prepared DSO (X data matrix with row and column labels plus biological/blank class labels) into the MATLAB-based Galaxy-M PCA tool built on PLS-Toolbox. Run PCA on the X matrix, letting the tool automatically determine the number of components to retain from cumulative explained variance when the component count is not user-specified. Extract the sample scores for each retained principal component, then apply a Student's t-test if there are two classes, or a one-way ANOVA followed by Tukey-Kramer post hoc comparisons if there are more than two classes. Write the per-PC test statistics and significance results to a summary statistic text file; no graphical output is produced by design, because the authors judge the statistical tests of separation more reliable than visual interpretation.

## Related tools

- **Galaxy-M** (Provides the PCA with Scores Test tool wrapper and MATLAB-based scripts that run the per-PC t-test / ANOVA with Tukey-Kramer post hoc analysis and write the summary statistic file) — https://github.com/Viant-Metabolomics/Galaxy-M
- **PLS-Toolbox** (Underlying MATLAB multivariate toolbox used for the PCA computation whose retained component scores feed the univariate tests) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Matlab** (Runtime/development environment (MATLAB Compiler Runtime 8.3, Matlab 2014a) for the compiled PCA with Scores Test executables in Galaxy-M) — https://github.com/Viant-Metabolomics/Galaxy-M

## Evaluation signals

- The script completes and the summary statistic text file is produced.
- The summary file contains class-separation test results for every retained principal component.
- When no component count is specified, the retained number of components is consistent with the cumulative-explained-variance criterion.
- Test type matches class count: Student's t-test for exactly two classes; one-way ANOVA plus Tukey-Kramer post hoc comparisons when there are more than two.
- No graphical output is emitted (per-PC statistics in text form are the expected sole artifact).

## Limitations

- There is no graphical output from this script; interpretation relies entirely on the statistical tests of separation rather than visual examination of scores plots.
- No univariate statistics tools are included in the current Galaxy-M implementation outside this PCA scores-test tool.
- The tool depends on the MATLAB Compiler Runtime (version 8.3) and PLS-Toolbox (version 8.0.2 in the README; version 7.0.3 cited in the article), which are optional/licensed components requiring installation.

## Evidence

- [abstract] There is no graphical output from this script; we believe that the subsequent statistical tests of separation are more reliable than visual interpretation: "There is no graphical output from this script; we believe that the subsequent statistical tests of separation are more reliable than visual interpretation"
- [abstract] Note that no univariate statistics tools are included in the current implementation: "Note that no univariate statistics tools are included in the current implementation"
- [readme] PLS-Toolbox for PCA tools (version 8.0.2): "PLS-Toolbox for PCA tools (version 8.0.2)"
- [readme] MATLAB Runtime is a standalone set of shared libraries that enables the execution of compiled MATLAB applications or components on computers that do not have MATLAB installed.: "The MATLAB Runtime is a standalone set of shared libraries that enables the execution of compiled MATLAB applications or components on computers that do not have MATLAB installed."
