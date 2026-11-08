---
name: students-t-test-class-comparison
description: Use when PCA sample scores from a preprocessed metabolomics data matrix
  (DSO) are available and you need per-component evidence of whether biological and
  blank (or other) sample classes are statistically separated.
when_to_use_negative:
- Input is not yet a prepared DSO (e.g. raw .dat transient files or unaligned XCMS
  feature tables) — data cleansing, filtering, and DSO creation must precede this
  test
- You expect plots or graphical PCA output — this tool explicitly produces none
- You need standalone univariate statistics on individual metabolite features rather
  than tests on PCA scores — the article notes 'no univariate statistics tools are
  included in the current implementation'
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Galaxy-M
  role: Hosts the PCA with Scores Test tool that runs PCA and applies the t-test/ANOVA
    on component scores
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: PLS-Toolbox
  role: MATLAB multivariate library underlying the PCA computation whose scores are
    subsequently tested
  repo: http://www.eigenvector.com/software/pls_toolbox.htm
- name: Matlab
  role: Runtime environment (compiled via MATLAB Compiler Runtime 8.3 / R2014a) in
    which the scores-test script executes
- name: Galaxy
  role: Web platform wrapping the tool so the test can be run reproducibly in a workflow
    history
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:0aa94fcc3d4e2a9a91557c580562b39ae986298dad8889313a1b4575d8c15229
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - Matlab
  - PLS-Toolbox
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# students-t-test-class-comparison

## Summary

Statistically evaluates class separation on principal component scores by applying a Student's t-test (two classes) or one-way ANOVA with Tukey-Kramer post hoc comparisons (more than two classes) in the Galaxy-M PCA with Scores Test tool, replacing visual PCA interpretation with significance testing.

## When to use

Use when PCA sample scores from a preprocessed metabolomics data matrix (DSO) are available and you need per-component evidence of whether biological and blank (or other) sample classes are statistically separated.

## When NOT to use

- Input is not yet a prepared DSO (e.g. raw .dat transient files or unaligned XCMS feature tables) — data cleansing, filtering, and DSO creation must precede this test
- You expect plots or graphical PCA output — this tool explicitly produces none
- You need standalone univariate statistics on individual metabolite features rather than tests on PCA scores — the article notes 'no univariate statistics tools are included in the current implementation'

## Inputs

- DSO: combined X data matrix file with row and column label information and class labels describing whether each sample is biological or blank
- Optional user-specified number of principal components to retain (otherwise auto-determined from cumulative explained variance)
- Class label assignment for samples (2 classes for t-test; >2 classes triggers ANOVA + Tukey-Kramer)

## Outputs

- Summary statistic text file containing per-PC class-separation test statistics and significance results for every retained principal component

## How to apply

Load the prepared DSO — the X data matrix with row and column labels plus biological/blank class labels — into the MATLAB-based Galaxy-M PCA tool built on PLS-Toolbox. Run PCA, automatically choosing the number of components to retain by interrogating cumulative explained variance when the user does not specify a component count. Extract the sample scores for each retained principal component, then for each PC test class separation on the scores: a Student's t-test when there are exactly two classes, or a one-way ANOVA followed by Tukey-Kramer post hoc comparisons when there are more than two. Write the per-PC test statistics and significance results to a summary statistic text file; by design no graphical output is produced, since the authors consider the statistical tests of separation more reliable than visual interpretation.

## Related tools

- **Galaxy-M** (Hosts the PCA with Scores Test tool that runs PCA and applies the t-test/ANOVA on component scores) — https://github.com/Viant-Metabolomics/Galaxy-M
- **PLS-Toolbox** (MATLAB multivariate library underlying the PCA computation whose scores are subsequently tested) — http://www.eigenvector.com/software/pls_toolbox.htm
- **Matlab** (Runtime environment (compiled via MATLAB Compiler Runtime 8.3 / R2014a) in which the scores-test script executes)
- **Galaxy** (Web platform wrapping the tool so the test can be run reproducibly in a workflow history) — https://github.com/galaxyproject/galaxy

## Evaluation signals

- The script completes and a summary statistic text file is produced containing class-separation test results for every retained PC
- The file contains a separate test statistic per retained principal component, not a single global value
- With exactly two labelled classes the reported test is a Student's t-test; with more than two classes the output shows ANOVA with Tukey-Kramer post hoc comparisons
- When the component count was not user-specified, the retained number of PCs matches the cumulative explained variance criterion chosen by the tool
- No graphical files (plots, loadings/scores images) are emitted — absence of graphical output is expected behaviour

## Limitations

- There is no graphical output from this script; separation is judged only through the statistical tests rather than visual inspection of scores plots
- The tests operate on PCA scores, so they reflect class separation along retained components and do not provide univariate statistics on individual metabolites
- Component selection depends on cumulative explained variance when no component count is given, which may not suit datasets where discriminating information lies in low-variance components

## Evidence

- [abstract] No graphical output from PCA scores test: "There is no graphical output from this script; we believe that the subsequent statistical tests of separation are more reliable than visual interpretation"
- [abstract] PCA with associated statistical evaluation: "principal components analysis (PCA) with associated statistical evaluation"
- [abstract] No univariate statistics tools: "Note that no univariate statistics tools are included in the current implementation"
- [readme] PLS-Toolbox version used for PCA tools: "PLS-Toolbox for PCA tools (version 8.0.2)](http://www.eigenvector.com/software/pls_toolbox.htm) (Optional: ftool development and use of"
