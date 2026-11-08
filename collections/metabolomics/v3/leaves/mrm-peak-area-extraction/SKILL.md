---
name: mrm-peak-area-extraction
description: Use when analyzing multiple reaction monitoring (MRM) mass spectrometry
  data where peak areas must be extracted per transition across a file batch, or when
  cross-checking automated peak picking against manual integration for quantification
  accuracy.
when_to_use_negative:
- Input is already a processed peak-area or feature table rather than raw/converted
  MRM chromatogram data
- The data are untargeted DDA/DIA acquisitions rather than MRM/SRM targeted transitions
- The goal is developing new peak-extraction algorithms — the paper notes MRMPro relies
  on classical algorithms and improved peak extraction is only planned future work
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MRMPro
  role: Web-based MRM data analysis platform used to extract peak areas, align peaks
    via retention time drift estimation, and perform batch manual inspection
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: Skyline
  role: Reference quantification tool whose default and manually inspected peak integrations
    provide comparison peak areas for R-square validation
- name: AirdPro
  role: GUI client to convert vendor MRM raw files into Aird format for MRMPro analysis
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: AirdSDK
  role: Java/C#/Python SDK to programmatically read Aird-format MRM chromatogram data
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:14586268795b16dbe6e1e209971dba624f583c804222f11f2ae879228c0829bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - AirdSDK
  - MRMPro
  - Skyline
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mrm-peak-area-extraction

## Summary

Extract and cross-validate peak areas from MRM chromatograms (e.g. Drug_light and Drug_heavy transitions) using MRMPro and Skyline, then compare tool outputs via least-squares linear calibration R-square values. This skill operationalizes MRMPro's claimed higher quantification accuracy over Skyline's default peak integration.

## When to use

Use when analyzing multiple reaction monitoring (MRM) mass spectrometry data where peak areas must be extracted per transition across a file batch, or when cross-checking automated peak picking against manual integration for quantification accuracy.

## When NOT to use

- Input is already a processed peak-area or feature table rather than raw/converted MRM chromatogram data
- The data are untargeted DDA/DIA acquisitions rather than MRM/SRM targeted transitions
- The goal is developing new peak-extraction algorithms — the paper notes MRMPro relies on classical algorithms and improved peak extraction is only planned future work

## Inputs

- Vendor MRM raw files or mzML/mzXML converted files (113-file Skyline small-molecule quantification test dataset)
- Aird-formatted MRM data (chromatogram index) converted via AirdPro
- MRM transition definitions (e.g. Drug_light and Drug_heavy transitions)
- Skyline default and manually integrated peak area exports

## Outputs

- Per-transition peak area tables for all files (MRMPro and Skyline)
- Least-squares linear calibration curves (peak area vs. concentration) with R-square values per tool/transition condition
- Per-transition R-square comparison and pass/fail flags (e.g. r2_comparison.csv)
- Final identification results exportable to xlsx format

## How to apply

Load the MRM dataset (e.g. the 113-file Skyline small-molecule quantification test dataset) and convert vendor files to Aird format with AirdPro, which supports MRM acquisition mode. Extract peak areas per MRM transition with MRMPro, and in parallel extract areas with Skyline using default peak integration; where discrepancies appear (e.g. Skyline baseline determination errors on Drug_heavy), manually inspect and re-integrate peaks in Skyline. Fit linear calibration curves (peak area vs. concentration) by least-squares regression per transition/tool condition and compute R-square values to compare the two tools. Use MRMPro's retention time drift estimation based on chromatogram similarity for peak alignment and automatic EIC grouping to support batch manual inspection. Verification is by comparing computed R-squares against reference values (Drug_light 0.9998, Drug_heavy 0.9961, manually inspected Drug_heavy 0.9993).

## Related tools

- **MRMPro** (Web-based MRM data analysis platform used to extract peak areas, align peaks via retention time drift estimation, and perform batch manual inspection) — github.com/CSi-Studio/MRMPro-Doc
- **Skyline** (Reference quantification tool whose default and manually inspected peak integrations provide comparison peak areas for R-square validation)
- **AirdPro** (GUI client to convert vendor MRM raw files into Aird format for MRMPro analysis) — https://github.com/CSi-Studio/AirdPro/releases/
- **AirdSDK** (Java/C#/Python SDK to programmatically read Aird-format MRM chromatogram data) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
pip install AirdSDK
```

## Evaluation signals

- MRMPro vs. Skyline default peak areas for Drug_light correlate at R-square ≈ 0.9998
- MRMPro vs. Skyline default peak areas for Drug_heavy correlate at R-square ≈ 0.9961 (Skyline baseline determination errors expected)
- MRMPro vs. manually integrated Skyline Drug_heavy results correlate at R-square ≈ 0.9993
- r2_comparison.csv contains per-transition R-squares with pass/fail flags matching reference values
- Final identification results export to xlsx format without missing peak groups after EIC grouping and alignment

## Limitations

- MRMPro relies on classical/traditional algorithms; incorporation of more advanced algorithms to improve peak extraction accuracy is only planned future work
- No peak picking algorithm achieves perfect accuracy, so manual inspection remains necessary and its cost grows with sample queue size
- Skyline default integration can produce baseline determination errors (as seen for Drug_heavy), requiring manual re-integration before fair comparison

## Evidence

- [abstract] MRMPro higher accuracy than Skyline: "Compared with Skyline, MRMPro has higher quantification accuracy and better manual inspection support."
- [intro] MRMPro purpose: "Here we propose MRMPro, a free and well-designed web tool focused on MRM data analysis and manual inspection in large volumes."
- [abstract] Retention time drift workflow: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [discussion] Classical algorithms limitation: "Despite demonstrating higher quantification accuracy compared to Skyline, MRMPro is built upon systematic optimizations of traditional algorithms"
- [readme] AirdPro MRM support: [evidence span withheld — anchor status: external_doc]
