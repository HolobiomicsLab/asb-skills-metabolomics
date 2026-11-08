---
name: r-squared-goodness-of-fit-verification
description: Use when comparing quantification outputs from two MRM analysis tools
  (e.g. MRMPro vs. Skyline) across a common dilution/file series and you need a numeric
  goodness-of-fit (R-squared) check of peak area vs. concentration curves against
  reference thresholds.
when_to_use_negative:
- Input is already a validated calibration/feature table with no need for tool-vs-tool
  agreement testing
- The dataset lacks a concentration series (peak areas only, no dilution/points to
  regress against)
- Data are not MRM transition peak areas (e.g. DDA/DIA identification results), where
  peak-area-vs-concentration calibration curves are not the analysis target
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MRMPro
  role: Web-based MRM data analysis platform whose quantification output (Drug_light/Drug_heavy
    peak areas) supplies one side of the R-squared comparison
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: Skyline
  role: Reference tool providing default and manually integrated peak areas to correlate
    against MRMPro results
- name: AirdPro
  role: GUI client for converting vendor MRM files to Aird format so the quantification
    test dataset can be analyzed downstream
  repo: https://github.com/CSi-Studio/AirdPro/releases/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:387b54eed3ef0310dd5957614c3f80fe960e760bb9c367af3722a193f4b0ae23
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MRMPro
  - Skyline
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# R-squared goodness-of-fit verification

## Summary

Fit least-squares linear calibration curves (peak area vs. concentration) for MRM transitions and verify tool agreement by comparing computed R-squared values against reported references, as done when benchmarking MRMPro against Skyline on the 113-file Skyline small-molecule quantification test dataset.

## When to use

Use when comparing quantification outputs from two MRM analysis tools (e.g. MRMPro vs. Skyline) across a common dilution/file series and you need a numeric goodness-of-fit (R-squared) check of peak area vs. concentration curves against reference thresholds.

## When NOT to use

- Input is already a validated calibration/feature table with no need for tool-vs-tool agreement testing
- The dataset lacks a concentration series (peak areas only, no dilution/points to regress against)
- Data are not MRM transition peak areas (e.g. DDA/DIA identification results), where peak-area-vs-concentration calibration curves are not the analysis target

## Inputs

- 113-file Skyline small-molecule quantification test dataset with Drug_light and Drug_heavy MRM transitions
- MRMPro peak area (quantification) output per transition per file
- Skyline peak areas under default peak integration settings
- Skyline peak areas under manual inspection/manual integration for the Drug_heavy transition

## Outputs

- Per-transition R-squared comparison table with pass/fail verification flags (r2_comparison.csv)
- Linear calibration curves (peak area vs. concentration) per tool/integration condition

## How to apply

Load the 113-file Skyline small-molecule quantification test dataset containing the Drug_light and Drug_heavy MRM transitions, then extract peak areas for both transitions from the MRMPro analysis output and from Skyline using its default peak integration settings. For each transition under each tool/integration condition, fit a linear calibration curve (peak area vs. concentration) by least-squares regression and compute the R-squared value. Compare computed R-squared values against the reported references: 0.9998 for MRMPro Drug_light vs. Skyline default, 0.9961 for MRMPro Drug_heavy vs. Skyline default (attributed to baseline determination errors in Skyline), and 0.9993 for MRMPro vs. manually inspected/integrated Skyline Drug_heavy results. Emit per-transition R-square comparisons with pass/fail verification flags (e.g. to r2_comparison.csv); a low R-squared against default settings that rises after manual re-integration indicates the discrepancy lies in the reference tool's peak picking, not in MRMPro.

## Related tools

- **MRMPro** (Web-based MRM data analysis platform whose quantification output (Drug_light/Drug_heavy peak areas) supplies one side of the R-squared comparison) — github.com/CSi-Studio/MRMPro-Doc
- **Skyline** (Reference tool providing default and manually integrated peak areas to correlate against MRMPro results)
- **AirdPro** (GUI client for converting vendor MRM files to Aird format so the quantification test dataset can be analyzed downstream) — https://github.com/CSi-Studio/AirdPro/releases/

## Examples

```
python fit_r2.py --dataset skyline_smallmol_113 --transitions Drug_light,Drug_heavy --tools MRMPro,Skyline --refs 0.9998,0.9961,0.9993 --out r2_comparison.csv
```

## Evaluation signals

- Drug_light R-squared between MRMPro and Skyline default integration reproduces 0.9998
- Drug_heavy R-squared between MRMPro and Skyline default integration falls near the reported 0.9961
- After manual inspection/re-integration in Skyline, Drug_heavy R-squared against MRMPro initial results rises to approximately 0.9993
- All three computed R-squared values meet or exceed their reference thresholds (pass flags) in r2_comparison.csv
- Every row of r2_comparison.csv contains an R-squared value for each transition/tool condition and a pass/fail flag (schema check)

## Limitations

- Discrepant R-squared under default settings can stem from baseline determination errors in the reference tool (as seen for Skyline Drug_heavy), so default-vs-default agreement alone is not sufficient to attribute error
- MRMPro relies on classical algorithms rather than more advanced analytical algorithms; peak extraction accuracy improvements are only planned future work
- Manual inspection of MRM data remains necessary in most scenarios, and its cost grows as the sample queue grows

## Evidence

- [abstract] MRMPro vs Skyline correlation accuracy: "Compared with Skyline, MRMPro has higher quantification accuracy and better manual inspection support."
- [discussion] Higher quantification accuracy vs Skyline: "Despite demonstrating higher quantification accuracy compared to Skyline, MRMPro is built upon systematic optimizations of traditional algorithms"
- [readme] MRM data conversion to Aird format: [evidence span withheld — anchor status: external_doc]
