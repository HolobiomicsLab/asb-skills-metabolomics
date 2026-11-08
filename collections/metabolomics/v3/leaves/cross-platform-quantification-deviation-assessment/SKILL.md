---
name: cross-platform-quantification-deviation-assessment
description: Use when a designed standard-mixture benchmark (known SB:SA dilution
  ratios) has been processed with MetaPro run-1 quantification reports (.xlsx) on
  two or more LC-MS platforms and you must quantify how closely observed log2 fold-changes
  track the design.
when_to_use_negative:
- Quantification results have already undergone manual inspection/integration — this
  skill evaluates raw first-run output specifically
- Input is untargeted feature-table data with no designed mixture ratios or Gm control
  annotations, so no expected log2 fold-change values exist to compare against
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaPro
  role: Produces the first-run relative quantification report (.xlsx) of the benchmark
    standard mixtures on both TripleTOF and QE HF datasets via its semi-targeted Target
    Search workflow
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: pandas
  role: Loads the run-1 .xlsx quantification reports and computes per-group, per-platform
    log2 fold-change statistics
- name: AirdPro
  role: Converts vendor instrument files to Aird format so they can be imported into
    MetaPro for quantification of the benchmark datasets
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:ea439311d9d54a3c9f6849c58eb192287677bd370846ea3e04761213f3b38b3e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - pandas
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# cross-platform-quantification-deviation-assessment

## Summary

Assesses whether first-run (raw, uninspected) relative quantification from MetaPro reproduces designed SB:SA log2 fold changes of a benchmark standard mixture on both TripleTOF 6600 and QE HF platforms, by comparing observed group-mean log2 fold-changes against designed values (-4, -2, -1, 1, 2, 4) and a Gm control group near 0.

## When to use

Use when a designed standard-mixture benchmark (known SB:SA dilution ratios) has been processed with MetaPro run-1 quantification reports (.xlsx) on two or more LC-MS platforms and you must quantify how closely observed log2 fold-changes track the design.

## When NOT to use

- Quantification results have already undergone manual inspection/integration — this skill evaluates raw first-run output specifically
- Input is untargeted feature-table data with no designed mixture ratios or Gm control annotations, so no expected log2 fold-change values exist to compare against

## Inputs

- Run-1 MetaPro-exported quantification report (.xlsx) for the TripleTOF 6600 dataset
- Run-1 MetaPro-exported quantification report (.xlsx) for the QE HF dataset
- Benchmark library annotation distinguishing the Gm control group from the 130-compound differential group
- Designed SB:SA dilution group assignments (1/16, 1/4, 1/2, 2/1, 4/1, 16/1)

## Outputs

- Per-group, per-platform statistics table (fold_change_stats.csv) with mean observed log2 fold-change, signed bias, and absolute error
- Gm control group log2 fold-change estimate expected to center near 0 on both platforms

## How to apply

Load the run-1 MetaPro-exported quantification report (.xlsx) with pandas for both the TripleTOF and QE HF platforms. Partition compounds into the Gm control group and the 130-compound differential group according to the benchmark library annotation. For each designed dilution group (1/16, 1/4, 1/2, 2/1, 4/1, 16/1; expected log2 values -4, -2, -1, 1, 2, 4), compute the mean observed log2 fold-change per group per platform. Quantify deviation of observed from designed log2 fold-change using signed bias and absolute error for every group and platform, and verify the Gm control group centers near 0. Write the per-group, per-platform statistics to fold_change_stats.csv; agreement of observed with designed values on both platforms demonstrates MetaPro's quantification accuracy and stability without manual inspection.

## Related tools

- **MetaPro** (Produces the first-run relative quantification report (.xlsx) of the benchmark standard mixtures on both TripleTOF and QE HF datasets via its semi-targeted Target Search workflow) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **pandas** (Loads the run-1 .xlsx quantification reports and computes per-group, per-platform log2 fold-change statistics)
- **AirdPro** (Converts vendor instrument files to Aird format so they can be imported into MetaPro for quantification of the benchmark datasets) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1

## Examples

```
import pandas as pd; df = pd.read_excel('metapro_run1_report.xlsx'); df['log2FC'] = np.log2(df['SA']/df['SB']); stats = df.groupby(['platform','group'])['log2FC'].mean(); stats.to_csv('fold_change_stats.csv')
```

## Evaluation signals

- Gm control group's observed log2 fold-change is close to 0 on both TripleTOF and QE HF platforms
- Differential groups' observed mean log2 fold-changes match designed values of -4, -2, -1, 1, 2, 4 with quantified deviations (signed bias and absolute error) reported
- fold_change_stats.csv contains one row per dilution group per platform with both bias metrics filled
- Agreement holds for both platforms, indicating cross-platform stability of run-1 quantification (as shown in Fig. 4)
- Distribution comparison of log-transformed relative quantification results before and after manual inspection shows no substantive shift

## Limitations

- Deviation quantification applies only to designed benchmark mixtures with known SB:SA ratios; it cannot validate quantification of unknown biological samples
- Evaluation targets raw first-run output; results may differ after manual integration or Reset integration params/Reset TargetRT operations in MetaPro
- Findings are specific to TripleTOF 6600 and QE HF DDA data processed by MetaPro v1.1.8; other instruments or acquisition methods are not covered

## Evidence

- [results] Comparing the log-transformed relative quantification result distributions before and after manual inspection, MetaPro demonstrated its quantification accuracy and stability: [evidence span withheld — anchor status: not_found]
- [methods] Designed dilution groups and expected log2 values: [evidence span withheld — anchor status: not_found]
- [methods] Datasets used for the two platforms: "The data contains 2 datasets: - **TripleTOF dataset** - 8 converted instrument data (16 files)"
- [abstract] Semi-targeted workflow rationale: "A semi-targeted analysis workflow is designed based on the concept of combining artificial judgment and algorithm inference."
