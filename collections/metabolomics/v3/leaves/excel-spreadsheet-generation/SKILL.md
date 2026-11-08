---
name: excel-spreadsheet-generation
description: Use when MALDI-ToF-MS spectra acquired at optically guided target points
  (mzML files) must be reduced to per-sample ion counts over specified m/z windows
  and exported as a structured .xlsx spreadsheet.
when_to_use_negative:
- Samples smaller than 300 μm, where microMS is stated to be more effective than the
  macroMS workflow
- The mzML spectra have not been generated (no CompassXport conversion step performed)
  or no m/z windows for the analytes of interest are defined
- An already-compiled per-sample feature/ion-count table exists — re-summing raw spectra
  would duplicate the step
tools:
- name: macroMS
  role: Web platform providing the data analysis and visualization tools whose pipeline
    sums ion counts over defined m/z ranges and generates the Excel output
  repo: github.com/macroMS/macroMS_web
- name: CompassXport
  role: Bruker converter used to turn .fid raw data files into the open mzML format
    read by the Python pipeline
- name: pymzml
  role: Python library used to read the converted mzML spectra before summing ion
    counts within m/z windows
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:de44821ab6043f2f3fe50029503e892c9944f22a76470caaaff959e5971640c9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CompassXport
  - macroMS
  - pymzml
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# excel-spreadsheet-generation

## Summary

Reduces MALDI-ToF-MS mass spectra (Bruker .fid converted to mzML) into a compiled Excel (.xlsx) table of summed ion counts over user-defined m/z ranges, one row per sample, enabling downstream screening for compounds of interest in the macroMS pipeline.

## When to use

Use when MALDI-ToF-MS spectra acquired at optically guided target points (mzML files) must be reduced to per-sample ion counts over specified m/z windows and exported as a structured .xlsx spreadsheet.

## When NOT to use

- Samples smaller than 300 μm, where microMS is stated to be more effective than the macroMS workflow
- The mzML spectra have not been generated (no CompassXport conversion step performed) or no m/z windows for the analytes of interest are defined
- An already-compiled per-sample feature/ion-count table exists — re-summing raw spectra would duplicate the step

## Inputs

- Bruker .fid raw MALDI-ToF-MS data files
- CompassXport-converted mzML spectra files
- User-supplied m/z window(s) for the target analyte(s)
- Geometry file with sample identifiers from the macroMS web interface

## Outputs

- Excel (.xlsx) spreadsheet listing ion counts for the m/z ranges for all mass spectra (one row per sample, one column per m/z range)

## How to apply

First convert each Bruker .fid raw data file to the open mzML format using Bruker's CompassXport converter, then read the converted mzML spectra in Python with the pymzml library. For each spectrum, integrate (sum) the ion counts within the user-supplied m/z window(s) corresponding to the target analyte(s). Compile the summed intensities into a table with one row per sample and one column per m/z range, labeling rows with the sample identifiers from the geometry file produced by the macroMS web workflow (sample finding, target filtering, device coordinate generation). Finally, write the compiled table to an Excel (.xlsx) spreadsheet so that visualization tools can aid locating samples containing compounds of interest.

## Related tools

- **macroMS** (Web platform providing the data analysis and visualization tools whose pipeline sums ion counts over defined m/z ranges and generates the Excel output) — github.com/macroMS/macroMS_web
- **CompassXport** (Bruker converter used to turn .fid raw data files into the open mzML format read by the Python pipeline)
- **pymzml** (Python library used to read the converted mzML spectra before summing ion counts within m/z windows)

## Examples

```
import pymzml; sp = pymzml.run.Reader('sample.mzML'); counts = [sum(i for m,z in s.peaks('raw') if mz_lo <= z <= mz_hi) for s in sp]
```

## Evaluation signals

- The generated Excel file has the expected structure: one row per acquired sample and one column per m/z range
- Row labels in the spreadsheet match the sample identifiers from the geometry file
- Each cell equals the summed (integrated) intensity of peaks within the specified m/z window, verifiable by re-computing sums from the mzML peaks with pymzml
- The number of rows matches the number of spectra acquired at the selected target locations from the custom geometry file
- m/z windows applied correspond exactly to the user-supplied ranges for the target analytes (e.g., MCFA or fatty acyl species)

## Limitations

- macroMS provides only limited data analysis; it is a screening aid for locating samples containing compounds of interest, not a full quantitative MS analysis platform
- No version information for macroMS is stated, and the pipeline details (exact m/z tolerances, integration method) are not fully documented in the article text
- The workflow is designed for macroscopic (>300 μm) samples; smaller samples should use microMS

## Evidence

- [full_text] In the macroMS data analysis pipeline, the ion counts over the defined m/z ranges are summed in the Python environment, and an Excel file listing ion counts for the m/z ranges for all of the mass spectra is generated.: [evidence span withheld — anchor status: not_found]
