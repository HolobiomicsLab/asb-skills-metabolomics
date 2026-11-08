---
name: maldi-tof-spectral-peak-detection
description: Use when you have a permethylated released N-glycan MALDI-TOF spectrum
  (e.g., Super-DHB matrix) and need to assign glycan identities to detected peaks
  by accurate-mass search against a glycan mass database, replacing manual peak-by-peak
  analysis.
when_to_use_negative:
- The data are LC-MS/MS Orbitrap (OrbiTrap) acquisitions of fluorescent-labeled N-glycans
  — use the app's MS1 and MS2 Peak Confirmation tabs instead of the MALDI Peak ID
  tab.
- The glycans of interest are O-linked, or absent from the database — the app's scope
  is N-linked glycans and has no de novo identification capability.
- The goal is MS/MS fragment annotation rather than intact-mass peak identification.
edam_operation: http://edamontology.org/operation_3232
tools:
- name: GlyKAn AZ app
  role: MATLAB app whose MALDI Peak ID tab performs built-in peak detection and mass
    search of the MALDI-TOF spectrum against the glycan mass database
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Runtime environment in which the GlyKAn AZ app is launched and executed
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MALDI-TOF
  role: Instrument generating the permethylated N-glycan spectra that serve as the
    input to the peak-detection workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:d93a5795976ac47a8f814be3d7848a4419ed27b891a3a7ae3dc582c0002a0645
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - GlyKAn AZ app
  - MALDI-TOF
  - MATLAB
  edam_operation: http://edamontology.org/operation_3232
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# maldi-tof-spectral-peak-detection

## Summary

Automated detection and identification of glycan peaks in permethylated N-linked glycan MALDI-TOF spectra using the GlyKAn AZ MATLAB app's MALDI Peak ID tab, which matches observed peak masses against a glycan mass database of theoretical masses (Da) and reproduces manual analyst assignments.

## When to use

Use when you have a permethylated released N-glycan MALDI-TOF spectrum (e.g., Super-DHB matrix) and need to assign glycan identities to detected peaks by accurate-mass search against a glycan mass database, replacing manual peak-by-peak analysis.

## When NOT to use

- The data are LC-MS/MS Orbitrap (OrbiTrap) acquisitions of fluorescent-labeled N-glycans — use the app's MS1 and MS2 Peak Confirmation tabs instead of the MALDI Peak ID tab.
- The glycans of interest are O-linked, or absent from the database — the app's scope is N-linked glycans and has no de novo identification capability.
- The goal is MS/MS fragment annotation rather than intact-mass peak identification.

## Inputs

- Permethylated N-glycan MALDI-TOF spectrum file (glycans released with PNGase F, HILIC SPE enriched, permethylated with DMSO/NaOH/CH3I, mixed with Super-DHB matrix)
- Glycan mass database (glycan names and theoretical masses in Da) shipped with the GlyKAn AZ app
- Optional manual analysis reference list of confirmed glycan assignments (e.g., the 49-glycan reference set)

## Outputs

- Peak-identification output table mapping detected MALDI-TOF peak masses to assigned glycan names
- List of false positives (doubly assigned peaks) at each mass accuracy threshold tested

## How to apply

Retrieve the example MALDI data set and glycan mass database shipped with the GlyKAn AZ application (github.com/ZaylaSchaeffer/GlyKAn-AZ-application), launch the app in MATLAB, and open the MALDI Peak ID tab. Load the MALDI-TOF spectrum file together with the glycan mass database containing glycan names and theoretical masses in Da, then run the app's built-in MALDI peak-detection and mass-search algorithm to match observed peak masses against theoretical glycan masses. Sweep mass accuracy thresholds (e.g., 10 ppm vs 50 ppm) and relative intensity thresholds to find the setting that recovers all manually confirmed glycans with the fewest false positives; the paper's LC-MS/MS benchmark found 25% relative intensity with 10 ppm optimal for the analogous MS1 tab. Export the peak-identification output table and compare it against the manual reference list; acceptance for the example set was recovery of all 49 manually confirmed glycans plus one extra peak that was unassigned during manual analysis, with false positives arising only from doubly assigned peaks.

## Related tools

- **GlyKAn AZ app** (MATLAB app whose MALDI Peak ID tab performs built-in peak detection and mass search of the MALDI-TOF spectrum against the glycan mass database) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Runtime environment in which the GlyKAn AZ app is launched and executed) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MALDI-TOF** (Instrument generating the permethylated N-glycan spectra that serve as the input to the peak-detection workflow)

## Evaluation signals

- The output table contains all 49 manually confirmed glycans from the example permethylated N-glycan data set.
- Exactly one additional peak (the one unassigned during manual analysis) is labeled by the app, and it receives the same glycan assignment at every mass accuracy threshold tested.
- Any false positives arise only from doubly assigned peaks, not from random mismatches.
- Assignment lists are stable across threshold sweeps (10 ppm vs 50 ppm mass accuracy), indicating the mass-search tolerance is not driving the identifications.
- Result matches the app's reported validation of identifying all glycan species that were previously identified manually.

## Limitations

- No de novo glycan identification: the app can only identify glycans already present in the supplied mass database.
- Scope is limited to N-linked glycans; O-linked glycans are not supported.
- False positives can arise from doubly assigned peaks in the MALDI data set.

## Evidence

- [abstract] matrix-assisted laser desorption/ionization-time of flight (MALDI-TOF) MS data and was successfully validated by identifying all glycan species that were previously identified manually: "matrix-assisted laser desorption/ionization-time of flight (MALDI-TOF) MS data and was successfully validated by identifying all glycan species that were previously identified manually"
- [results] at every mass accuracy threshold tested, the app labeled the same unidentified peak with the same glycan, with false positives arising only from doubly assigned peaks: [evidence span withheld — anchor status: not_found]
- [other] Load the example MALDI-TOF spectrum file and the accompanying glycan mass database (glycan names and theoretical masses in Da) into the tab.: [evidence span withheld — anchor status: not_found]
- [readme] This app was developed by Zayla Schaeffer and Ashna Dhingra at AstraZeneca, and is available under MIT license (see License.txt).: "This app was developed by Zayla Schaeffer and Ashna Dhingra at AstraZeneca, and is available under MIT license (see License.txt)."
