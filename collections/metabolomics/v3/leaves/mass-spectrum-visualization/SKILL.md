---
name: mass-spectrum-visualization
description: Use when MS data is already stored in the MZA (HDF5) format and a static
  PNG figure of an MS1 spectrum or a chromatogram trace (e.g., TIC) is needed over
  a defined m/z or retention-time range.
when_to_use_negative:
- The raw MS data is still in a vendor format (Agilent .d, Bruker ion mobility .d,
  Thermo .raw, or mzML) — it must first be converted to MZA with the mza converter
  tool before mzapy.view plotting can access the arrays.
- The goal is raw data extraction, calibration, signal processing, or peak finding
  rather than figure generation — mzapy provides separate utilities for those tasks.
- Interactive or exploratory plotting is required and no output image file is desired
  — omitting figname triggers display rather than saving, which may not fit automated
  workflows.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: mzapy
  role: Provides the mzapy.view plotting functions (plot_spectrum, plot_chrom) that
    consume arrays extracted via the mzapy.MZA interface and save figures when figname
    is set
  repo: https://github.com/PNNL-m-q/mzapy
- name: MZA
  role: HDF5-based file format whose MZA file supplies the spectra and chromatogram
    arrays to be plotted; produced by the mza converter from vendor raw files
  repo: https://github.com/PNNL-m-q/mza
- name: HDF5
  role: Underlying storage format of MZA files, read via HDF5 libraries (e.g., h5py)
    to access per-scan intensity and m/z arrays
- name: Python
  role: Implementation language of mzapy; the visualization workflow is a Python snippet
    using the MZA interface and mzapy.view module
- name: h5py
  role: Dependency required for Python access to MZA (HDF5) datasets during array
    extraction prior to plotting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:4dc77078a452e80bd8a7560f79b9b5b41fe4f0a47f3c8121838c37a5c0a77d30
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - HDF5
  - MZA
  - Python
  - h5py
  - mzapy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# mass-spectrum-visualization

## Summary

Generate publication-ready static figures (MS1 spectra and chromatograms) from spectra extracted out of MZA (HDF5-based) files using the mzapy.view plotting utilities (plot_spectrum, plot_chrom), which save to an image file when the figname kwarg is provided.

## When to use

Use when MS data is already stored in the MZA (HDF5) format and a static PNG figure of an MS1 spectrum or a chromatogram trace (e.g., TIC) is needed over a defined m/z or retention-time range.

## When NOT to use

- The raw MS data is still in a vendor format (Agilent .d, Bruker ion mobility .d, Thermo .raw, or mzML) — it must first be converted to MZA with the mza converter tool before mzapy.view plotting can access the arrays.
- The goal is raw data extraction, calibration, signal processing, or peak finding rather than figure generation — mzapy provides separate utilities for those tasks.
- Interactive or exploratory plotting is required and no output image file is desired — omitting figname triggers display rather than saving, which may not fit automated workflows.

## Inputs

- MZA file (HDF5-based mass spectrometry data, e.g., example.h5)
- Extracted MS1 m/z and intensity arrays (via mzapy MZA.collect_ms1_arrays_by_rt)
- Chromatogram trace arrays such as TIC (via mzapy MZA.get_tic)
- Display range for m/z (spectra) or retention time (chromatograms)

## Outputs

- Saved image file of the plotted spectrum or chromatogram (e.g., PNG specified via figname='tic_plot.png')

## How to apply

First open the MZA file with the mzapy.MZA interface (e.g., MZA('./data/example.h5')). Extract the arrays to plot: an MS1 spectrum's m/z and intensity arrays via collect_ms1_arrays_by_rt, or a chromatogram trace such as the TIC via get_tic. Pass those arrays to the matching mzapy.view function — plot_spectrum for spectra or plot_chrom for chromatograms — supplying the m/z or retention-time display range to window the view. Set the figname kwarg to an image file path (e.g., figname='tic_plot.png') so the function saves the figure to disk instead of displaying it interactively; this is the key decision point between file export and interactive display. Validate that the output figure file exists on disk and is a readable image in the expected PNG format. Python environments using the MZA tooling should have matplotlib available alongside h5py and hdf5plugin, per the MZA README's requirements.

## Related tools

- **mzapy** (Provides the mzapy.view plotting functions (plot_spectrum, plot_chrom) that consume arrays extracted via the mzapy.MZA interface and save figures when figname is set) — https://github.com/PNNL-m-q/mzapy
- **MZA** (HDF5-based file format whose MZA file supplies the spectra and chromatogram arrays to be plotted; produced by the mza converter from vendor raw files) — https://github.com/PNNL-m-q/mza
- **HDF5** (Underlying storage format of MZA files, read via HDF5 libraries (e.g., h5py) to access per-scan intensity and m/z arrays)
- **Python** (Implementation language of mzapy; the visualization workflow is a Python snippet using the MZA interface and mzapy.view module)
- **h5py** (Dependency required for Python access to MZA (HDF5) datasets during array extraction prior to plotting)

## Examples

```
from mzapy import MZA, view; mza = MZA('./data/example.h5'); mz, inten = mza.collect_ms1_arrays_by_rt(rt_start, rt_stop); view.plot_spectrum(mz, inten, mz_range=(mz_lo, mz_hi), figname='spectrum.png')
```

## Evaluation signals

- The output figure file specified by figname (e.g., tic_plot.png) exists on disk after the call.
- The output file is a readable image file in the expected PNG format (openable by an image reader).
- The plotted content corresponds to the extracted arrays: MS1 m/z/intensity from collect_ms1_arrays_by_rt or TIC from get_tic.
- The displayed m/z or retention-time window matches the display range passed to plot_spectrum or plot_chrom.
- The Python environment includes the required plotting dependency (matplotlib) alongside h5py, hdf5plugin, and numpy.

## Limitations

- For ion mobility spectra, the m/z dimension is stored as indexes (mzbins) into a shared Full_mz_array rather than direct m/z values, which must be handled when plotting IM-frame data.
- MZA files with too many spectra may use partitions, so array datasets may live under an MzaPath-indexed group (e.g., Arrays_intensity1/630 rather than Arrays_intensity/630), affecting which arrays are fetched for plotting.
- Spectra are stored omitting zero-intensity values (jagged per-scan arrays), so extracted arrays are not on a uniform grid and may need interpolation for certain visualizations.
- Saving to file requires the figname kwarg; otherwise the function displays the plot rather than producing a persistent output.

## Evidence

- [readme] Example scripts for visualization of mobility distributions from total (or mass extracted) ion intensities in the raw data in mza files: [evidence span withheld — anchor status: external_doc]
- [readme] Python requires the h5py, hdf5plugin, numpy and matplotlib packages (see requirements.txt).: [evidence span withheld — anchor status: external_doc]
