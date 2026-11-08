---
name: fragment-adduct-annotation
description: 'Use when confirming fluorescent-labeled N-linked glycan peaks from LC-MS/MS
  data: after MS1 screening produces an output Excel file and candidate glycan-RT
  pairs need MS2 fragment-level annotation to verify identity.'
when_to_use_negative:
- The glycans of interest are O-linked — the app's scope is limited to N-linked glycans
- De novo glycan identification is needed for species not in the database — the app
  only identifies glycans already located in the database
- Multiple mzXML files must be processed simultaneously — this is a planned future
  improvement, not implemented
edam_operation: http://edamontology.org/operation_3203
tools:
- name: GlyKAn AZ app
  role: Runs the MS2 Peak Confirmation tab that performs fragment matching, threshold
    filtering, and adduct-aware annotation of MS2 spectra
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Platform on which the GlyKAn AZ app implementing the annotation routine is
    built
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:7521519b7250d28866a6bf761e73365bd3144d335d4a7587d03f4c69c118c4a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - GlyKAn AZ app
  - MATLAB
  edam_operation: http://edamontology.org/operation_3203
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# fragment-adduct-annotation

## Summary

Annotate MS2 fragmentation spectra for a selected glycan–RT pair by matching observed fragment peaks against a curated MS2 Fragment Database within a mass-accuracy tolerance, keeping peaks above a relative-intensity threshold, and labeling each m/z with up to two candidate fragments including adduct forms (+Na, +K, -OH).

## When to use

Use when confirming fluorescent-labeled N-linked glycan peaks from LC-MS/MS data: after MS1 screening produces an output Excel file and candidate glycan-RT pairs need MS2 fragment-level annotation to verify identity.

## When NOT to use

- The glycans of interest are O-linked — the app's scope is limited to N-linked glycans
- De novo glycan identification is needed for species not in the database — the app only identifies glycans already located in the database
- Multiple mzXML files must be processed simultaneously — this is a planned future improvement, not implemented

## Inputs

- MS1 output Excel file listing candidate glycan-RT pairs
- MS2 Fragment Database (fragment names, theoretical masses, adduct forms)
- MS2 fragment image repository for spectrum rendering
- mzXML-format MS/MS data (converted from raw files with ProteoWizard)

## Outputs

- Annotated and labeled MS2 fragmentation spectrum per selected glycan-RT pair
- ms2_annotated_spectra.xlsx containing fragment annotations

## How to apply

Load the MS1 output Excel file, the MS2 Fragment Database (glycan names, theoretical masses, fragment entries), and the MS2 fragment image repository into the MS2 Peak Confirmation tab of the GlyKAn AZ app (MATLAB). For each user-selected glycan-RT pair, extract MS2 scans starting at the MS1 RT minus 0.1 min and locate the fragment scan whose precursor m/z and charge match the target glycan. Match observed fragment peaks against MS2 Fragment Database entries within the mass-accuracy tolerance, keeping only peaks above the relative-intensity threshold (e.g. the 3% relative-abundance cutoff used in validation). Annotate each matching m/z with up to two candidate fragments from the parent glycan, including adduct forms (+Na, +K, -OH), then render the annotated and labeled MS2 fragmentation spectrum and write results to ms2_annotated_spectra.xlsx. The rationale is that fragment-level confirmation distinguishes co-eluting isomers of identical mass that MS1 screening alone cannot resolve.

## Related tools

- **GlyKAn AZ app** (Runs the MS2 Peak Confirmation tab that performs fragment matching, threshold filtering, and adduct-aware annotation of MS2 spectra) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Platform on which the GlyKAn AZ app implementing the annotation routine is built) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application

## Evaluation signals

- Every annotated peak in the output must fall above the relative-intensity threshold and match a database fragment within the mass-accuracy tolerance
- Each annotated m/z carries at most two candidate fragment labels
- Adduct forms (+Na, +K, -OH) are present among annotations where applicable
- Annotated fragment counts are comparable to manual analysis (e.g. 18 of 24 peaks above 3% relative abundance annotated for glycan A1G1F(α1,3) at RT 13.77 min)
- Output file ms2_annotated_spectra.xlsx is produced with spectra rendered from the fragment image repository

## Limitations

- The isomer peak retention time range is a fixed value, not customizable
- The app cannot process multiple mzXML files simultaneously
- Only N-linked glycans in the database can be annotated; no de novo identification
- The GlyKAn AZ Instructions Manual (Additional file 1) content was not provided, limiting reproducibility of GUI steps

## Evidence

- [intro] This output file, the MS2 Fragment Database, and the MS2 fragment image repository can then be loaded into the MS2 Peak Confirmation tab.: "This output file, the MS2 Fragment Database, and the MS2 fragment image repository can then be loaded into the MS2 Peak Confirmation tab."
- [intro] The app then returns annotated and labeled MS2 fragmentation spectra, providing the analyst with a method for glycan confirmation for the selected glycan peaks.: "The app then returns annotated and labeled MS2 fragmentation spectra, providing the analyst with a method for glycan confirmation for the selected glycan peaks."
- [results] of the 24 total distinct peaks falling above a relative abundance percentage of 3%, 18 fragment peaks were annotated: "of the 24 total distinct peaks falling above a relative abundance percentage of 3%, 18 fragment peaks were annotated"
- [abstract] The GlyKAn AZ app can automatically annotate the MS/MS spectra, yet this data display feature remains flexible and customizable by users: "The GlyKAn AZ app can automatically annotate the MS/MS spectra, yet this data display feature remains flexible and customizable by users"
- [readme] Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data.: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data."
