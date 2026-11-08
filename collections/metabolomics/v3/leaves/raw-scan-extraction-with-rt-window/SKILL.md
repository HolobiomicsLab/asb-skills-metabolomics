---
name: raw-scan-extraction-with-rt-window
description: Use when a targeted GC-CI-MS stable-isotope-labelling run needs its raw
  scan data inspected at a specific compound's retention time — e.
when_to_use_negative:
- When the goal is final isotopologue quantification rather than visual QC — the quantitative
  step consumes the mzML data separately and does not need the rawPlot heatmap.
- When the sample was acquired in low-resolution SIM/NS mode on a triple-quadrupole;
  this visualization targets high-resolution full-scan data where individual isotopologue
  m/z values are resolvable.
- When the target metabolite is absent from the formulaTable — no RT/m/z anchor exists
  for the window extraction.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: isoSCAN
  role: Provides the `rawPlot` function that opens the mzML via XCMS/mzR, extracts
    scans within the RT window around the target formula row, and renders the scan-intensity
    heatmap.
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Runtime environment in which isoSCAN and its rawPlot QC visualization are
    invoked from the console or a script.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:3f26b8c32ba32eea807ad84db17925840d8dc3a67dd886af7dea1334a88ad7a1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Raw-scan extraction with RT window

## Summary

A QC step in the isoSCAN GC-CI-MS workflow that opens a vendor-converted mzML file for a target metabolite and extracts the raw scan-by-scan intensity matrix within a bounded retention-time window, rendering it as a m/z-by-retention-time heatmap coloured by scan intensity to confirm that labelled and unlabelled isotopologues of the targeted ion cluster are present before quantitative processing.

## When to use

Use when a targeted GC-CI-MS stable-isotope-labelling run needs its raw scan data inspected at a specific compound's retention time — e.g. to verify that the expected [M+H]+ isotopic envelope of a derivatized metabolite (from the formulaTable) is present in a high-resolution Orbitrap or qTOF mzML file before running full isotopologue quantification.

## When NOT to use

- When the goal is final isotopologue quantification rather than visual QC — the quantitative step consumes the mzML data separately and does not need the rawPlot heatmap.
- When the sample was acquired in low-resolution SIM/NS mode on a triple-quadrupole; this visualization targets high-resolution full-scan data where individual isotopologue m/z values are resolvable.
- When the target metabolite is absent from the formulaTable — no RT/m/z anchor exists for the window extraction.

## Inputs

- High-resolution GC-CI-MS raw data file in mzML format (from GC-Orbitrap or GC-(q)TOF instruments), converted via XCMS/mzR-readable format
- A formula-table row from isoSCAN's `formulaTable_orbi` specifying the target compound's m/z, molecular formula, and expected isotopologues
- A retention-time window width parameter (RTwin, e.g. 5 s)

## Outputs

- A 'spectra heatmap' image (e.g. spectra_heatmap.png) plotting m/z versus retention time for the extracted scans, coloured by raw scan intensity
- A diagnostic visual confirmation of which isotopologues of the target metabolite are detected within the ±RTwin window

## How to apply

Select the compound of interest from the isoSCAN `formulaTable` (e.g. the first row of the Orbitrap sample table, carrying the target m/z, molecular formula and its isotopologues). Open the sample's mzML file via XCMS/mzR through `isoSCAN::rawPlot`, supplying the formula row and an `RTwin` of 5 seconds so that only scans falling within ±5 s of the compound's retention time are pulled from the raw data. `rawPlot` then builds a scan-by-scan intensity matrix with m/z on one axis and retention time on the other, colouring each point by raw scan intensity, producing a 'spectra heatmap'. Inspect the heatmap for the presence of the labelled peaks across the RT window — each expected isotopologue of the [M+H]+ cluster (generated from the isobutane CI ion) should appear as an intensity-coloured point at the correct m/z and retention time; absence or mis-position of these points indicates a retention-time mis-assignment or a missing isotopologue and should trigger re-checking the formula table or the raw file before proceeding to isotopologue quantification.

## Related tools

- **isoSCAN** (Provides the `rawPlot` function that opens the mzML via XCMS/mzR, extracts scans within the RT window around the target formula row, and renders the scan-intensity heatmap.) — https://github.com/jcapelladesto/isoSCAN
- **R** (Runtime environment in which isoSCAN and its rawPlot QC visualization are invoked from the console or a script.)

## Examples

```
rawPlot(files = files_hr[[1]], formulaTable_orbi[1, ], RTwin = 5)
```

## Evaluation signals

- The rendered heatmap must show the target metabolite's expected isotopologue m/z values as labelled, intensity-coloured points clustered at a single retention time within the ±RTwin window.
- The heatmap's axes must be m/z on one axis and retention time on the other, with colour encoding raw scan intensity — any transpose or colour-coding mismatch indicates the wrong formulaTable row was used.
- Only scans falling within ±RTwin (e.g. 5 s) of the compound's RT should appear; points far outside this window indicate an RT mis-assignment.
- The heatmap should be reproducible from the same mzML and formula row — re-running `rawPlot` with identical parameters must yield the same visual result.
- If a labelled isotope peak is missing from the heatmap, the downstream isotopologue quantification should show zero or anomalously low abundance for that isotopologue, corroborating the QC failure.

## Limitations

- The extraction is anchored to a single formula-table row; if the compound's retention time in the sample file drifts outside the ±RTwin window, the heatmap will be empty or partial, giving a false impression of missing isotopologues.
- rawPlot visualizes raw scan intensities without intensity normalization or background subtraction, so the heatmap is a QC view rather than a quantitative measurement.
- The ±5 s RTwin default is a heuristic — narrow windows may miss isotopologue peaks at scan boundaries, while very wide windows dilute the visual signal with noise.
- The heatmap is only meaningful for high-resolution data where individual isotopologue m/z values are distinguishable; in low-resolution QqQ SIM data the cluster is collapsed and the visualization offers little diagnostic value.

## Evidence

- [full_text] rawPlot prints a 'spectra heatmap' with m/z and retention time on the x,y-axes respectively, and points coloured according to the scan intensity value: [evidence span withheld — anchor status: not_found]
- [results] isoSCAN is an open-source R-based computational tool: [evidence span withheld — anchor status: not_found]
