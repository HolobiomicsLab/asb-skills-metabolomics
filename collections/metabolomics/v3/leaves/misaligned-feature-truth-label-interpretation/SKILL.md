---
name: misaligned-feature-truth-label-interpretation
description: Use when ncGTW has flagged potentially misaligned feature groups from
  XCMS output and you must decide which flagged features are genuine misalignments
  worth keeping, using CV improvement from ncGTW-guided peak-filling as the truth
  signal.
when_to_use_negative:
- When the goal is only to detect misalignment, not to judge which detected features
  are real — use the ncGTW misalignment-detection module alone instead
- When no paired before/after peak-filling comparison can be computed (e.g., raw profiles
  or XCMS output are unavailable), since truth labels hinge on CV improvement
- When working with features already confirmed well-aligned by the single XCMS warping
  function, which would not enter this labeling workflow
tools:
- name: ncGTW
  role: Supplies the misalignment-detection flags, per-sample warping functions, and
    compCV function used to compute per-feature CV for truth labeling
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Provides the baseline fillPeaks run with the single warping function and the
    realigned fillPeaks run fed by ncGTW warping functions
- name: R
  role: Execution environment; ncGTW is an R/Bioconductor package installed via BiocManager
    or devtools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:a086a05ecbc598a8ef52e5e03784a2942f648e7d30aced49b7bc17cf981763e0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - R
  - XCMS
  - ncGTW
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# misaligned-feature-truth-label-interpretation

## Summary

Interprets features flagged by the ncGTW misalignment-detection module by labeling each as a true positive (circle) or false positive (cross) based on whether substituting the ncGTW per-sample warping functions into XCMS peak-filling actually lowers the coefficient of variation (compCV) for that feature.

## When to use

Use when ncGTW has flagged potentially misaligned feature groups from XCMS output and you must decide which flagged features are genuine misalignments worth keeping, using CV improvement from ncGTW-guided peak-filling as the truth signal.

## When NOT to use

- When the goal is only to detect misalignment, not to judge which detected features are real — use the ncGTW misalignment-detection module alone instead
- When no paired before/after peak-filling comparison can be computed (e.g., raw profiles or XCMS output are unavailable), since truth labels hinge on CV improvement
- When working with features already confirmed well-aligned by the single XCMS warping function, which would not enter this labeling workflow

## Inputs

- XCMS grouped feature table with flagged (potentially misaligned) feature groups
- ncGTW per-sample warping functions for the flagged features (from the realignment step)
- Raw LC-MS profiles for the samples
- Baseline filled feature table from XCMS fillPeaks with the original single warping function

## Outputs

- Per-feature CV values before and after ncGTW-guided fillPeaks (compCV over samples)
- cv_comparison.csv: table distinguishing true positives (circles) from false positives (crosses) of misalignment detection
- Set of true-positive misaligned features retained for downstream analysis

## How to apply

Run XCMS fillPeaks twice on the flagged features: once with the original single XCMS warping function (baseline) and once with the ncGTW per-sample warping functions substituted via the replaced fillPeaksChromPar routine, retrieving missing peaks at the ncGTW-specified locations. Compute the per-feature coefficient of variation of intensity over samples with the ncGTW compCV function on both filled tables. Label a flagged feature a true positive when the CV drops substantially after ncGTW realignment (in the reference run, CV fell from 0.369 to 0.229 and from 0.351 to 0.119 for the two misaligned features); features showing no meaningful CV reduction are labeled false positives (crosses) and screened out for further analysis. Assemble the labeled comparison table (circles vs crosses) as cv_comparison.csv. The rationale is that ncGTW deliberately 'accepts' some false positives at detection time to keep the false negative rate low, so post-alignment CV behavior is the practical screening criterion.

## Related tools

- **ncGTW** (Supplies the misalignment-detection flags, per-sample warping functions, and compCV function used to compute per-feature CV for truth labeling) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Provides the baseline fillPeaks run with the single warping function and the realigned fillPeaks run fed by ncGTW warping functions)
- **R** (Execution environment; ncGTW is an R/Bioconductor package installed via BiocManager or devtools)

## Examples

```
devtools::install_github("ChiungTingWu/ncGTW")
```

## Evaluation signals

- True-positive features must show consistent CV reduction after ncGTW-guided fillPeaks (e.g., the reported pattern 0.369→0.229 and 0.351→0.119 on the Rotterdam/MESA-style data)
- cv_comparison.csv exists and every flagged feature carries an explicit circle (true positive) or cross (false positive) label
- The proportion of labels judged false positive should be checked against the expected elevated false-positive rate (article notes the Rotterdam false positive rate exceeds the theoretical threshold of 0.05)
- The labeled subset of true positives should be a strict subset of the features flagged by the ncGTW misalignment-detection module (e.g., the 57 potentially misaligned features)
- CV must be computed with ncGTW compCV on identically configured fillPeaks runs differing only in the substituted warping functions

## Limitations

- ncGTW intentionally accepts false positives at detection time; the exact mechanism/criteria for screening them out later is not fully detailed in the article, so CV-based labeling is an operational proxy
- The false positive rate in real data (e.g., Rotterdam) is much higher than the theoretical threshold of 0.05, so many flagged features will be labeled crosses
- ncGTW can distort peaks during realignment, which can confound CV-based quality judgments; parameter settings may need adjustment

## Evidence
