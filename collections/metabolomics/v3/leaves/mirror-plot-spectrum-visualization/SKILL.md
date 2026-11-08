---
name: mirror-plot-spectrum-visualization
description: Use when consensus/deconvolved MS2 spectra have been searched against
  reference databases and you need visual verification of top-scoring compound matches,
  or when compiling annotated features for a QC case study.
when_to_use_negative:
- Input is raw MS1 spectra or a pre-built feature/peak table rather than deconvolved
  MS2 spectra with precursor m/z and retention time — the skill presupposes the MS2
  Peak Annotation search step has already been run.
- Only one spectrum or one compound needs to be visualized with no reference library
  to match against — there is nothing to overlay in a mirror plot.
- The goal is statistical analysis, pathway analysis, or mGWAS/dose-response modeling
  — this skill is purely for MS2 peak annotation verification, not downstream analysis
  modules.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnalyst 6.0
  role: Web/CLI platform providing the MS2 Peak Annotation module, reference databases,
    and mirror-plot visualization of spectrum matches
  repo: https://www.metaboanalyst.ca
- name: MetaboAnalystR 4.0
  role: R package implementing the same MS2 search and mirror-plot workflow locally;
    the R command history downloaded from MetaboAnalyst reproduces the web results
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:116242da53b26a2cc84a6612d0eec66dbd6d8fd83d3a347d92aa76f098148428
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Mirror-plot spectrum visualization

## Summary

Generates mirror plots that overlay each deconvolved query MS2 spectrum against its best-matching reference spectrum from MetaboAnalyst 6.0's MS2 Peak Annotation module, providing visual verification of compound identifications produced by the spectral search. It pairs a compound identification table with inspection-ready overlay figures.

## When to use

Use when consensus/deconvolved MS2 spectra have been searched against reference databases and you need visual verification of top-scoring compound matches, or when compiling annotated features for a QC case study.

## When NOT to use

- Input is raw MS1 spectra or a pre-built feature/peak table rather than deconvolved MS2 spectra with precursor m/z and retention time — the skill presupposes the MS2 Peak Annotation search step has already been run.
- Only one spectrum or one compound needs to be visualized with no reference library to match against — there is nothing to overlay in a mirror plot.
- The goal is statistical analysis, pathway analysis, or mGWAS/dose-response modeling — this skill is purely for MS2 peak annotation verification, not downstream analysis modules.

## Inputs

- Deconvolved/consensus MS2 spectra (MS2 data from pooled QC acquisitions, with precursor m/z and retention time associations reconstructed during DDA or DIA deconvolution)
- MetaboAnalystR bundled spectra databases (~1.5 million MS2 spectra)
- MS2 reference spectra with fragment-level annotations

## Outputs

- Compound identification table (feature identifier, precursor m/z, retention time, matched compound name, database accession, similarity score) exported as compound_identification_table.csv
- Mirror plots overlaying each query MS2 spectrum against its best-matching reference spectrum for top-scoring annotations

## How to apply

First, run the MS2 Peak Annotation search in MetaboAnalystR 4.0 (R base > 3.6.1): load the deconvolved/consensus MS2 spectra — reconstructed during DDA/DIA deconvolution so each spectrum carries its precursor m/z and retention time — and search them against the bundled spectra databases (~1.5 million MS2 spectra) bundled with MetaboAnalystR. Candidate matches are scored by spectral similarity (dot-product or spectral entropy), constrained by precursor ion agreement, yielding a compound identification table with feature identifier, precursor m/z, retention time, matched compound name, database accession, and similarity score; export it as compound_identification_table.csv. Then generate mirror plots for the top-scoring annotations, overlaying the query MS2 peaks against the matched reference MS2 peaks so shared fragment-level annotations can be inspected visually. Judge success by whether each annotated feature's mirror plot shows matching precursor m/z and consistent fragment alignment, and whether the identification coverage is consistent with MetaboAnalystR 4.0's reported gain — > 40% true-positive-rate improvement without increased false identifications.

## Related tools

- **MetaboAnalyst 6.0** (Web/CLI platform providing the MS2 Peak Annotation module, reference databases, and mirror-plot visualization of spectrum matches) — https://www.metaboanalyst.ca
- **MetaboAnalystR 4.0** (R package implementing the same MS2 search and mirror-plot workflow locally; the R command history downloaded from MetaboAnalyst reproduces the web results) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
R: library(MetaboAnalystR); SearchMS2PeakAnnotation(mSet, spectra_db = 'MoNA', similarity = 'dot_product', min_sim = 0.7, out_dir = 'mirror_plots/')
```

## Evaluation signals

- Every annotated feature in compound_identification_table.csv must carry a matched compound name, database accession, and similarity score — no partially filled rows.
- Mirror plot must show the query spectrum's precursor m/z matching the reference spectrum's precursor m/z; mismatched precursors indicate a false annotation.
- Fragment-level annotations in the mirror plot should align shared fragments between query and reference; sparse overlap signals a low-confidence match.
- Benchmark expectation: identification coverage should reflect the reported > 40% true-positive-rate improvement for DDA and DIA datasets without increased false identifications.
- The module is reported to detect and identify > 10% more high-quality MS and MS/MS features, so the annotation count should be consistent with this coverage gain.

## Limitations

- Requires MS2 spectra that have been deconvolved from DIA data to reconstruct associations with precursor ions; unprocessed DIA spectra will not map correctly.
- The compound identification is scored as a potential named chemical identity — the module's annotation coverage is bounded by the bundled reference databases and knowledgebases.
- Windows users need Rtools; Linux users need libcairo2-dev, libnetcdf-dev, libxml2, libxt-dev, libssl-dev; Mac OS users need Xcode and GNU Fortran for the MetaboAnalystR installation.
- MetaboAnalystR 4.0's compatibility with the latest R version (v4.0.0) is still under evaluation.

## Evidence

- [abstract] MS2 peak annotation via reference-database search produces mirror plots: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
- [intro] MS2 spectra from pooled QC aid compound identification: "To assist with compound identification, tandem MS (called MS/MS or MS2) spectra are often collected from pooled QC samples"
- [readme] MetaboAnalystR ships ~1.5 million MS2 reference spectra: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [readme] MetaboAnalystR's MS/MS deconvolution and compound annotation for DDA/DIA underpins this skill: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
- [readme] MetaboAnalystR improves chemical identification coverage and accuracy: "MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications"
- [intro] DIA data require deconvolution to reconstruct precursor associations: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [readme] MetaboAnalystR synchronized with web server for reproducibility: "users will be able to reproduce the same results from their local computers using the corresponding R command history downloaded from MetaboAnalyst"
