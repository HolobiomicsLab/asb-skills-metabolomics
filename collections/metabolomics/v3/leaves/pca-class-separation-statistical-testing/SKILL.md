---
name: pca-class-separation-statistical-testing
description: Decision support for statistically evaluating whether metabolomic sample
  groups separate in a PCA scores plot when no univariate statistics or graphical
  diagnostics are available in the workflow.
when_to_use_negative:
- When univariate per-feature statistics are needed — the article notes no univariate
  statistics tools are included in the current Galaxy-M implementation
- When graphical PCA diagnostics (scores plots) are the primary decision criterion
  — the PCA tool has no graphical output and the authors deem statistical tests of
  separation more reliable
- When inputs are raw spectra rather than a processed, filtered, imputed, normalized
  and labeled X matrix from Create DSO
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0092
tools:
- name: PLS-Toolbox
  role: PCA engine underlying the Galaxy-M PCA tool (version 7.0.3)
- name: Galaxy-M
  role: Hosts the PCA and Scores Test tools; PCA executable runs via the MATLAB Compiler
    Runtime
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: MI-Pack
  role: Downstream metabolite annotation paired with Scores Test results in the published
    histories
  repo: https://github.com/Viant-Metabolomics/MI-Pack
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:de140202152fd0640a02f65cc1f6d82eaa5eaf35511b992a32d9e6f5468c6b28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy-M
  - MI-Pack
  - PLS-Toolbox
  - SimStitch
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# PCA class-separation statistical testing

## Summary

Applies the Galaxy-M PCA tool built on PLS-Toolbox compiled executables, then runs the Scores Test to compute t-test p-values for class separation on the first two principal components, as used in the DIMS (MTBLS79) and LC-MS (MTBLS146) case studies.

## When to use

Decision support for statistically evaluating whether metabolomic sample groups separate in a PCA scores plot when no univariate statistics or graphical diagnostics are available in the workflow.

## When NOT to use

- When univariate per-feature statistics are needed — the article notes no univariate statistics tools are included in the current Galaxy-M implementation
- When graphical PCA diagnostics (scores plots) are the primary decision criterion — the PCA tool has no graphical output and the authors deem statistical tests of separation more reliable
- When inputs are raw spectra rather than a processed, filtered, imputed, normalized and labeled X matrix from Create DSO

## Inputs

- X data matrix file with row and column labels from Create DSO
- Class labels distinguishing biological from blank samples
- Pre-installed Galaxy-M PCA tool (PLS-Toolbox compiled executable run via MATLAB Compiler Runtime)

## Outputs

- Text file containing the Scores Test t-test p-value for class separation on the first two principal components
- PCA model (e.g. seven PCs retaining ≥70% of variance in the LC-MS MTBLS146 case)

## How to apply

After building the X matrix with Create DSO (row/column labels plus biological-vs-blank class labels), run PCA via the Galaxy-M PCA tool, a PLS-Toolbox 7.0.3-based executable executed through the MATLAB Compiler Runtime. The tool computes the principal component model but produces no graphical output — the article explicitly states the Scores Test is preferred over visual interpretation. Then invoke the Scores Test tool to compute the significance of class separation on the first two PCs via a t-test, saving the p-value as a text file. In the DIMS MTBLS79 case study (cow vs sheep heart subset with QC and blank), this yields a p-value list for significance of separation of the two biological classes. In the LC-MS MTBLS146 case study, PCA retained seven principal components describing at least 70% of the variance. Validate by comparing the Scores Test p-value table against the published history outputs (Additional file 1 Tables S2 and S4).

## Related tools

- **PLS-Toolbox** (PCA engine underlying the Galaxy-M PCA tool (version 7.0.3))
- **Galaxy-M** (Hosts the PCA and Scores Test tools; PCA executable runs via the MATLAB Compiler Runtime) — https://github.com/Viant-Metabolomics/Galaxy-M
- **MI-Pack** (Downstream metabolite annotation paired with Scores Test results in the published histories) — https://github.com/Viant-Metabolomics/MI-Pack

## Examples

```
Run the 'PCA with Scores Test' tool in Galaxy-M on the Create DSO output to generate the Scores Test p-value text file for the LCMS workflow history.
```

## Evaluation signals

- Scores Test output is a text file of t-test p-values for separation of biological classes on PC1 and PC2
- In the LC-MS MTBLS146 case study, the retained PCA model has seven PCs describing at least 70% of the variance
- Scores Test table matches published Additional file 1: Table S4 (LC-MS) or the p-value list reproduced in the DIMS MTBLS79 history
- X matrix fed to PCA is a DSO assembled by Create DSO with biological-vs-blank class labels, not raw spectra
- No graphical output is expected from the PCA tool itself; absence of plots is normal, not a failure

## Limitations

- The Galaxy-M PCA tool produces no graphical output — no scores plots can be inspected directly
- No univariate statistics tools are included in the current implementation, so per-feature tests must be done elsewhere
- The Scores Test evaluates only class separation on the first two principal components, not arbitrary numbers of PCs
- Exact p-value reproducibility depends on using the same filtered, imputed, normalized X matrix produced upstream in the Galaxy-M workflow

## Evidence

- [abstract] Scores Test: performs statistical tests of separation between classes in the PCA model, using the number of principal components to retain that is selected by the user: [evidence span withheld — anchor status: not_found]
- [abstract] There is no graphical output from this script; we believe that the subsequent statistical tests of separation are more reliable than visual interpretation: "There is no graphical output from this script; we believe that the subsequent statistical tests of separation are more reliable than visual interpretation"
- [abstract] Note that no univariate statistics tools are included in the current implementation: "Note that no univariate statistics tools are included in the current implementation"
- [abstract] Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank: "Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank"
- [results] The PCA model retained seven principal components, describing a cumulative variance of 71%: [evidence span withheld — anchor status: not_found]
- [methods] To use the PCA tools, users must first install PLS-Toolbox and the MATLAB Compiler Runtime (MCR), following the published Galaxy-M instructions: [evidence span withheld — anchor status: not_found]
