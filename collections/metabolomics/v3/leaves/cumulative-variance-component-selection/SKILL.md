---
name: cumulative-variance-component-selection
description: Use when running PCA on a prepared metabolomics X data matrix (DSO) and
  the number of components is not specified by the user, so the component count must
  be determined from cumulative explained variance.
when_to_use_negative:
- The data is still raw spectra or an unfiltered peak table — the input must be a
  cleansed, prepared DSO with biological/blank class labels
- Graphical PCA output (scores plots, loadings plots, scree plots) is required — this
  script produces no graphical output by design
- A general-purpose univariate statistics pipeline is needed — the article notes no
  univariate statistics tools are included in the current implementation outside this
  PCA-coupled testing
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3173
tools:
- name: Galaxy-M PCA tool (PLS-Toolbox/MATLAB executable)
  role: Implements PCA on the DSO, auto-selects the retained component count from
    cumulative explained variance, and runs t-test/ANOVA+Tukey-Kramer on the scores
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: PLS-Toolbox
  role: Underlying multivariate library providing the PCA computation
- name: Create DSO
  role: Upstream Galaxy-M tool that combines the X data matrix with row/column labels
    and biological/blank class labels to form the tool's input
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:d90a7d92b75e5695d026effb23e886b96801b8a211006b9f6378d791519dd0fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Create DSO
  - Galaxy-M PCA tool (PLS-Toolbox/MATLAB executable)
  - PLS-Toolbox
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cumulative-variance-component-selection

## Summary

Automatic selection of the number of principal components to retain in PCA by interrogating cumulative explained variance per component, as implemented in the Galaxy-M 'PCA with Scores Test' tool built on PLS-Toolbox/MATLAB, followed by statistical testing of each retained component's scores.

## When to use

Use when running PCA on a prepared metabolomics X data matrix (DSO) and the number of components is not specified by the user, so the component count must be determined from cumulative explained variance.

## When NOT to use

- The data is still raw spectra or an unfiltered peak table — the input must be a cleansed, prepared DSO with biological/blank class labels
- Graphical PCA output (scores plots, loadings plots, scree plots) is required — this script produces no graphical output by design
- A general-purpose univariate statistics pipeline is needed — the article notes no univariate statistics tools are included in the current implementation outside this PCA-coupled testing

## Inputs

- Prepared DSO (dataset object) combining the X data matrix file with row and column labels and class labels describing whether each sample is biological or blank
- Optionally, a user-specified number of principal components to retain (otherwise determined automatically from cumulative explained variance)

## Outputs

- Summary statistic text file containing per-PC class-separation test statistics and significance results (t-test or ANOVA/Tukey-Kramer)
- Sample scores for each retained principal component (intermediate)

## How to apply

Load the prepared DSO — the X data matrix with row and column labels plus biological/blank class labels created by the Create DSO tool — into the Galaxy-M PCA tool, which wraps PLS-Toolbox (version 7.0.3 in the publication, 8.0.2 in the repository) compiled to a MATLAB executable. When the user does not specify a component count, the tool automatically chooses the optimum number of components by interrogating the cumulative variance explained by each component rather than requiring manual inspection of scree plots. It then extracts the sample scores for each retained principal component and applies a univariate test of class separation on the scores: a Student's t-test when there are two classes, or a one-way ANOVA followed by Tukey-Kramer post hoc comparisons when there are more than two classes. Per-PC test statistics and significance results are written to a summary statistic text file. The rationale is that the statistical tests of separation on the scores are considered more reliable than visual interpretation, so no graphical output is produced.

## Related tools

- **Galaxy-M PCA tool (PLS-Toolbox/MATLAB executable)** (Implements PCA on the DSO, auto-selects the retained component count from cumulative explained variance, and runs t-test/ANOVA+Tukey-Kramer on the scores) — https://github.com/Viant-Metabolomics/Galaxy-M
- **PLS-Toolbox** (Underlying multivariate library providing the PCA computation)
- **Create DSO** (Upstream Galaxy-M tool that combines the X data matrix with row/column labels and biological/blank class labels to form the tool's input) — https://github.com/Viant-Metabolomics/Galaxy-M

## Evaluation signals

- The script completes and a summary statistic text file is produced containing class-separation test results for every retained PC
- The number of retained components reported is consistent with the cumulative explained variance profile of the X matrix (when the count was auto-selected)
- The number of test results in the output equals the number of retained components (one t-test or one ANOVA+Tukey-Kramer set per PC)
- Test type matches the class structure: Student's t-test for two classes, one-way ANOVA with Tukey-Kramer post hoc comparisons for more than two
- No graphical outputs are expected; absence of plots is the correct behavior, not a failure

## Limitations

- There is no graphical output from this script; the authors believe the statistical tests of separation are more reliable than visual interpretation, so no scree/scores plots are available for manual inspection
- No univariate statistics tools are included in the current implementation beyond the tests coupled to this PCA tool
- Requires the DSO preparation step (Create DSO) to have been run; MATLAB Compiler Runtime (MCR v8.3 / R2014a) and PLS-Toolbox are needed for source-code use

## Evidence

- [other] When the number of components is not specified, the tool first chooses the optimum number of components by interrogating the cumulative variance in each component, then applies a univariate statistical test to the scores.: [evidence span withheld — anchor status: not_found]
- [abstract] There is no graphical output from this script; the statistical tests of separation are considered more reliable than visual interpretation.: "There is no graphical output from this script; we believe that the subsequent statistical tests of separation are more reliable than visual interpretation"
- [abstract] Create DSO combines the X data matrix file with row and column label information and class labels describing biological or blank samples.: "Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank"
- [readme] PLS-Toolbox for PCA tools (version 8.0.2) is listed as an optional requirement in the repository README.: "[PLS-Toolbox for PCA tools (version 8.0.2)](http://www.eigenvector.com/software/pls_toolbox.htm)"
