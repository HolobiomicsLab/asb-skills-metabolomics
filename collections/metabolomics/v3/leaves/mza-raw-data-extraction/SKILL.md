---
name: mza-raw-data-extraction
description: Use when raw MS data in vendor formats (Agilent .d, Bruker TIMS .d, Thermo
  .raw, mzML) has been converted to MZA (.h5/HDF5) and arrays such as MS1 spectra,
  TIC, XICs, or arrival time distributions must be extracted programmatically.
when_to_use_negative:
- The input is still a proprietary vendor file (.d, .raw, mzML) that has not been
  converted — run the mza executable first
- The task is data conversion rather than extraction/access — use the stand-alone
  mza converter tool (github.com/PNNL-m-q/mza) instead of the mzapy access interface
- The data is not mass spectrometry data in MZA/HDF5 form (e.g., already-summarized
  feature tables), which this interface does not consume
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: mzapy
  role: Pure-Python interface used to open MZA files and extract MS1 spectra, TIC,
    XIC, and arrival-time-distribution arrays
  repo: https://github.com/PNNL-m-q/mzapy
- name: MZA (mza converter)
  role: Command-line executable that converts vendor MS files into the MZA HDF5 data
    structure this skill reads
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Underlying HDF5 access dependency mzapy relies on for reading MZA datasets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:63923f3ebe44fe3e100446657a771e78b642c9a6f680f09a6a11a945bc938348
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - MZA
  - MZA (mza converter)
  - Python
  - h5py
  - mzapy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# mza-raw-data-extraction

## Summary

Extract unprocessed mass spectrometry data from the HDF5-based MZA file format using the mzapy Python package, which provides a pure-Python interface to jagged m/z and intensity arrays plus per-spectrum metadata for multidimensional (LC, ion mobility, MS/MS) data.

## When to use

Use when raw MS data in vendor formats (Agilent .d, Bruker TIMS .d, Thermo .raw, mzML) has been converted to MZA (.h5/HDF5) and arrays such as MS1 spectra, TIC, XICs, or arrival time distributions must be extracted programmatically.

## When NOT to use

- The input is still a proprietary vendor file (.d, .raw, mzML) that has not been converted — run the mza executable first
- The task is data conversion rather than extraction/access — use the stand-alone mza converter tool (github.com/PNNL-m-q/mza) instead of the mzapy access interface
- The data is not mass spectrometry data in MZA/HDF5 form (e.g., already-summarized feature tables), which this interface does not consume

## Inputs

- MZA file (.h5, HDF5-based) produced by the mza converter from Agilent '.d', Bruker ion mobility '.d', Thermo '.raw', or mzML input
- Target accessor parameters: retention-time or m/z ranges, scan selection via the metadata table (Scan, MSLevel, RetentionTime, IonMobilityBin columns)

## Outputs

- 1D m/z and intensity arrays (jagged per-spectrum, zero-intensity values omitted at storage level)
- Chromatogram traces such as TIC from get_tic
- Extracted ion chromatogram and arrival-time-distribution arrays from collect_xic_arrays_by_mz and collect_atd_arrays_by_rt_mz

## How to apply

Open the MZA file with the mzapy.MZA interface (e.g., MZA('./data/example.h5')); internally the file is a metadata table plus HDF5 groups (Arrays_intensity, Arrays_mz, and for ion mobility data Arrays_mzbin/Full_mz_array) holding 1D jagged arrays per scan, keyed by the 'Scan' column. Use the accessor methods matching the target view: collect_ms1_arrays_by_rt for MS1 spectra over a retention-time window, get_tic for total ion chromatograms, collect_xic_arrays_by_mz for extracted ion chromatograms, and collect_atd_arrays_by_rt_mz for ion-mobility arrival time distributions. Because spectra are stored omitting zero-intensity values, the interface reconstructs usable m/z/intensity pairs for downstream plotting or processing (e.g., passing arrays to mzapy.view plot_spectrum/plot_chrom). For files with many scans, respect the MzaPath partition column (e.g., Arrays_intensity1/630 for MzaPath=1), which exists because too many datasets in one HDF5 group degrades read performance.

## Related tools

- **mzapy** (Pure-Python interface used to open MZA files and extract MS1 spectra, TIC, XIC, and arrival-time-distribution arrays) — https://github.com/PNNL-m-q/mzapy
- **MZA (mza converter)** (Command-line executable that converts vendor MS files into the MZA HDF5 data structure this skill reads) — https://github.com/PNNL-m-q/mza
- **h5py** (Underlying HDF5 access dependency mzapy relies on for reading MZA datasets)

## Examples

```
from mzapy import MZA; mza = MZA('./data/example.h5'); rt, tic = mza.get_tic(); mz, inten = mza.collect_ms1_arrays_by_rt(rtmin, rtmax)
```

## Evaluation signals

- Confirm the MZA file opens via mzapy.MZA and its metadata table exposes expected columns (Scan, MSLevel, RetentionTime, IonMobilityBin)
- Extracted array lengths must match between the m/z and intensity arrays for the same Scan value
- Extracted intensity arrays should reflect the converter's intensityThreshold — signals below that minimum must be absent
- For ion mobility scans, m/z values come via Arrays_mzbin indexes into the shared Full_mz_array; verify index bounds against Full_mz_array length
- Downstream use (e.g., plotting via mzapy.view) renders sensible spectra/chromatograms from the extracted arrays

## Limitations

- MZA stores spectra with zero-intensity values omitted, so consumers must handle jagged arrays rather than uniform spectra
- Ion mobility spectra store m/z as indexes (mzbins) into a shared Full_mz_array rather than explicit values
- Files with very many spectra may be partitioned (MzaPath column), requiring partition-aware dataset paths like Arrays_intensity1/630
- IonMobilityBin = 0 represents the total frame spectrum (IM dimension summed away), which can be mistaken for a full IM-resolved scan
- Analysis of IMS data has been described as challenging because incorporating the IMS dimension requires upgrading existing computational pipelines

## Evidence

- [readme] MZA file structure is a metadata table and two groups with 1D jagged arrays stored individually as HDF5 datasets per spectrum: [evidence span withheld — anchor status: external_doc]
- [readme] Spectra are stored omitting zero-intensity values and as two jagged arrays, one in each corresponding group and named as the "Scan" value in the metadata table: [evidence span withheld — anchor status: external_doc]
- [readme] Partitions may be created for files with too many spectra because HDF5 is more efficient with multiple groups: [evidence span withheld — anchor status: external_doc]
- [methods] Specialized extraction accessors for XIC and ATD arrays in mzapy: [evidence span withheld — anchor status: not_found]
