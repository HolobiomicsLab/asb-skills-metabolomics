---
name: log2-fold-change-calculation
description: Use when you have a MetaPro-exported run-1 quantification report (.xlsx)
  from a benchmark mixture with designed SB:SA dilution ratios (e.g. 1/16 to 16/1
  plus Gm controls) and need to verify quantification accuracy per platform.
when_to_use_negative:
- Input is already an aggregated fold-change or statistics table rather than a raw
  MetaPro run-1 .xlsx quantification report
- The data is untargeted or lacks a designed dilution series with annotated control
  (Gm) and differential compound groups, so no expected log2 values exist to validate
  against
- The goal is post-inspection or manually re-integrated quantification, when the skill
  is defined on first-run, uninspected MetaPro results
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaPro
  role: Produces the run-1 relative quantification report (.xlsx) whose compound-level
    SB:SA log2 fold changes are computed and validated; results are compared before
    and after manual inspection
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro
  role: Converts vendor instrument files (e.g. TripleTOF and QE HF data) into Aird
    format that feeds the MetaPro Target Search analysis underlying the quantification
    report
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:f717aab07fa346066115e44e11435dec1b7cacc1c621c60b643eb511d54c5e50
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# log2-fold-change-calculation

## Summary

Computes and validates observed log2 fold changes from MetaPro-exported quantification reports against a designed SB:SA mixture dilution series, on both TripleTOF 6600 and QE HF HRMS platforms. It quantifies how faithfully first-run (raw, uninspected) MetaPro relative quantification reproduces the benchmark's expected log2 ratios.

## When to use

Use when you have a MetaPro-exported run-1 quantification report (.xlsx) from a benchmark mixture with designed SB:SA dilution ratios (e.g. 1/16 to 16/1 plus Gm controls) and need to verify quantification accuracy per platform.

## When NOT to use

- Input is already an aggregated fold-change or statistics table rather than a raw MetaPro run-1 .xlsx quantification report
- The data is untargeted or lacks a designed dilution series with annotated control (Gm) and differential compound groups, so no expected log2 values exist to validate against
- The goal is post-inspection or manually re-integrated quantification, when the skill is defined on first-run, uninspected MetaPro results

## Inputs

- MetaPro run-1 (first-run, raw/uninspected) exported quantification report in .xlsx format, one per HRMS platform (TripleTOF 6600 and QE HF)
- Benchmark library annotation assigning compounds to the Gm control group or the 130-compound differential dilution groups

## Outputs

- fold_change_stats.csv containing per-group, per-platform statistics: mean observed log2 fold-change, designed log2 value (-4, -2, -1, 1, 2, 4), signed bias, and absolute error
- Validation assessment that the Gm group's log2 fold-change is close to 0 and differential groups match designed values on both platforms

## How to apply

Load the run-1 MetaPro-exported quantification report (.xlsx) with pandas for both the TripleTOF and QE HF platforms. Partition compounds into the Gm control group and the 130-compound differential group using the benchmark library annotation. For each designed dilution group (1/16, 1/4, 1/2, 2/1, 4/1, 16/1), compute the mean observed log2 fold-change per group per platform and compare it against the expected log2 values of -4, -2, -1, 1, 2, 4. Quantify deviation as signed bias and absolute error for every group and platform, and verify the Gm control group centers near 0. Write the per-group, per-platform statistics to fold_change_stats.csv; the rationale is that raw (pre-manual-inspection) results that track the designed ratios demonstrate MetaPro's quantification accuracy and stability.

## Related tools

- **MetaPro** (Produces the run-1 relative quantification report (.xlsx) whose compound-level SB:SA log2 fold changes are computed and validated; results are compared before and after manual inspection) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro** (Converts vendor instrument files (e.g. TripleTOF and QE HF data) into Aird format that feeds the MetaPro Target Search analysis underlying the quantification report) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1

## Examples

```
python -c "import pandas as pd; df=pd.read_excel('MetaPro_run1_quant.xlsx'); df.groupby(['platform','dilution_group'])['log2FC'].mean().to_csv('fold_change_stats.csv')"
```

## Evaluation signals

- Gm control group's mean observed log2 fold-change is close to 0 on both platforms
- Observed log2 fold-changes of the differential groups match the designed values -4, -2, -1, 1, 2, 4 with quantified (signed bias, absolute error) deviations
- fold_change_stats.csv contains one row per designed dilution group per platform, covering both the TripleTOF 6600 and QE HF datasets
- Run-1 result distributions are compared before and after manual inspection to demonstrate stability (as in Fig. 4)
- Groups are partitioned strictly per the benchmark library annotation (Gm control vs 130 differential compounds)

## Limitations

- MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method, so the skill applies to semi-targeted DDA-derived quantification only
- Raw run-1 results may shift after manual inspection (e.g. Reset integration params, Reset TargetRT, Manual Integration), so this validation reflects first-run, uninspected accuracy rather than curated results
- Quantification accuracy is assessed against a designed mixture benchmark; it does not by itself validate identification accuracy, which depends on spectral library quality

## Evidence

- [results] comparing the log-transformed relative quantification result distributions before and after manual inspection, MetaPro demonstrated its quantification accuracy and stability: [evidence span withheld — anchor status: not_found]
- [other] compute the mean observed log2 fold-change per group per platform: [evidence span withheld — anchor status: not_found]
- [other] the Gm group's log2 fold-change should be close to 0: [evidence span withheld — anchor status: not_found]
- [methods] MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method: "MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method."
