---
name: gnps-matching-peak-scoring
description: Use when dereplicating preprocessed MS2 .mzML query spectra against GNPS,
  HMDB, and MassBank spectral libraries and scoring candidate metabolite matches per
  LC-MS2 feature.
when_to_use_negative:
- Input is not tandem MS2 spectral data (.mzML) needing spectral-library dereplication
  — e.g. when the goal is compound-structure annotation via compound databases, where
  SIRIUS/MetFrag dereplication or RDKit candidate selection is the relevant step
- Query spectra are absent or poorly represented in the three public spectral libraries
  — the article notes there is a large mismatch between experimental spectra in spectral
  databases and structures in compound databases, leading to a low number of annotated
  features
- You need deterministic candidate-ranking thresholds — the article only established
  score gates for the spectral-database segment, not for downstream in silico annotation
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Spectra
  role: R package that integrates the spectral databases and provides the cosine similarity
    (dot product) spectral-matching metric and preprocessing for query and reference
    MS2 spectra
  repo: https://rformassspectrometry.github.io/Spectra/
- name: MAW-R
  role: R segment of MAW implementing the spectral database dereplication workflow,
    including GNPS-score matching and MAW-defined mz_score/int_score metrics
  repo: https://github.com/zmahnoor14/MAW
- name: maw-r
  role: Docker image (zmahnoor/maw-r:1.0.8) providing the reproducible containerized
    environment in which the Spectra-based dereplication runs
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:f33da8016e4397d200d4b890c7353e7c7279d7367837735603502e2deedbb4ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAW-R
  - Spectra
  - maw-r
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gnps-matching-peak-scoring

## Summary

Spectral database dereplication step of MAW-R that matches query MS2 spectra (.mzML) against GNPS, HMDB, and MassBank spectral libraries inside the R Spectra framework, scoring GNPS candidates with the GNPS altered matching-peak similarity score (threshold 0.85) and HMDB/MassBank candidates with the unaltered dot-product cosine similarity (threshold 0.70), gated by MAW-defined mz_score/int_score metrics at a 0.5 threshold.

## When to use

Use when dereplicating preprocessed MS2 .mzML query spectra against GNPS, HMDB, and MassBank spectral libraries and scoring candidate metabolite matches per LC-MS2 feature.

## When NOT to use

- Input is not tandem MS2 spectral data (.mzML) needing spectral-library dereplication — e.g. when the goal is compound-structure annotation via compound databases, where SIRIUS/MetFrag dereplication or RDKit candidate selection is the relevant step
- Query spectra are absent or poorly represented in the three public spectral libraries — the article notes there is a large mismatch between experimental spectra in spectral databases and structures in compound databases, leading to a low number of annotated features
- You need deterministic candidate-ranking thresholds — the article only established score gates for the spectral-database segment, not for downstream in silico annotation

## Inputs

- Query MS2 spectra in .mzML format
- gnps.rda spectral database R-object (Zenodo record 7519270)
- hmdb.rda spectral database R-object (Zenodo record 7519270)
- mbankNIST.rda spectral database R-object (Zenodo record 7519270)
- Precursor m/z values per LC-MS2 feature

## Outputs

- spectral_results.csv with per-feature GNPS/HMDB/MassBank match entries in the spectral_dereplication folder
- Per-database spectral match directories

## How to apply

Load the deposited spectral database R-objects gnps.rda, hmdb.rda, and mbankNIST.rda (Zenodo record 7519270) plus the query .mzML file into the Spectra package running in the maw-r Docker container (zmahnoor/maw-r:1.0.8, linux/amd64). Preprocess each MS2 spectrum by normalizing peak intensities and removing peaks below 5% relative intensity. Match against GNPS using the GNPS altered matching-peak similarity score accepted at a 0.85 threshold, and against HMDB and MassBank with the unaltered dot-product cosine similarity at a 0.70 threshold, constraining candidates by precursor matching within 15 ppm. Compute the MAW-defined mz_score and int_score metrics for matched candidates and retain only those with a matching score of at least 0.5 in addition to high cosine similarity. Write per-feature dereplication results and per-database match directories to the spectral_dereplication folder as spectral_results.csv.

## Related tools

- **Spectra** (R package that integrates the spectral databases and provides the cosine similarity (dot product) spectral-matching metric and preprocessing for query and reference MS2 spectra) — https://rformassspectrometry.github.io/Spectra/
- **MAW-R** (R segment of MAW implementing the spectral database dereplication workflow, including GNPS-score matching and MAW-defined mz_score/int_score metrics) — https://github.com/zmahnoor14/MAW
- **maw-r** (Docker image (zmahnoor/maw-r:1.0.8) providing the reproducible containerized environment in which the Spectra-based dereplication runs) — https://github.com/zmahnoor14/MAW

## Examples

```
docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64
```

## Evaluation signals

- spectral_results.csv exists and contains per-feature GNPS/HMDB/MassBank match entries
- All GNPS matches meet the 0.85 altered matching-peak similarity threshold; HMDB and MassBank matches meet the 0.70 dot-product cosine threshold
- Precursor agreement between query and library spectra falls within 15 ppm
- Retained candidates satisfy the 0.50 MAW-metric (mz_score/int_score) threshold combined with high cosine similarity scores
- Per-database match directories are present under the spectral_dereplication folder

## Limitations

- The MAW-defined 0.5 matching-score gate combined with high cosine similarity can miss candidates with low cosine similarity scores as false negatives
- Cosine similarity metrics can fail under differing fragmentation patterns or peak shifts; the article concludes that spectral matching needs better metrics
- Public spectral databases are not completely standardized and can misannotate (e.g. betaine annotated as reduced-state trimethylglycine), and only spectra present in the databases can be annotated

## Evidence

- [results] GNPS matches use a scoring threshold of 0.85, while HMDB and MassBank spectra are compared with a dot product with a default threshold of 0.70.: [evidence span withheld — anchor status: not_found]
- [discussion] For GNPS, this similarity score is altered as described in the method section and the spectral matching with GNPS gives better results: "For GNPS, this similarity score is altered as described in the method section and the spectral matching with GNPS gives better results"
- [discussion] Spectra uses the most common spectral matching metric which is the cosine similarity score ... For HMDB and MassBank, this metric is used unaltered: [evidence span withheld — anchor status: not_found]
- [discussion] All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [methods] Download the databases gnps.rda, hmdb.rda, and mbankNIST.rda from https://zenodo.org/record/7519270: "Download the databases gnps.rda, hmdb.rda, and mbankNIST.rda from https://zenodo.org/record/7519270"
- [abstract] The databases are integrated via the R package Spectra: "The databases are integrated via the R package Spectra"
- [methods] docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64: "docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64"
