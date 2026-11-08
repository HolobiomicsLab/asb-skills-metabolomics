---
name: ms2-spectra-deconvolution
description: Use when raw DDA or SWATH-DIA MS/MS spectra contain co-eluting or chimeric
  fragment ions that must be reassigned to their precursor ions before compound annotation
  or functional interpretation.
when_to_use_negative:
- Input is already a deconvoluted consensus spectra table or a feature intensity matrix
  — the deconvolution step is unnecessary
- Data are MS1-only peak lists with no tandem MS spectra — use the MS1 peak detection/quantification
  module (asari/OptiLCMS) instead
- Targeted MRM/SRM data with one precursor-fragment transition per analyte, where
  no chimeric spectra occur
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnalystR 4.0
  role: R package implementing the streamlined MS/MS spectra deconvolution and compound
    annotation module for DDA and DIA, bundled with the ~1.5M-spectra reference database
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalyst 6.0
  role: Web platform exposing the enhanced Spectra Processing module with auto-optimized
    MS2 deconvolution for DDA (DecoID-derived) and SWATH-DIA (DecoMetDIA-derived)
    data
- name: R
  role: Runtime environment (R base > 3.6.1 recommended) in which MetaboAnalystR deconvolution
    functions execute
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:48308a36c58e6f065d60126673507ddfd53f1adc8049ed46d35946a71bdcd2ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- ms2-spectral-deconvolution
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ms2-spectra-deconvolution

## Summary

Deconvolves contaminated/chimeric MS2 spectra from DDA and SWATH-DIA acquisitions into consensus precursor-associated spectra, using DecoID-style database-dependent regression for DDA and the DecoMetDIA algorithm (Rcpp/C++) for DIA, as implemented in MetaboAnalystR 4.0 / MetaboAnalyst 6.0's Spectra Processing module.

## When to use

Use when raw DDA or SWATH-DIA MS/MS spectra contain co-eluting or chimeric fragment ions that must be reassigned to their precursor ions before compound annotation or functional interpretation.

## When NOT to use

- Input is already a deconvoluted consensus spectra table or a feature intensity matrix — the deconvolution step is unnecessary
- Data are MS1-only peak lists with no tandem MS spectra — use the MS1 peak detection/quantification module (asari/OptiLCMS) instead
- Targeted MRM/SRM data with one precursor-fragment transition per analyte, where no chimeric spectra occur

## Inputs

- Replicate DDA MS2 spectra files (tandem MS, often collected from pooled QC samples)
- Replicate SWATH-DIA MS2 spectra files with isolation-window information
- MetaboAnalystR-bundled MS2 reference spectra database (~1.5 million spectra)

## Outputs

- Deconvoluted MS2 spectra with reconstructed precursor-fragment associations
- Consensus spectra merged across replicate injections (m/z/RT-clustered fragments, averaged intensities)
- Fragment-level compound annotations against the MS2 reference library

## How to apply

Load replicate DDA/DIA MS2 spectra and the MetaboAnalystR-bundled reference database (~1.5 million MS2 spectra) in R with MetaboAnalystR 4.0. For DDA acquisitions, apply a DecoID-style database-dependent regression: correlate co-eluting fragment ion traces and solve a sparse regression against reference library spectra to assign fragments to their precursor ions. For SWATH-DIA acquisitions, run the DecoMetDIA algorithm re-written in C++ via Rcpp, correlating extracted-ion chromatograms within each isolation window to deconvolute chimeric MS2 spectra and reconstruct precursor-fragment associations. Merge deconvoluted spectra across replicate injections into consensus spectra by clustering fragments on m/z/retention time and averaging intensities, then annotate each consensus spectrum fragment against the reference spectral library for fragment-level compound assignments. The rationale is that DIA data generally require deconvolution to reconstruct precursor associations, and that incorporating MS2 processing and annotation into untargeted workflows greatly improves compound annotations.

## Related tools

- **MetaboAnalystR 4.0** (R package implementing the streamlined MS/MS spectra deconvolution and compound annotation module for DDA and DIA, bundled with the ~1.5M-spectra reference database) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalyst 6.0** (Web platform exposing the enhanced Spectra Processing module with auto-optimized MS2 deconvolution for DDA (DecoID-derived) and SWATH-DIA (DecoMetDIA-derived) data)
- **R** (Runtime environment (R base > 3.6.1 recommended) in which MetaboAnalystR deconvolution functions execute)

## Examples

```
install.packages('devtools'); library(devtools); devtools::install_github('xia-lab/MetaboAnalystR', build = TRUE, build_vignettes = TRUE, build_manual =T)
```

## Evaluation signals

- confirm consensus_spectra.csv exists and contains precursor-association and compound-annotation fields
- verify via expert review that deconvoluted spectra of known reference compounds match their library spectra
- benchmark against the README claim that MetaboAnalystR 4.0 increases the true positive rate of chemical identification by >40% for DDA and DIA datasets without increasing false identifications
- check that serial dilutions detect >10% more high-quality MS and MS/MS features relative to baseline processing
- confirm fragment ions from co-eluting/isolation-window contaminants are no longer attributed to the wrong precursor in the deconvoluted output

## Limitations

- DDA deconvolution is database-dependent (DecoID-style), so compounds absent from the ~1.5 million-spectra reference library cannot be deconvolved against references
- SWATH-DIA deconvolution depends on correlating extracted-ion chromatograms within each isolation window; poorly resolved co-elution within a window may leave residual chimericity
- The article lists improving metabolome annotations as future work, indicating current annotation coverage is imperfect

## Evidence

- [readme] a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA): "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
- [readme] MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra): "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [intro] DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [intro] Incorporating MS2 processing and annotation into untargeted metabolomics workflows can greatly improve compound annotations and functional interpretation: "Incorporating MS2 processing and annotation into untargeted metabolomics workflows can greatly improve compound annotations and functional interpretation."
- [readme] MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications: "For both DDA and DIA datasets, MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications."
