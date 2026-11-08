---
name: ms2-spectral-library-annotation
description: Use when raw LC-MS/MS (DDA or DIA) spectra have been processed to features
  and you need compound-level annotation of MS2 peaks via spectral library matching.
when_to_use_negative:
- Input is already a curated feature table with compound IDs — go straight to downstream
  statistical or functional analysis modules
- Only MS1 data are available (no MS2 spectra collected) — MS2 Peak Annotation requires
  tandem-MS spectra
- Targeted metabolomics with a predefined compound panel — use targeted processing
  rather than untargeted MS2 library matching
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnalyst 6.0
  role: Web platform hosting the MS2 Peak Annotation module with comprehensive MS2
    reference databases and fragment-level annotation
- name: MetaboAnalystR 4.0
  role: R package exposing the same MS2 annotation functions locally, synchronized
    with the MetaboAnalyst web server
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: asari
  role: LC-MS1 spectra processing algorithm that precedes MS2 annotation in the MetaboAnalyst
    6.0 workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:04a54d6ceaa9f64e059627390f51682e626a86994399e61178cc2ceb64097342
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# ms2-spectral-library-annotation

## Summary

Annotate deconvoluted MS2 peaks against MetaboAnalyst 6.0's MS2 reference spectra databases to assign compound identities to features, a key step in MetaboAnalyst 6.0's enhanced untargeted metabolomics workflow.

## When to use

Use when raw LC-MS/MS (DDA or DIA) spectra have been processed to features and you need compound-level annotation of MS2 peaks via spectral library matching.

## When NOT to use

- Input is already a curated feature table with compound IDs — go straight to downstream statistical or functional analysis modules
- Only MS1 data are available (no MS2 spectra collected) — MS2 Peak Annotation requires tandem-MS spectra
- Targeted metabolomics with a predefined compound panel — use targeted processing rather than untargeted MS2 library matching

## Inputs

- Raw LC-MS/MS spectral files (mzML/mzData, DDA or DIA) from samples plus pooled-QC MS2 acquisitions
- Deconvoluted MS2 peak lists from the Spectra Processing module (feature_table.csv)

## Outputs

- Compound annotations for MS2 features (compound names/mapping against the MS2 reference library)
- Count of features with at least one named chemical identity

## How to apply

First run MetaboAnalyst 6.0's Spectra Processing module (asari for LC-MS1 peak detection and quantification, plus MS2 deconvolution for pooled-QC DIA data) to obtain a feature table with associated deconvoluted MS2 spectra. Then apply the MS2 Peak Annotation module, which matches experimental MS2 spectra against the package's spectra database of ~1.5 million MS2 reference spectra to assign compound identities at fragment level. MetaboAnalystR 4.0 exposes the same functionality in R via functions accessible through the MetaboAnalyst web server's command history. Correct application is judged by the number of features assigned at least one named chemical identity (benchmark: 4,959 of 27,209 features in the human islet case study) and by the increased true-positive identification rate reported (>40% for DDA and DIA without more false IDs).

## Related tools

- **MetaboAnalyst 6.0** (Web platform hosting the MS2 Peak Annotation module with comprehensive MS2 reference databases and fragment-level annotation)
- **MetaboAnalystR 4.0** (R package exposing the same MS2 annotation functions locally, synchronized with the MetaboAnalyst web server) — https://github.com/xia-lab/MetaboAnalystR
- **asari** (LC-MS1 spectra processing algorithm that precedes MS2 annotation in the MetaboAnalyst 6.0 workflow)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE)
```

## Evaluation signals

- Number of features with at least one named chemical identity — benchmark ~4,959 of 27,209 in the islet case study
- True-positive rate of chemical identification should increase by >40% for both DDA and DIA without increasing false identifications
- Deconvoluted MS2 features should successfully reconstruct precursor-ion associations for pooled-QC DIA data before annotation
- Feature table regenerated through the asari-based pipeline should match the deposited reference (27,209 MS1 features, 4,959 named)

## Limitations

- MS2 annotation relies on the quality of pooled-QC MS2 spectra; poor or missing pooled-QC acquisitions reduce identifiability
- DIA data require deconvolution before annotation; skipping this step produces incorrect precursor-MS2 associations
- The article's future work notes metabolome annotations still need improvement, indicating annotation coverage is incomplete

## Evidence

- [abstract] MS2 Peak Annotation module matches deconvoluted MS2 peaks against MS2 reference libraries: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
- [readme] MS2 spectra database size in MetaboAnalystR 4.0: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [intro] Pooled-QC collection of MS2 spectra to support compound identification: "tandem MS (called MS/MS or MS2) spectra are often collected from pooled QC samples during the experiments"
- [abstract] MS2 spectra and deconvolution workflow in MetaboAnalyst 6.0: "a significantly enhanced Spectra Processing module with support for MS2 data and the asari algorithm"
- [results] Benchmark counts from the islet case study: [evidence span withheld — anchor status: not_found]
- [intro] Deconvolution requirement for DIA data: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
