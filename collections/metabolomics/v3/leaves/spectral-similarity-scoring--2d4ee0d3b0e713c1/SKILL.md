---
name: spectral-similarity-scoring
description: Use when SWATH-DIA pooled QC consensus MS2 spectra (with reconstructed
  precursor m/z and retention time) need named compound annotations, and a compound
  identification table with similarity scores is the deliverable.
when_to_use_negative:
- Input MS2 spectra still require deconvolution — DIA data must first have precursor-ion
  associations reconstructed before library searching
- Goal is MS1 feature detection/quantification from raw spectra rather than compound
  identification from already-derived MS2 spectra
- Only MS1 peak tables are available with no tandem MS data — spectral similarity
  scoring cannot be applied without MS2 spectra
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR
  role: R package providing the MS2 peak annotation/compound identification module,
    spectra databases, and similarity-scoring functions used to build the compound
    identification table
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalyst 6.0
  role: Web platform whose MS2 Peak Annotation module searches consensus MS2 spectra
    against curated MS2 reference databases and produces the identification table
    with downloadable mirror plots
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:72b70b9d3c7657c31fc6ad30dc0add62ae68b432b635dfa14ef9242af4b242e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - ProteoWizard
  - R
  - mzR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6521739130434783
n_claims: 23
---

# spectral-similarity-scoring

## Summary

Score deconvolved consensus MS2 spectra against curated reference spectra databases (~1.5 million MS2 spectra) bundled with MetaboAnalyst 6.0/MetaboAnalystR 4.0 to assign named compound identities to detected MS1 features, using spectral similarity (e.g. dot-product style) scoring constrained by precursor ion agreement.

## When to use

Use when SWATH-DIA pooled QC consensus MS2 spectra (with reconstructed precursor m/z and retention time) need named compound annotations, and a compound identification table with similarity scores is the deliverable.

## When NOT to use

- Input MS2 spectra still require deconvolution — DIA data must first have precursor-ion associations reconstructed before library searching
- Goal is MS1 feature detection/quantification from raw spectra rather than compound identification from already-derived MS2 spectra
- Only MS1 peak tables are available with no tandem MS data — spectral similarity scoring cannot be applied without MS2 spectra

## Inputs

- Deconvolved/consensus MS2 spectra from SWATH-DIA pooled QC acquisitions, with precursor m/z and retention time associations reconstructed
- MetaboAnalystR knowledgebases (~500,000 entries of metabolite sets) and bundled spectra databases (~1.5 million MS2 spectra)

## Outputs

- compound_identification_table.csv with feature identifier, precursor m/z, retention time, matched compound name, database accession, and similarity score
- Mirror plots overlaying each query MS2 spectrum against its best-matching reference spectrum for top-scoring annotations

## How to apply

Load the deconvolved/consensus MS2 spectra into MetaboAnalystR 4.0 under R base > 3.6.1, ensuring each spectrum carries its precursor m/z and retention time reconstructed during the DIA deconvolution step. Search each consensus MS2 spectrum against the bundled spectra databases (~1.5 million MS2 spectra) that accompany the MS2 Peak Annotation module. Score candidate matches using spectral similarity scoring (dot-product style) restricted to candidates whose precursor ion agrees, and retain the best-scoring named compound per feature. Assemble the results into a compound identification table (feature identifier, precursor m/z, retention time, compound name, database accession, similarity score) and export it as compound_identification_table.csv; mirror plots overlaying query and best-matching reference spectra support the top annotations.

## Related tools

- **MetaboAnalystR** (R package providing the MS2 peak annotation/compound identification module, spectra databases, and similarity-scoring functions used to build the compound identification table) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalyst 6.0** (Web platform whose MS2 Peak Annotation module searches consensus MS2 spectra against curated MS2 reference databases and produces the identification table with downloadable mirror plots)

## Examples

```
install.packages("devtools"); library(devtools); devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- Every annotated feature in compound_identification_table.csv carries a named chemical identity, a database accession, and a similarity score
- Best-matching candidates agree with the query's reconstructed precursor m/z before similarity scoring is applied
- Annotation coverage is consistent with the reported improvement: > 10% more high-quality MS and MS/MS features detected and identified
- True positive rate of chemical identification increases by > 40% relative to baseline without an increase in false identifications
- Mirror plots visually overlay query and reference peaks for the top-scoring annotations, supporting manual verification of matches

## Limitations

- Scoring quality depends on correct DIA deconvolution; DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions
- Coverage is bounded by the bundled spectra databases (~1.5 million MS2 spectra); the authors note metabolome annotations still need improvement as future work
- MetaboAnalystR 4.0 requires R base > 3.6.1 and compilation dependencies (e.g. libcairo2-dev, libnetcdf-dev on Ubuntu; Xcode and GNU Fortran on Mac OS)

## Evidence

- [methods] MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server"
- [intro] DIA deconvolution prerequisite: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [readme] Benchmark coverage claim: "MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications"
- [readme] Spectra databases size: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
