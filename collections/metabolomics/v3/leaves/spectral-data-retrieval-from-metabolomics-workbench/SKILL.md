---
name: spectral-data-retrieval-from-metabolomics-workbench
description: Use when raw SWATH-DIA metabolomics runs must be fetched from Metabolomics
  Workbench (e.g. ST002796/ST002798) as centroid mzML files and parsed into MS1 features
  plus SWATH MS2 windows before spectra deconvolution.
when_to_use_negative:
- The spectra are already available locally as a processed feature table or peak list
  rather than raw centroid mzML — no retrieval or mzR parsing is needed.
- The acquisition is DDA rather than SWATH-DIA; chimeric DDA spectra are handled by
  the separate PerformDDADeconvolution workflow instead.
- The data are profile-mode mzML; the workflow requires centroid mode to be confirmed
  before parsing.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetaboAnalystR 4.0
  role: Target pipeline whose SWATH-DIA deconvolution module (PerformDIADeconvolution,
    Rcpp/C++ core following DecoMetDIA) consumes the retrieved spectra
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: mzR
  role: Loads the retrieved mzML files and provides the MS1 feature table and SWATH
    MS2 windows
- name: XCMS
  role: Reference implementation (reconstructChromPeakSpectra) used to benchmark deconvolution
    on the same retrieved SWATH-DIA input
- name: R
  role: Open-source environment in which the retrieval and downstream unified LC-MS
    workflow run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:89d1766b49a94a98e9a285fc3964086ca5ea825bfe43680f9ae4ce5efa6282c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - R
  - XCMS
  - mzR
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectral-data-retrieval-from-metabolomics-workbench

## Summary

Retrieves deposited raw SWATH-DIA metabolomics spectra from Metabolomics Workbench (accessions ST002796 and ST002798) in centroid mzML mode and loads them via mzR so that the MS1 feature table and sequential SWATH MS2 windows can be parsed for downstream deconvolution with PerformDIADeconvolution in MetaboAnalystR 4.0.

## When to use

Use when raw SWATH-DIA metabolomics runs must be fetched from Metabolomics Workbench (e.g. ST002796/ST002798) as centroid mzML files and parsed into MS1 features plus SWATH MS2 windows before spectra deconvolution.

## When NOT to use

- The spectra are already available locally as a processed feature table or peak list rather than raw centroid mzML — no retrieval or mzR parsing is needed.
- The acquisition is DDA rather than SWATH-DIA; chimeric DDA spectra are handled by the separate PerformDDADeconvolution workflow instead.
- The data are profile-mode mzML; the workflow requires centroid mode to be confirmed before parsing.

## Inputs

- Metabolomics Workbench study accessions (e.g. ST002796, ST002798)
- Raw SWATH-DIA LC-MS runs in centroid mzML format

## Outputs

- Centroid mzML files locally loaded via mzR
- Parsed MS1 feature table
- Sequential SWATH MS2 window definitions (10 variable-size windows, 1.0 m/z neighbor overlap, ~0.9 s cycle time)

## How to apply

Retrieve the deposited raw SWATH-DIA metabolomics spectra from Metabolomics Workbench using accessions ST002796 and ST002798, and confirm the files are in centroid mzML mode before any processing. Load the mzML data with the mzR package and parse the MS1 feature table together with the sequential SWATH MS2 windows — in this dataset these are 10 variable-size windows with 1.0 m/z neighbor overlap and a ~0.9 s cycle time. The parsed MS1 features and their corresponding SWATH windows then feed the SWATH-DIA deconvolution module, which follows the DecoMetDIA approach re-implemented with an Rcpp/C++ core behind the PerformDIADeconvolution interface in MetaboAnalystR 4.0. Retrieval is only the first step of the pipeline: the downstream benchmark is performed against XCMS's reconstructChromPeakSpectra on the same input, so the retrieved files must be complete, untransformed raw spectra rather than pre-processed peak tables.

## Related tools

- **MetaboAnalystR 4.0** (Target pipeline whose SWATH-DIA deconvolution module (PerformDIADeconvolution, Rcpp/C++ core following DecoMetDIA) consumes the retrieved spectra) — https://github.com/xia-lab/MetaboAnalystR
- **mzR** (Loads the retrieved mzML files and provides the MS1 feature table and SWATH MS2 windows)
- **XCMS** (Reference implementation (reconstructChromPeakSpectra) used to benchmark deconvolution on the same retrieved SWATH-DIA input)
- **R** (Open-source environment in which the retrieval and downstream unified LC-MS workflow run)

## Examples

```
library(mzR); ms <- mzR::openMSfile("ST002796_swath.mzML"); ms1 <- mzR::peaks(ms); ms2 <- mzR::MSn(ms)
```

## Evaluation signals

- Retrieved files correspond to Metabolomics Workbench accessions ST002796 and ST002798 and are confirmed centroid mzML.
- mzR parsing yields an MS1 feature table plus exactly 10 variable-size SWATH MS2 windows with 1.0 m/z neighbor overlap.
- Cycle time of the SWATH windows is approximately 0.9 s, matching the dataset specification.
- The parsed features/windows feed PerformDIADeconvolution without re-conversion, and the run succeeds when the cluster containing each original MS feature is exported as a pseudo-MS2 spectrum in the .msp output.
- Deconvoluted spectra from the retrieved data are consistent with the XCMS reconstructChromPeakSpectra reference on the same dataset.

## Limitations

- The accessions cited (ST002796, ST002798) cover SWATH-DIA metabolomics data with 10 variable windows; other acquisition schemes or window layouts would need re-verification before use.
- The original DecoMetDIA approach had a computational bottleneck; the downstream module mitigates it via Rcpp/C++ and parallel computing, but retrieval does not by itself resolve runtime limits.
- Integrated LC-MS1 and MS2 spectra processing remains a bottleneck in computational efficiency and parameter-setting expertise, so retrieved raw data still require knowledgeable parameter choices downstream.

## Evidence

- [other] Retrieve the deposited raw SWATH-DIA metabolomics spectra from Metabolomics Workbench (accessions ST002796 and ST002798) and confirm they are in centroid mzML mode.: [evidence span withheld — anchor status: not_found]
- [other] Load the mzML data with the mzR package and parse the MS1 feature table plus the sequential SWATH MS2 windows (10 variable-size windows with 1.0 m/z neighbor overlap, ~0.9 s cycle time).: [evidence span withheld — anchor status: not_found]
- [methods] The deconvolution of SWATH-DIA data can be achieved with the function, PerformDIADeconvolution.: "The deconvolution of SWATH-DIA data can be achieved with the function, PerformDIADeconvolution."
- [abstract] Efficient MS2 spectra deconvolution and compound identification for data-dependent or data-independent acquisition.: "efficient MS2 spectra deconvolution and compound identification for data-dependent or data-independent acquisition"
