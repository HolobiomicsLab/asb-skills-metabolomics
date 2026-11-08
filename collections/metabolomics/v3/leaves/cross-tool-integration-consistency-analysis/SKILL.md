---
name: cross-tool-integration-consistency-analysis
description: Use when the same MRM acquisition dataset (e.g., 113 Skyline small-molecule
  test files) has been processed by two tools and you must verify that quantification
  results (peak areas per transition such as Drug_light/Drug_heavy) agree within a
  defined correlation threshold.
when_to_use_negative:
- Only one tool has processed the data and no cross-tool reference or replicate result
  exists to correlate against
- The analysis goal is peak picking quality itself rather than consistency of quantification
  outputs between tools
- Transitions lack matched identities or a shared file set between the two tool outputs,
  making area-level correlation meaningless
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMPro
  role: Primary MRM quantification tool; provides peak areas for both transitions
    and serves as one side of the cross-tool correlation
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: Skyline
  role: Reference quantification tool; provides default-integration and manually integrated
    peak areas for comparison against MRMPro
- name: AirdPro
  role: Conversion client that turns vendor raw files into Aird format consumable
    by the MRMPro ecosystem, including MRM acquisition data
  repo: https://github.com/CSi-Studio/AirdPro/releases/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:2962484ef9a2f9058d8931b7facc0173c83761e58aaf5ca5098f81a94ae9171d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MRMPro
  - Skyline
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-tool-integration-consistency-analysis

## Summary

Benchmark quantification consistency between two MRM data analysis tools (e.g., MRMPro and Skyline) by extracting peak areas for matched transitions across a shared file set and comparing fitted calibration curves via R-square values, flagging discrepancies to baseline-determination or peak-integration errors.

## When to use

Use when the same MRM acquisition dataset (e.g., 113 Skyline small-molecule test files) has been processed by two tools and you must verify that quantification results (peak areas per transition such as Drug_light/Drug_heavy) agree within a defined correlation threshold.

## When NOT to use

- Only one tool has processed the data and no cross-tool reference or replicate result exists to correlate against
- The analysis goal is peak picking quality itself rather than consistency of quantification outputs between tools
- Transitions lack matched identities or a shared file set between the two tool outputs, making area-level correlation meaningless

## Inputs

- 113-file Skyline small-molecule quantification test dataset with Drug_light and Drug_heavy MRM transitions
- MRMPro analysis output with per-file peak areas per transition
- Skyline output with default peak integration peak areas
- Skyline output with manually inspected/integrated peak areas (for error attribution)

## Outputs

- Per-transition R-square values comparing tool pairs (e.g., 0.9998, 0.9961, 0.9993)
- r2_comparison.csv with per-transition R-square comparisons and pass/fail verification flags
- Fitted linear calibration curves (peak area vs. concentration) per tool/integration condition

## How to apply

Load the shared dataset — here, 113 files containing Drug_light and Drug_heavy MRM transitions — and extract peak areas from both tools' outputs: MRMPro's analysis output and Skyline's default peak integration. Fit linear calibration curves (peak area vs. concentration) by least-squares regression for each transition under each tool/integration condition and compute R-square values. Compare against reported references: 0.9998 (Drug_light, MRMPro vs Skyline default), 0.9961 (Drug_heavy, MRMPro vs Skyline default, the deficit attributed to baseline determination errors in Skyline), and 0.9993 (MRMPro vs manually integrated Skyline Drug_heavy). When a transition's R-square is depressed, repeat the comparison against manually inspected/integrated results in the reference tool to confirm the discrepancy is a tool artifact rather than a data problem. Write per-transition R-square comparisons with pass/fail verification flags to a CSV such as r2_comparison.csv.

## Related tools

- **MRMPro** (Primary MRM quantification tool; provides peak areas for both transitions and serves as one side of the cross-tool correlation) — github.com/CSi-Studio/MRMPro-Doc
- **Skyline** (Reference quantification tool; provides default-integration and manually integrated peak areas for comparison against MRMPro)
- **AirdPro** (Conversion client that turns vendor raw files into Aird format consumable by the MRMPro ecosystem, including MRM acquisition data) — https://github.com/CSi-Studio/AirdPro/releases/

## Examples

```
import scipy.stats as ss; r2 = ss.linregress(mrmpro_areas, skyline_areas).rvalue**2
```

## Evaluation signals

- Drug_light R-square between MRMPro and Skyline default integration reproduces the reference value 0.9998
- Drug_heavy R-square against Skyline default integration is near but below the Drug_light value (reference 0.9961), attributable to Skyline baseline determination errors
- Drug_heavy R-square between MRMPro initial results and manually integrated Skyline results recovers to approximately 0.9993, confirming error attribution
- All per-transition pass/fail flags in r2_comparison.csv match the expected reference thresholds
- Fitted calibration curves are linear least-squares fits of peak area vs. concentration for every transition/tool pair with no unmatched files

## Limitations

- Skyline's default peak integration can introduce baseline determination errors (observed as the Drug_heavy R-square drop to 0.9961), requiring manual re-integration to isolate the cause
- No peak picking algorithm achieves perfect accuracy, so cross-tool discrepancies may reflect genuine peak picking errors needing manual inspection rather than tool failure
- MRMPro relies on classical algorithms; further improvements to peak extraction accuracy are deferred to future work, so residual disagreements may persist

## Evidence

- [abstract] Compared with Skyline, MRMPro has higher quantification accuracy and better manual inspection support.: "Compared with Skyline, MRMPro has higher quantification accuracy and better manual inspection support."
- [abstract] no peak picking algorithm can achieve perfect accuracy, manual inspection is necessary to correct the errors: "no peak picking algorithm can achieve perfect accuracy, manual inspection is necessary to correct the errors"
- [discussion] Despite demonstrating higher quantification accuracy compared to Skyline, MRMPro is built upon systematic optimizations of traditional algorithms: "Despite demonstrating higher quantification accuracy compared to Skyline, MRMPro is built upon systematic optimizations of traditional algorithms"
- [discussion] we plan to incorporate more advanced analytical algorithms to improve the accuracy of peak extraction: "we plan to incorporate more advanced analytical algorithms to improve the accuracy of peak extraction and further enhance the efficiency of manual inspection"
- [readme] You should use the AirdPro client to transfer the vendor files into Aird format: [evidence span withheld — anchor status: not_found]
