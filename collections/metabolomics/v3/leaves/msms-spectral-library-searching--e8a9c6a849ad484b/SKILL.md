---
name: msms-spectral-library-searching
description: Use when consensus MS2 spectra already associated with precursor m/z
  and retention time (e.g. deconvolved DIA data or DDA pooled QC spectra) must be
  annotated with named chemical identities via spectral library matching.
when_to_use_negative:
- Input is raw MS1 spectra still needing peak detection/quantification — use the MetaboAnalyst
  spectra processing module (asari/OptiLCMS) first, not library searching
- MS2 spectra from DIA data have not yet been deconvolved to reconstruct precursor
  ion associations — deconvolution must precede library searching
- The goal is fragment-level structural annotation without requiring whole-spectrum
  library matches to named compounds — the MS2 Peak Annotation module's scoring is
  designed for library matches
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_0092
tools:
- name: MetaboAnalystR 4.0
  role: R implementation of the MS2 spectra deconvolution, compound annotation, and
    spectral library search workflow executed locally
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalyst 6.0
  role: Web platform hosting the MS2 Peak Annotation module based on comprehensive
    MS2 reference databases with fragment-level annotation
- name: R
  role: Runtime environment (R base > 3.6.1 recommended) for MetaboAnalystR library
    searching
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:ed19052062a8e3e279f4413a4b2cc88d7c415b492334da140db157772369ba7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# msms-spectral-library-searching

## Summary

Search deconvolved/consensus MS2 spectra (from DDA or DIA pooled QC acquisitions) against MetaboAnalyst 6.0's comprehensive MS2 reference databases (~1.5 million MS2 spectra) to produce named compound identifications with similarity scores and mirror plots.

## When to use

Use when consensus MS2 spectra already associated with precursor m/z and retention time (e.g. deconvolved DIA data or DDA pooled QC spectra) must be annotated with named chemical identities via spectral library matching.

## When NOT to use

- Input is raw MS1 spectra still needing peak detection/quantification — use the MetaboAnalyst spectra processing module (asari/OptiLCMS) first, not library searching
- MS2 spectra from DIA data have not yet been deconvolved to reconstruct precursor ion associations — deconvolution must precede library searching
- The goal is fragment-level structural annotation without requiring whole-spectrum library matches to named compounds — the MS2 Peak Annotation module's scoring is designed for library matches

## Inputs

- Deconvolved/consensus MS2 spectra with reconstructed precursor m/z and retention time associations (DDA or DIA, e.g. from pooled QC acquisitions)
- MetaboAnalystR bundled spectra databases (~1.5 million MS2 spectra) and knowledgebases (~500,000 entries of metabolite sets)

## Outputs

- Compound identification table (feature identifier, precursor m/z, retention time, matched compound name, database accession, similarity score), e.g. compound_identification_table.csv
- Mirror plots overlaying query MS2 spectra against best-matching reference spectra for top-scoring annotations

## How to apply

Load the deconvolved consensus MS2 spectra — with precursor ion associations reconstructed during the DDA/DIA deconvolution step — into MetaboAnalystR (R base > 3.6.1 recommended). Search each consensus MS2 spectrum against the bundled spectra databases (~1.5 million MS2 spectra) that accompany the MS2 Peak Annotation module, which supports fragment-level annotation. Score candidates with spectral similarity (dot-product style scoring) constrained by precursor ion agreement, and retain the best-scoring named compound per feature. Assemble a compound identification table containing feature identifier, precursor m/z, retention time, matched compound name, database accession, and similarity score, and export it (e.g. compound_identification_table.csv). For top-scoring annotations, generate mirror plots overlaying each query MS2 spectrum against its best-matching reference spectrum. The value of this step is grounded in the benchmark claim that MS2 annotation can detect and identify > 10% more high-quality MS and MS/MS features and raise the true positive rate of chemical identification by > 40% without increasing false identifications.

## Related tools

- **MetaboAnalystR 4.0** (R implementation of the MS2 spectra deconvolution, compound annotation, and spectral library search workflow executed locally) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalyst 6.0** (Web platform hosting the MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation)
- **R** (Runtime environment (R base > 3.6.1 recommended) for MetaboAnalystR library searching)

## Examples

```
install.packages("devtools"); library(devtools); devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T)
```

## Evaluation signals

- Every annotated feature in the output table receives at least one potential named chemical identity (matched compound name plus database accession)
- Each match is constrained by precursor ion agreement between query and reference entry, not spectral similarity alone
- Annotation coverage should be consistent with the reported benchmark improvement (> 10% more high-quality MS and MS/MS features identified; true positive rate of chemical identification raised by > 40% without increased false identifications)
- Mirror plots for top-scoring annotations visibly overlay query and reference MS2 peaks, allowing visual confirmation of match quality
- Compound identification table contains all required fields: feature identifier, precursor m/z, retention time, matched compound name, database accession, and similarity score

## Limitations

- DIA MS2 data generally require deconvolution first to reconstruct associations with their precursor ions — library searching fails or is misleading without this step
- Identification is limited to entries present in the bundled spectra databases (~1.5 million MS2 spectra); coverage of the metabolome annotations remains imperfect and is stated as future work ('we will continue to improve metabolome annotations')
- MetaboAnalystR compatibility with the latest R versions beyond > 3.6.1 is stated as still under evaluation

## Evidence

- [abstract] MS2 Peak Annotation module based on comprehensive MS2 reference databases: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
- [intro] MS2 annotation improves compound annotations: "Incorporating MS2 processing and annotation into untargeted metabolomics workflows can greatly improve compound annotations and functional interpretation"
- [methods] Bundled MS2 spectra databases size: "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [intro] DIA deconvolution prerequisite: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [readme] DDA/DIA deconvolution and compound annotation module: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
- [readme] Benchmark on identification accuracy: "increase the true positive rate of chemical identification by > 40% without increasing false identifications"
