---
name: msp-spectral-file-export
description: Use when deconvolved DDA or DIA MS2 spectra (e.g. from PerformDDADeconvolution
  or PerformDIADeconvolution) have been merged into consensus spectra per MS1 feature
  and need to be written to a .msp file for database searching.
when_to_use_negative:
- Input is already an exported consensus_spectra.msp — go directly to reference-library
  searching via PerformDBSearchingBatch
- You only have raw vendor or mzML spectra that have not yet been deconvolved — run
  PerformDDADeconvolution or PerformDIADeconvolution first
- Spectra are single-acquisition (no replicate grouping) and do not require frequency-based
  fragment filtering
tools:
- name: MetaboAnalystR 4.0
  role: Provides PerformSpectrumConsenus for merging replicate MS2 spectra and exporting
    consensus_spectra.msp
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Open-source environment in which the MetaboAnalystR 4.0 consensus and export
    functions are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:8304794e96eb06215c99863c99f7054a5c5ca6631087ce9ec1393f7c303f5fd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# msp-spectral-file-export

## Summary

Export replicate-merged consensus MS2 spectra from MetaboAnalystR 4.0 into consensus_spectra.msp so downstream reference-library searching can consume them in a standard NIST MSP format.

## When to use

Use when deconvolved DDA or DIA MS2 spectra (e.g. from PerformDDADeconvolution or PerformDIADeconvolution) have been merged into consensus spectra per MS1 feature and need to be written to a .msp file for database searching.

## When NOT to use

- Input is already an exported consensus_spectra.msp — go directly to reference-library searching via PerformDBSearchingBatch
- You only have raw vendor or mzML spectra that have not yet been deconvolved — run PerformDDADeconvolution or PerformDIADeconvolution first
- Spectra are single-acquisition (no replicate grouping) and do not require frequency-based fragment filtering

## Inputs

- Deconvolved MS2 spectra from PerformDDADeconvolution or PerformDIADeconvolution (DDA or SWATH-DIA data)
- Replicate-grouped MS1 peak/feature table linking spectra to the same MS1 peak
- User-defined fragment frequency threshold (e.g., 50%)
- Optional SQLite reference spectra library list (L) per precursor for database-assisted consensus

## Outputs

- consensus_spectra.msp (NIST MSP-format file with one consensus spectrum per replicate-grouped MS1 feature)

## How to apply

First run the spectrum consensus step with PerformSpectrumConsenus, which groups all replicate MS2 spectra belonging to the same MS1 peak and merges them into one spectrum. During consensus, apply the user-defined fragment frequency threshold (e.g., 50%): a fragment is kept only if its frequency across replicates exceeds the threshold; optionally, sub-threshold fragments can be rescued if their m/z is found in the SQLite reference spectra list (L) for that precursor and observed in more than 2 replicates. Normalize the intensities of the surviving fragments before merging. Finally, export the merged spectra with PerformSpectrumConsenus to consensus_spectra.msp in NIST MSP format, one consensus spectrum per replicate-grouped MS1 feature, ready for PerformDBSearchingBatch.

## Related tools

- **MetaboAnalystR 4.0** (Provides PerformSpectrumConsenus for merging replicate MS2 spectra and exporting consensus_spectra.msp) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Open-source environment in which the MetaboAnalystR 4.0 consensus and export functions are executed)

## Examples

```
mSet <- PerformSpectrumConsenus(mSet)  # writes consensus_spectra.msp for downstream reference-library searching
```

## Evaluation signals

- consensus_spectra.msp exists in the output directory
- File parses as a valid NIST MSP file containing one consensus spectrum per replicate-grouped MS1 feature
- No fragment below the user-defined frequency threshold (e.g., 50%) is present unless rescued by the database-assisted rule (found in reference list L and observed in >2 replicates)
- Fragment intensities in the exported spectra are normalized across the merged replicates

## Limitations

- The frequency-threshold filtering can discard true fragments that appear in few replicates; only those matching the SQLite reference spectra list and seen in more than 2 replicates are recovered
- The exported .msp inherits the quality of the upstream deconvolution (DDA chimeric-spectrum and SWATH-DIA deconvolution) — errors made there propagate into the consensus spectra
- Consensus output is intended specifically as input for downstream reference-library searching via PerformDBSearchingBatch; it is not a general-purpose spectral file converter

## Evidence

- [methods] The spectrum consensus can be achieved with the function, PerformSpectrumConsenus.: "The spectrum consensus can be achieved with the function, PerformSpectrumConsenus."
- [methods] If the frequency of an individual fragment is above a user-defined threshold (e.g., 50%), it is kept; otherwise, the fragment is removed.: "If the frequency of an individual fragment is above a user-defined threshold (e.g., 50%), it is kept; otherwise, the fragment is removed."
- [methods] The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution.: "The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution."
- [readme] devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T): "devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T)"
