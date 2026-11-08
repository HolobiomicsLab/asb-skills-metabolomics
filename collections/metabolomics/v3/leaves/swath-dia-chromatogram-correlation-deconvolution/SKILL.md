---
name: swath-dia-chromatogram-correlation-deconvolution
description: Use when processing SWATH/DIA tandem MS data where MS2 fragments must
  be reassigned to their precursor ions before compound annotation, e.g. pooled-QC
  DIA spectra in untargeted metabolomics.
when_to_use_negative:
- Input is DDA data — use the DecoID-style database-dependent regression deconvolution
  instead of the DecoMetDIA chromatogram-correlation method
- Data are MS1-only (no tandem MS spectra) — deconvolution is not applicable; use
  feature detection/quantification instead
- A feature table already exists and the goal is downstream statistics or pathway
  analysis rather than spectral deconvolution
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnalystR 4.0
  role: Provides the R functions and bundled ~1.5 million MS2 spectra database underlying
    the SWATH-DIA deconvolution and MS2 compound annotation workflow
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalyst 6.0
  role: Web platform exposing the enhanced Spectra Processing module with MS2 deconvolution
    support for DIA data
- name: R
  role: Runtime environment (R base version > 3.6.1 recommended) in which the deconvolution
    is executed via MetaboAnalystR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:f7a75e360239b291dc10c3f95d3778791593d6018e557fff76f36be09096e4be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# swath-dia-chromatogram-correlation-deconvolution

## Summary

Deconvolves chimeric SWATH-DIA MS2 spectra by correlating extracted-ion chromatograms within each isolation window to reconstruct precursor-fragment associations, as implemented in MetaboAnalyst 6.0 / MetaboAnalystR 4.0 via a C++ (Rcpp) re-implementation of the DecoMetDIA algorithm.

## When to use

Use when processing SWATH/DIA tandem MS data where MS2 fragments must be reassigned to their precursor ions before compound annotation, e.g. pooled-QC DIA spectra in untargeted metabolomics.

## When NOT to use

- Input is DDA data — use the DecoID-style database-dependent regression deconvolution instead of the DecoMetDIA chromatogram-correlation method
- Data are MS1-only (no tandem MS spectra) — deconvolution is not applicable; use feature detection/quantification instead
- A feature table already exists and the goal is downstream statistics or pathway analysis rather than spectral deconvolution

## Inputs

- Replicate SWATH-DIA MS2 spectra files (raw tandem MS data)
- MetaboAnalystR-bundled MS2 reference spectra database (~1.5 million spectra)

## Outputs

- consensus_spectra.csv with precursor-association and compound-annotation fields
- Deconvoluted MS2 spectra with reconstructed precursor-fragment associations
- Fragment-level compound annotations against the reference spectral library

## How to apply

Load replicate DIA MS2 spectra files together with the MetaboAnalystR-bundled MS2 reference spectra database (~1.5 million spectra) in R using MetaboAnalystR 4.0. For each SWATH isolation window, the DecoMetDIA-style algorithm (re-written in C++ via Rcpp) correlates extracted-ion chromatograms of co-eluting fragment traces to deconvolute chimeric MS2 spectra and reconstruct precursor-fragment associations. Merge deconvoluted spectra across replicate injections into consensus spectra by clustering fragments on m/z and retention time and averaging intensities. Annotate each consensus spectrum fragment against the reference spectral library to obtain fragment-level compound assignments; validation confirms consensus_spectra.csv contains precursor-association and compound-annotation fields and that deconvoluted spectra of known reference compounds match library spectra.

## Related tools

- **MetaboAnalystR 4.0** (Provides the R functions and bundled ~1.5 million MS2 spectra database underlying the SWATH-DIA deconvolution and MS2 compound annotation workflow) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalyst 6.0** (Web platform exposing the enhanced Spectra Processing module with MS2 deconvolution support for DIA data)
- **R** (Runtime environment (R base version > 3.6.1 recommended) in which the deconvolution is executed via MetaboAnalystR)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T)
```

## Evaluation signals

- consensus_spectra.csv exists and contains precursor-association and compound-annotation fields
- Deconvoluted spectra of known reference compounds match their library spectra (expert-review check)
- Fragments across replicate injections cluster cleanly on m/z and retention time into consensus spectra
- Benchmark expectation: for DDA and DIA datasets, MetaboAnalystR 4.0 increases the true positive rate of chemical identification by > 40% without increasing false identifications

## Limitations

- DIA data generally require deconvolution to reconstruct precursor-ion associations, so raw DIA MS2 spectra cannot be annotated directly
- Metabolome annotations still need improvement, stated as future work in the article's discussion
- The web server and R package must stay synchronized; local results depend on the bundled spectra databases and API service availability

## Evidence

- [readme] README deconvolution scope: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
- [readme] MS2 spectra database size: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [intro] DIA deconvolution rationale: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [readme] Benchmark accuracy claim: "For both DDA and DIA datasets, MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications."
- [abstract] Spectra processing module: "a significantly enhanced Spectra Processing module with support for MS2 data and the asari algorithm"
