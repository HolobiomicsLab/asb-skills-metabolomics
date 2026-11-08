---
name: flexible-plotting-interface-configuration
description: Use when you have m/z and intensity arrays or a chromatogram trace (e.g.,
  TIC via get_tic) extracted from an MZA .h5 file and need to generate a spectrum
  or chromatogram figure, optionally saved as a PNG via figname.
when_to_use_negative:
- When the input is a proprietary vendor raw file (Agilent .d, Bruker .d, Thermo .raw)
  not yet converted to MZA — convert first with the mza tool
- When the analysis goal is peak finding, calibration, or signal processing rather
  than visualization — use mzapy.peaks or mzapy.calibration utilities instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: mzapy
  role: provides the mzapy.view plotting module (plot_spectrum, plot_chrom) applied
    to arrays extracted via mzapy.MZA
  repo: https://github.com/PNNL-m-q/mzapy
- name: MZA
  role: HDF5-based source data structure whose extracted arrays feed the plotting
    functions
  repo: https://github.com/PNNL-m-q/mza
- name: HDF5
  role: underlying file format of the MZA container read by mzapy
- name: Python
  role: implementation language; plotting uses matplotlib per MZA requirements.txt
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:f0e6bdf89a688cda6c6d7828380779f08cd0c784576cbc36c3537fbd467e6172
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - HDF5
  - MZA
  - Python
  - mzapy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# flexible-plotting-interface-configuration

## Summary

mzapy.view supplies plotting utilities (e.g., plot_spectrum and plot_chrom) that render mass spectra and chromatogram traces extracted from MZA (HDF5-based) files, either interactively or saved directly to an image file via the figname kwarg.

## When to use

Use when you have m/z and intensity arrays or a chromatogram trace (e.g., TIC via get_tic) extracted from an MZA .h5 file and need to generate a spectrum or chromatogram figure, optionally saved as a PNG via figname.

## When NOT to use

- When the input is a proprietary vendor raw file (Agilent .d, Bruker .d, Thermo .raw) not yet converted to MZA — convert first with the mza tool
- When the analysis goal is peak finding, calibration, or signal processing rather than visualization — use mzapy.peaks or mzapy.calibration utilities instead

## Inputs

- MZA file (HDF5-based .h5) opened via mzapy.MZA
- m/z and intensity arrays extracted via collect_ms1_arrays_by_rt
- chromatogram trace (e.g., TIC) extracted via get_tic
- m/z or retention-time display range
- figname kwarg: output image file path (e.g., 'tic_plot.png')

## Outputs

- rendered spectrum plot (plot_spectrum) from m/z/intensity arrays
- rendered chromatogram plot (plot_chrom) from an RT-based trace
- saved image file (e.g., tic_plot.png) when figname is set

## How to apply

Open the raw MS data file in MZA (HDF5-based) format using the mzapy.MZA interface (e.g., MZA('./data/example.h5')). Extract the arrays to plot — the MS1 spectrum (m/z and intensity arrays via collect_ms1_arrays_by_rt) or a chromatogram trace such as TIC via get_tic. Pass the extracted arrays to the corresponding mzapy.view function: plot_spectrum for spectra or plot_chrom for chromatograms, supplying an m/z or retention-time display range. To save rather than display the figure, set the figname kwarg to an image file path (e.g., figname='tic_plot.png'). Validate by confirming the output figure file exists on disk and is a readable PNG.

## Related tools

- **mzapy** (provides the mzapy.view plotting module (plot_spectrum, plot_chrom) applied to arrays extracted via mzapy.MZA) — https://github.com/PNNL-m-q/mzapy
- **MZA** (HDF5-based source data structure whose extracted arrays feed the plotting functions) — https://github.com/PNNL-m-q/mza
- **HDF5** (underlying file format of the MZA container read by mzapy)
- **Python** (implementation language; plotting uses matplotlib per MZA requirements.txt)

## Examples

```
from mzapy import MZA, view; m = MZA('./data/example.h5'); rt, tic = m.get_tic(); view.plot_chrom(rt, tic, figname='tic_plot.png')
```

## Evaluation signals

- The output figure file named by figname exists on disk
- The saved figure is a readable image file in the expected PNG format
- The displayed m/z or retention-time axis matches the requested display range
- The plotted spectrum/chromatogram values correspond to the arrays extracted from the MZA file (e.g., TIC trace from get_tic)

## Limitations

- When figname is not supplied the function displays the plot instead of saving it, so a headless/automated run may produce no file output
- Plotting operates on extracted arrays; the MZA converter stores spectra omitting zero-intensity values, and IM spectra store m/z as mzbins indexes into Full_mz_array, which must be resolved before plotting

## Evidence

- [abstract] In addition to raw data extraction, mzapy contains supporting utilities enabling tasks including calibration, signal processing, peak finding, and generating plots: "In addition to raw data extraction, mzapy contains supporting utilities enabling tasks including calibration, signal processing, peak finding, and generating plots"
- [methods] mzapy.view functions save to file via figname: [evidence span withheld — anchor status: not_found]
- [readme] MZA access requirements include plotting libraries: [evidence span withheld — anchor status: external_doc]
- [abstract] MZA is HDF5-based and opened via mzapy.MZA: "MZA, a new and simple mass spectrometry data structure based on the broadly supported HDF5 format"
