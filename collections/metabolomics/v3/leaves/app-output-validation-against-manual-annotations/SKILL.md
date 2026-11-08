---
name: app-output-validation-against-manual-annotations
description: 'Use when an automated glycan-identification or peak-assignment tool
  must be validated before adoption: a manually curated glycan list exists for the
  same spectrum, and you need to confirm the tool''s output reproduces it (all true
  positives recovered, extra calls explainable).'
when_to_use_negative:
- The spectrum or glycan class is out of the tool's scope (GlyKAn AZ is limited to
  N-linked glycans; O-linked glycans are unsupported).
- No manually curated reference annotation exists for the same data set — use an orthogonal
  validation instead.
- The goal is de novo glycan identification; the app only matches glycans already
  present in its database.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0080
tools:
- name: GlyKAn AZ app
  role: Tool under validation; its MALDI Peak ID tab performs peak detection and mass-search
    against the glycan mass database
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Runtime environment in which the GlyKAn AZ app is launched and the MALDI Peak
    ID tab is executed
- name: MALDI-TOF
  role: Instrument that acquired the permethylated N-glycan spectrum used as the validation
    data set
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:7c85f869e12743bb30d296dd9d474de39b482410c6622af333f253ab4a4eb211
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - GlyKAn AZ app
  - MALDI-TOF
  - MATLAB
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# App output validation against manual annotations

## Summary

A benchmarking procedure that verifies a glycan-identification app's automated peak calls against an expert manual reference, by confirming the app recovers all manually confirmed glycans (plus any known-unassigned peaks) without false negatives. Here it validates the GlyKAn AZ MATLAB app on a permethylated N-glycan MALDI-TOF data set.

## When to use

Use when an automated glycan-identification or peak-assignment tool must be validated before adoption: a manually curated glycan list exists for the same spectrum, and you need to confirm the tool's output reproduces it (all true positives recovered, extra calls explainable).

## When NOT to use

- The spectrum or glycan class is out of the tool's scope (GlyKAn AZ is limited to N-linked glycans; O-linked glycans are unsupported).
- No manually curated reference annotation exists for the same data set — use an orthogonal validation instead.
- The goal is de novo glycan identification; the app only matches glycans already present in its database.

## Inputs

- Example MALDI-TOF spectrum file of permethylated N-glycans shipped with the GlyKAn AZ app
- Glycan mass database (glycan names and theoretical masses in Da)
- Manually curated reference list of confirmed glycan identifications (49 glycans plus one known unassigned peak)

## Outputs

- Exported app peak-identification output table
- Threshold-sweep comparison table (true positives and false positives vs. mass accuracy threshold)
- Validation pass/fail assessment against the manual reference

## How to apply

Retrieve the example data shipped with the tool (here the permethylated N-glycan MALDI-TOF spectrum and glycan mass database of glycan names and theoretical masses in Da from github.com/ZaylaSchaeffer/GlyKAn-AZ-application) and the corresponding manual reference list. Load the spectrum and database into the relevant tool tab (for GlyKAn AZ, the MALDI Peak ID tab) and run the built-in peak-detection and mass-search matching. Export the tool's peak-identification output table and sweep the mass accuracy thresholds (the paper tested multiple ppm settings); at each threshold, compare the tool-selected glycan list against the manual list. Acceptance criteria: every manually confirmed glycan appears in the tool output, and any additional peak matches a peak that was known to be unassigned in manual analysis (e.g. a doubly assigned peak) rather than an unexplained false positive. In the reference validation, the app labeled the same unidentified peak with the same glycan at every mass accuracy threshold tested, with false positives arising only from doubly assigned peaks, reproducing the 49 manually confirmed glycans plus the single unassigned peak.

## Related tools

- **GlyKAn AZ app** (Tool under validation; its MALDI Peak ID tab performs peak detection and mass-search against the glycan mass database) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Runtime environment in which the GlyKAn AZ app is launched and the MALDI Peak ID tab is executed)
- **MALDI-TOF** (Instrument that acquired the permethylated N-glycan spectrum used as the validation data set)

## Evaluation signals

- Output table contains all 49 manually confirmed glycans at every mass accuracy threshold tested.
- Exactly one additional peak beyond the 49, matching the peak left unassigned during manual analysis, with the same glycan assignment across thresholds.
- Any false positives in the output are attributable to doubly assigned peaks, not spurious matches.
- No manually confirmed glycan is missing from the app output at any threshold (zero false negatives).
- App validation is consistent with the paper's claim of identifying all glycan species previously identified manually.

## Limitations

- The app requires the glycan to already exist in its database; no de novo identification capability.
- App cannot process multiple mzXML files simultaneously (planned future improvement).
- Validation conclusions are specific to the shipped example data set and its manual reference; performance on new data sets must be re-established.
- App scope is limited to N-linked glycans.

## Evidence

- [results] MALDI-TOF validation reproducing 49 manual glycans plus one unassigned peak: [evidence span withheld — anchor status: not_found]
- [abstract] Overall validation claim: "was successfully validated by identifying all glycan species that were previously identified manually"
- [intro] Database contents loaded into the app: "the MS1 Database containing glycan names, theoretical masses (in daltons [Da]), and optionally expected RTs (in minutes) are first loaded into the MS1 tab in the app"
- [intro] Output review step: "The MS1 output Excel file that the app returns can then be reviewed to determine which glycans are likely to be present in the sample."
- [intro] MALDI sample preparation defines the validation data set: "The released glycans were enriched and purified by HILIC solid-phase extraction. The enriched fraction was then dried and derivatized by a permethylation process"
- [readme] Source availability: "This app was developed by Zayla Schaeffer and Ashna Dhingra at AstraZeneca, and is available under MIT license"
