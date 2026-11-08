---
name: metabolomics-quantification-report-processing
description: Use when a MetaPro run-1 quantification export (.xlsx) is available for
  a benchmark mixture design with known SB:SA dilution ratios (1/16 to 16/1) and you
  need to verify that observed log2 fold changes track the designed values before
  manual peak inspection.
when_to_use_negative:
- The data are still vendor instrument files (e.g., unconverted .raw/.wiff2) that
  have not been processed by AirdPro conversion and MetaPro Target Search, so no quantification
  report exists yet.
- The study has no designed dilution standards or control groups, so there is no ground-truth
  fold change to validate quantification against.
- The goal is interactive peak-shape QC (manual integration, Reset TargetRT) rather
  than numerical validation of run-1 quantification accuracy.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MetaPro
  role: Generates the run-1 semi-targeted quantification report via Target Search
    whose log2 fold changes are validated against the designed SB:SA ratios
  repo: https://github.com/CSi-Studio/MetaPro
- name: AirdPro
  role: 'Pre-step: converts vendor instrument files (DDA) to Aird format that MetaPro
    analyzes for both the TripleTOF and QE HF datasets'
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:2647fab72d913ddb0e890e59c3f933697b03ea3463b332c808b20a94d6eee80a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/MetaPro
  tools:
  - AirdPro
  - MetaPro
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metabolomics-quantification-report-processing

## Summary

Validates the quantification accuracy of a first-run (raw, uninspected) MetaPro LC-MS semi-targeted analysis by comparing log2-transformed relative quantification fold changes against a benchmark standard-mixture design (Gm controls plus 130 compounds at designed SB:SA ratios) on both TripleTOF 6600 and QE HF platforms.

## When to use

Use when a MetaPro run-1 quantification export (.xlsx) is available for a benchmark mixture design with known SB:SA dilution ratios (1/16 to 16/1) and you need to verify that observed log2 fold changes track the designed values before manual peak inspection.

## When NOT to use

- The data are still vendor instrument files (e.g., unconverted .raw/.wiff2) that have not been processed by AirdPro conversion and MetaPro Target Search, so no quantification report exists yet.
- The study has no designed dilution standards or control groups, so there is no ground-truth fold change to validate quantification against.
- The goal is interactive peak-shape QC (manual integration, Reset TargetRT) rather than numerical validation of run-1 quantification accuracy.

## Inputs

- MetaPro run-1 (raw, uninspected) exported quantification report in .xlsx format, one per platform
- Benchmark library annotation mapping compounds to the Gm control group and the 130-compound differential group
- Designed SB:SA dilution scheme (1/16, 1/4, 1/2, 2/1, 4/1, 16/1) for the standard mixtures

## Outputs

- Per-group, per-platform mean observed log2 fold-change table (fold_change_stats.csv)
- Signed bias and absolute error statistics for each dilution group on both TripleTOF and QE HF datasets

## How to apply

Load the run-1 MetaPro-exported quantification report (.xlsx) with pandas for each platform (TripleTOF 6600 and QE HF). Partition compounds into the Gm control group and the 130-compound differential group using the benchmark library annotation. For each designed dilution group (1/16, 1/4, 1/2, 2/1, 4/1, 16/1, i.e., expected log2 values -4, -2, -1, 1, 2, 4), compute the mean observed log2 fold-change per group per platform. Quantify deviation of observed from designed values as signed bias and absolute error, and verify the Gm control group centers near 0. Write the per-group, per-platform statistics to fold_change_stats.csv; tracking of designed ratios in run-1 (before manual inspection) demonstrates quantification accuracy and stability per Fig. 4 of the MetaPro paper.

## Related tools

- **MetaPro** (Generates the run-1 semi-targeted quantification report via Target Search whose log2 fold changes are validated against the designed SB:SA ratios) — https://github.com/CSi-Studio/MetaPro
- **AirdPro** (Pre-step: converts vendor instrument files (DDA) to Aird format that MetaPro analyzes for both the TripleTOF and QE HF datasets) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1

## Examples

```
import pandas as pd; df = pd.read_excel('run1_quantification.xlsx'); groups = df.groupby(['platform','dilution_group'])['log2_fold_change'].mean()
```

## Evaluation signals

- The Gm control group's observed log2 fold-change is close to 0 on both platforms.
- Observed log2 fold-changes for the 130-compound differential groups match the designed values -4, -2, -1, 1, 2, 4 within the quantified deviations.
- fold_change_stats.csv contains one row per dilution group per platform with both signed bias and absolute error columns populated.
- Run-1 (pre-inspection) result distributions align with post-manual-inspection distributions, mirroring the Fig. 4 comparison in the paper.
- The same pattern holds consistently on both the TripleTOF 6600 and QE HF datasets (16 and 20 converted files respectively).

## Limitations

- The validation relies on a benchmark library design (Gm controls and designed dilutions); it cannot certify quantification accuracy for compounds lacking such ground-truth standards.
- Manual inspection remains indispensable: the paper stresses that many tools 'directly output the quantification results without offering manual modification functions,' so run-1 accuracy is an approximate, distribution-level check rather than per-peak correctness.

## Evidence

- [methods] Load the run-1 MetaPro-exported quantification report (.xlsx) with pandas for both the TripleTOF and QE HF platforms.: [evidence span withheld — anchor status: not_found]
- [methods] designed dilution groups: [evidence span withheld — anchor status: not_found]
- [results] accuracy and stability finding: [evidence span withheld — anchor status: not_found]
- [intro] manual modification limitation: "many pieces of metabolomics software directly output the quantification results without offering manual modification functions toward the quantification results"
