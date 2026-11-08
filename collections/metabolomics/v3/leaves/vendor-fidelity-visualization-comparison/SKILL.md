---
name: vendor-fidelity-visualization-comparison
description: Use when raw LC-IM-MS data has been converted to .mza and you need to
  recreate a vendor-browser view (e.g., mobility heatmap for RT 52–52.5 min) and compare
  speed against the vendor software.
when_to_use_negative:
- The input file is still a vendor raw format (.d, .raw) or plain mzML and has not
  been converted to the .mza HDF5 structure with Arrays_mzbin/Full_mz_array for IM
  spectra
- The analysis needs m/z values per spectrum rather than mzbins (e.g., non-IM MS1/MS2
  spectra stored as Arrays_mz), where the bin-indexed IM heatmap path does not apply
- You need quantitative CCS values rather than raw mobility distributions — use the
  CCS/z conversion scripts in EMSL-Computing/CCSz-demo instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZA
  role: Converted the vendor LC-SLIM-MS raw data into the .mza HDF5 structure that
    the Python visualization reads, enabling the fast heatmap generation compared
    to the vendor software
  repo: https://github.com/PNNL-m-q/mza
- name: HDF5
  role: File structure accessed with h5py to read the metadata table (RetentionTime,
    IonMobilityTime) and the jagged intensity/mzbin arrays per Scan
- name: Python
  role: h5py/numpy/matplotlib scripts that subset spectra by RT, build the mobility-time
    matrix, render the heatmap, and time the generation
- name: CCSz-demo
  role: Repository providing the example scripts for visualization of mobility distributions
    from total (or mass extracted) ion intensities in mza files, mirrored by the heatmap
    workflow
  repo: https://github.com/EMSL-Computing/CCSz-demo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:c44980a2d6313574c0f11688196bfffa8acccac7cb99710fa36eab0961670514
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - CCSz-demo
  - HDF5
  - MZA
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# vendor-fidelity-visualization-comparison

## Summary

Reproduce a vendor-software visualization (e.g., an ion mobility heatmap for an LC-IM-MS retention-time subset) from open MZA/HDF5 data in Python, and benchmark generation time against the vendor tool to confirm fidelity and speed.

## When to use

Use when raw LC-IM-MS data has been converted to .mza and you need to recreate a vendor-browser view (e.g., mobility heatmap for RT 52–52.5 min) and compare speed against the vendor software.

## When NOT to use

- The input file is still a vendor raw format (.d, .raw) or plain mzML and has not been converted to the .mza HDF5 structure with Arrays_mzbin/Full_mz_array for IM spectra
- The analysis needs m/z values per spectrum rather than mzbins (e.g., non-IM MS1/MS2 spectra stored as Arrays_mz), where the bin-indexed IM heatmap path does not apply
- You need quantitative CCS values rather than raw mobility distributions — use the CCS/z conversion scripts in EMSL-Computing/CCSz-demo instead

## Inputs

- MZA .mza HDF5 file (metadata table, Arrays_intensity, Arrays_mzbin, Full_mz_array) from LC-SLIM-MS data
- Retention-time window of interest (e.g., 52–52.5 minutes)
- Vendor software heatmap output used as the fidelity reference

## Outputs

- Ion mobility heatmap figure (mobility arrival time in ms vs m/z) saved via matplotlib
- Measured heatmap generation time in seconds (expected ~3 s for the MZA path vs 8.89 s vendor)

## How to apply

Read the .mza HDF5 file with h5py and select spectra whose RetentionTime (in minutes) falls within the target window (e.g., 52–52.5 min). For IM spectra, extract mzbins from Arrays_mzbin indexed against Full_mz_array together with matching intensity arrays from Arrays_intensity, using the Scan value in the metadata table to join them. Assemble the mobility-time matrix by mapping IonMobilityTime (arrival time in milliseconds for SLIM) against m/z and aggregating total or mass-extracted ion intensities per spectrum, mirroring the example visualization scripts in the repository (CCSz-demo). Plot the heatmap with matplotlib, then time the whole generation from file read to figure completion; the paper reports ~3 s from MZA data versus 8.89 s with the vendor software. Judge correctness by verifying the rendered heatmap displays the same information as the vendor figure and that elapsed time matches the reported MZA timing rather than the vendor path.

## Related tools

- **MZA** (Converted the vendor LC-SLIM-MS raw data into the .mza HDF5 structure that the Python visualization reads, enabling the fast heatmap generation compared to the vendor software) — https://github.com/PNNL-m-q/mza
- **HDF5** (File structure accessed with h5py to read the metadata table (RetentionTime, IonMobilityTime) and the jagged intensity/mzbin arrays per Scan)
- **Python** (h5py/numpy/matplotlib scripts that subset spectra by RT, build the mobility-time matrix, render the heatmap, and time the generation)
- **CCSz-demo** (Repository providing the example scripts for visualization of mobility distributions from total (or mass extracted) ion intensities in mza files, mirrored by the heatmap workflow) — https://github.com/EMSL-Computing/CCSz-demo

## Examples

```
import h5py; f = h5py.File('LC-SLIM-MS.mza','r'); idx = [i for i,rt in enumerate(f['RetentionTime']) if 52 <= rt <= 52.5]
```

## Evaluation signals

- The rendered heatmap displays the same information as the vendor software figure for the same RT window (fidelity check)
- Generation time from file read to figure completion is approximately 3 s for the MZA path, not the 8.89 s reported for the vendor software
- Selected spectra's RetentionTime values all fall strictly within 52–52.5 minutes
- Each extracted Arrays_intensity/Scan array matches the array length of the corresponding Arrays_mzbin/Scan, and mzbins index into Full_mz_array
- The heatmap figure is successfully saved by the matplotlib script

## Limitations

- IonMobilityTime is in milliseconds for DT and SLIM but is 1/k0 in Vs/cm2 for TimsTOF, so axis units differ across IM platforms
- Partitions may be created for files with too many spectra (MzaPath column), so array access may need group-specific paths like Arrays_intensity1/630
- IM spectra store m/z as bin indexes into Full_mz_array rather than per-spectrum m/z values, and IonMobilityBin = 0 represents the total frame spectrum ignoring the IM dimension

## Evidence

- [results] The ion mobility heatmap view for the subset of spectra at RT 52-52.5 min was generated from MZA data in Python in 3 s, faster than the 8.89 s required with the vendor software: [evidence span withheld — anchor status: not_found]
