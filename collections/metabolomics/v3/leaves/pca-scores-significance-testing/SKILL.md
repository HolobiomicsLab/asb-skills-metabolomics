---
name: pca-scores-significance-testing
description: Use when a cleaned, class-labelled X data matrix (DSO with row/column
  labels and biological/blank class labels) is ready and you need PCA with per-component
  statistical evidence of class separation rather than visual scores plots.
when_to_use_negative:
- The X matrix has not yet been converted into a DSO with row/column labels and biological/blank
  class labels (run the Galaxy-M 'Create DSO' step first)
- A graphical output such as scores or loadings plots is required — this tool produces
  none by design
- The input is raw mass spectral data (e.g. DIMS transients or LC-MS features) that
  still requires peak picking, replicate/blank/sample filtering, missing value imputation,
  and normalization/scaling
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: PLS-Toolbox
  role: Provides the multivariate PCA engine (version 7.0.3 in the article; 8.0.2
    per README) used to compute components and scores inside the Galaxy-M 'PCA with
    Scores Test' tool
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy-M
  role: Wraps the MATLAB-compiled PCA with Scores Test tool as a Galaxy tool for reproducible
    metabolomics workflows; supplies the DSO input conventions and summary-file output
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Matlab
  role: Runtime/compiler (R2014a, MCR 8.3) under which the PCA with Scores Test tool
    executes the component selection, score extraction, and t-test/ANOVA statistics
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:fe58a487d69575ce43d284d3532b98bcacaf05b1ac3fc41b180996a7d611dc85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy-M
  - Matlab
  - PLS-Toolbox
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# PCA scores significance testing

## Summary

Runs PCA on a prepared metabolomics data matrix (DSO) in Galaxy-M using the MATLAB/PLS-Toolbox 'PCA with Scores Test' tool, automatically selects the number of components from cumulative explained variance, and statistically evaluates each retained PC's sample scores for class separation (Student's t-test or one-way ANOVA with Tukey-Kramer post hoc), writing results to a summary text file.

## When to use

Use when a cleaned, class-labelled X data matrix (DSO with row/column labels and biological/blank class labels) is ready and you need PCA with per-component statistical evidence of class separation rather than visual scores plots.

## When NOT to use

- The X matrix has not yet been converted into a DSO with row/column labels and biological/blank class labels (run the Galaxy-M 'Create DSO' step first)
- A graphical output such as scores or loadings plots is required — this tool produces none by design
- The input is raw mass spectral data (e.g. DIMS transients or LC-MS features) that still requires peak picking, replicate/blank/sample filtering, missing value imputation, and normalization/scaling

## Inputs

- Prepared DSO (Data Structure Object): X data matrix with row and column labels plus class labels describing whether each sample is biological or blank
- Optional user-specified number of principal components to retain

## Outputs

- Summary statistic text file containing per-PC test statistics and class-separation significance results for every retained principal component
- Sample scores for each retained principal component (no graphical output)

## How to apply

Load the prepared DSO (the X data matrix combined with row and column labels plus class labels marking samples as biological or blank) into the Galaxy-M PCA tool built on MATLAB and PLS-Toolbox. When the number of components is not user-specified, the tool determines the optimal component count by interrogating the cumulative variance explained by each component. It then extracts the sample scores for each retained PC and applies a univariate test per component: a Student's t-test when there are two classes, or a one-way ANOVA followed by Tukey-Kramer post hoc comparisons when there are more than two. The per-PC test statistics and significance results are written to a summary statistic text file; deliberately no graphical output is produced because the statistical tests of separation are considered more reliable than visual interpretation. Note the tool operates on PLS-Toolbox (version 7.0.3 in the published workflow, 8.0.2 per the README) run under MATLAB R2014a / MCR 8.3 within Galaxy.

## Related tools

- **PLS-Toolbox** (Provides the multivariate PCA engine (version 7.0.3 in the article; 8.0.2 per README) used to compute components and scores inside the Galaxy-M 'PCA with Scores Test' tool) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy-M** (Wraps the MATLAB-compiled PCA with Scores Test tool as a Galaxy tool for reproducible metabolomics workflows; supplies the DSO input conventions and summary-file output) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Matlab** (Runtime/compiler (R2014a, MCR 8.3) under which the PCA with Scores Test tool executes the component selection, score extraction, and t-test/ANOVA statistics) — https://github.com/Viant-Metabolomics/Galaxy-M

## Evaluation signals

- The tool script completes without error and the summary statistic text file is produced
- The summary text file contains class-separation test results for every retained PC (no missing per-component entries)
- The test applied matches the class structure: Student's t-test for two classes; one-way ANOVA with Tukey-Kramer post hoc comparisons for more than two classes
- The retained component count agrees with the cumulative-explained-variance criterion when no component number was user-specified
- No graphical output is produced (its absence is expected, not a failure)

## Limitations

- There is no graphical output from this script; interpretation relies entirely on the statistical tests of separation rather than visual inspection of scores plots
- No univariate statistics tools are included elsewhere in the current Galaxy-M implementation, so this PCA tool's built-in tests are the only per-class separation statistics
- Requires the MATLAB Compiler Runtime (MCR 8.3) / MATLAB 2014a and licensed PLS-Toolbox (7.0.3 or 8.0.2), which complicates installation outside the pre-installed GigaDB virtual machine

## Evidence

- [abstract] principal components analysis (PCA) with associated statistical evaluation: "principal components analysis (PCA) with associated statistical evaluation"
- [abstract] Create DSO description: "Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank"
- [abstract] No graphical output rationale: "There is no graphical output from this script; we believe that the subsequent statistical tests of separation are more reliable than visual interpretation"
- [abstract] PLS-Toolbox version in article: "PLS-Toolbox for multivariate tools (version 7.0.3)"
- [readme] PLS-Toolbox version in README: "[PLS-Toolbox for PCA tools (version 8.0.2)](http://www.eigenvector.com/software/pls_toolbox.htm)"
