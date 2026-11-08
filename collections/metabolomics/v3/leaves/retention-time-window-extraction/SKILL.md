---
name: retention-time-window-extraction
description: Use when confirming a candidate glycan identified in MS1 by pulling the
  MS2 fragmentation scans in a narrow window around the glycan's MS1 retention time
  (MS1 RT minus 0.1 min) from an mzXML-derived LC-MS/MS run.
when_to_use_negative:
- The glycan of interest is O-linked or absent from the MS2 Fragment Database — the
  app's scope is limited to N-linked glycans and offers no de novo identification
- The glycan-RT pair was not first confirmed in MS1 (no MS1 output Excel file with
  a retention time to anchor the window)
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GlyKAn AZ app
  role: Hosts the MS2 Peak Confirmation tab that performs the RT-window MS2 scan extraction
    and fragment annotation
  repo: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: MATLAB
  role: Implementation platform for the GlyKAn AZ app and its MS2 Peak Confirmation
    processing routine
  repo: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: ProteoWizard
  role: Upstream conversion of raw MS files to mzXML format that the app loads
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:ada6e2cb092dc2f4e958bace9c0a12b789ec67e74fc93ae3d47cdefe778fcd6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
  tools:
  - GlyKAn AZ
  - GlyKAn AZ app
  - MATLAB
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Retention-time-window extraction

## Summary

In GlyKAn AZ's MS2 Peak Confirmation tab, for each user-selected glycan-RT pair, MS2 scans are extracted starting at the MS1 retention time minus 0.1 min, then the fragment scan whose precursor m/z and charge match the target glycan is located for peak confirmation.

## When to use

Use when confirming a candidate glycan identified in MS1 by pulling the MS2 fragmentation scans in a narrow window around the glycan's MS1 retention time (MS1 RT minus 0.1 min) from an mzXML-derived LC-MS/MS run.

## When NOT to use

- The glycan of interest is O-linked or absent from the MS2 Fragment Database — the app's scope is limited to N-linked glycans and offers no de novo identification
- The glycan-RT pair was not first confirmed in MS1 (no MS1 output Excel file with a retention time to anchor the window)

## Inputs

- MS1 output Excel file from the GlyKAn AZ MS1 tab (glycan names, theoretical masses, identified RTs)
- MS2 Fragment Database (glycan fragment names and theoretical masses)
- MS2 fragment image repository
- User-selected glycan-RT pairs and user-defined thresholds (3% relative-intensity cutoff, mass-accuracy tolerance)

## Outputs

- Annotated and labeled MS2 fragmentation spectra for the selected glycan-RT pairs
- ms2_annotated_spectra.xlsx output file with fragment peak annotations

## How to apply

First, review the MS1 output Excel file to identify glycan-RT pairs likely present in the sample, then load that file, the MS2 Fragment Database, and the MS2 fragment image repository into the MS2 Peak Confirmation tab. For each user-selected glycan-RT pair, extract MS2 scans beginning at the MS1 RT minus 0.1 min and locate the fragment scan whose precursor m/z and charge match the target glycan. Within that scan, match observed fragment peaks against MS2 Fragment Database entries within the mass-accuracy tolerance, keeping only peaks above the 3% relative-intensity threshold, and annotate each matching m/z with up to two candidate fragments (including adduct forms +Na, +K, -OH) from the parent glycan. The rationale is that the narrow RT window anchored to the MS1-confirmed peak ensures the MS2 spectrum corresponds to the same chromatographic feature, enabling glycan confirmation via annotated fragmentation spectra rendered to ms2_annotated_spectra.xlsx.

## Related tools

- **GlyKAn AZ app** (Hosts the MS2 Peak Confirmation tab that performs the RT-window MS2 scan extraction and fragment annotation) — https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **MATLAB** (Implementation platform for the GlyKAn AZ app and its MS2 Peak Confirmation processing routine) — https://github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **ProteoWizard** (Upstream conversion of raw MS files to mzXML format that the app loads)

## Evaluation signals

- The extracted MS2 scan's precursor m/z and charge must match the target glycan's MS1-confirmed values
- Window start must be exactly MS1 RT minus 0.1 min for each selected glycan-RT pair
- Only fragment peaks above the 3% relative-abundance threshold and within the mass-accuracy tolerance should be annotated; at most two candidate fragments per m/z
- For the published validation dataset, glycan A1G1F(α1,3) at RT 13.77 min yielded 18 of 24 distinct peaks (>3% relative abundance) annotated — a useful benchmark
- Output must be written to ms2_annotated_spectra.xlsx with rendered, labeled fragmentation spectra

## Limitations

- The isomer peak retention time range is a fixed value, not customizable
- The app cannot process multiple mzXML files simultaneously (planned future improvement)
- Annotation depends on pre-built MS2 Fragment Database entries; fragments for new glycans must first be generated via the Fragment Generator

## Evidence

- [results] MS2 scan extraction window anchored to MS1 RT: [evidence span withheld — anchor status: not_found]
- [intro] Loading workflow into MS2 tab: [evidence span withheld — anchor status: not_found]
- [intro] Output spectra: "The app then returns annotated and labeled MS2 fragmentation spectra, providing the analyst with a method for glycan confirmation for the selected glycan peaks."
- [results] Benchmark annotation count: "of the 24 total distinct peaks falling above a relative abundance percentage of 3%, 18 fragment peaks were annotated"
- [readme] README provenance: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data."
