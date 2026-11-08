---
name: spectral-database-dereplication
description: Use when untargeted LC-MS/MS data in .mzML format needs spectral library
  dereplication against GNPS/HMDB/MassBank as the first annotation stage of MAW, before
  in silico tools like SIRIUS or MetFrag.
when_to_use_negative:
- When the goal is de novo structural annotation of compounds absent from public spectral
  libraries — MAW notes a 'huge mismatch between submitted experimental spectra to
  spectral databases and chemical structures present in compound databases', so use
  in silico tools (SIRIUS/MetFrag) instead
- When input is already a feature table or dereplicated candidate list rather than
  raw MS2 .mzML files
- When the candidate space is natural products needing specialized compound databases
  (e.g. COCONUT with MetFrag) rather than experimental spectral matches
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Spectra
  role: R package used to load .mzML data and the .rda database objects, preprocess
    spectra, and compute cosine/dot-product similarity matches against GNPS, HMDB,
    and MassBank
  repo: https://github.com/zmahnoor14/MAW
- name: MAW-R
  role: R segment of MAW that runs the spectral dereplication module, applies the
    0.85/0.70/0.50 thresholds, and writes spectral_results.csv plus the SIRIUS input
    table
  repo: https://github.com/zmahnoor14/MAW
- name: maw-r
  role: Docker image (zmahnoor/maw-r:1.0.8, pulled with --platform=linux/amd64) in
    which the dereplication R script is executed
  repo: https://github.com/zmahnoor14/MAW
- name: SIRIUS
  role: Downstream in silico annotation tool whose input table (MS1DATA_SiriusP.tsv)
    is generated from this dereplication step
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_007
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:2c50122ea81e64751ac9721f4801f7159cf23b05760ed5264ef5f8b08506ed5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - MAW-R
  - MetFrag
  - SIRIUS
  - Spectra
  - maw-py
  - maw-r
  - maw-sirius5-old
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# spectral-database-dereplication

## Summary

MAW's R-side (MAW-R) spectral database dereplication step, which matches MS2 spectra from .mzML files against GNPS, HMDB, and MassBank via the Spectra package to produce per-feature dereplication results and generate the SIRIUS input table.

## When to use

Use when untargeted LC-MS/MS data in .mzML format needs spectral library dereplication against GNPS/HMDB/MassBank as the first annotation stage of MAW, before in silico tools like SIRIUS or MetFrag.

## When NOT to use

- When the goal is de novo structural annotation of compounds absent from public spectral libraries — MAW notes a 'huge mismatch between submitted experimental spectra to spectral databases and chemical structures present in compound databases', so use in silico tools (SIRIUS/MetFrag) instead
- When input is already a feature table or dereplicated candidate list rather than raw MS2 .mzML files
- When the candidate space is natural products needing specialized compound databases (e.g. COCONUT with MetFrag) rather than experimental spectral matches

## Inputs

- MS2 data in .mzML format
- gnps.rda spectral database R-object (Zenodo record 7519270)
- hmdb.rda spectral database R-object (Zenodo record 7519270)
- mbankNIST.rda spectral database R-object (Zenodo record 7519270)

## Outputs

- spectral_results.csv with per-feature GNPS/HMDB/MassBank dereplication matches
- per-database dereplication match directories
- MS1DATA_SiriusP.tsv and .ms files under insilico/SIRIUS/no_isotope for downstream SIRIUS annotation

## How to apply

Load the deposited spectral database R-objects gnps.rda, hmdb.rda, and mbankNIST.rda (Zenodo record 7519270) and the query .mzML file into R using the Spectra package inside the maw-r Docker container. Preprocess each MS2 spectrum by normalizing peak intensities and removing peaks with relative intensity below 5%. Match spectra against GNPS using the altered GNPS matching-peak similarity score with a 0.85 threshold, and against HMDB and MassBank using the unaltered dot-product cosine similarity at a 0.70 threshold, with precursor matching at 15 ppm. Compute the MAW-defined mz_score and int_score metrics for matched candidates and retain only those with a matching score of at least 0.5 in addition to a high cosine similarity score. Write per-feature results to the spectral_dereplication folder as spectral_results.csv, together with the per-database match directories and the MS1DATA_SiriusP.tsv table that feeds the SIRIUS stage.

## Related tools

- **Spectra** (R package used to load .mzML data and the .rda database objects, preprocess spectra, and compute cosine/dot-product similarity matches against GNPS, HMDB, and MassBank) — https://github.com/zmahnoor14/MAW
- **MAW-R** (R segment of MAW that runs the spectral dereplication module, applies the 0.85/0.70/0.50 thresholds, and writes spectral_results.csv plus the SIRIUS input table) — https://github.com/zmahnoor14/MAW
- **maw-r** (Docker image (zmahnoor/maw-r:1.0.8, pulled with --platform=linux/amd64) in which the dereplication R script is executed) — https://github.com/zmahnoor14/MAW
- **SIRIUS** (Downstream in silico annotation tool whose input table (MS1DATA_SiriusP.tsv) is generated from this dereplication step) — https://github.com/zmahnoor14/MAW

## Examples

```
docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64
```

## Evaluation signals

- spectral_results.csv exists and contains per-feature GNPS/HMDB/MassBank match entries
- Retained candidates satisfy the MAW-metric threshold of at least 0.5 combined with high cosine similarity scores
- Precursor matches respect the 15 ppm tolerance used for spectral database dereplication
- On the 37-feature standards dataset, Spectra spectral matching should reach 75.7% of MAW's annotations (the reported benchmark)
- MS1DATA_SiriusP.tsv with .ms files is produced under insilico/SIRIUS/no_isotope so the SIRIUS stage can run

## Limitations

- The MAW-defined 0.5 threshold may miss candidates with low cosine similarity scores as false negatives, even though it reduces false positives
- Public spectral databases are not completely standardized and can only annotate spectra present in the databases (e.g. betaine annotated as its reduced form, trimethylglycine)
- Cosine similarity metrics fail under differing fragmentation patterns or peak shifts, so spectral matching needs better metrics

## Evidence

- [abstract] The databases are integrated via the R package Spectra: "The databases are integrated via the R package Spectra"
- [discussion] Spectra uses the most common spectral matching metric which is the cosine similarity score ... For HMDB and MassBank, this metric is used unaltered: [evidence span withheld — anchor status: not_found]
- [discussion] For GNPS, this similarity score is altered as described in the method section and the spectral matching with GNPS gives better results: "For GNPS, this similarity score is altered as described in the method section and the spectral matching with GNPS gives better results"
- [discussion] All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [methods] Download the databases gnps.rda, hmdb.rda, and mbankNIST.rda from https://zenodo.org/record/7519270: "Download the databases gnps.rda, hmdb.rda, and mbankNIST.rda from https://zenodo.org/record/7519270"
