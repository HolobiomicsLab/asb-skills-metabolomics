---
name: chromatogram-trace-plotting
description: Use when you have retention-time and intensity arrays extracted from
  an MZA (HDF5-based) file with mzapy (e.g., via get_tic or collect_xic_arrays_by_mz)
  and need a visual chromatogram saved as an image like PNG.
when_to_use_negative:
- You need a spectrum (m/z vs intensity) plot instead of a retention-time trace —
  use mzapy.view.plot_spectrum on collect_ms1_arrays_by_rt output instead
- The data is still in a proprietary vendor format (.raw, Agilent .d, Bruker ion mobility
  .d) and has not been converted to MZA/HDF5 with the mza converter
- Interactive visualization or downstream numerical analysis of the trace is the goal
  rather than producing a saved figure file
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzapy
  role: Provides the MZA interface (mzapy.MZA) for extracting chromatogram arrays
    and the mzapy.view.plot_chrom plotting utility with the figname kwarg for saving
    the figure
  repo: https://github.com/PNNL-m-q/mzapy
- name: MZA
  role: HDF5-based MS data format that supplies the raw spectra from which chromatogram
    trace arrays are extracted
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Standardized HDF5 dependency used by mzapy to read the MZA file structure
    containing the trace data
- name: matplotlib
  role: Plotting backend required for mzapy.view figure generation; needed in the
    MZA example environment per requirements.txt
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:3634ba7e9d01450825abd029c6f21a45daab524090dced6fec5e2e6f00d3eee1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - MZA
  - Python
  - h5py
  - matplotlib
  - mzapy
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Chromatogram trace plotting

## Summary

Use mzapy.view's plot_chrom utility to render a chromatogram trace (e.g., TIC or XIC arrays extracted from an MZA/HDF5 file via get_tic or collect_xic_arrays_by_mz) as a saved image file rather than an interactive display.

## When to use

Use when you have retention-time and intensity arrays extracted from an MZA (HDF5-based) file with mzapy (e.g., via get_tic or collect_xic_arrays_by_mz) and need a visual chromatogram saved as an image like PNG.

## When NOT to use

- You need a spectrum (m/z vs intensity) plot instead of a retention-time trace — use mzapy.view.plot_spectrum on collect_ms1_arrays_by_rt output instead
- The data is still in a proprietary vendor format (.raw, Agilent .d, Bruker ion mobility .d) and has not been converted to MZA/HDF5 with the mza converter
- Interactive visualization or downstream numerical analysis of the trace is the goal rather than producing a saved figure file

## Inputs

- MZA file (HDF5-based MS data, e.g., './data/example.h5') opened via mzapy.MZA
- Retention-time and intensity arrays for a chromatogram trace (e.g., TIC from get_tic or XIC from collect_xic_arrays_by_mz)
- Retention-time display range (x-axis limits)

## Outputs

- Saved image file of the chromatogram plot (e.g., 'tic_plot.png') written to the figname path

## How to apply

First open the raw MS data file in the MZA (HDF5-based) format with the mzapy.MZA interface, e.g., MZA('./data/example.h5'). Extract the arrays to plot — for a chromatogram, a trace such as TIC via get_tic or an extracted ion chromatogram via collect_xic_arrays_by_mz, which return retention-time and intensity arrays. Pass these arrays to the corresponding mzapy.view plotting function plot_chrom, supplying a retention-time display range and setting the figname kwarg to an image file path (e.g., figname='tic_plot.png') so the function saves the figure to disk instead of displaying it. The rationale is that mzapy contains supporting utilities for generating plots in addition to raw data extraction, and matplotlib is a declared dependency of the MZA data-access workflow, so no extra plotting stack is needed.

## Related tools

- **mzapy** (Provides the MZA interface (mzapy.MZA) for extracting chromatogram arrays and the mzapy.view.plot_chrom plotting utility with the figname kwarg for saving the figure) — https://github.com/PNNL-m-q/mzapy
- **MZA** (HDF5-based MS data format that supplies the raw spectra from which chromatogram trace arrays are extracted) — https://github.com/PNNL-m-q/mza
- **h5py** (Standardized HDF5 dependency used by mzapy to read the MZA file structure containing the trace data)
- **matplotlib** (Plotting backend required for mzapy.view figure generation; needed in the MZA example environment per requirements.txt)

## Examples

```
from mzapy import MZA, view; mza = MZA('./data/example.h5'); rt, tic = mza.get_tic(); view.plot_chrom(rt, tic, figname='tic_plot.png')
```

## Evaluation signals

- The output figure file named by the figname kwarg (e.g., 'tic_plot.png') exists on disk after the call
- The output file is a readable image file in the expected PNG format
- The plotted x-axis corresponds to the requested retention-time display range
- The plotted trace matches the extracted intensity arrays (e.g., TIC values) from the source MZA file

## Limitations

- The skill covers chromatogram traces only; MS1 spectra require the sibling plot_spectrum utility with m/z display ranges
- When figname is not supplied, plot_chrom displays the plot rather than saving it, so a figname path must be set for file-based workflows
- Requires the source data already converted to the MZA format; ion mobility spectra store m/z as mzbins indexes into a Full_mz_array, which is a different access path

## Evidence

- [abstract] In addition to raw data extraction, mzapy contains supporting utilities enabling tasks including calibration, signal processing, peak finding, and generating plots: "In addition to raw data extraction, mzapy contains supporting utilities enabling tasks including calibration, signal processing, peak finding, and generating plots"
- [readme] MZA HDF5 access requires matplotlib: [evidence span withheld — anchor status: external_doc]
- [abstract] mzapy source availability: "The software source code is freely available at https://github.com/PNNL-m-q/mzapy"
- [methods] Extracted Ion Chromatograms collection function: [evidence span withheld — anchor status: not_found]
