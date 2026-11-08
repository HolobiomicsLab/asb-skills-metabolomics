---
name: small-molecule-mrm-quantification-comparison
description: Use when two MRM quantification tools (e.g. MRMPro and Skyline) must
  be compared on the same small-molecule MRM acquisition files, and per-transition
  peak-area correlations (R-square vs concentration) quantify agreement.
when_to_use_negative:
- Input data are proteomics/omics DIA or DDA runs rather than small-molecule MRM transitions
  — MRMPro and this comparison target MRM acquisition data
- You only need peak picking or compression performance evaluation (e.g. Aird format
  compression ratios), not cross-tool quantification accuracy comparison
- Skyline results have already been manually inspected and corrected, so baseline
  determination errors are absent and the Drug_heavy discrepancy signal cannot be
  evaluated
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MRMPro
  role: Web-based MRM data analysis platform producing the reference peak areas for
    Drug_light and Drug_heavy transitions
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: Skyline
  role: Comparison tool providing default and manually integrated peak areas used
    as the quantification-accuracy baseline
- name: AirdPro
  role: GUI client for converting vendor MRM raw files (wiff2, MRM/SRM acquisition)
    into Aird format consumed by MRMPro
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java/C#/Python SDK for programmatically reading Aird chromatogram data (chromatogramIndex
    for MRM mode) during peak-area extraction
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:ca9127656e45ef34ff7a619946bc0133e36085281706827987e79a051ea848ef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - MRMPro
  - Skyline
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# small-molecule-mrm-quantification-comparison

## Summary

Reproduce and verify MRMPro vs Skyline quantification agreement on the Skyline small-molecule MRM test dataset (113 files) by comparing Drug_light and Drug_heavy peak-area calibration R-square values. It benchmarks quantification accuracy between an automated MRM tool and a manual-inspection reference tool.

## When to use

Use when two MRM quantification tools (e.g. MRMPro and Skyline) must be compared on the same small-molecule MRM acquisition files, and per-transition peak-area correlations (R-square vs concentration) quantify agreement.

## When NOT to use

- Input data are proteomics/omics DIA or DDA runs rather than small-molecule MRM transitions — MRMPro and this comparison target MRM acquisition data
- You only need peak picking or compression performance evaluation (e.g. Aird format compression ratios), not cross-tool quantification accuracy comparison
- Skyline results have already been manually inspected and corrected, so baseline determination errors are absent and the Drug_heavy discrepancy signal cannot be evaluated

## Inputs

- 113-file Skyline small-molecule quantification test dataset (vendor MRM raw files, convertible to Aird via AirdPro)
- MRMPro analysis output with Drug_light and Drug_heavy peak areas
- Skyline peak-integration results (default settings and manually integrated Drug_heavy results)
- Concentration metadata for calibration-curve fitting

## Outputs

- Per-transition R-square comparison table (r2_comparison.csv) with reference values 0.9998 (Drug_light), 0.9961 (Drug_heavy default), 0.9993 (Drug_heavy manual) and pass/fail verification flags
- Least-squares calibration curves (peak area vs. concentration) per transition/tool condition

## How to apply

Load the 113-file Skyline small-molecule quantification test dataset containing the Drug_light and Drug_heavy MRM transitions, converting vendor files with AirdPro if needed since MRMPro reads Aird data. Extract peak areas from the MRMPro analysis output for both transitions across all 113 files, then extract corresponding peak areas from Skyline using its default peak integration settings. Fit linear calibration curves (peak area vs. concentration) by least-squares regression for each transition/tool condition and compute R-square values. Compare against reference targets: 0.9998 for Drug_light (MRMPro vs Skyline default), 0.9961 for Drug_heavy (MRMPro vs Skyline default, lowered by Skyline baseline determination errors), and 0.9993 for MRMPro vs manually inspected Skyline Drug_heavy results. Write per-transition R-square comparisons with pass/fail flags to r2_comparison.csv; a large deviation on Drug_heavy typically indicates Skyline baseline determination errors rather than MRMPro failure.

## Related tools

- **MRMPro** (Web-based MRM data analysis platform producing the reference peak areas for Drug_light and Drug_heavy transitions) — github.com/CSi-Studio/MRMPro-Doc
- **Skyline** (Comparison tool providing default and manually integrated peak areas used as the quantification-accuracy baseline)
- **AirdPro** (GUI client for converting vendor MRM raw files (wiff2, MRM/SRM acquisition) into Aird format consumed by MRMPro) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java/C#/Python SDK for programmatically reading Aird chromatogram data (chromatogramIndex for MRM mode) during peak-area extraction) — https://github.com/CSi-Studio/Aird-SDK

## Evaluation signals

- Drug_light R-square between MRMPro and Skyline default integration equals 0.9998 (tolerance per pass/fail flags)
- Drug_heavy R-square vs Skyline default equals 0.9961; residual deviation is attributable to Skyline baseline determination errors, not MRMPro
- Drug_heavy R-square between MRMPro initial results and Skyline manually integrated results reaches 0.9993
- r2_comparison.csv contains exactly the three per-transition comparisons plus pass/fail flags, and all reference targets fall within the defined tolerance
- Peak areas are extracted for all 113 files for both transitions with no missing-file gaps

## Limitations

- Skyline's default baseline determination can introduce errors on Drug_heavy, lowering apparent correlation; manual re-integration in Skyline is needed to confirm MRMPro accuracy
- MRMPro is built upon systematic optimizations of classical/traditional algorithms; more advanced peak-extraction algorithms were only planned future work, so peak extraction accuracy itself is not improved
- Manual inspection cost grows as the sample queue grows, and MRMPro is currently dedicated to improving manual-inspection efficiency rather than broader analytical capability

## Evidence

- [results] MRMPro vs Skyline default Drug_light R-square: [evidence span withheld — anchor status: not_found]
- [results] Manually integrated Drug_heavy confirms MRMPro accuracy: [evidence span withheld — anchor status: not_found]
- [abstract] Overall accuracy claim from abstract: "Compared with Skyline, MRMPro has higher quantification accuracy and better manual inspection support."
- [discussion] Classical algorithm limitation from discussion: "MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework"
- [readme] AirdPro MRM support from README: [evidence span withheld — anchor status: external_doc]
