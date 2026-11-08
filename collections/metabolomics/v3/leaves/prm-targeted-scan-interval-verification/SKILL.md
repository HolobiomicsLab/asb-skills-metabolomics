---
name: prm-targeted-scan-interval-verification
description: Use when a Thermo .raw file is claimed to be PRM/targeted and you must
  confirm regularly spaced MS2 scans for a precursor (e.g. FTMS + c NSI Full ms2 487.2567@hcd27.00)
  via scan-index deltas.
when_to_use_negative:
- The acquisition is conventional DDA shotgun LC-MS/MS, where irregular MS2 spacing
  is expected and a constant delta would not be meaningful
- The data has already been converted to an exchange format like mzML via ProteoWizard/MSconvert,
  so rawrr's RawFileReader access is unnecessary
- The analysis goal is quantification or XIC extraction around the target mass at
  10 ppm using MS1-level scans rather than verifying the acquisition schedule
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: rawrr
  role: Provides rawrr::readIndex over the rawrr.exe/RawFileReader wrapper to return
    scan numbers, scan types, and filter strings used to detect targeted MS2 scans
  repo: https://github.com/fgcz/rawR
- name: R
  role: Environment in which readIndex output is subset, sorted, and diff() is applied
    to compute consecutive scan deltas
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:184bfc5e36f2ffed7672cb9c72c40f8c637084d93c38c8a48d37ca1d52d81944
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawR
  tools:
  - R
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# prm-targeted-scan-interval-verification

## Summary

Verifies parallel reaction monitoring (PRM) acquisition by extracting the scan index of a Thermo Fisher .raw file with rawrr::readIndex and showing that consecutive targeted MS2 scans are separated by a constant scan delta (e.g. 22 scans for the 487.2567 iRT precursor), confirming a regular PRM cycle.

## When to use

Use when a Thermo .raw file is claimed to be PRM/targeted and you must confirm regularly spaced MS2 scans for a precursor (e.g. FTMS + c NSI Full ms2 487.2567@hcd27.00) via scan-index deltas.

## When NOT to use

- The acquisition is conventional DDA shotgun LC-MS/MS, where irregular MS2 spacing is expected and a constant delta would not be meaningful
- The data has already been converted to an exchange format like mzML via ProteoWizard/MSconvert, so rawrr's RawFileReader access is unnecessary
- The analysis goal is quantification or XIC extraction around the target mass at 10 ppm using MS1-level scans rather than verifying the acquisition schedule

## Inputs

- Thermo Fisher Scientific Orbitrap raw file (e.g. 20181113_010_autoQC01.raw)
- Targeted MS2 filter string pattern for the PRM method (e.g. 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]')
- Expected cycle delta (22 scans) for pass/fail comparison

## Outputs

- scan_deltas.csv with two columns: scan number and delta between consecutive targeted MS2 scans
- Binary confirmation that all deltas equal 22 (PRM cycle confirmed) or a flag for deviations

## How to apply

Load the example raw file 20181113_010_autoQC01.raw (from MassIVE dataset MSV000086542 or via the Bioconductor data package tartare, ExperimentHub record EH4547) in R. Call rawrr::readIndex, which invokes the rawrr.exe wrapper over the Thermo Fisher RawFileReader .NET assembly, to obtain scan numbers, scan types, and filter strings for all 2.1881 x 10^4 scans of the 35 min LC-MS run. Subset the targeted MS2 scans matching the PRM filter string (e.g. 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'). Sort the scan-number vector and compute the difference between consecutive targeted scans with diff(); the decision point is whether every delta equals 22, i.e. one PRM acquisition cycle. Write the two-column table (scan number, delta) to scan_deltas.csv and flag any deviation from the constant delta.

## Related tools

- **rawrr** (Provides rawrr::readIndex over the rawrr.exe/RawFileReader wrapper to return scan numbers, scan types, and filter strings used to detect targeted MS2 scans) — https://github.com/fgcz/rawR
- **R** (Environment in which readIndex output is subset, sorted, and diff() is applied to compute consecutive scan deltas)

## Examples

```
idx <- rawrr::readIndex('20181113_010_autoQC01.raw'); ms2 <- sort(idx$ScanNumber[idx$ScanType=='MS2' & grepl('Full ms2 487.2567@hcd27.00', idx$FilterString)]); all(diff(ms2)==22)
```

## Evaluation signals

- scan_deltas.csv exists and contains the scan-number and delta columns
- Every recorded delta equals 22; any deviation is flagged
- Subsetting selects only MS2 scans whose filter string matches the targeted pattern 'FTMS + c NSI Full ms2 487.2567@hcd27.00'
- The scan-number vector is sorted and diff() is computed on consecutive values of that vector, not on the raw index order
- The total number of indexed scans is consistent with the 2.1881 x 10^4 scans of the 35 min run reported for this file

## Limitations

- rawrrSpectrum/rawrr content partially depends on the instrument model and installed instrument control software version
- On Windows, the decimal symbol must be configured as '.' for numeric vectors read back from the rawrr.exe wrapper to have correct lengths
- Access requires the vendor-provided RawFileReader .NET assembly invoked via rawrr.exe; rawR/rrawrr provides direct raw-file access rather than working on mzML or preprocessed data

## Evidence
