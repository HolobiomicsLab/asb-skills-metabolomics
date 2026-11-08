---
name: chromatographic-calibration-curve-fitting
description: Use when comparing quantification accuracy of two MRM analysis tools
  (e.g., MRMPro vs. Skyline) on a shared multi-file dataset with defined transitions,
  where peak areas can be regressed against known concentrations.
when_to_use_negative:
- The dataset lacks known concentrations, so no calibration curve (peak area vs. concentration)
  can be regressed
- Only one tool's peak areas are available — the skill is a comparative benchmark
  requiring paired areas from two tools or two integration conditions
- The acquisition is not targeted MRM (e.g., DDA/DIA discovery data), where transition-level
  peak-area calibration curves of this form do not apply
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMPro
  role: Web-based MRM data analysis platform whose extracted peak areas are regressed
    into calibration curves; provides the reference quantification results
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: Skyline
  role: Provides comparison peak areas under default peak integration settings and
    manually integrated areas for transitions where default baseline determination
    fails
- name: AirdPro
  role: GUI client for converting vendor MRM raw files to the Aird format consumed
    by the MRMPro analysis stack
  repo: https://github.com/CSi-Studio/AirdPro/releases/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:d561902734e440506255a4cc7ccdc19a358e2922151dff16405d6eda5004cc03
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MRMPro
  - Skyline
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# chromatographic-calibration-curve-fitting

## Summary

Benchmark quantification accuracy of MRM peak-integration tools by fitting least-squares calibration curves (peak area vs. concentration) for each transition and comparing R-square values, as done for MRMPro vs. Skyline on the Drug_light/Drug_heavy transitions.

## When to use

Use when comparing quantification accuracy of two MRM analysis tools (e.g., MRMPro vs. Skyline) on a shared multi-file dataset with defined transitions, where peak areas can be regressed against known concentrations.

## When NOT to use

- The dataset lacks known concentrations, so no calibration curve (peak area vs. concentration) can be regressed
- Only one tool's peak areas are available — the skill is a comparative benchmark requiring paired areas from two tools or two integration conditions
- The acquisition is not targeted MRM (e.g., DDA/DIA discovery data), where transition-level peak-area calibration curves of this form do not apply

## Inputs

- 113-file Skyline small-molecule quantification test dataset with Drug_light and Drug_heavy MRM transitions
- MRMPro analysis output with per-file peak areas for each transition
- Skyline peak areas under default integration settings, plus manually integrated peak areas for transitions with baseline determination errors

## Outputs

- Linear calibration curves (peak area vs. concentration) per tool/transition condition with least-squares fits
- R-square values per tool/transition comparison (e.g., 0.9998, 0.9961, 0.9993)
- Per-transition R-square comparison table with pass/fail verification flags (e.g., r2_comparison.csv)

## How to apply

Load the multi-file MRM dataset (here 113 files from the Skyline small-molecule quantification test dataset) containing the transitions of interest, e.g., Drug_light and Drug_heavy. Extract peak areas for each transition from the MRMPro analysis output and from Skyline under its default peak integration settings. For each tool/transition condition, fit linear calibration curves (peak area vs. concentration) by least-squares regression and compute R-square values. Compare computed R-squares against reference values — 0.9998 (MRMPro vs. Skyline default, Drug_light), 0.9961 (MRMPro vs. Skyline default, Drug_heavy) — and, where a tool shows baseline determination errors (Skyline on Drug_heavy), re-extract manually integrated peak areas and recompute (reference R-square 0.9993 for MRMPro vs. manually inspected Skyline). Higher R-square on a given tool's areas supports that tool's quantification accuracy; MRMPro demonstrated higher quantification accuracy compared to Skyline.

## Related tools

- **MRMPro** (Web-based MRM data analysis platform whose extracted peak areas are regressed into calibration curves; provides the reference quantification results) — github.com/CSi-Studio/MRMPro-Doc
- **Skyline** (Provides comparison peak areas under default peak integration settings and manually integrated areas for transitions where default baseline determination fails)
- **AirdPro** (GUI client for converting vendor MRM raw files to the Aird format consumed by the MRMPro analysis stack) — https://github.com/CSi-Studio/AirdPro/releases/

## Examples

```
python fit_r2.py --transitions Drug_light,Drug_heavy --mrmpro-areas mrmpro_areas.csv --skyline-areas skyline_default.csv --skyline-manual skyline_manual.csv --n-files 113 --out r2_comparison.csv
```

## Evaluation signals

- Drug_light R-square for MRMPro vs. Skyline default integration reproduces 0.9998
- Drug_heavy R-square for MRMPro vs. Skyline default integration is near but below the Drug_light value (reported 0.9961), attributable to Skyline baseline determination errors
- After manual re-integration in Skyline, the Drug_heavy R-square rises to approximately 0.9993
- r2_comparison.csv contains one row per tool/transition condition with the computed R-square and a pass/fail flag against the reported reference values
- Each calibration fit is a least-squares linear regression over all 113 files with peak area as response and concentration as predictor

## Limitations

- Skyline's default peak integration can produce baseline determination errors (as seen for Drug_heavy), depressing the apparent correlation; manual inspection/integration is necessary to correct peak picking errors
- No peak picking algorithm achieves perfect accuracy, so manual inspection cost grows as the sample queue grows
- MRMPro relies on classical algorithms and peak extraction accuracy improvements are deferred to future work
- Supplementary analysis parameters and standard curve goodness-of-fit indicators (Additional file 1, Appendix S1) are not included inline, complicating exact reproduction

## Evidence

- [full_text] Benchmark finding for Drug_light: [evidence span withheld — anchor status: not_found]
- [full_text] Benchmark finding for Drug_heavy and manual re-integration: [evidence span withheld — anchor status: not_found]
- [abstract] Overall accuracy claim: "Compared with Skyline, MRMPro has higher quantification accuracy and better manual inspection support."
- [abstract] Manual inspection rationale: "no peak picking algorithm can achieve perfect accuracy, manual inspection is necessary to correct the errors"
- [abstract] Export of results for subsequent analysis: "The final identification results can be exported to a common.xlsx format for subsequent analysis."
