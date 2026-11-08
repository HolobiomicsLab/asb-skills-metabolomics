---
name: bruker-fid-to-mzml-conversion
description: Use when raw Bruker .fid MALDI-ToF mass spectra (e.g., acquired at optically
  guided macroMS target points) must be converted to open mzML for downstream Python
  processing such as pymzml ion-count integration.
when_to_use_negative:
- The spectra are already in mzML or another open format directly readable by pymzml
  — no CompassXport conversion step is needed.
- The raw data come from a non-Bruker instrument/format; CompassXport converts Bruker
  .fid files, not other vendors' raw formats.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: macroMS
  role: Web platform whose data-analysis pipeline consumes the converted mzML spectra
    for m/z-window ion counting and Excel/visualization output
  repo: github.com/macroMS/macroMS_web
- name: CompassXport
  role: Bruker converter run on each .fid raw data file to produce the open mzML format
- name: pymzml
  role: Python library used to read the converted mzML spectra for ion-count integration
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:e1768f6855a58f22575fca16f49078967319d6e6650c7ebc97cd16595911f9eb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CompassXport
  - macroMS
  - pymzml
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bruker-fid-to-mzml-conversion

## Summary

Converts Bruker .fid raw MALDI-ToF-MS data files into the open mzML format using Bruker's CompassXport, so the spectra can be read in Python with pymzml and integrated over user-supplied m/z windows in the macroMS data-analysis pipeline.

## When to use

Use when raw Bruker .fid MALDI-ToF mass spectra (e.g., acquired at optically guided macroMS target points) must be converted to open mzML for downstream Python processing such as pymzml ion-count integration.

## When NOT to use

- The spectra are already in mzML or another open format directly readable by pymzml — no CompassXport conversion step is needed.
- The raw data come from a non-Bruker instrument/format; CompassXport converts Bruker .fid files, not other vendors' raw formats.

## Inputs

- Bruker .fid raw MALDI-ToF mass spectra files (one per acquired sample/target point)
- Custom geometry file from macroMS defining sample identifiers and target locations

## Outputs

- mzML-format mass spectra (one converted file per .fid input)
- Downstream Excel (.xlsx) table of summed ion counts per m/z range per sample

## How to apply

For each Bruker .fid raw data file, run Bruker's CompassXport converter to emit an mzML file containing the spectrum. Read the converted mzML spectra in Python with the pymzml library. For each spectrum, integrate (sum) the ion counts within the user-supplied m/z window(s) corresponding to the target analyte(s), then compile the summed intensities into a table with one row per sample (labeled with sample identifiers from the geometry file) and one column per m/z range. This conversion step is what unlocks open-format, vendor-independent processing of the spectra acquired at the geometry-file target locations, enabling macroMS's downstream Excel output and visualization tools.

## Related tools

- **macroMS** (Web platform whose data-analysis pipeline consumes the converted mzML spectra for m/z-window ion counting and Excel/visualization output) — github.com/macroMS/macroMS_web
- **CompassXport** (Bruker converter run on each .fid raw data file to produce the open mzML format)
- **pymzml** (Python library used to read the converted mzML spectra for ion-count integration)

## Evaluation signals

- Each Bruker .fid file yields a corresponding mzML file that pymzml can open without errors.
- Number of mzML spectra matches the number of acquired sample points in the macroMS geometry file.
- The downstream Excel output has the expected row/column structure: one row per acquired sample, one column per m/z range.
- Rows in the ion-count table are labeled with the sample identifiers from the geometry file.
- Summed ion intensities fall within plausible counts for the selected m/z windows (sanity-check against raw spectrum peaks).

## Limitations

- The macroMS_web repository README provided contains no usage documentation, so conversion parameters must be taken from CompassXport's Bruker documentation.
- macroMS provides only limited data analysis; downstream analysis is constrained to m/z-window ion counting and visualization rather than full spectral processing.
- CompassXport is Bruker software, so the conversion is tied to Bruker instruments and licensing.

## Evidence

- [abstract] input the custom geometry files into an MS device to acquire spectra at the selected locations: "input the custom geometry files into an MS device to acquire spectra at the selected locations"
- [abstract] perform limited data analysis and use visualization tools to aid locating samples containing compounds of interest: "perform limited data analysis and use visualization tools to aid locating samples containing compounds of interest"
- [discussion] Equipped with several data analysis and visualization tools for the mass spectra generated for the samples, macroMS provides a pipeline for completing MS screens: "Equipped with several data analysis and visualization tools for the mass spectra generated for the samples, macroMS provides a pipeline for completing MS screens"
