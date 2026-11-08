---
name: compound-annotation-table-generation
description: Use when consensus MS2 spectra with reconstructed precursor m/z and retention
  time are available and named chemical identities are needed for detected MS1 features.
when_to_use_negative:
- Input is only an MS1 feature table without associated deconvolved consensus MS2
  spectra — no spectral evidence exists for compound identification
- DIA MS2 spectra have not yet been deconvolved to reconstruct precursor-ion associations
- The goal is de novo identification without reference spectra or when confirmed (not
  annotated) compound identities are required
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnalyst 6.0
  role: Provides the MS2 Peak Annotation module that searches consensus MS2 spectra
    against curated MS2 reference databases with fragment-level annotation and produces
    mirror plots
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalystR 4.0
  role: R package implementing the MS/MS spectra deconvolution and compound annotation
    workflow locally, bundling the ~1.5 million MS2 spectra reference databases
  repo: https://github.com/xia-lab/MetaboAnalystR.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:1d8f29657a57c6112482a9c13c1ff5779237b993a45b545767870818ac751479
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# compound-annotation-table-generation

## Summary

Searches deconvolved consensus MS2 spectra against MetaboAnalyst 6.0's curated MS2 reference databases (~1.5 million spectra) using dot-product or spectral-entropy similarity, and assembles a compound identification table plus mirror plots for the best-scoring named compounds per MS1 feature.

## When to use

Use when consensus MS2 spectra with reconstructed precursor m/z and retention time are available and named chemical identities are needed for detected MS1 features.

## When NOT to use

- Input is only an MS1 feature table without associated deconvolved consensus MS2 spectra — no spectral evidence exists for compound identification
- DIA MS2 spectra have not yet been deconvolved to reconstruct precursor-ion associations
- The goal is de novo identification without reference spectra or when confirmed (not annotated) compound identities are required

## Inputs

- Deconvolved/consensus MS2 spectra with precursor m/z and retention time associations (from DDA or DIA pooled-QC acquisitions)
- MetaboAnalystR bundled MS2 reference spectra databases (~1.5 million MS2 spectra)

## Outputs

- compound_identification_table.csv containing feature identifier, precursor m/z, retention time, matched compound name, database accession, and similarity score
- Mirror plots overlaying query MS2 spectra against best-matching reference spectra for top-scoring annotations

## How to apply

Load the deconvolved consensus MS2 spectra into MetaboAnalystR (R base > 3.6.1); spectra must retain precursor m/z and retention-time associations reconstructed during the DDA/DIA deconvolution step. Search each consensus spectrum against the bundled spectra databases (~1.5 million MS2 spectra) using the MS2 Peak Annotation module, scoring candidates with dot-product or spectral entropy similarity constrained by precursor ion agreement, and retain the best-scoring named compound per feature. Assemble the compound identification table with feature identifier, precursor m/z, retention time, matched compound name, database accession, and similarity score, and export it as compound_identification_table.csv. Generate mirror plots overlaying each query MS2 spectrum against its best-matching reference spectrum for the top-scoring annotations. Treat matches as potential (not confirmed) identities, consistent with the module's reported improvement in identification coverage rather than validated identifications.

## Related tools

- **MetaboAnalyst 6.0** (Provides the MS2 Peak Annotation module that searches consensus MS2 spectra against curated MS2 reference databases with fragment-level annotation and produces mirror plots) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalystR 4.0** (R package implementing the MS/MS spectra deconvolution and compound annotation workflow locally, bundling the ~1.5 million MS2 spectra reference databases) — https://github.com/xia-lab/MetaboAnalystR.git

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T)
```

## Evaluation signals

- Every annotated feature receives at least one potential named chemical identity with a similarity score from the dot-product or spectral-entropy search
- The table contains all expected columns: feature identifier, precursor m/z, retention time, matched compound name, database accession, and similarity score
- Matches satisfy precursor ion agreement in addition to spectral similarity
- Annotation coverage is consistent with the reported benchmark of > 10% more high-quality MS and MS/MS features detected and identified, and a > 40% increase in true positive identification rate without increased false identifications
- A mirror plot exists for each top-scoring annotation, overlaying the query spectrum against the best-matching reference spectrum

## Limitations

- Annotations are potential identities from database searching, not confirmed compound identifications
- Identification coverage depends on the reference databases; the article notes metabolome annotations still need improvement as future work
- DIA data must first be deconvolved to reconstruct precursor-ion associations before searching

## Evidence

- [abstract] a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
- [methods] spectra databases (~1.5 million MS2 spectra): "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [readme] streamlined MS/MS spectra deconvolution and compound annotation module for both DDA or DIA: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA);"
- [readme] increase the true positive rate of chemical identification by > 40% without increasing false identifications: "MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications."
- [intro] DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
