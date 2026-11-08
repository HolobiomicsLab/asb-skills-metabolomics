---
name: swath-dia-ms2-deconvolution
description: Use when raw LC-MS/MS files contain DIA/SWATH MS2 scans (typically acquired
  on pooled-QC samples) and feature-level compound annotation requires associating
  fragment ions to precursor MS1 features before searching MS2 reference spectra.
when_to_use_negative:
- Input is already a processed feature table or peak list — use downstream statistical/annotation
  modules instead of spectra deconvolution
- MS2 spectra were acquired in DDA mode, where precursor associations are recorded
  at acquisition time and deconvolution is unnecessary
- The analysis goal is targeted metabolomics quantification rather than untargeted
  LC-MS feature detection and MS2 annotation
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnalyst 6.0
  role: Web platform providing the enhanced Spectra Processing module (asari-based
    MS1 peak detection plus DIA MS2 deconvolution) and MS2 Peak Annotation
- name: asari
  role: LC-MS1 peak detection and quantification algorithm invoked upstream of MS2
    deconvolution within the Spectra Processing module
- name: MetaboAnalystR 4.0
  role: R package exposing the same MS/MS spectra deconvolution and compound annotation
    functions locally for reproducible pipelines
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:0286bc455f975a73ea16461f21bf0490e52921ea8fa124dbb88d90148e1aaf49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  - asari
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# SWATH-DIA MS2 deconvolution

## Summary

Deconvolution of data-independent acquisition (DIA/SWATH) MS2 spectra collected from pooled-QC samples to reconstruct fragment-to-precursor ion associations, followed by MS2-based compound annotation, as implemented in MetaboAnalyst 6.0's Spectra Processing workflow with the asari algorithm.

## When to use

Use when raw LC-MS/MS files contain DIA/SWATH MS2 scans (typically acquired on pooled-QC samples) and feature-level compound annotation requires associating fragment ions to precursor MS1 features before searching MS2 reference spectra.

## When NOT to use

- Input is already a processed feature table or peak list — use downstream statistical/annotation modules instead of spectra deconvolution
- MS2 spectra were acquired in DDA mode, where precursor associations are recorded at acquisition time and deconvolution is unnecessary
- The analysis goal is targeted metabolomics quantification rather than untargeted LC-MS feature detection and MS2 annotation

## Inputs

- Raw LC-MS/MS2 spectral files (SWATH-DIA acquisitions, e.g. mzML/centroided raw scans) from pooled-QC samples
- Raw LC-MS1 spectral files from biological samples (high/low-glucose islet exposures in the case study)
- MetaboAnalyst MS2 reference spectra database (~1.5 million MS2 spectra)

## Outputs

- feature_table.csv of detected MS1 features with quantification
- Deconvoluted MS2 spectra with reconstructed fragment-to-precursor associations
- Feature-to-compound identity annotations against the ~1.5M MS2 reference spectra database

## How to apply

Retrieve the deposited raw LC-MS/MS spectral files (e.g. mzML/centroided vendor conversions) and first run the MetaboAnalyst 6.0 Spectra Processing module with the asari algorithm to perform LC-MS1 peak detection and quantification. Then deconvolute the DIA MS2 spectra from the pooled-QC acquisitions to reconstruct associations between fragment ions and their precursor features, using MetaboAnalystR 4.0's streamlined MS/MS spectra deconvolution module (local R commands mirror the web server's command history for reproducibility). Annotate the deconvoluted MS2 features against the bundled spectra database of ~1.5 million MS2 reference spectra to assign named compound identities. Decision point: DDA data do not require this deconvolution step, whereas DIA data generally require it to reconstruct precursor associations. In the paper's islet case study this pipeline detected 27,209 MS1 features, of which 4,959 were identified with at least one potential named chemical identity.

## Related tools

- **MetaboAnalyst 6.0** (Web platform providing the enhanced Spectra Processing module (asari-based MS1 peak detection plus DIA MS2 deconvolution) and MS2 Peak Annotation)
- **asari** (LC-MS1 peak detection and quantification algorithm invoked upstream of MS2 deconvolution within the Spectra Processing module)
- **MetaboAnalystR 4.0** (R package exposing the same MS/MS spectra deconvolution and compound annotation functions locally for reproducible pipelines) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- Regenerated feature_table.csv matches the reported benchmark of 27,209 detected MS1 features for the islet case-study data
- Count of features with at least one named chemical identity matches the reported 4,959 within tolerance of the deposited reference
- Each deconvoluted MS2 spectrum carries a valid fragment-to-precursor association to an asari-detected MS1 feature
- Annotation assignments are traceable to entries in the ~1.5 million MS2 reference spectra database
- Local MetaboAnalystR output reproduces the MetaboAnalyst web server results given the same R command history

## Limitations

- DIA deconvolution adds computational overhead and is only required because DIA MS2 data lack precursor associations; DDA data bypass this step
- Metabolome annotations remain imperfect — the authors state 'we will continue to improve metabolome annotations' as future work
- Reproduction depends on deposited raw files and pooled-QC SWATH-DIA acquisitions being present; feature counts are only benchmarks for the specific case-study data

## Evidence

- [intro] DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [intro] MS2 from pooled QC assist compound identification: "To assist with compound identification, tandem MS (called MS/MS or MS2) spectra are often collected from pooled QC samples during the experiments"
- [abstract] MS2 Peak Annotation against reference databases with fragment-level annotation: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
- [readme] MetaboAnalystR 4.0 deconvolution and annotation module for DDA or DIA: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
- [methods] ~1.5 million MS2 spectra knowledgebase in MetaboAnalystR: "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
