---
name: ion-mobility-spectrum-conversion
description: Use when raw IM-MS data (Agilent .d, Bruker TimsTOF .d, Thermo .raw,
  or mzML) must be converted to a cross-platform HDF5 structure for Python/R access,
  especially when the ion mobility dimension must be preserved.
when_to_use_negative:
- The goal is LC-MS proteomics feature extraction with lossy representations or IM
  dimension support is irrelevant — the paper notes MS2AI is limited to LC-MS proteomics
  and does not support the IM dimension, whereas MZA preserves it.
- The input is already an .mza file or another HDF5-based structure — data access
  via h5py/rhdf5 scripts suffices; no reconversion is needed.
- Vendor-specific processing is required that the stand-alone converter does not expose
  beyond file/extension/intensityThreshold parameters.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZA
  role: Stand-alone command-line executable that converts multidimensional MS data
    (including mzML) from vendor formats to the MZA HDF5 file structure with metadata
    table and per-spectrum array groups
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Python library used in example scripts to read and validate the .mza HDF5
    datasets and groups
  repo: https://github.com/PNNL-m-q/mza
- name: rhdf5
  role: R package (Bioconductor) used in example scripts to access .mza file datasets
  repo: https://github.com/PNNL-m-q/mza
- name: CCSz-demo
  role: Demo repository with Python/R scripts that read MZA files and visualize mobility
    distributions converted to CCS/z space
  repo: https://github.com/EMSL-Computing/CCSz-demo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:f04728dead31ba33e187639fb470b0dd15cea2770e962fc066d870e5f4357d31
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - CCSz-demo
  - MZA
  - Python
  - h5py
  - pymzML
  - rhdf5
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# ion-mobility-spectrum-conversion

## Summary

Convert multidimensional ion mobility mass spectrometry (IM-MS) data, including mzML inputs, into the MZA HDF5 file structure using the MZA converter, storing IM spectra as m/z bin indexes into a shared Full_mz_array plus per-spectrum intensity arrays and a metadata table.

## When to use

Use when raw IM-MS data (Agilent .d, Bruker TimsTOF .d, Thermo .raw, or mzML) must be converted to a cross-platform HDF5 structure for Python/R access, especially when the ion mobility dimension must be preserved.

## When NOT to use

- The goal is LC-MS proteomics feature extraction with lossy representations or IM dimension support is irrelevant — the paper notes MS2AI is limited to LC-MS proteomics and does not support the IM dimension, whereas MZA preserves it.
- The input is already an .mza file or another HDF5-based structure — data access via h5py/rhdf5 scripts suffices; no reconversion is needed.
- Vendor-specific processing is required that the stand-alone converter does not expose beyond file/extension/intensityThreshold parameters.

## Inputs

- Agilent .d MassHunter file (with or without ion mobility)
- Bruker TimsTOF ion mobility .d file
- Thermo .raw file
- mzML file
- Optional parameters: -extension (.mzML, .raw, or .d) for batch conversion; -intensityThreshold (minimum intensity)

## Outputs

- .mza HDF5 file containing a metadata table (one row per spectrum with Scan, MSLevel, Polarity, Activation, CollisionEnergy, RetentionTime, IonMobilityFrame, IonMobilityBin, IonMobilityTime, etc.)
- Arrays_intensity HDF5 group of 1D intensity datasets named by Scan
- Arrays_mz HDF5 group of 1D m/z datasets named by Scan (non-IM spectra)
- Arrays_mzbin HDF5 group of 1D m/z bin index datasets and a shared Full_mz_array dataset (IM spectra)
- Optional CCScalDT / CCScalSLIM / CCScalTIMS HDF5 datasets with CCS calibration coefficients

## How to apply

Run the stand-alone mza executable with -file pointing to the raw MS file and optionally -extension for batch conversion of all files in a path, plus -intensityThreshold to exclude signals below a minimum intensity (e.g., 20 in the README example). The converter emits an HDF5 .mza file with a metadata table (one row per spectrum; columns include Scan, MSLevel, Activation, CollisionEnergy, RetentionTime, IonMobilityFrame, IonMobilityBin, IonMobilityTime) and groups Arrays_intensity and Arrays_mz holding jagged 1D datasets named by scan number with zero-intensity values omitted. For ion mobility spectra, build a single shared Full_mz_array and store per-spectrum m/z bin indexes in Arrays_mzbin instead of raw m/z values, mapping IonMobilityBin=0 to the total frame spectrum summed over the IM dimension. CCS calibration coefficients, if detected, are embedded as HDF5 datasets: CCScalDT=[Tfix, Beta] for Agilent DT, CCScalSLIM=[C0..C3], and CCScalTIMS=[C0..C9] for Bruker TimsTOF. On Unix-like systems, run the Windows executable via Wine or a Docker container with wine. Validate by reading the .mza back with h5py/rhdf5 and confirming the metadata table, group structure, and scan-named datasets are present and correctly shaped.

## Related tools

- **MZA** (Stand-alone command-line executable that converts multidimensional MS data (including mzML) from vendor formats to the MZA HDF5 file structure with metadata table and per-spectrum array groups) — https://github.com/PNNL-m-q/mza
- **h5py** (Python library used in example scripts to read and validate the .mza HDF5 datasets and groups) — https://github.com/PNNL-m-q/mza
- **rhdf5** (R package (Bioconductor) used in example scripts to access .mza file datasets) — https://github.com/PNNL-m-q/mza
- **CCSz-demo** (Demo repository with Python/R scripts that read MZA files and visualize mobility distributions converted to CCS/z space) — https://github.com/EMSL-Computing/CCSz-demo

## Examples

```
mza -file test_data\LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20
```

## Evaluation signals

- Read the .mza back with h5py: the metadata table, Arrays_intensity/Arrays_mz (and Arrays_mzbin + Full_mz_array for IM data) groups exist, and each dataset name matches a Scan value in the metadata.
- Each spectrum's intensity and m/z arrays have equal length, and no zero-intensity values are stored (zero-intensity filtering is applied).
- For IM spectra, m/z values resolve correctly as Arrays_mzbin indexes into Full_mz_array, and IonMobilityBin=0 corresponds to the total frame (IM-dimension-summed) spectrum.
- Spot-check a scan (e.g., Scan 630 → Arrays_intensity/630) against the raw file's spectrum values and metadata columns such as RetentionTime (minutes) and IonMobilityTime (ms for DT/SLIM; 1/k0 in Vs/cm2 for TimsTOF).
- CCS calibration datasets (CCScalDT, CCScalSLIM, or CCScalTIMS) appear in the file only when calibration coefficients were detected during conversion.

## Limitations

- Improvements in speed and file size are still needed for MZA; no quantitative speed/file-size benchmarks are provided.
- Files with too many spectra may require partitions (MzaPath column indexes the HDF5 group), because having too many datasets in a single group slows HDF5 reading performance.
- The MZA executable is Windows-based; Unix-like systems require Wine (tested 6.0.3 with wine-mono 7.3.0) or a Docker container.
- No changelog or version number is stated for the MZA repository.

## Evidence
