---
name: permethylated-n-glycan-mass-assignment
description: Use when you have a permethylated N-glycan MALDI-TOF spectrum (e.g.,
  PNGase-F released, HILIC SPE enriched, permethylated with DMSO/NaOH/CH3I, mixed
  with Super-DHB matrix) and a glycan name/theoretical-mass (Da) database, and need
  to call which glycans are present by accurate mass matching.
when_to_use_negative:
- The data are LC-MS/MS Orbitrap runs of fluorescently labeled (2-AB or ProCA) N-glycans
  — use the app's MS1/MS2 tabs instead of the MALDI Peak ID tab
- Glycans of interest are not in the database — the app performs mass search only
  and has no de novo glycan identification capability
- The analytes are O-linked glycans — the app's scope is limited to N-linked glycans
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: GlyKAn AZ app
  role: Hosts the MALDI Peak ID tab used to run peak detection and mass search on
    permethylated N-glycan MALDI-TOF spectra
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Runtime environment in which the GlyKAn AZ app is launched
- name: MALDI-TOF
  role: Instrument producing the permethylated N-glycan mass spectra that are input
    to the mass assignment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:2c53d2e0a850aea880eba15db70a1c60308edcbf14791959096e6486ede8356d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - GlyKAn AZ app
  - MALDI-TOF
  - MATLAB
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# permethylated-n-glycan-mass-assignment

## Summary

Assign permethylated N-glycan species in MALDI-TOF mass spectra by searching observed peak masses against a theoretical glycan mass database using the MALDI Peak ID tab of the GlyKAn AZ MATLAB app. The skill replaces manual peak-by-mass glycan calling and was validated by reproducing a full manual analysis (49 confirmed glycans plus one unassigned peak).

## When to use

Use when you have a permethylated N-glycan MALDI-TOF spectrum (e.g., PNGase-F released, HILIC SPE enriched, permethylated with DMSO/NaOH/CH3I, mixed with Super-DHB matrix) and a glycan name/theoretical-mass (Da) database, and need to call which glycans are present by accurate mass matching.

## When NOT to use

- The data are LC-MS/MS Orbitrap runs of fluorescently labeled (2-AB or ProCA) N-glycans — use the app's MS1/MS2 tabs instead of the MALDI Peak ID tab
- Glycans of interest are not in the database — the app performs mass search only and has no de novo glycan identification capability
- The analytes are O-linked glycans — the app's scope is limited to N-linked glycans

## Inputs

- MALDI-TOF mass spectrum of permethylated N-linked glycans (prepared via PNGase-F release, HILIC SPE enrichment, permethylation with DMSO/NaOH/CH3I, LLE with dichloromethane/water, resuspension in 50% methanol, mixing with Super-DHB matrix)
- Glycan mass database (glycan names and theoretical masses in Da) shipped with the GlyKAn AZ application

## Outputs

- Peak-identification output table mapping observed m/z peaks to assigned glycan species at the chosen mass accuracy threshold

## How to apply

Retrieve the example MALDI data set and glycan mass database shipped with the GlyKAn AZ app (github.com/ZaylaSchaeffer/GlyKAn-AZ-application), launch the app in MATLAB, and open the MALDI Peak ID tab. Load the MALDI-TOF spectrum file together with the glycan mass database (glycan names and theoretical masses in Da). Run the built-in MALDI peak-detection and mass-search algorithm to match observed peak masses against theoretical glycan masses at a chosen mass accuracy threshold (the validation swept multiple ppm thresholds), then export the peak-identification output table. Compare the app-selected glycan list against a manual reference: acceptance requires all manually confirmed glycans be recovered; residual extra assignments may arise from peaks whose masses are doubly assigned to two glycans (false positives), not from algorithmic error. Sweep the mass accuracy threshold to confirm assignments are stable across settings — in the reference validation, the app labeled the same unidentified peak with the same glycan at every threshold tested.

## Related tools

- **GlyKAn AZ app** (Hosts the MALDI Peak ID tab used to run peak detection and mass search on permethylated N-glycan MALDI-TOF spectra) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Runtime environment in which the GlyKAn AZ app is launched)
- **MALDI-TOF** (Instrument producing the permethylated N-glycan mass spectra that are input to the mass assignment)

## Evaluation signals

- The output table contains all 49 manually confirmed glycans from the reference permethylated N-glycan MALDI-TOF data set
- The single peak that was unassigned during manual analysis receives a consistent glycan assignment at every mass accuracy threshold tested
- Any false positives can be attributed to peaks whose masses are doubly assigned to two glycans, not to algorithmic misassignment
- Re-running across multiple ppm thresholds yields the same core glycan assignments (stability check)
- For analogous validation runs, the app reproduces glycan species previously identified manually, consistent with the app's reported validation

## Limitations

- False positives can arise from peaks whose masses correspond to more than one glycan (doubly assigned peaks)
- Mass assignments depend entirely on the supplied database; the app cannot identify glycans absent from the database (no de novo capability)
- No software version for the GlyKAn AZ app is stated in the article or repository README

## Evidence

- [abstract] matrix-assisted laser desorption/ionization-time of flight (MALDI-TOF) MS data and was successfully validated by identifying all glycan species that were previously identified manually: "matrix-assisted laser desorption/ionization-time of flight (MALDI-TOF) MS data and was successfully validated by identifying all glycan species that were previously identified manually"
- [intro] The released glycans were enriched and purified by HILIC solid-phase extraction.: "The released glycans were enriched and purified by HILIC solid-phase extraction. The enriched fraction was then dried and derivatized by a permethylation process"
- [results] for the permethylated N-glycan MALDI-TOF data set, at every mass accuracy threshold tested, the app labeled the same unidentified peak with the same glycan, with false positives arising only from doubly assigned peaks: [evidence span withheld — anchor status: not_found]
- [discussion] The inclusion of a de novo approach could also enhance the app's ability to identify glycans not already located in the database.: "The inclusion of a de novo approach could also enhance the app’s ability to identify glycans not already located in the database."
- [readme] Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data.: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan 
identification process from LC-MS/MS data."
