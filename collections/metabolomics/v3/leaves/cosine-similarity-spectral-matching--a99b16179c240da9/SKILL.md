---
name: cosine-similarity-spectral-matching
description: Use when dereplicating untargeted LC-MS2 .mzML features against public
  MS2 spectral libraries (GNPS, HMDB, MassBank) and you need library-level metabolite
  annotations with per-match similarity scores.
when_to_use_negative:
- Query spectra have fragmentation patterns or peak shifts that break cosine similarity
  — the authors note cosine metrics fail here and better spectral matching metrics
  are needed
- No library spectrum exists for the feature — public spectral databases are not standardized
  and can only annotate spectra present in them, so compound database dereplication
  (SIRIUS) is the alternative route
- Input is already annotated candidates needing ranking/selection — that is final
  candidate selection via RDKit in MAW-Py, not spectral matching
tools:
- name: Spectra
  role: R package that integrates the spectral databases and provides the cosine similarity
    metric and comparison infrastructure for spectral matching
  repo: https://rformassspectrometry.github.io/Spectra/
- name: MAW-R
  role: Runs the spectral database dereplication segment (preprocessing, GNPS/HMDB/MassBank
    matching) inside the maw-r Docker container
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:101d8f99e9fb32c975e145fbb38c24b56d1d5403c5afdeec940b407537774649
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAW-R
  - Spectra
  - maw-r
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cosine-similarity-spectral-matching

## Summary

MAW's spectral database dereplication scheme: preprocess MS2 .mzML query spectra and pre-built spectral library R-objects (GNPS, HMDB, MassBank) with the R Spectra package, then match candidates using cosine/dot-product similarity — the GNPS-altered score at a 0.85 threshold and the unaltered dot product for HMDB/MassBank at 0.70 — plus MAW-defined mz_score/int_score metrics.

## When to use

Use when dereplicating untargeted LC-MS2 .mzML features against public MS2 spectral libraries (GNPS, HMDB, MassBank) and you need library-level metabolite annotations with per-match similarity scores.

## When NOT to use

- Query spectra have fragmentation patterns or peak shifts that break cosine similarity — the authors note cosine metrics fail here and better spectral matching metrics are needed
- No library spectrum exists for the feature — public spectral databases are not standardized and can only annotate spectra present in them, so compound database dereplication (SIRIUS) is the alternative route
- Input is already annotated candidates needing ranking/selection — that is final candidate selection via RDKit in MAW-Py, not spectral matching

## Inputs

- Query MS2 spectra in .mzML format
- Spectral library R-objects gnps.rda, hmdb.rda, mbankNIST.rda (Zenodo record 7519270)
- Feature (precursor m/z / retention time) table from MS2 preprocessing

## Outputs

- spectral_dereplication/spectral_results.csv with per-feature GNPS/HMDB/MassBank matches
- Per-database match directories
- Retained candidate annotations passing the 0.50 MAW-metric threshold with high cosine similarity

## How to apply

Load the deposited spectral library R-objects (gnps.rda, hmdb.rda, mbankNIST.rda from Zenodo record 7519270) and the query .mzML file with the Spectra package inside the maw-r Docker container. Normalize peak intensities and remove peaks with relative intensity below 5% for each MS2 spectrum, query and library alike. Match against GNPS with the GNPS-altered matching-peak similarity score (threshold 0.85) and against HMDB and MassBank with the unaltered dot-product cosine similarity (default threshold 0.70), requiring precursor agreement at 15 ppm. Compute MAW-defined mz_score and int_score metrics and retain candidates with a matching score of at least 0.5 in addition to high cosine similarity; write per-feature results plus per-database match directories to spectral_dereplication/spectral_results.csv.

## Related tools

- **Spectra** (R package that integrates the spectral databases and provides the cosine similarity metric and comparison infrastructure for spectral matching) — https://rformassspectrometry.github.io/Spectra/
- **MAW-R** (Runs the spectral database dereplication segment (preprocessing, GNPS/HMDB/MassBank matching) inside the maw-r Docker container) — https://github.com/zmahnoor14/MAW

## Examples

```
docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64
```

## Evaluation signals

- spectral_results.csv exists in the spectral_dereplication folder and contains per-feature GNPS, HMDB, and MassBank match entries
- GNPS matches meet the 0.85 altered-score threshold; HMDB/MassBank dot-product matches meet the 0.70 threshold with precursor matching at 15 ppm
- Retained candidates satisfy the MAW-defined metrics (mz_score, int_score) of at least 0.5 combined with high cosine similarity
- Query and library spectra were both preprocessed (normalized intensities, peaks <5% relative intensity removed) before comparison

## Limitations

- Cosine similarity fails under differing fragmentation patterns or peak shifts; the authors conclude spectral matching needs better metrics
- The MAW-defined 0.5 threshold can turn candidates with low cosine similarity into false negatives
- Public spectral databases are not completely standardized — e.g. betaine was annotated as reduced-state trimethylglycine — and only spectra present in the libraries can be annotated
- Large mismatch between experimental spectra in spectral databases and structures in compound databases limits the number of annotated features

## Evidence

- [discussion] Cosine/dot-product rationale: [evidence span withheld — anchor status: not_found]
- [discussion] MAW metric threshold: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [methods] Library data provenance: "Download the databases gnps.rda, hmdb.rda, and mbankNIST.rda from https://zenodo.org/record/7519270"
- [discussion] Known metric limitation: "We conclude that spectral matching needs better metrics."
- [readme] Role within MAW: "It performs spectral database dereplication using R Package"
