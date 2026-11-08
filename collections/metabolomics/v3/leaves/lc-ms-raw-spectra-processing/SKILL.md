---
name: lc-ms-raw-spectra-processing
description: Use when starting from raw LC-MS1/MS2 spectral files (e.g. mzML/CDF vendor
  conversions of untargeted metabolomics or exposomics runs, including pooled-QC SWATH-DIA
  acquisitions) and you need feature detection, quantification, and MS2 deconvolution
  before annotation or statistics.
when_to_use_negative:
- Input is already a peak/feature table rather than raw MS spectra — proceed directly
  to MS2 Peak Annotation or Statistical Analysis modules instead
- Targeted metabolomics data that do not need untargeted MS1 feature detection or
  DIA MS2 deconvolution
- MS2-only fragments without precursor/pooled-QC context, since DIA deconvolution
  requires reconstructing fragment-to-precursor associations from pooled QC spectra
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalyst 6.0
  role: Unified platform whose enhanced Spectra Processing module performs LC-MS1
    peak detection/quantification and MS2 deconvolution
- name: asari
  role: Algorithm invoked by the Spectra Processing module for LC-MS spectra processing
    (MS1 feature detection and quantification)
- name: MetaboAnalystR 4.0
  role: R package reproducing the web server's spectra-processing workflow locally,
    including OptiLCMS-based feature detection and MS/MS deconvolution for DDA and
    DIA
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: OptiLCMS
  role: R dependency installed from GitHub supporting the local LC-MS feature detection
    workflow
  repo: https://github.com/xia-lab/OptiLCMS
- name: R
  role: Runtime environment (version > 3.6.1 recommended) for the local MetaboAnalystR
    pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:e6d81a9f26b05a24587174c09c4410f6f4aefa3bc46f5277794a40f02e9b80d6
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# lc-ms-raw-spectra-processing

## Summary

Process raw LC-MS/MS spectral files (MS1 peak detection/quantification plus MS2 DDA/DIA deconvolution) using MetaboAnalyst 6.0's Spectra Processing module powered by the asari algorithm, as the entry point of the unified metabolomics workflow from raw spectra to compound identification.

## When to use

Use when starting from raw LC-MS1/MS2 spectral files (e.g. mzML/CDF vendor conversions of untargeted metabolomics or exposomics runs, including pooled-QC SWATH-DIA acquisitions) and you need feature detection, quantification, and MS2 deconvolution before annotation or statistics.

## When NOT to use

- Input is already a peak/feature table rather than raw MS spectra — proceed directly to MS2 Peak Annotation or Statistical Analysis modules instead
- Targeted metabolomics data that do not need untargeted MS1 feature detection or DIA MS2 deconvolution
- MS2-only fragments without precursor/pooled-QC context, since DIA deconvolution requires reconstructing fragment-to-precursor associations from pooled QC spectra

## Inputs

- Raw LC-MS/MS spectral files for study samples (untargeted metabolomics or exposomics runs)
- Pooled-QC acquisitions containing tandem MS (MS2) spectra, including SWATH-DIA data
- Optionally blank samples for blank.subtraction via FilterVariable parameters

## Outputs

- feature_table.csv with detected/quantified MS1 features
- Deconvoluted MS2 spectra with reconstructed fragment-to-precursor associations (DDA and DIA)
- Feature table annotated downstream with compound identities via MS2 reference database search

## How to apply

Retrieve the deposited raw LC-MS/MS spectral files for the study samples plus the pooled-QC acquisitions that carry tandem MS. Run MetaboAnalyst 6.0's Spectra Processing module (locally via MetaboAnalystR 4.0 in R > 3.6.1, or the web server) with the asari algorithm to perform auto-optimized LC-MS1 peak detection and quantification on the raw spectra; the README notes optional preprocessing helpers such as CentroidCheck, Empty MS scan removal, MS data centroiding, and Signal Drift Correction before feature detection. For data-independent acquisition (DIA) data, deconvolute the MS2 spectra collected from pooled-QC samples to reconstruct fragment-to-precursor associations (MetaboAnalystR 4.0 supports both DDA and DIA deconvolution). The module outputs a feature table (e.g. feature_table.csv) of detected MS1 features ready for downstream MS2 peak annotation against the ~1.5 million MS2 spectra database and for statistical analysis. Validate by comparing regenerated feature counts against published benchmarks, e.g. the islet case study's 27,209 MS1 features with 4,959 carrying a named compound identity after annotation.

## Related tools

- **MetaboAnalyst 6.0** (Unified platform whose enhanced Spectra Processing module performs LC-MS1 peak detection/quantification and MS2 deconvolution)
- **asari** (Algorithm invoked by the Spectra Processing module for LC-MS spectra processing (MS1 feature detection and quantification))
- **MetaboAnalystR 4.0** (R package reproducing the web server's spectra-processing workflow locally, including OptiLCMS-based feature detection and MS/MS deconvolution for DDA and DIA) — https://github.com/xia-lab/MetaboAnalystR
- **OptiLCMS** (R dependency installed from GitHub supporting the local LC-MS feature detection workflow) — https://github.com/xia-lab/OptiLCMS
- **R** (Runtime environment (version > 3.6.1 recommended) for the local MetaboAnalystR pipeline)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE); library(MetaboAnalystR)
```

## Evaluation signals

- Regenerated feature_table.csv must match reported benchmark counts — e.g. 27,209 MS1 features and 4,959 features with at least one named chemical identity for the human islet case study
- Empty MS scans removed and spectra centroided before feature detection (CentroidCheck should pass on converted files)
- DIA MS2 deconvolution yields fragment-to-precursor associations for pooled-QC spectra rather than unassigned fragments
- Detected features can be matched against the ~1.5 million MS2 reference spectra database, with identified features carrying at least one potential named compound identity
- Benchmarks expect >10% more high-quality MS and MS/MS features and >40% true-positive chemical identification rate relative to prior workflows

## Limitations

- MS2 spectra are typically only available from pooled-QC samples, so compound identification depends on those acquisitions being present
- DIA data require explicit deconvolution to reconstruct precursor associations; skipping this step prevents reliable annotation
- MetaboAnalyst authors note metabolome annotations still need improvement, and installation requires system libraries (libcairo2-dev, libnetcdf-dev, libxml2, libxt-dev, libssl-dev on Linux) with compatibility of the latest R versions still under evaluation

## Evidence

- [abstract] Spectra processing with asari + MS2 deconvolution: "a significantly enhanced Spectra Processing module with support for MS2 data and the asari algorithm"
- [intro] asari for LC-MS spectra processing: "A significantly enhanced spectra processing workflow with the addition of asari algorithm for LC–MS spectra processing"
- [intro] DIA deconvolution rationale: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [intro] Pooled QC MS2 collection: "To assist with compound identification, tandem MS (called MS/MS or MS2) spectra are often collected from pooled QC samples during the experiments"
- [methods] R runtime and install: "R base with version > 3.6.1 is recommended."
- [methods] MS2 spectral database scale: "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [readme] README: three key features of MetaboAnalystR 4.0: "1) an auto-optimized feature detection and quantification module for LC-MS1 spectra processing; 2) a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent"
- [readme] README: benchmark claim: "Serial dilutions demonstrate that MetaboAnalystR 4.0 can accurately detect and identify > 10% more high-quality MS and MS/MS features."
- [results] Task card benchmark finding (via source card): [evidence span withheld — anchor status: not_found]
- [discussion] Workflow scope statement: "The workflow spans from raw MS spectra processing to compound identification to functional analysis"
