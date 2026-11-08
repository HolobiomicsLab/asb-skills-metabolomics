---
name: pymzml-spectral-data-parsing
description: Use when raw MALDI-ToF mass spectra exist as Bruker .fid files (or converted
  mzML) and you need per-sample ion counts summed over defined m/z ranges for screening
  or visualization.
when_to_use_negative:
- Samples smaller than 300 μm, where the article states microMS is more effective
  than the macroMS workflow
- Tasks requiring comprehensive spectral deconvolution or peak picking rather than
  simple ion-count integration over preset m/z windows
edam_operation: http://edamontology.org/operation_3214
tools:
- name: pymzml
  role: Python library used to read the converted mzML mass spectra
- name: CompassXport
  role: Bruker converter that converts .fid raw data files into the open mzML format
    before parsing
- name: macroMS
  role: Web platform whose data analysis stage consumes the parsed spectra and generates
    the ion-count Excel table
  repo: github.com/macroMS/macroMS_web
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:9dd2dd3aa7ed021ed0a68e52baaf2eae4a782eb87cf59706f6c3988b22cdf47e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CompassXport
  - macroMS
  - pymzml
  edam_operation: http://edamontology.org/operation_3214
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pymzml-spectral-data-parsing

## Summary

Parse mzML mass spectra in Python with the pymzml library and integrate (sum) ion counts within user-supplied m/z windows, as done in the macroMS MALDI-ToF-MS data analysis pipeline that converts Bruker .fid files to mzML via CompassXport.

## When to use

Use when raw MALDI-ToF mass spectra exist as Bruker .fid files (or converted mzML) and you need per-sample ion counts summed over defined m/z ranges for screening or visualization.

## When NOT to use

- Samples smaller than 300 μm, where the article states microMS is more effective than the macroMS workflow
- Tasks requiring comprehensive spectral deconvolution or peak picking rather than simple ion-count integration over preset m/z windows

## Inputs

- Bruker .fid MALDI-ToF raw data files converted to mzML via CompassXport
- User-supplied m/z window(s) corresponding to the target analyte(s)
- macroMS geometry file with sample identifiers

## Outputs

- Excel (.xlsx) spreadsheet listing summed ion counts, one row per sample and one column per m/z range, with rows labeled by sample identifiers
- Per-spectrum summed ion intensities for each target m/z range

## How to apply

First convert each Bruker .fid raw data file to the open mzML format with Bruker's CompassXport converter. Read the converted mzML spectra in Python with the pymzml library. For each spectrum, integrate (sum) the ion counts within the user-supplied m/z window(s) corresponding to the target analyte(s) — e.g., medium-chain fatty acyl ions in the Saccharomyces cerevisiae or E. coli / Pseudomonas fluorescens screens. Compile the summed intensities into a table with one row per sample and one column per m/z range, labeling rows with sample identifiers from the macroMS geometry file, and write the table to an Excel (.xlsx) spreadsheet. This forms the limited data analysis and visualization stage of the macroMS pipeline that helps locate samples containing compounds of interest.

## Related tools

- **pymzml** (Python library used to read the converted mzML mass spectra)
- **CompassXport** (Bruker converter that converts .fid raw data files into the open mzML format before parsing)
- **macroMS** (Web platform whose data analysis stage consumes the parsed spectra and generates the ion-count Excel table) — github.com/macroMS/macroMS_web

## Examples

```
import pymzml; run = pymzml.run.Reader('sample.mzML'); counts = {s.ID: sum(i for mz, i in s.peaks if mz0 <= mz <= mz1) for s in run}
```

## Evaluation signals

- Verify each Bruker .fid file has a corresponding CompassXport-produced mzML file that pymzml can open without errors
- Check that the generated Excel file has the expected row/column structure (one row per acquired sample, one column per m/z range)
- Confirm rows in the Excel table are labeled with the sample identifiers from the geometry file
- Confirm summed ion counts fall within plausible detector intensity ranges and match per-m/z-window integration, not peak-picked intensities
- Verify the m/z windows used match the target analyte(s) (e.g., medium-chain fatty acyl ions) specified by the user

## Limitations

- The macroMS data analysis is explicitly 'limited' — suitable for screening specific compounds, not full spectral analysis
- m/z windows must be supplied by the user; the pipeline does not automatically identify analytes
- Requires prior conversion of Bruker .fid files to mzML via CompassXport; it cannot read .fid directly

## Evidence

- [other] Read the converted mzML spectra with the pymzml library in Python.: [evidence span withheld — anchor status: not_found]
- [other] the ion counts over the defined m/z ranges are summed in the Python environment, and an Excel file listing ion counts for the m/z ranges for all of the mass spectra is generated: [evidence span withheld — anchor status: not_found]
