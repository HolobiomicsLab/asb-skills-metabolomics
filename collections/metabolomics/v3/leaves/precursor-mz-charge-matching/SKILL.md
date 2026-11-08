---
name: precursor-mz-charge-matching
description: 'Use when confirming N-linked glycan identifications from LC-MS/MS mzXML
  data: an MS1 output Excel file lists candidate glycan-RT pairs, and the MS2 fragment
  scan for the matching precursor m/z and charge must be located for fragment annotation.'
when_to_use_negative:
- The glycans of interest are O-linked rather than N-linked, since the app's scope
  is limited to N-linked glycans
- De novo glycan identification is needed, because the app can only identify glycans
  already present in its database
- Multiple mzXML files must be processed simultaneously, which the app does not currently
  support
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GlyKAn AZ app
  role: MATLAB app whose MS2 Peak Confirmation tab performs the precursor m/z and
    charge matching and renders annotated MS2 fragmentation spectra
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Runtime environment for the GlyKAn AZ application that executes the MS2 Peak
    Confirmation processing routine
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:b8edaff59b51d4f547ebe10f81c2ec289dc2a7a4d82a127187c33b1dac67d88d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - GlyKAn AZ app
  - MATLAB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 29
---

# precursor-mz-charge-matching

## Summary

In GlyKAn AZ's MS2 Peak Confirmation tab, for each user-selected glycan-RT pair the app extracts MS2 scans starting at the MS1 RT minus 0.1 min and locates the fragment scan whose precursor m/z and charge match the target glycan, then annotates matched fragment peaks. This anchors MS2 confirmation to the correct precursor scan and enables fragment-level glycan verification.

## When to use

Use when confirming N-linked glycan identifications from LC-MS/MS mzXML data: an MS1 output Excel file lists candidate glycan-RT pairs, and the MS2 fragment scan for the matching precursor m/z and charge must be located for fragment annotation.

## When NOT to use

- The glycans of interest are O-linked rather than N-linked, since the app's scope is limited to N-linked glycans
- De novo glycan identification is needed, because the app can only identify glycans already present in its database
- Multiple mzXML files must be processed simultaneously, which the app does not currently support

## Inputs

- MS1 output Excel file (glycan-RT pairs from the MS1 tab)
- MS2 Fragment Database (fragment names, theoretical masses)
- MS2 fragment image repository
- mzXML file (converted from raw Orbitrap MS data with ProteoWizard)

## Outputs

- ms2_annotated_spectra.xlsx (annotated and labeled MS2 fragmentation spectra for the selected glycan-RT pairs)

## How to apply

Load the MS1 output Excel file, the MS2 Fragment Database, and the MS2 fragment image repository into the MS2 Peak Confirmation tab of the GlyKAn AZ MATLAB app. For each user-selected glycan-RT pair, extract MS2 scans starting at the MS1 RT minus 0.1 min and select the fragment scan whose precursor m/z and charge match the target glycan. Match observed fragment peaks against MS2 Fragment Database entries within the mass-accuracy tolerance, keeping only peaks above the 3% relative-intensity threshold. Annotate each matching m/z with up to two candidate fragments from the parent glycan, including adduct forms (+Na, +K, -OH), then render the annotated spectrum and write results to ms2_annotated_spectra.xlsx. The rationale is that accurate-mass matching of the precursor (per the MS1/MS2 mass search algorithms) plus fragment matching confirms the fluorescent-labeled glycan species; in the published dataset this annotated 18 of 24 distinct peaks above 3% relative abundance for glycan A1G1F(α1,3) at RT 13.77 min.

## Related tools

- **GlyKAn AZ app** (MATLAB app whose MS2 Peak Confirmation tab performs the precursor m/z and charge matching and renders annotated MS2 fragmentation spectra) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Runtime environment for the GlyKAn AZ application that executes the MS2 Peak Confirmation processing routine) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application

## Evaluation signals

- The selected fragment scan's precursor m/z and charge match the target glycan from the MS1 output for the user-selected glycan-RT pair
- MS2 scan extraction begins at the MS1 RT minus 0.1 min for each selected pair
- Only fragment peaks above the 3% relative-intensity threshold and within the mass-accuracy tolerance are annotated, with at most two candidate fragments (including +Na, +K, -OH adducts) per m/z
- Results are written to ms2_annotated_spectra.xlsx and match the reference result where 18 of 24 distinct peaks above 3% relative abundance were annotated for A1G1F(α1,3) at RT 13.77 min
- The annotated spectra agree with the RTs reported by manual analyst confirmation (all 20 true positives identified at the same RTs as manual analysis)

## Limitations

- The relative-intensity threshold and mass-accuracy tolerance are user-defined; suboptimal settings change true/false positive counts (e.g., at 25% threshold, 50 ppm gave 20 TP / 8 FP vs 10 ppm giving 20 TP / 3 FP)
- The app cannot process multiple mzXML files simultaneously
- No de novo identification: glycans absent from the MS2 Fragment Database cannot be annotated
- The isomer peak retention time range is a fixed value, not customizable

## Evidence

- [methods] For each user-selected glycan–RT pair, extract MS2 scans starting at the MS1 RT minus 0.1 min and locate the fragment scan whose precursor m/z and charge match the target glycan.: [evidence span withheld — anchor status: not_found]
- [intro] The app then returns annotated and labeled MS2 fragmentation spectra, providing the analyst with a method for glycan confirmation for the selected glycan peaks.: "The app then returns annotated and labeled MS2 fragmentation spectra, providing the analyst with a method for glycan confirmation for the selected glycan peaks."
- [results] of the 24 total distinct peaks falling above a relative abundance percentage of 3%, 18 fragment peaks were annotated: "of the 24 total distinct peaks falling above a relative abundance percentage of 3%, 18 fragment peaks were annotated"
- [abstract] MS1 and MS2 mass search algorithms along with glycan databases were developed to confirm the fluorescent labeled N-linked glycan species based on accurate mass: "MS1 and MS2 mass search algorithms along with glycan databases were developed to confirm the fluorescent labeled N-linked glycan species based on accurate mass"
- [readme] Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data.: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data."
