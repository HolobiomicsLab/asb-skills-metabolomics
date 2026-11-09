---
name: dda-msms-precursor-spectrum-retrieval
description: Use when a DDA MS/MS dataset has been converted to MZA (HDF5) and you
  need to pull out the fragmented MS2 spectrum (with m/z, intensity, and ion mobility
  arrival time where present) for a specific precursor identified in the metadata.
when_to_use_negative:
- The data is still in a proprietary vendor format (.raw, .d, mzML) and has not yet
  been converted to MZA — use the mza converter first
- The file was not acquired in DDA mode or contains no MS2 spectra (MSLevel 1 only)
  — there is no MS/MS spectrum to retrieve
- You need centroided peak lists or identifications rather than raw spectrum arrays
  — mzapy provides raw data extraction, not database search results
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: mzapy
  role: Python package whose mzapy.dda module provides MsmsReaderDda with get_msms_spectrum
    and drop_scans for extracting the precursor MS2 spectrum from MZA files
  repo: https://github.com/PNNL-m-q/mzapy
- name: MZA
  role: Source data format/structure (HDF5-based) whose metadata table and jagged
    spectrum arrays the reader accesses
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: HDF5 dependency through which mzapy reads the MZA file's metadata table and
    per-spectrum 1D arrays
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:1cca58cba4fa8c3cfb784f894981ed0745b9fe6c784903e761dd5eb5250d73f8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - MZA
  - h5py
  - mzapy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# dda-msms-precursor-spectrum-retrieval

## Summary

Retrieves the MS/MS (MS2) spectrum for a named precursor from a DDA mass spectrometry file converted to the MZA HDF5 format, using the mzapy.dda.MsmsReaderDda class. Handles identification of the precursor's scans from the MZA metadata table, optional removal of erroneous scans, and extraction of m/z, intensity, and ion mobility arrival-time arrays.

## When to use

Use when a DDA MS/MS dataset has been converted to MZA (HDF5) and you need to pull out the fragmented MS2 spectrum (with m/z, intensity, and ion mobility arrival time where present) for a specific precursor identified in the metadata.

## When NOT to use

- The data is still in a proprietary vendor format (.raw, .d, mzML) and has not yet been converted to MZA — use the mza converter first
- The file was not acquired in DDA mode or contains no MS2 spectra (MSLevel 1 only) — there is no MS/MS spectrum to retrieve
- You need centroided peak lists or identifications rather than raw spectrum arrays — mzapy provides raw data extraction, not database search results

## Inputs

- MZA-format file (HDF5) converted from a DDA MS/MS acquisition (e.g. from Thermo .raw, Agilent .d, Bruker IM .d, or mzML via the mza converter)
- Target precursor identity (e.g. precursor monoisotopic m/z or a precursor name resolvable from the MZA metadata table)
- Optional list of erroneous scan numbers to drop from the precursor's scans

## Outputs

- MS/MS spectrum arrays for the target precursor: m/z, intensity, and ion mobility arrival (drift) time dimensions where present
- In-memory spectrum object suitable for downstream mzapy processing (peak finding, calibration, plotting) or serialization

## How to apply

Open the MZA (HDF5) file with mzapy.MZA.__init__, optionally passing cache_metadata_headers=[] for the smallest memory footprint, and read the cached metadata to locate the scan numbers belonging to the target precursor (identified via columns such as PrecursorMonoisotopicMz, PrecursorCharge, and RetentionTime). If the metadata contains erroneous scans for that precursor, register them with mzapy.dda.MsmsReaderDda.drop_scans so they are excluded. Call MsmsReaderDda.get_msms_spectrum to extract the precursor's MS2 spectrum, retaining the m/z, intensity, and drift time (arrival time) dimensions where present, then close the file handle with mzapy.MZA.close. The rationale is that MZA stores each spectrum as jagged 1D arrays named by its Scan value in the Arrays_intensity and Arrays_mz (or Arrays_mzbin for IM spectra) groups, so the reader resolves those HDF5 datasets by scan number from the metadata table.

## Related tools

- **mzapy** (Python package whose mzapy.dda module provides MsmsReaderDda with get_msms_spectrum and drop_scans for extracting the precursor MS2 spectrum from MZA files) — https://github.com/PNNL-m-q/mzapy
- **MZA** (Source data format/structure (HDF5-based) whose metadata table and jagged spectrum arrays the reader accesses) — https://github.com/PNNL-m-q/mza
- **h5py** (HDF5 dependency through which mzapy reads the MZA file's metadata table and per-spectrum 1D arrays)

## Examples

```
from mzapy import MZA; from mzapy.dda import MsmsReaderDda; mza = MZA('sample.mza', cache_metadata_headers=[]); msms = MsmsReaderDda(mza); spec = msms.get_msms_spectrum(precursor)
```

## Evaluation signals

- The returned spectrum's m/z and intensity arrays are non-empty and correspond to scans whose metadata MSLevel is 2 and whose precursor fields match the target precursor
- Scans flagged as erroneous were dropped: scan numbers present in drop_scans do not contribute to the extracted spectrum
- For ion mobility data, the drift time (arrival time) dimension is present and its units match the MZA convention (ms for DT and SLIM; 1/k0 in Vs/cm2 for TimsTOF)
- The MZA file handle is closed after extraction (mzapy.MZA.close) without leaving open HDF5 handles
- Extracted arrays serialize cleanly (e.g. via numpy.savez) and reload without error, with lengths consistent across the m/z and intensity dimensions

## Limitations

- MZA spectra are stored omitting zero-intensity values, so extracted arrays are jagged/compressed and intensities of zero are absent
- Ion mobility m/z data is stored as indexes (Arrays_mzbin) into a shared Full_mz_array, so m/z reconstruction requires mapping through the mzbins index
- Files with very many spectra may be partitioned into multiple HDF5 groups (MzaPath index, e.g. Arrays_intensity1/630), which must be resolved when accessing arrays
- IonMobilityBin = 0 represents the total frame spectrum (IM dimension summed/ignored), which is not a true mobility-resolved MS2 spectrum

## Evidence

- [methods] Specialized DDA reader classes exist in mzapy.dda: "This module contains a couple of specialized classes for extracting data from DDA MS/MS MZA files."
- [readme] MZA HDF5 layout accessed by the reader: [evidence span withheld — anchor status: external_doc]
- [readme] Ion mobility arrival time semantics in the metadata table: [evidence span withheld — anchor status: external_doc]
- [abstract] mzapy's stated purpose for raw MS data extraction: "we present a Python package, mzapy, for efficient extraction and processing of mass spectrometry data in the MZA format"
