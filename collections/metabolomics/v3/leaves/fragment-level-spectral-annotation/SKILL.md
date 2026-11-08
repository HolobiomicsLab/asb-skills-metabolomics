---
name: fragment-level-spectral-annotation
description: Use when MS2 spectra from DDA or SWATH-DIA LC-MS runs (often pooled QC
  samples) have been deconvoluted into consensus spectra and need fragment-level compound
  assignments against a reference spectral library.
when_to_use_negative:
- Input is already an MS1-only feature table (peaks/intensities) with no tandem MS
  spectra — the MS2 Peak Annotation module requires deconvoluted MS2 data
- MS2 spectra have not yet been deconvoluted, e.g. raw SWATH-DIA chimeric spectra
  — DIA data require deconvolution to reconstruct precursor associations before annotation
- A targeted assay where compound identities are already known and only quantification
  is needed
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnalyst 6.0
  role: Web platform providing the MS2 Peak Annotation module with fragment-level
    annotation against comprehensive MS2 reference databases
- name: MetaboAnalystR 4.0
  role: R implementation of the MS/MS spectra deconvolution and compound annotation
    module (DecoID for DDA, DecoMetDIA for DIA) with bundled ~1.5 million MS2 spectra
    databases for local large-scale processing
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:7b70f987c8da9ed1245f304af4aef0bf3d78886acc4773844b31a4bfa3976136
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# fragment-level-spectral-annotation

## Summary

Annotate deconvoluted MS2 consensus spectra at the fragment level against MetaboAnalyst's comprehensive MS2 reference databases (~1.5 million spectra) to assign compounds and improve identification coverage in untargeted metabolomics.

## When to use

Use when MS2 spectra from DDA or SWATH-DIA LC-MS runs (often pooled QC samples) have been deconvoluted into consensus spectra and need fragment-level compound assignments against a reference spectral library.

## When NOT to use

- Input is already an MS1-only feature table (peaks/intensities) with no tandem MS spectra — the MS2 Peak Annotation module requires deconvoluted MS2 data
- MS2 spectra have not yet been deconvoluted, e.g. raw SWATH-DIA chimeric spectra — DIA data require deconvolution to reconstruct precursor associations before annotation
- A targeted assay where compound identities are already known and only quantification is needed

## Inputs

- Replicate DDA or SWATH-DIA MS2 spectra files from pooled QC samples (raw LC-MS/MS acquisitions)
- MetaboAnalystR-bundled MS2 reference spectra database (~1.5 million spectra) and knowledgebases (~500,000 metabolite set entries)

## Outputs

- consensus_spectra.csv containing precursor-association and fragment-level compound-annotation fields
- Consensus MS2 spectra with fragment-to-precursor associations and compound assignments per fragment

## How to apply

First deconvolute chimeric MS2 spectra: for DDA acquisitions apply a DecoID-style database-dependent sparse regression correlating co-eluting fragment ion traces against reference library spectra to assign fragments to precursor ions; for SWATH-DIA use the DecoMetDIA algorithm (re-written in C++ via Rcpp) correlating extracted-ion chromatograms within each isolation window. Merge deconvoluted spectra across replicate injections into consensus spectra by clustering fragments on m/z and retention time and averaging intensities. Then use the MetaboAnalyst 6.0 MS2 Peak Annotation module (via MetaboAnalystR 4.0 in R) to annotate each consensus-spectrum fragment against the bundled ~1.5 million MS2 reference spectra database, producing fragment-level compound assignments. Rationale: incorporating MS2 processing and annotation into untargeted workflows greatly improves compound annotations and functional interpretation, and the paper reports >40% higher true-positive identification rates for DDA and DIA data without increasing false identifications.

## Related tools

- **MetaboAnalyst 6.0** (Web platform providing the MS2 Peak Annotation module with fragment-level annotation against comprehensive MS2 reference databases)
- **MetaboAnalystR 4.0** (R implementation of the MS/MS spectra deconvolution and compound annotation module (DecoID for DDA, DecoMetDIA for DIA) with bundled ~1.5 million MS2 spectra databases for local large-scale processing) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
install.packages("devtools"); library(devtools); devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T)
```

## Evaluation signals

- consensus_spectra.csv exists and contains precursor-association and compound-annotation fields
- Expert review confirms deconvoluted spectra of known reference compounds match their library spectra
- Every consensus-spectrum fragment carries a fragment-level annotation against the reference MS2 spectral library
- Benchmark expectation: >10% more high-quality MS and MS/MS features detected and identified in serial dilutions
- Benchmark expectation: >40% increase in true positive chemical identification rate for DDA and DIA datasets without increased false identifications

## Limitations

- MS2 spectra are typically collected only from pooled QC samples during experiments, limiting annotation to spectra acquired on those pooled injections
- Metabolome annotations still need improvement and are listed as future work by the authors
- DIA data cannot be annotated directly — deconvolution is required first to reconstruct associations with precursor ions

## Evidence

- [abstract] MS2 Peak Annotation module purpose: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
- [intro] Benefit of MS2 annotation in untargeted workflows: "Incorporating MS2 processing and annotation into untargeted metabolomics workflows can greatly improve compound annotations and functional interpretation."
- [intro] DIA deconvolution prerequisite: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [readme] MetaboAnalystR annotation module scope: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA);"
- [readme] Reference spectra database size: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
