---
name: retention-time-window-filtering
description: Use when a raw LC-IM-MS dataset has been converted to .mza and the analysis
  target is a narrow chromatographic region (e.g., a 0.5 min RT window) whose spectra
  must be isolated before mobility or m/z visualization.
when_to_use_negative:
- The data is still in vendor format (.d, .raw) or mzML and has not been converted
  to the MZA HDF5 structure, so there is no RetentionTime metadata table or per-scan
  Arrays_intensity/Arrays_mzbin groups to filter
- The analysis targets the full acquisition rather than a chromatographic region,
  in which case windowing would discard usable spectra
- The selection criterion is m/z or intensity based rather than chromatographic (e.g.,
  the -intensityThreshold conversion filter already handles intensity cutoffs at conversion
  time)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZA
  role: Produces the .mza HDF5 file whose metadata and per-scan array groups the RT
    filtering operates on
  repo: https://github.com/PNNL-m-q/mza
- name: HDF5
  role: File format underlying the .mza structure; filtered spectra are accessed via
    generic HDF5 libraries
  repo: https://github.com/PNNL-m-q/mza
- name: Python
  role: Language used with h5py to read the metadata, apply the RT mask, and pull
    arrays for selected scans
  repo: https://github.com/PNNL-m-q/mza
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:e0fc48208fdc1cb0e33d61cae9f0cf7c01ff6eb47267b25a71bd3aac78f2d5d2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - HDF5
  - MZA
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# retention-time-window-filtering

## Summary

Selects spectra from an MZA (.mza HDF5) file whose RetentionTime metadata falls within a specified LC window (e.g., 52–52.5 minutes) so that downstream IM-MS views are computed on a spatially defined chromatographic subset rather than the whole acquisition.

## When to use

Use when a raw LC-IM-MS dataset has been converted to .mza and the analysis target is a narrow chromatographic region (e.g., a 0.5 min RT window) whose spectra must be isolated before mobility or m/z visualization.

## When NOT to use

- The data is still in vendor format (.d, .raw) or mzML and has not been converted to the MZA HDF5 structure, so there is no RetentionTime metadata table or per-scan Arrays_intensity/Arrays_mzbin groups to filter
- The analysis targets the full acquisition rather than a chromatographic region, in which case windowing would discard usable spectra
- The selection criterion is m/z or intensity based rather than chromatographic (e.g., the -intensityThreshold conversion filter already handles intensity cutoffs at conversion time)

## Inputs

- .mza HDF5 file (MZA-converted LC-SLIM-MS raw data) with Metadata table, Arrays_intensity, Arrays_mzbin, and Full_mz_array groups
- RT window bounds in minutes (e.g., 52 and 52.5)

## Outputs

- Subset of spectra (Scan identifiers with their intensity and mzbin arrays) whose RetentionTime lies within the window
- Downstream products such as an ion mobility vs m/z heatmap matrix for the filtered subset

## How to apply

Read the .mza file with h5py and load the metadata table, where each row is a spectrum and RetentionTime is recorded in minutes. Apply a boolean mask (e.g., 52 <= RetentionTime <= 52.5) over the metadata rows to select scans in the window. For each selected scan, retrieve the jagged 1D arrays from the Arrays_intensity and Arrays_mzbin groups, resolving m/z values through the shared Full_mz_array dataset since IM spectra store m/z as bin indexes. The filtered scan subset then feeds aggregation steps such as assembling the mobility (IonMobilityTime, ms) vs m/z matrix for heatmap rendering. Rationale: filtering at the metadata level before touching intensity arrays avoids reading all spectra, and the MZA structure stores spectra as per-scan HDF5 datasets named by their Scan value, so the mask directly determines which datasets to open.

## Related tools

- **MZA** (Produces the .mza HDF5 file whose metadata and per-scan array groups the RT filtering operates on) — https://github.com/PNNL-m-q/mza
- **HDF5** (File format underlying the .mza structure; filtered spectra are accessed via generic HDF5 libraries) — https://github.com/PNNL-m-q/mza
- **Python** (Language used with h5py to read the metadata, apply the RT mask, and pull arrays for selected scans) — https://github.com/PNNL-m-q/mza

## Examples

```
import h5py; f=h5py.File('LC-SLIM-MS.mza','r'); rt=f['Metadata']['RetentionTime'][...]; scans=[s for s,r in zip(f['Metadata']['Scan'][...],rt) if 52<=r<=52.5]
```

## Evaluation signals

- Every retained scan's RetentionTime in the metadata satisfies the window bounds (e.g., all values within 52–52.5 minutes)
- Each selected Scan value resolves to a dataset in Arrays_intensity and Arrays_mzbin (or a partitioned group such as Arrays_intensity1 when MzaPath is set)
- For IM spectra, m/z values reconstructed via Full_mz_array[Arrays_mzbin/scan] fall within the file's common m/z axis
- Downstream heatmap rendered from the subset is visually equivalent to the vendor software view for the same RT window, with generation time around ~3 s in Python vs 8.89 s for the vendor path

## Limitations

- Partitions may be created for files with too many spectra, so scan arrays can live in partitioned groups (MzaPath index) rather than a single group; the filter must respect MzaPath when opening datasets
- IonMobilityBin = 0 represents the total frame spectrum (IM dimension ignored), so such rows need separate handling if only IM-resolved spectra are wanted
- RetentionTime is stored in minutes, so window bounds must be supplied in the same unit

## Evidence
