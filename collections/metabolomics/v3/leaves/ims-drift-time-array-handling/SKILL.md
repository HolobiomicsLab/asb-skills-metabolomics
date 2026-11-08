---
name: ims-drift-time-array-handling
description: Use when raw IMS-MS data in MZA (HDF5) format must be extracted with
  the drift-time dimension intact — e.g. building arrival time distributions or precursor
  arrays retaining mz, intensity, retention time, and drift time.
when_to_use_negative:
- The MZA file lacks an ion mobility dimension (plain LC-MS) — there is no drift-time
  field to preserve and the IonMobilityBin/IonMobilityTime semantics do not apply.
- Data are already extracted 1D/2D arrays or a feature table — use mzapy's peak utilities
  (find_peaks_1d_localmax, find_peaks_1d_gauss, calc_peak_area, lerp_2d) on existing
  arrays instead of re-reading raw MZA spectra.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: mzapy
  role: 'Reads MZA (HDF5) files and provides the IM-aware extraction methods that
    retain the drift-time dimension: mzapy.MZA.collect_ms1_arrays_by_rt, collect_ms1_df_by_rt,
    collect_atd_arrays_by_rt_mz, and mzapy.dda.MsmsReaderDda.get_msms_spectrum'
  repo: https://github.com/PNNL-m-q/mzapy
- name: mza
  role: Command-line converter that writes vendor raw files (Agilent .d with IM, Bruker
    timsTOF, Thermo .raw, mzML) into the MZA structure whose Arrays_mzbin, Full_mz_array,
    and IonMobilityTime fields encode the IMS dimension
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: HDF5 access dependency mzapy relies on to read MZA datasets such as Arrays_mzbin
    and Full_mz_array
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:61521d54ff1f8431234c9baa82569b34d064fdbe694c516ad5033e9936b1e026
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - MZA
  - h5py
  - mza
  - mzapy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ims-drift-time-array-handling

## Summary

Extracts the ion mobility (drift/arrival time) dimension from MZA (HDF5) mass spectrometry files with mzapy, correctly handling MZA's Arrays_mzbin/Full_mz_array storage and instrument-specific IonMobilityTime units so that extracted arrays retain mz, intensity, retention time, and drift time.

## When to use

Use when raw IMS-MS data in MZA (HDF5) format must be extracted with the drift-time dimension intact — e.g. building arrival time distributions or precursor arrays retaining mz, intensity, retention time, and drift time.

## When NOT to use

- The MZA file lacks an ion mobility dimension (plain LC-MS) — there is no drift-time field to preserve and the IonMobilityBin/IonMobilityTime semantics do not apply.
- Data are already extracted 1D/2D arrays or a feature table — use mzapy's peak utilities (find_peaks_1d_localmax, find_peaks_1d_gauss, calc_peak_area, lerp_2d) on existing arrays instead of re-reading raw MZA spectra.

## Inputs

- MZA file (HDF5-based) converted from vendor raw data — Agilent '.d' (with ion mobility), Bruker ion mobility '.d' (timsTOF), Thermo '.raw', or mzML
- MZA metadata table (mzapy.MZA.metadata) with columns Scan, MSLevel, RetentionTime, IonMobilityFrame, IonMobilityBin, IonMobilityTime, PrecursorMonoisotopicMz, PrecursorCharge
- Target precursor definition (name or precursor m/z) and optional list of erroneous scans to drop

## Outputs

- IM-resolved MS1 arrays (mz, i, rt, dt) from mzapy.MZA.collect_ms1_arrays_by_rt / collect_ms1_df_by_rt
- Arrival time distribution (ATD) arrays from mzapy.MZA.collect_atd_arrays_by_rt_mz
- Precursor MS/MS spectrum arrays from mzapy.dda.MsmsReaderDda.get_msms_spectrum
- Serialized extracted_precursor_arrays.npz with named fields (mz, i, rt, dt) via numpy.savez

## How to apply

Open the MZA (HDF5) file with mzapy.MZA.__init__, setting cache_metadata_headers (pass [] for the smallest memory footprint), and read mzapy.MZA.metadata to locate the scans of interest, dropping erroneous scans with mzapy.dda.MsmsReaderDda.drop_scans if needed. For ion mobility spectra, map the mzbins in Arrays_mzbin through the shared Full_mz_array dataset to recover true m/z values, since MZA stores the IM m/z dimension as indexes rather than per-scan m/z arrays. Treat scans with IonMobilityBin = 0 as total frame spectra (IM dimension ignored/summed) and keep them separate from mobility-resolved bins; interpret IonMobilityTime as arrival time in milliseconds for Agilent DT and SLIM data but as 1/k0 (Vs/cm2) for Bruker TimsTOF. Extract IM-resolved MS1 arrays with mzapy.MZA.collect_ms1_arrays_by_rt / collect_ms1_df_by_rt, arrival time distributions with collect_atd_arrays_by_rt_mz, and precursor MS/MS with mzapy.dda.MsmsReaderDda.get_msms_spectrum, always retaining the dt dimension. Before converting arrival times to collision cross sections, check for CCS calibration coefficients written at conversion time (CCScalDT = [Tfix, Beta], CCScalSLIM = [C0, C1, C2, C3], CCScalTIMS = [C0, C1, C2, C3, C4, C5, C6, C7, C8, C9]). Close the handle with mzapy.MZA.close and serialize the arrays as named fields (mz, i, rt, dt) in extracted_precursor_arrays.npz via numpy.savez.

## Related tools

- **mzapy** (Reads MZA (HDF5) files and provides the IM-aware extraction methods that retain the drift-time dimension: mzapy.MZA.collect_ms1_arrays_by_rt, collect_ms1_df_by_rt, collect_atd_arrays_by_rt_mz, and mzapy.dda.MsmsReaderDda.get_msms_spectrum) — https://github.com/PNNL-m-q/mzapy
- **mza** (Command-line converter that writes vendor raw files (Agilent .d with IM, Bruker timsTOF, Thermo .raw, mzML) into the MZA structure whose Arrays_mzbin, Full_mz_array, and IonMobilityTime fields encode the IMS dimension) — https://github.com/PNNL-m-q/mza
- **h5py** (HDF5 access dependency mzapy relies on to read MZA datasets such as Arrays_mzbin and Full_mz_array)

## Examples

```
import mzapy; mza = mzapy.MZA('sample.mza', cache_metadata_headers=[])
```

## Evaluation signals

- Extracted arrays carry a dt (drift/arrival time) field whose units match the IonMobilityTime metadata: milliseconds for DT/SLIM, 1/k0 (Vs/cm2) for TimsTOF.
- m/z values recovered from Arrays_mzbin indexes equal the corresponding Full_mz_array values, with no contamination from frame-total spectra.
- Scans with IonMobilityBin = 0 (total frame spectrum, IM dimension ignored) are excluded or handled separately from mobility-resolved bins.
- The serialized extracted_precursor_arrays.npz loads without error and contains the expected named fields (mz, i, rt, dt) spanning the precursor's scans.
- ATD arrays from collect_atd_arrays_by_rt_mz cover the requested retention-time and m/z window with plausible arrival-time ranges.

## Limitations

- IonMobilityTime is instrument-dependent: arrival time in milliseconds for DT and SLIM but 1/k0 in Vs/cm2 for TimsTOF, so values are not directly interchangeable.
- For IM spectra the m/z dimension is stored as indexes (mzbins) into Full_mz_array, not as literal per-scan m/z values — plotting mzbins as m/z is wrong.
- IonMobilityBin = 0 marks the total frame spectrum or summed spectra in the frame (i.e., ignores the IM dimension); mixing it with mobility-resolved bins corrupts arrival time distributions.
- Files with too many spectra may be partitioned (MzaPath index), changing dataset paths to e.g. Arrays_intensity1/630, Arrays_mzbin1/630.

## Evidence

- [methods] Arrival Time Distributions ... collect_atd_arrays_by_rt_mz: [evidence span withheld — anchor status: not_found]
- [readme] For IM spectra the m/z dimension is stored as indexes (mzbins):: [evidence span withheld — anchor status: external_doc]
- [readme] Note: IonMobilityBin = 0, represents the total frame spectrum or summed spectra in the frame (i.e., ignores IM dimension).: [evidence span withheld — anchor status: external_doc]
- [readme] | IonMobilityTime | numeric    | Arrival time in milliseconds for DT and SLIM; 1/k0 in Vs/cm2 for TimsTOF |: [evidence span withheld — anchor status: external_doc]
