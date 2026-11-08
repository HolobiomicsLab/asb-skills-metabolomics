---
name: mz-intensity-range-integration
description: Use when you have MALDI-ToF-MS spectra (e.g., Bruker .fid converted to
  mzML) acquired at many target points and need total ion counts for specific m/z
  ranges per sample for screening.
when_to_use_negative:
- Samples smaller than 300 μm — the article states microMS is more effective for such
  targets
- You need full image reconstruction or spatial visualization of ion distributions
  rather than per-sample ion-count tables
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: macroMS
  role: Web platform whose data analysis pipeline performs the m/z-range ion counting
    and Excel export
  repo: github.com/macroMS/macroMS_web
- name: CompassXport
  role: Bruker converter that turns .fid raw spectra into the open mzML format consumed
    by the pipeline
- name: pymzml
  role: Python library used to read the converted mzML spectra and sum ion counts
    within m/z windows
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:7cb7d5c8908bfc4f98f72aa631422fc030a0bf4d3db3af74b8b8cb10f1c0a449
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CompassXport
  - macroMS
  - pymzml
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mz-intensity-range-integration

## Summary

Sum MALDI-ToF-MS ion counts within user-defined m/z windows for each acquired sample and compile the results into a per-sample, per-m/z-range Excel table, as implemented in the macroMS data analysis pipeline.

## When to use

Use when you have MALDI-ToF-MS spectra (e.g., Bruker .fid converted to mzML) acquired at many target points and need total ion counts for specific m/z ranges per sample for screening.

## When NOT to use

- Samples smaller than 300 μm — the article states microMS is more effective for such targets
- You need full image reconstruction or spatial visualization of ion distributions rather than per-sample ion-count tables

## Inputs

- Bruker .fid raw MALDI-ToF mass spectra converted to mzML (via CompassXport)
- User-supplied m/z window(s) defining the target analyte ion ranges
- Sample identifiers / geometry file mapping spectra to sample positions

## Outputs

- Excel (.xlsx) table with one row per acquired sample and one column per m/z range, containing summed ion counts

## How to apply

Convert each Bruker .fid raw spectrum to mzML with CompassXport, then read the mzML files with pymzml in Python. For each spectrum, sum the ion intensities within the user-supplied m/z window(s) corresponding to the target analyte(s) (e.g., medium-chain fatty acyl ions in the Saccharomyces cerevisiae mutant library screen). Compile the summed intensities into a table with one row per sample and one column per m/z range, labeling rows with sample identifiers from the geometry file, and write the table to an .xlsx spreadsheet. This 'limited data analysis' step is intentionally simple—it ranks samples by ion counts so visualization tools can aid locating samples containing compounds of interest.

## Related tools

- **macroMS** (Web platform whose data analysis pipeline performs the m/z-range ion counting and Excel export) — github.com/macroMS/macroMS_web
- **CompassXport** (Bruker converter that turns .fid raw spectra into the open mzML format consumed by the pipeline)
- **pymzml** (Python library used to read the converted mzML spectra and sum ion counts within m/z windows)

## Examples

```
import pymzml; run = pymzml.run.Reader('sample.mzML'); ion_counts = sum(i for mz, i in run[0].peaks(mz_min=530.0, mz_max=540.0))
```

## Evaluation signals

- The generated Excel file has the expected row/column structure: one row per acquired sample, one column per m/z range
- Row labels match the sample identifiers from the geometry file
- Each cell value equals the summed intensity within the specified m/z window and is non-negative
- The number of rows equals the number of converted mzML spectra (one per .fid sample)
- Screening throughput remains consistent with the ~2 s/sample acquisition rate before integration

## Limitations

- This is 'limited data analysis' — it produces ion-count tables for target compounds, not full spectral deconvolution or spatial imaging
- m/z windows must be supplied by the user; the pipeline does not automatically identify analyte peaks
- No quantitative benchmark or uncertainty is given for the pipeline outputs

## Evidence

- [full_text] the ion counts over the defined m/z ranges are summed in the Python environment, and an Excel file listing ion counts for the m/z ranges for all of the mass spectra is generated: [evidence span withheld — anchor status: not_found]
