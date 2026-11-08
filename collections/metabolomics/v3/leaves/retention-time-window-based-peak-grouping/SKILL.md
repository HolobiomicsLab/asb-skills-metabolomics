---
name: retention-time-window-based-peak-grouping
description: Use when processing LC–MS/MS mzXML files of released N-glycans where
  multiple peaks share the same m/z and need to be separated and counted as distinct
  isomer peaks within a fixed retention-time window.
when_to_use_negative:
- When the goal is de novo glycan identification for species not present in the MS1
  Database — the app is limited to glycans already in the database and has no de novo
  capability.
- When analyzing O-linked glycans — the app's scope is explicitly limited to N-linked
  glycans.
- When multiple mzXML files need to be processed simultaneously — processing multiple
  mzXML files at once is a planned future improvement, not implemented.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: GlyKAn AZ
  role: MATLAB app whose MS1 tab performs the peak search and RT-window isomer grouping,
    then writes the MS1 output Excel file.
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Implementation language of the app's MS1 search algorithm, including the slope-based
    isomer detection logic.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:f91c4df70484365f63b6cca9bf673fdab4f8cf4169b31b8bcc98b56441f042ff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - MATLAB
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# retention-time-window-based-peak-grouping

## Summary

The MS1 Peak Identification step of GlyKAn AZ that groups isomeric glycan peaks sharing an m/z by scanning a ±1.75 min retention-time window around the representative apex using a slope-based edge detector (3-of-4 positive then 3-of-4 negative), so that co-eluting isomers of the same glycan mass are enumerated as separate chromatographic peaks.

## When to use

Use when processing LC–MS/MS mzXML files of released N-glycans where multiple peaks share the same m/z and need to be separated and counted as distinct isomer peaks within a fixed retention-time window.

## When NOT to use

- When the goal is de novo glycan identification for species not present in the MS1 Database — the app is limited to glycans already in the database and has no de novo capability.
- When analyzing O-linked glycans — the app's scope is explicitly limited to N-linked glycans.
- When multiple mzXML files need to be processed simultaneously — processing multiple mzXML files at once is a planned future improvement, not implemented.

## Inputs

- mzXML file from Orbitrap ESI-MS LC-MS/MS acquisition of released, fluorescently labeled N-glycans
- MS1 Database containing glycan names, theoretical masses in daltons, and optionally expected retention times in minutes
- User-specified mass-accuracy tolerance (e.g. 10 ppm or 50 ppm)

## Outputs

- MS1 output Excel file listing glycan name, charge state, theoretical mass, observed m/z, mass error, retention time, and intensity for each detected peak and isomer peak

## How to apply

Load the mzXML file plus the MS1 Database of glycan names, theoretical masses in Da and optional expected RTs in minutes, then for each glycan and charge state find the highest-intensity peak within the user-specified mass-accuracy tolerance (10 ppm was optimal in the paper's evaluation). Treat that apex as the representative peak and walk the extracted ion chromatogram of intensity vs RT within a fixed ±1.75 min window, calling an isomer peak where 3-of-4 consecutive points show a positive intensity slope (rising edge) followed by 3-of-4 consecutive points showing a negative slope (falling edge). Filter the resulting isomer peaks with a relative-intensity threshold (25% was the optimal setting, giving 20 true positives / 0 false positives at 10 ppm). Output an Excel table of glycan name, charge state, theoretical mass, observed m/z, mass error, RT and intensity for all confirmed peaks and isomers for downstream MS2 Peak Confirmation. The rationale is that isomers have identical mass but distinct RTs, so grouping by RT window plus slope-detection separates co-eluting species that a mass-only search would merge.

## Related tools

- **GlyKAn AZ** (MATLAB app whose MS1 tab performs the peak search and RT-window isomer grouping, then writes the MS1 output Excel file.) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Implementation language of the app's MS1 search algorithm, including the slope-based isomer detection logic.)

## Evaluation signals

- The search recovers all 20 manually confirmed glycans at their manually assigned RTs with minimal false positives — at 25% relative intensity and 10 ppm the paper reports 20 true positives / 0 false positives.
- Isomer peaks are detected only within the fixed ±1.75 min RT window around the representative apex; peaks outside that window must not be grouped.
- A detected isomer peak must exhibit the slope criterion: 3-of-4 consecutive points positive slope followed by 3-of-4 consecutive points negative slope.
- Isomer peaks falling below the 25% relative-intensity threshold are excluded from the output.
- The output Excel file must contain columns for glycan name, charge state, theoretical mass, observed m/z, mass error, RT and intensity for every confirmed peak.

## Limitations

- The isomer peak retention time range is a fixed value (±1.75 min), not customizable, which may not suit all chromatographic separations.
- Isomer detection depends on the slope-based criterion, so very sharp or poorly resolved peaks may be missed or over-counted.
- The mass-accuracy tolerance and relative-intensity threshold must be tuned per dataset; e.g. 50 ppm at 25% yielded 20 TP / 8 FP whereas 10 ppm gave 20 TP / 3 FP, and 100% relative intensity dropped true positives to 6.
- No de novo glycan identification capability — species absent from the MS1 Database cannot be found.

## Evidence

- [methods] For each matched glycan, select the highest-intensity peak as the representative peak at its chromatographic apex.: [evidence span withheld — anchor status: not_found]
- [results] isomer peaks detected at the same m/z within a fixed ±1.75 min retention-time window using a slope-based criterion (3 of 4 positive then 3 of 4 negative): [evidence span withheld — anchor status: not_found]
- [results] 25% relative threshold optimized in the paper, with 10 ppm mass accuracy giving the optimal result: "A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset, containing the fewest false positives while still identifying all 20"
- [discussion] The isomer peak retention time range is a fixed value, not customizable.: "including the isomer peak retention time range, which is currently a fixed value that meets most use cases."
- [discussion] No de novo glycan identification; app limited to glycans in the database.: "The inclusion of a de novo approach could also enhance the app’s ability to identify glycans not already located in the database."
- [intro] mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons), and optionally expected RTs (in minutes) are first loaded into the MS1 tab in the app: "The mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons [Da]), and optionally expected RTs (in minutes) are first loaded into the MS1 tab in the app"
