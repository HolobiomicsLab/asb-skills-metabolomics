---
name: xic-isomer-peak-detection-slope-criteria
description: Decision support for distinguishing co-eluting glycan isomers sharing
  an identical m/z in LC-MS extracted ion chromatograms, where mass accuracy alone
  cannot separate them and slope-based edge detection plus a relative intensity threshold
  is needed.
when_to_use_negative:
- Isomer peaks are separated by more than the fixed ±1.75 min retention-time window,
  so a different deconvolution or chromatographic-resolution approach is needed.
- The user needs de novo glycan identification rather than detection of isomers for
  glycans already in the MS1 Database.
- The sample is O-linked glycans; the app's scope is limited to N-linked glycans.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GlyKAn AZ
  role: MATLAB-based app whose MS1 Peak Identification routine implements the slope-based
    isomer peak detection and relative-intensity filtering over mzXML and MS1 Database
    inputs.
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Programming environment in which the slope-based isomer peak-calling algorithm
    is implemented and executed.
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:9ab0bf5e68283674fd838ad8c3643c47ac3d5a6780d5879fa037bbcff1939b2b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - MATLAB
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# XIC isomer peak detection via slope criteria

## Summary

A chromatographic peak-calling method that detects isomer peaks co-eluting at the same m/z within a fixed ±1.75 min retention-time window by scanning the extracted ion chromatogram for a rising edge (3-of-4 consecutive positive intensity slopes) followed by a falling edge (3-of-4 consecutive negative slopes), then applying a relative-intensity threshold filter.

## When to use

Decision support for distinguishing co-eluting glycan isomers sharing an identical m/z in LC-MS extracted ion chromatograms, where mass accuracy alone cannot separate them and slope-based edge detection plus a relative intensity threshold is needed.

## When NOT to use

- Isomer peaks are separated by more than the fixed ±1.75 min retention-time window, so a different deconvolution or chromatographic-resolution approach is needed.
- The user needs de novo glycan identification rather than detection of isomers for glycans already in the MS1 Database.
- The sample is O-linked glycans; the app's scope is limited to N-linked glycans.

## Inputs

- Extracted ion chromatogram (intensity vs. RT) at the matched m/z from the mzXML file
- Representative peak apex RT from the MS1 search result
- User-specified mass-accuracy tolerance in ppm
- Fixed isomer RT window (±1.75 min)
- Relative intensity threshold (e.g. 25%)

## Outputs

- List of isomer peaks at the same m/z with RT and intensity
- Table of glycan name, charge state, theoretical mass, observed m/z, mass error, RT, and intensity
- MS1 output Excel file for downstream MS2 peak confirmation

## How to apply

After the MS1 search has located the highest-intensity peak matching a glycan and charge state within the user-specified mass-accuracy tolerance (10 ppm was optimal for the evaluated mAb dataset), extract the intensity-versus-retention-time chromatogram at that m/z. Walk the XIC and call a peak where 3-of-4 consecutive points show a positive intensity slope (rising edge) followed by 3-of-4 consecutive points showing a negative slope (falling edge). Restrict the search to peaks at the same m/z within the fixed ±1.75 min RT window around the representative apex, then apply a relative intensity threshold (25% was optimal) to retain only well-supported isomer peaks. Assemble the retained peaks into a table of glycan name, charge state, theoretical mass, observed m/z, mass error, RT, and intensity for the MS1 output Excel file.

## Related tools

- **GlyKAn AZ** (MATLAB-based app whose MS1 Peak Identification routine implements the slope-based isomer peak detection and relative-intensity filtering over mzXML and MS1 Database inputs.) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Programming environment in which the slope-based isomer peak-calling algorithm is implemented and executed.) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application

## Examples

```
% In MATLAB GlyKAn AZ app: load mzXML and MS1 Database; MS1 search runs the slope-based isomer detection at ±1.75 min, 10 ppm, 25% relative intensity and writes the MS1 output Excel file.
```

## Evaluation signals

- Confirmed isomer peaks must fall within the fixed ±1.75 min RT window of the representative apex.
- At the optimal settings for the evaluated mAb dataset (25% relative intensity, 10 ppm mass accuracy), the search should recover all 20 manually confirmed glycans at their manually assigned RTs with 0 false positives.
- Across threshold tests at 25% relative intensity: 10 ppm mass accuracy yields 20 TP / 3 FP (or 0 FP in the paper's optimum), while 50 ppm yields 20 TP / 8 FP — mass accuracy is the sensitive parameter.
- Each retained isomer peak must satisfy the 3-of-4 positive-then-negative slope criterion on the XIC.
- Output table must contain glycan name, charge state, theoretical mass, observed m/z, mass error, RT, and intensity columns for the MS1 output Excel file.

## Limitations

- The isomer peak retention time range is a fixed value (±1.75 min), not customizable, which may miss isomers outside that window.
- The relative intensity threshold trades off true positives and false positives: at 100% relative intensity only 6 TP / 6 FP were recovered, showing the threshold's effect on sensitivity.
- Only glycans already present in the MS1 Database can be detected; no de novo glycan identification is supported.
- App cannot process multiple mzXML files simultaneously (a planned future improvement).
- Scope is limited to N-linked glycans from Orbitrap and MALDI-TOF MS data; O-linked glycans are not supported.

## Evidence

- [results] Slope-based isomer peak calling criterion and fixed RT window: "three isomer peaks were identified at RTs of 13.39, 13.77, and 14.07 min"
- [methods] Slope-based isomer peak calling criterion: [evidence span withheld — anchor status: not_found]
- [results] Relative intensity threshold parameter and optimal value: "A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset"
- [discussion] Fixed non-customizable isomer RT window as a limitation: "including the isomer peak retention time range, which is currently a fixed value that meets most use cases."
- [discussion] No de novo identification limitation: "The inclusion of a de novo approach could also enhance the app’s ability to identify glycans not already located in the database."
