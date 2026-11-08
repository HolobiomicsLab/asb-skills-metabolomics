---
name: data-access-performance-timing
description: Use when you must quantify data-access or visualization speed for raw
  LC-IM-MS spectra stored in .mza files — e.g., benchmarking MZA/HDF5 read-and-plot
  latency against vendor software on a defined RT window.
when_to_use_negative:
- Input is still a proprietary vendor file (.raw, Agilent/Bruker .d) that has not
  been converted to .mza — h5py cannot read it directly, so run MZA data conversion
  first.
- You need a throughput benchmark of the MZA converter itself (conversion speed/file
  size) rather than downstream data-access and visualization latency.
- The analysis targets spectra without the IM dimension stored (IonMobilityBin = 0
  total-frame spectra), where a mobility heatmap is not meaningful.
tools:
- name: MZA
  role: Converter that produced the .mza HDF5 file whose data access is being timed
  repo: https://github.com/PNNL-m-q/mza
- name: HDF5
  role: Underlying hierarchical storage format enabling cross-platform read of the
    .mza file
- name: Python
  role: Language used with h5py/hdf5plugin/numpy/matplotlib to read spectra, build
    the mobility-time matrix, plot the heatmap, and time the workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:6ed161ed505f56b24483bf992828e2613200349c5255c8403f62cc3b1d381173
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - HDF5
  - MZA
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# data-access-performance-timing

## Summary

Times how fast raw multidimensional MS spectra stored in the MZA HDF5 (.mza) format can be read and rendered into an ion-mobility heatmap view with Python/h5py, benchmarking open-format data access against the vendor software path (reported ~3 s vs 8.89 s).

## When to use

Use when you must quantify data-access or visualization speed for raw LC-IM-MS spectra stored in .mza files — e.g., benchmarking MZA/HDF5 read-and-plot latency against vendor software on a defined RT window.

## When NOT to use

- Input is still a proprietary vendor file (.raw, Agilent/Bruker .d) that has not been converted to .mza — h5py cannot read it directly, so run MZA data conversion first.
- You need a throughput benchmark of the MZA converter itself (conversion speed/file size) rather than downstream data-access and visualization latency.
- The analysis targets spectra without the IM dimension stored (IonMobilityBin = 0 total-frame spectra), where a mobility heatmap is not meaningful.

## Inputs

- MZA-format .mza HDF5 file containing IM-MS spectra (metadata table, Arrays_intensity, Arrays_mzbin, Full_mz_array)
- RT window definition in minutes (e.g., 52–52.5 min)
- Vendor-software timing baseline for the equivalent heatmap task (e.g., 8.89 s)

## Outputs

- Ion mobility heatmap figure (IonMobilityTime vs m/z, intensities aggregated per spectrum) saved from matplotlib
- Elapsed generation time in seconds measured from file read to figure completion
- Side-by-side timing comparison: MZA/Python (~3 s) vs vendor software (8.89 s)

## How to apply

Read the .mza file with h5py (plus hdf5plugin), selecting spectra whose RetentionTime (minutes) falls inside the target window (e.g., RT 52–52.5 min). For IM spectra, pull the m/z dimension as bin indexes from Arrays_mzbin and resolve them against the file-wide Full_mz_array, pairing each index array with its intensity array from Arrays_intensity; note IonMobilityBin = 0 marks the total frame spectrum ignoring the IM dimension. Assemble a mobility-time matrix mapping IonMobilityTime (ms for DT and SLIM; 1/k0 in Vs/cm2 for TimsTOF) against m/z, aggregating total or mass-extracted ion intensities per spectrum, then render the heatmap with matplotlib following the example scripts in the MZA/CCSz-demo visualization folders. Start the timer before the h5py read and stop it after the figure completes, recording elapsed seconds and comparing against the vendor-software timing (MZA ~3 s vs vendor 8.89 s for the same subset). If the file uses partitions (MzaPath column), address arrays as Arrays_intensity{N}/Scan and Arrays_mzbin{N}/Scan.

## Related tools

- **MZA** (Converter that produced the .mza HDF5 file whose data access is being timed) — https://github.com/PNNL-m-q/mza
- **HDF5** (Underlying hierarchical storage format enabling cross-platform read of the .mza file)
- **Python** (Language used with h5py/hdf5plugin/numpy/matplotlib to read spectra, build the mobility-time matrix, plot the heatmap, and time the workflow)

## Examples

```
import h5py, time; t0=time.time(); f=h5py.File('run.mza','r'); mz=f['Full_mz_array'][:]; inten=f['Arrays_intensity/630'][()]; mzbin=f['Arrays_mzbin/630'][()]; print(time.time()-t0)
```

## Evaluation signals

- Recorded generation time is approximately 3 s for the MZA/Python path, faster than the 8.89 s vendor-software baseline.
- The rendered heatmap figure is actually saved and displays the same information as the vendor software figure for the same spectra.
- All selected spectra satisfy 52 ≤ RetentionTime ≤ 52.5 minutes from the metadata table.
- Arrays are accessed with keys matching the metadata Scan values (e.g., Arrays_intensity/630, Arrays_mzbin/630), and mzbin indexes resolve correctly into Full_mz_array.
- IonMobilityBin = 0 spectra are treated as total-frame (IM-ignored) sums, consistent with the MZA file structure.

## Limitations

- Improvements in speed and file size are still needed for MZA, and the article reports no quantitative speed/file-size benchmarks for the converter itself.
- HDF5 read performance degrades when too many datasets sit in one group; large files may be partitioned (MzaPath column), requiring prefixed group access such as Arrays_intensity1/630.
- IonMobilityTime units differ by platform (ms for DT and SLIM; 1/k0 in Vs/cm2 for TimsTOF), so cross-platform heatmap axes must be interpreted carefully.
- Timing results are specific to the subset analyzed (e.g., RT 52–52.5 min) and hardware/software stack, and do not generalize to arbitrary windows or file sizes.

## Evidence

- [readme] Spectra are stored omitting zero-intensity values and as two jagged arrays, one in each corresponding group and named as the "Scan" value in the metadata table:: "Spectra are stored omitting zero-intensity values and as two jagged arrays, one in each corresponding group and named as the "Scan" value in the metadata table:"
- [readme] Note: IonMobilityBin = 0, represents the total frame spectrum or summed spectra in the frame (i.e., ignores IM dimension).: "Note: IonMobilityBin = 0, represents the total frame spectrum or summed spectra in the frame (i.e., ignores IM dimension)."
- [readme] Example scripts for visualization of mobility distributions from total (or mass extracted) ion intensities in the raw data in mza files: "Example scripts for visualization of mobility distributions from total (or mass extracted) ion intensities in the raw data in mza files"
- [discussion] Direct and easy access to the raw MS-data is critical to perform tasks like feature engineering and data transformations: "Direct and easy access to the raw MS-data is critical to perform tasks like feature engineering and data transformations"
