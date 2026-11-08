---
name: dso-metabolomics-matrix-handling
description: Use when a cleansed peak table (matrix of samples x peaks) plus sample
  and metabolite labels and biological/blank class assignments must be packaged for
  PLS-Toolbox multivariate tools in Galaxy-M.
when_to_use_negative:
- Input is raw mass spectral data (.dat transients or LC-MS raw files) rather than
  a cleansed, filtered feature matrix
- The analysis requires univariate statistics on individual peaks — the Galaxy-M implementation
  includes no univariate statistics tools outside the per-PC score tests
- Graphical PCA output (scores plots) is required — the PCA script produces no graphical
  output by design
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: Galaxy-M
  role: Provides the Create DSO tool that combines the X data matrix with row/column
    labels and biological/blank class labels into the DSO format consumed by multivariate
    tools
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: PLS-Toolbox
  role: Multivariate library (version 7.0.3 in the paper, 8.0.2 in the README) whose
    MATLAB routines consume the DSO for PCA and associated statistical evaluation
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Matlab
  role: Runtime (2014a, with MCR 8.3) executing the compiled Galaxy-M PCA tool that
    loads the DSO and runs PCA on the X matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:d7d1c6ff3c207fa9f11eddf1ef2970ff0c080f73852591a3f1537966881e6ce5
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# dso-metabolomics-matrix-handling

## Summary

Construct and carry a Galaxy-M DSO (Data Structure Object) that bundles the X data matrix with row/column labels and biological/blank class labels so MATLAB/PLS-Toolbox multivariate tools (e.g. PCA with Scores Test) can consume a single annotated object.

## When to use

Use when a cleansed peak table (matrix of samples x peaks) plus sample and metabolite labels and biological/blank class assignments must be packaged for PLS-Toolbox multivariate tools in Galaxy-M.

## When NOT to use

- Input is raw mass spectral data (.dat transients or LC-MS raw files) rather than a cleansed, filtered feature matrix
- The analysis requires univariate statistics on individual peaks — the Galaxy-M implementation includes no univariate statistics tools outside the per-PC score tests
- Graphical PCA output (scores plots) is required — the PCA script produces no graphical output by design

## Inputs

- Cleansed X data matrix file (samples x peaks) from Galaxy-M filtering/imputation/scaling steps
- Row labels (sample names) and column labels (peak m/z or feature identifiers)
- Class labels describing whether each sample is biological or blank (CSV annotation files from the GalaxyM-TestData folder)

## Outputs

- DSO (Data Structure Object) containing the X matrix, row/column labels and class labels, loadable by Galaxy-M MATLAB/PLS-Toolbox tools
- Downstream: summary statistic text file with PCA scores and per-PC class-separation test results (t-test or ANOVA with Tukey-Kramer)

## How to apply

After Galaxy-M data cleansing (replicate, blank and sample filtering, missing value imputation, normalization and scaling), create a DSO that combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank. The DSO is the required input format for the MATLAB-compiled Galaxy-M tools built on PLS-Toolbox (e.g. PCA): downstream tools load the prepared DSO, run PCA on the X matrix, extract sample scores per retained PC, and apply class-separation tests on the labels held in the DSO. Keep labels consistent with the earlier filtering steps (e.g. blanks flagged during Blank Filtering) so class-based tests operate on the correct samples. Downstream PCA tool output is a summary statistic text file with per-PC test statistics; no graphical output is produced, so correctness rests on the DSO's labels and matrix structure.

## Related tools

- **Galaxy-M** (Provides the Create DSO tool that combines the X data matrix with row/column labels and biological/blank class labels into the DSO format consumed by multivariate tools) — https://github.com/Viant-Metabolomics/Galaxy-M
- **PLS-Toolbox** (Multivariate library (version 7.0.3 in the paper, 8.0.2 in the README) whose MATLAB routines consume the DSO for PCA and associated statistical evaluation) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Matlab** (Runtime (2014a, with MCR 8.3) executing the compiled Galaxy-M PCA tool that loads the DSO and runs PCA on the X matrix)

## Evaluation signals

- Script completes and the summary statistic text file is produced containing class-separation test results for every retained PC
- DSO row labels match the number of biological (and blank) samples entering after replicate/blank/sample filtering; column labels match surviving peaks
- Class labels in the DSO correctly mark each sample as biological or blank, consistent with the Blank Filtering annotation
- PCA on the DSO runs without 'Invalid file identifier'-type MATLAB runtime errors (which indicate WINE/MSFileReader or packaging problems upstream)
- Cumulative explained variance determines the number of retained components when the component count is not user-specified, and per-PC tests appear in the text output

## Limitations

- The DSO's class structure supports biological vs blank labels; class-separation testing reduces to a t-test (2 classes) or one-way ANOVA with Tukey-Kramer (>2 classes) on PC scores, not on individual features
- No univariate statistics tools are included in the current Galaxy-M implementation
- There is no graphical output from the PCA script; the authors state the statistical tests of separation are more reliable than visual interpretation
- MATLAB-compiled tools require MCR 8.3 (or MATLAB 2014a for source use) and, on 64-bit Ubuntu, WINE must be forced to 32-bit mode or the tools may fail with 'Invalid file identifier' errors

## Evidence

- [abstract] Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank: "Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank"
- [results] Load the prepared DSO into the MATLAB-based Galaxy-M PCA tool: [evidence span withheld — anchor status: not_found]
- [abstract] PLS-Toolbox for multivariate tools (version 7.0.3): "PLS-Toolbox for multivariate tools (version 7.0.3)"
- [abstract] There is no graphical output from this script: "There is no graphical output from this script; we believe that the subsequent statistical tests of separation are more reliable than visual interpretation"
- [readme] PLS-Toolbox for PCA tools (version 8.0.2): "[PLS-Toolbox for PCA tools (version 8.0.2)](http://www.eigenvector.com/software/pls_toolbox.htm)"
