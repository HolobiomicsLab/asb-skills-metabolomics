---
name: ms2-fragment-peak-annotation
description: Use when you have an MS1-identified glycan–RT pair from LC-MS/MS (Orbitrap
  or MALDI-TOF) data in mzXML format and need to confirm the glycan identity by annotating
  its MS2 fragment peaks.
when_to_use_negative:
- Your analytes are O-linked glycans — the app's scope is explicitly limited to fluorescent-labeled
  N-linked glycans
- The glycan of interest is not present in the MS2 Fragment Database — the app has
  no de novo identification capability
- You need to annotate peaks for a glycan lacking an MS1-identified RT (MS2 confirmation
  requires a user-selected glycan–RT pair as the starting point)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: GlyKAn AZ
  role: MATLAB app whose MS2 Peak Confirmation tab performs the fragment matching,
    annotation, and spectral rendering
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: GlyKAn AZ app
  role: Provides the MS2 Fragment Database and fragment image repository used for
    annotation and display
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: ProteoWizard
  role: Upstream conversion of Orbitrap raw MS files to mzXML format required by the
    app
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:65bfda484958d0f968f11a55d0fe67534752bd7c683f19f560b798e3ff890128
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - GlyKAn AZ app
  - MATLAB
  - OrbiTrap
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9736842105263158
n_claims: 38
---

# ms2-fragment-peak-annotation

## Summary

Annotate MS/MS fragmentation spectra for fluorescent-labeled N-linked glycans using the GlyKAn AZ app's MS2 Peak Confirmation tab, which matches observed fragment peaks against a curated MS2 Fragment Database and labels them in rendered spectra.

## When to use

Use when you have an MS1-identified glycan–RT pair from LC-MS/MS (Orbitrap or MALDI-TOF) data in mzXML format and need to confirm the glycan identity by annotating its MS2 fragment peaks.

## When NOT to use

- Your analytes are O-linked glycans — the app's scope is explicitly limited to fluorescent-labeled N-linked glycans
- The glycan of interest is not present in the MS2 Fragment Database — the app has no de novo identification capability
- You need to annotate peaks for a glycan lacking an MS1-identified RT (MS2 confirmation requires a user-selected glycan–RT pair as the starting point)

## Inputs

- MS1 output Excel file from the GlyKAn AZ MS1 Peak ID tab (glycan candidates with retention times)
- MS2 Fragment Database (theoretical fragment m/z values per glycan)
- MS2 fragment image repository for rendering labeled spectra
- mzXML-converted LC-MS/MS raw data (e.g., converted from Orbitrap raw files with ProteoWizard)

## Outputs

- Annotated and labeled MS2 fragmentation spectra per selected glycan–RT pair
- ms2_annotated_spectra.xlsx containing the annotated fragment peak lists

## How to apply

Load the MS1 output Excel file, the MS2 Fragment Database, and the MS2 fragment image repository into the MS2 Peak Confirmation tab, then select the glycan–RT pairs to confirm. For each pair, the app extracts MS2 scans starting at the MS1 RT minus 0.1 min and locates the fragment scan whose precursor m/z and charge match the target glycan. Observed fragment peaks are matched against MS2 Fragment Database entries within the mass-accuracy tolerance, keeping only peaks above the 3% relative-intensity threshold (the same threshold used when the paper annotated 18 of 24 distinct peaks for A1G1F(α1,3) at RT 13.77 min). Each matching m/z is annotated with up to two candidate fragments from the parent glycan, including adduct forms (+Na, +K, -OH). The app renders the annotated and labeled MS2 fragmentation spectrum using the fragment image repository and writes the annotated output to ms2_annotated_spectra.xlsx.

## Related tools

- **GlyKAn AZ** (MATLAB app whose MS2 Peak Confirmation tab performs the fragment matching, annotation, and spectral rendering) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **GlyKAn AZ app** (Provides the MS2 Fragment Database and fragment image repository used for annotation and display) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **ProteoWizard** (Upstream conversion of Orbitrap raw MS files to mzXML format required by the app)

## Evaluation signals

- For A1G1F(α1,3) at RT 13.77 min, 18 of 24 distinct peaks above 3% relative abundance should receive fragment annotations — a comparable annotation fraction is expected for similar glycan spectra
- Each annotated m/z carries at most two candidate fragment labels, potentially including +Na, +K, or -OH adduct forms
- Fragment matches must fall within the user-set mass-accuracy tolerance; no matches should exceed it
- All MS2 scans searched must start at the MS1 RT minus 0.1 min for the selected glycan–RT pair
- Output Excel file ms2_annotated_spectra.xlsx is produced with the annotated spectra, and the rendered spectrum images are consistent with the fragment image repository labels

## Limitations

- Only peaks whose relative intensity exceeds the 3% threshold are annotated, so low-abundance fragments are ignored
- Each m/z is annotated with up to two potential fragments — ambiguous assignments may remain
- The MS2 Fragment Database must already contain the glycan's fragmentation patterns; new glycans require the Fragment Generator functionality to expand the database
- Multiple mzXML files cannot be processed simultaneously (a planned future improvement)

## Evidence

- [results] MS2 scans starting at the MS1 RT minus 0.1 min: [evidence span withheld — anchor status: not_found]
- [results] 18 fragment peaks annotated: "of the 24 total distinct peaks falling above a relative abundance percentage of 3%, 18 fragment peaks were annotated"
- [intro] annotated and labeled MS2 fragmentation spectra: "The app then returns annotated and labeled MS2 fragmentation spectra, providing the analyst with a method for glycan confirmation for the selected glycan peaks."
- [intro] load MS2 Fragment Database and image repository: "This output file, the MS2 Fragment Database, and the MS2 fragment image repository can then be loaded into the MS2 Peak Confirmation tab."
- [readme] GlyKAn AZ purpose from README: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data."
