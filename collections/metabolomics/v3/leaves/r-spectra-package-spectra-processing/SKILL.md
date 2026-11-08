---
name: r-spectra-package-spectra-processing
description: Use when untargeted MS2 .mzML features must be annotated by spectral
  matching against GNPS, HMDB, or MassBank, requiring threshold-based dereplication
  with cosine similarity scoring.
when_to_use_negative:
- Input is already a feature table or candidate list rather than MS2 .mzML spectra
  requiring spectral matching
- The target compound database annotation step (e.g. SIRIUS/CSI:FingerID compound
  dereplication) is needed instead of spectral database dereplication
- Spectra are not represented in public spectral databases — the article notes the
  workflow 'can only annotate spectra present in the databases'
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Spectra
  role: R package used for MS2 spectrum preprocessing, spectral similarity scoring,
    and spectral database dereplication against GNPS, HMDB, and MassBank
  repo: https://github.com/zmahnoor14/MAW
- name: maw-r
  role: Docker container image (zmahnoor/maw-r:1.0.8) in which the R-based spectral
    dereplication step executes
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:dc8d982f73e89357d9af741a9a21b35136fab5a3244b3fcff3abc07225e7ee06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - Spectra
  - maw-r
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# r-spectra-package-spectra-processing

## Summary

Use the R Spectra package inside the MAW-R workflow to preprocess MS2 .mzML spectra and dereplicate them against GNPS, HMDB, and MassBank spectral databases, scoring matches with cosine/dot-product similarity plus MAW-defined metrics.

## When to use

Use when untargeted MS2 .mzML features must be annotated by spectral matching against GNPS, HMDB, or MassBank, requiring threshold-based dereplication with cosine similarity scoring.

## When NOT to use

- Input is already a feature table or candidate list rather than MS2 .mzML spectra requiring spectral matching
- The target compound database annotation step (e.g. SIRIUS/CSI:FingerID compound dereplication) is needed instead of spectral database dereplication
- Spectra are not represented in public spectral databases — the article notes the workflow 'can only annotate spectra present in the databases'

## Inputs

- MS2 .mzML data file (query spectra)
- gnps.rda spectral database R-object (Zenodo record 7519270)
- hmdb.rda spectral database R-object (Zenodo record 7519270)
- mbankNIST.rda spectral database R-object (Zenodo record 7519270)

## Outputs

- spectral_results.csv with per-feature dereplication results in the spectral_dereplication folder
- Per-database match directories (GNPS, HMDB, MassBank)

## How to apply

Load the deposited spectral database R-objects (gnps.rda, hmdb.rda, mbankNIST.rda from Zenodo record 7519270) and the query .mzML file into R using the Spectra package, running inside the maw-r Docker container (zmahnoor/maw-r:1.0.8). Preprocess each MS2 spectrum by normalizing peak intensities and removing peaks with relative intensity below 5%. Match preprocessed spectra against GNPS using the GNPS-altered matching-peak similarity score with a 0.85 threshold, and against HMDB and MassBank using the unaltered dot-product cosine similarity with a default 0.70 threshold, matching precursors at 15 ppm. Compute the MAW-defined mz_score and int_score metrics for matched candidates and retain only candidates with a matching score of at least 0.5 in addition to high cosine similarity. Write per-feature dereplication results with per-database match directories to the spectral_dereplication folder as spectral_results.csv.

## Related tools

- **Spectra** (R package used for MS2 spectrum preprocessing, spectral similarity scoring, and spectral database dereplication against GNPS, HMDB, and MassBank) — https://github.com/zmahnoor14/MAW
- **maw-r** (Docker container image (zmahnoor/maw-r:1.0.8) in which the R-based spectral dereplication step executes) — https://github.com/zmahnoor14/MAW

## Examples

```
docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64
```

## Evaluation signals

- spectral_results.csv exists and contains per-feature GNPS/HMDB/MassBank match entries
- GNPS matches satisfy the 0.85 score threshold; HMDB and MassBank matches satisfy the 0.70 cosine/dot-product threshold
- Precursor matching was performed at 15 ppm tolerance
- Retained candidates satisfy the 0.50 MAW-metric threshold combined with high cosine similarity scores
- Preprocessed spectra show peaks below 5% relative intensity removed and normalized intensities

## Limitations

- The MAW-defined 0.5 threshold may reduce false positives but can miss candidates with low cosine similarity scores as false negatives
- Public spectral databases are not completely standardized and can only annotate spectra present in the databases (e.g. betaine misannotation case)
- There is a large mismatch between submitted experimental spectra in spectral databases and chemical structures in compound databases, limiting the number of annotated features
- Cosine similarity metrics can fail under differing fragmentation patterns or peak shifts; the authors conclude spectral matching needs better metrics

## Evidence

- [discussion] Spectra uses the most common spectral matching metric which is the cosine similarity score: "Spectra uses the most common spectral matching metric which is the cosine similarity score"
- [discussion] MAW-defined metrics threshold: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [abstract] Databases integrated via Spectra: "The databases are integrated via the R package Spectra"
- [readme] Repository README on input format and Spectra usage: "The workflow takes MS2 .mzML format data files as an input in R. It performs spectral database dereplication using R Package [Spectra]"
