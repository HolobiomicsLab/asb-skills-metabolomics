---
name: ion-mobility-heatmap-visualization
description: Use when raw multidimensional MS data (e.g., Agilent .d or SLIM-MS LC-IM-MS
  runs) has been converted to .mza and you need to visualize mobility distributions
  or benchmark non-vendor rendering speed of an ion mobility heatmap.
when_to_use_negative:
- The data contains no ion mobility dimension (no Arrays_mzbin / Full_mz_array), e.g.,
  plain LC-MS spectra — there is no mobility axis to plot.
- You need CCS values or CCS/z-converted plots rather than raw arrival-time mobility
  distributions — use the CCSz-demo calibration/conversion workflow instead, since
  raw mza files store arrival times, not collision cross sections.
- The file has not yet been converted from a proprietary vendor format (.d, .raw,
  mzML) to MZA — run the mza converter first; the executable requires Wine on Unix-like
  systems.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MZA
  role: Provides the .mza HDF5 data structure (metadata table, Arrays_intensity, Arrays_mzbin,
    Full_mz_array) that the heatmap is rendered from; conversion from vendor raw formats
    is a prerequisite
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Reads the .mza HDF5 file in Python, selecting spectra by RetentionTime and
    extracting jagged mzbins/intensity arrays per Scan
- name: matplotlib
  role: Renders the mobility × m/z heatmap figure from the assembled ion-intensity
    matrix
- name: CCSz-demo
  role: Reference example scripts for visualization of mobility distributions from
    total (or mass extracted) ion intensities in mza files
  repo: https://github.com/EMSL-Computing/CCSz-demo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:98b6336c322e97e9a19ab24f474097a0c051cac1ef3dc8022d01ebe9b9f564bb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - CCSz-demo
  - MZA
  - Python
  - h5py
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# ion-mobility-heatmap-visualization

## Summary

Generate an ion mobility × m/z heatmap from LC-SLIM-MS raw data stored in MZA HDF5 format, reading spectra (mzbins + intensities) directly with h5py and plotting with matplotlib. Demonstrates that Python access to .mza files is fast enough (≈3 s for a 0.5 min RT window) to rival vendor software (8.89 s) while displaying the same information.

## When to use

Use when raw multidimensional MS data (e.g., Agilent .d or SLIM-MS LC-IM-MS runs) has been converted to .mza and you need to visualize mobility distributions or benchmark non-vendor rendering speed of an ion mobility heatmap.

## When NOT to use

- The data contains no ion mobility dimension (no Arrays_mzbin / Full_mz_array), e.g., plain LC-MS spectra — there is no mobility axis to plot.
- You need CCS values or CCS/z-converted plots rather than raw arrival-time mobility distributions — use the CCSz-demo calibration/conversion workflow instead, since raw mza files store arrival times, not collision cross sections.
- The file has not yet been converted from a proprietary vendor format (.d, .raw, mzML) to MZA — run the mza converter first; the executable requires Wine on Unix-like systems.

## Inputs

- MZA (.mza) HDF5 file converted from LC-IM-MS raw data (e.g., SLIM-MS Agilent .d), with metadata table, Arrays_intensity, Arrays_mzbin, and Full_mz_array datasets
- Retention-time window of interest (e.g., 52–52.5 min) for subsetting spectra
- Python environment with h5py, hdf5plugin, numpy, and matplotlib

## Outputs

- Ion mobility (arrival time, ms) × m/z heatmap figure rendered with matplotlib and saved to disk
- Elapsed generation time in seconds (reported ≈3 s for MZA/Python vs 8.89 s for vendor software)

## How to apply

First ensure the vendor raw file (Agilent .d, Bruker .d, Thermo .raw, or mzML) has been converted with mza so that spectra live in HDF5 groups: Arrays_intensity (1D intensity arrays per Scan), Arrays_mzbin (1D indexes into Full_mz_array) for IM spectra. Read the .mza file with h5py (plus hdf5plugin), select spectra whose RetentionTime column in the metadata table falls within the window of interest (e.g., 52–52.5 minutes), and extract each spectrum's mzbins with its corresponding intensity array, resolving m/z values through Full_mz_array. Assemble a mobility-time matrix mapping IonMobilityTime (arrival time in ms for DT and SLIM; 1/k0 in Vs/cm² for TimsTOF) against m/z, aggregating total or mass-extracted ion intensities per spectrum; exclude IonMobilityBin = 0 entries, since those are summed frame spectra that ignore the IM dimension. Plot the heatmap with matplotlib, mirroring the example scripts in the mza repository's visualization folder and the CCSz-demo repository, and time the operation from file read to completed figure. The rationale is that MZA's simple HDF5 layout lets any language access raw MS data without vendor libraries, which are Windows-bound and slow for ad hoc views.

## Related tools

- **MZA** (Provides the .mza HDF5 data structure (metadata table, Arrays_intensity, Arrays_mzbin, Full_mz_array) that the heatmap is rendered from; conversion from vendor raw formats is a prerequisite) — https://github.com/PNNL-m-q/mza
- **h5py** (Reads the .mza HDF5 file in Python, selecting spectra by RetentionTime and extracting jagged mzbins/intensity arrays per Scan)
- **matplotlib** (Renders the mobility × m/z heatmap figure from the assembled ion-intensity matrix)
- **CCSz-demo** (Reference example scripts for visualization of mobility distributions from total (or mass extracted) ion intensities in mza files) — https://github.com/EMSL-Computing/CCSz-demo

## Examples

```
import h5py; f=h5py.File('raw.mza','r'); sel=(f['Metadata']['RetentionTime'][:]>=52)&(f['Metadata']['RetentionTime'][:]<=52.5); idx=f['Full_mz_array'][:]; I=f['Arrays_intensity/630'][:]
```

## Evaluation signals

- The rendered heatmap figure is actually saved to disk after the matplotlib step completes.
- The recorded generation time from file read to figure completion is ≈3 s for the MZA path (not the 8.89 s vendor-software timing).
- Spectra are subset correctly: only Scans whose metadata RetentionTime lies within the 52–52.5 min window contribute to the matrix.
- The IM axis resolves m/z through Full_mz_array via Arrays_mzbin indexes, and IonMobilityBin = 0 summed-frame spectra are excluded from the mobility dimension.
- The displayed heatmap information matches what the vendor software shows for the same RT subset.

## Limitations

- The paper notes that 'improvements in speed and file size are still needed' for MZA, and the discussion gives no quantitative speed/file-size benchmarks itself.
- Files with too many spectra may use partitioned HDF5 groups (MzaPath column); having too many datasets in one group can slow data-reading performance, so array access paths must respect MzaPath.
- IonMobilityBin = 0 represents the total frame spectrum summed over the IM dimension and must be handled explicitly to avoid contaminating the mobility view.
- IonMobilityTime units differ by platform: ms for DT and SLIM, but 1/k0 in Vs/cm² for TimsTOF, so axis labels are platform-specific.
- No changelog or version number is available for the mza repository, complicating reproducibility tracking.

## Evidence
