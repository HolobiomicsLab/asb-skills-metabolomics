---
name: matplotlib-figure-export
description: Use when you have extracted m/z-intensity arrays or a chromatogram trace
  (e.g., TIC via get_tic) from an MZA .h5 file and need a saved image file (e.g.,
  PNG) of the MS1 spectrum or chromatogram rather than a live plot window.
when_to_use_negative:
- No plot image is required — arrays are consumed directly by downstream numeric analysis
  (e.g., peak finding with mzapy.peaks or mass calibration) rather than visualized
- Data is not in the MZA (HDF5) format and cannot be read through the mzapy.MZA interface
- Interactive figure display is desired — the figname kwarg routes output to a saved
  file instead of a display window
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: mzapy
  role: Provides the mzapy.view plotting functions (plot_spectrum, plot_chrom) that
    accept arrays extracted from MZA files and save figures via the figname kwarg
  repo: https://github.com/PNNL-m-q/mzapy
- name: MZA
  role: HDF5-based MS data structure whose extracted arrays (MS1 spectra, TIC) are
    the direct inputs to the plotting functions
  repo: https://github.com/PNNL-m-q/mza
- name: Python
  role: Implementation language of mzapy; the plotting workflow is executed as pure
    Python with matplotlib as the figure-rendering backend
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:7eabfd61fc99b65121dd32e616fb3f12148b706aa44c43b448b253f527553145
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - MZA
  - Python
  - mzapy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# matplotlib-figure-export

## Summary

Generates publication-ready MS plot images (spectra and chromatograms) by passing arrays extracted from an MZA (HDF5) file into the mzapy.view plotting functions, saving figures to image files via the figname kwarg instead of displaying them interactively.

## When to use

Use when you have extracted m/z-intensity arrays or a chromatogram trace (e.g., TIC via get_tic) from an MZA .h5 file and need a saved image file (e.g., PNG) of the MS1 spectrum or chromatogram rather than a live plot window.

## When NOT to use

- No plot image is required — arrays are consumed directly by downstream numeric analysis (e.g., peak finding with mzapy.peaks or mass calibration) rather than visualized
- Data is not in the MZA (HDF5) format and cannot be read through the mzapy.MZA interface
- Interactive figure display is desired — the figname kwarg routes output to a saved file instead of a display window

## Inputs

- MZA file in HDF5-based format (.h5) opened via mzapy.MZA
- Extracted MS1 spectrum arrays (m/z and intensity) from collect_ms1_arrays_by_rt
- Chromatogram trace (e.g., TIC) from get_tic
- Display range (m/z or retention time) and figname image path

## Outputs

- Saved figure image file (e.g., PNG) of an MS1 spectrum via plot_spectrum
- Saved figure image file (e.g., PNG) of a chromatogram (e.g., TIC) via plot_chrom

## How to apply

Open the raw MS data in the MZA format with the mzapy.MZA interface (e.g., MZA('./data/example.h5')), then extract the arrays to plot: the MS1 spectrum (m/z and intensity arrays via collect_ms1_arrays_by_rt) or a chromatogram trace such as TIC via get_tic. Pass the extracted arrays to the corresponding mzapy.view function — plot_spectrum for an m/z display range or plot_chrom for a retention-time display range — and set the figname kwarg to an image file path (e.g., figname='tic_plot.png') so the function saves the figure to disk rather than displaying it. mzapy is implemented in pure Python with standardized dependencies; the visualization examples in the mza README rely on matplotlib, so figures are standard matplotlib image outputs. The rationale is that mzapy contains plotting utilities alongside raw data extraction so downstream agents can produce visual QC artifacts (spectra, chromatograms, mobility distributions) without writing custom plotting code.

## Related tools

- **mzapy** (Provides the mzapy.view plotting functions (plot_spectrum, plot_chrom) that accept arrays extracted from MZA files and save figures via the figname kwarg) — https://github.com/PNNL-m-q/mzapy
- **MZA** (HDF5-based MS data structure whose extracted arrays (MS1 spectra, TIC) are the direct inputs to the plotting functions) — https://github.com/PNNL-m-q/mza
- **Python** (Implementation language of mzapy; the plotting workflow is executed as pure Python with matplotlib as the figure-rendering backend)

## Examples

```
mz = mzapy.MZA('./data/example.h5'); x, y = mz.collect_ms1_arrays_by_rt(rt); mzapy.view.plot_chrom(x, y, rt_range, figname='tic_plot.png')
```

## Evaluation signals

- The output figure file exists on disk at the path given in figname (e.g., tic_plot.png)
- The saved file is a readable image in the expected PNG format
- The plotted m/z or retention-time axis matches the display range supplied to plot_spectrum or plot_chrom
- The plotted data matches the arrays returned by collect_ms1_arrays_by_rt or get_tic (same shape and intensity envelope)
- Re-running the invocation reproduces a consistent figure from the same MZA file and parameters

## Limitations

- Saving via the figname kwarg replaces interactive display — if figname is not set the behavior differs (display rather than save)
- Spectra in MZA files are stored omitting zero-intensity values and as jagged arrays; for ion mobility spectra the m/z dimension is stored as indexes (mzbins) into Full_mz_array, so plotting IM-dimension data may need additional handling
- The mzapy package targets MZA-format data; it cannot directly plot proprietary vendor formats (e.g., Thermo .raw, Agilent .d) without prior conversion via the mza tool

## Evidence

- [readme] MZA is a stand-alone and self-contained command-line executable which converts multidimensional mass spectrometry (MS) data from files in proprietary vendor formats to the MZA simple data structure based on the HDF5 format.: [evidence span withheld — anchor status: external_doc]
- [readme] Spectra are stored omitting zero-intensity values and as two jagged arrays, one in each corresponding group and named as the "Scan" value in the metadata table: [evidence span withheld — anchor status: external_doc]
