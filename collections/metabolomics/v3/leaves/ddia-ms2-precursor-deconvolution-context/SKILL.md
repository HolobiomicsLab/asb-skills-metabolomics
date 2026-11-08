---
name: ddia-ms2-precursor-deconvolution-context
description: Use when raw LC-MS/MS data contain MS2 scans from pooled QC acquisitions
  whose precursor-ion associations must be reconstructed (especially DIA), before
  spectral library searching or compound annotation.
when_to_use_negative:
- Input is an already quantified MS1 feature table with no MS2 scans — there is nothing
  to deconvolve; use feature-detection/annotation skills instead
- The MS2 spectra are already deconvolved consensus spectra with precursor associations
  — proceed directly to MS2 spectral library searching rather than re-deconvolving
- The task is statistical analysis of complex multi-factor designs or exposomics dose-response
  analysis, which use processed feature tables rather than raw MS/MS acquisitions
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR 4.0
  role: Provides the R functions underlying the streamlined MS/MS spectra deconvolution
    and compound annotation module for both DDA and DIA data; runs locally with the
    bundled ~1.5 million MS2 spectra databases
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: OptiLCMS
  role: Companion R package installed via GitHub for LC-MS spectra processing (feature
    detection) that precedes MS2 deconvolution
  repo: https://github.com/xia-lab/OptiLCMS
- name: asari
  role: Algorithm underlying the enhanced Spectra Processing module with MS2 support
    in MetaboAnalyst 6.0
- name: MetaboAnalyst 6.0
  role: Web platform implementing the same spectra processing and MS2 deconvolution
    workflow; R command history can be downloaded to reproduce results locally with
    MetaboAnalystR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:a6c128e6ffae73d3306a77c291120bdc6c7fa36d0eef399c3299c0ed99032ee4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - OptiLCMS
  - R
  - asari
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# ddia-ms2-precursor-deconvolution-context

## Summary

Deconvolves tandem MS (MS2) spectra collected under DDA or DIA acquisition from pooled QC samples so that each consensus MS2 spectrum is re-associated with its precursor ion (m/z and retention time), enabling downstream compound annotation. This is the step in MetaboAnalyst 6.0 / MetaboAnalystR 4.0 that bridges raw MS/MS scans to searchable consensus spectra against ~1.5 million MS2 reference spectra.

## When to use

Use when raw LC-MS/MS data contain MS2 scans from pooled QC acquisitions whose precursor-ion associations must be reconstructed (especially DIA), before spectral library searching or compound annotation.

## When NOT to use

- Input is an already quantified MS1 feature table with no MS2 scans — there is nothing to deconvolve; use feature-detection/annotation skills instead
- The MS2 spectra are already deconvolved consensus spectra with precursor associations — proceed directly to MS2 spectral library searching rather than re-deconvolving
- The task is statistical analysis of complex multi-factor designs or exposomics dose-response analysis, which use processed feature tables rather than raw MS/MS acquisitions

## Inputs

- Raw LC-MS/MS acquisitions (DDA or DIA/SWATH) containing MS1 survey scans and MS2 spectra collected from pooled QC samples
- MS1 peak-picking results (e.g. from OptiLCMS feature detection) used to associate MS2 scans with precursor features

## Outputs

- Deconvolved consensus MS2 spectra, each annotated with its precursor m/z and retention time
- Deconvolved spectra feed directly into a compound identification table (feature ID, precursor m/z, RT, compound name, database accession, similarity score) with mirror plots

## How to apply

Run the spectra-processing/deconvolution module of MetaboAnalystR 4.0 (R base > 3.6.1; install the OptiLCMS companion for LC-MS1 peak detection) on the raw MS/MS acquisitions. For DIA data, deconvolution groups co-eluting MS2 fragment peaks across the SWATH/DIA windows and reconstructs which precursor m/z and retention time each MS2 spectrum belongs to; for DDA, the module consolidates the already-labeled precursor scans into consensus spectra. Combine deconvolved scans that share precursor m/z and co-elute into consensus MS2 spectra, retaining the precursor m/z and RT as metadata on each spectrum. These consensus spectra are then the direct input for MS2 database searching (dot-product or spectral-entropy similarity scoring with precursor-ion agreement constraints) against the bundled ~1.5 million MS2 spectra databases. The rationale is that MetaboAnalystR 4.0's streamlined deconvolution-plus-annotation pipeline is benchmarked to detect and identify > 10% more high-quality MS and MS/MS features and to raise the true positive rate of chemical identification by > 40% without increasing false identifications.

## Related tools

- **MetaboAnalystR 4.0** (Provides the R functions underlying the streamlined MS/MS spectra deconvolution and compound annotation module for both DDA and DIA data; runs locally with the bundled ~1.5 million MS2 spectra databases) — https://github.com/xia-lab/MetaboAnalystR
- **OptiLCMS** (Companion R package installed via GitHub for LC-MS spectra processing (feature detection) that precedes MS2 deconvolution) — https://github.com/xia-lab/OptiLCMS
- **asari** (Algorithm underlying the enhanced Spectra Processing module with MS2 support in MetaboAnalyst 6.0)
- **MetaboAnalyst 6.0** (Web platform implementing the same spectra processing and MS2 deconvolution workflow; R command history can be downloaded to reproduce results locally with MetaboAnalystR)

## Examples

```
install.packages("devtools"); library(devtools); devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- Every output consensus MS2 spectrum carries a reconstructed precursor m/z and retention time association (the defining property of successful DIA deconvolution)
- Downstream search of the deconvolved spectra against the bundled MS2 databases yields a compound identification table in which annotated features receive at least one potential named identity
- Benchmark expectation: the pipeline detects and identifies > 10% more high-quality MS and MS/MS features relative to the previous workflow
- Benchmark expectation: the true positive rate of chemical identification increases by > 40% without an increase in false identifications for both DDA and DIA datasets
- Serial-dilution checks show quantification and identification coverage consistent with the reported improved accuracy

## Limitations

- Deconvolution is required specifically because DIA data lack explicit precursor-spectrum links; the quality of reconstructed associations depends on co-elution/precursor reconstruction and directly gates downstream identification coverage
- MS2 spectra are typically collected only from pooled QC samples, so deconvolved spectra support compound identification but not per-sample quantification
- MetaboAnalystR documentation notes compatibility of the latest version (v4.0.0) is under evaluation and recommends R base > 3.6.1

## Evidence

- [intro] DIA deconvolution necessity: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [intro] MS2 from pooled QC: "To assist with compound identification, tandem MS (called MS/MS or MS2) spectra are often collected from pooled QC samples during the experiments"
- [readme] DDA/DIA deconvolution module in README: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
- [abstract] Enhanced spectra processing with MS2 and asari: "a significantly enhanced Spectra Processing module with support for MS2 data and the asari algorithm"
- [readme] Benchmark TPR improvement: "MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications"
