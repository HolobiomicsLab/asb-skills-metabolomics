---
name: precursor-mass-ppm-filtering
description: Use when dereplicating untargeted LC-MS2 features (.mzML input) against
  public spectral libraries (GNPS, HMDB, MassBank) inside MAW-R and you need reproducible
  precursor-ppm and similarity-score filtering of candidates.
when_to_use_negative:
- You are doing compound-structure candidate selection or chemical classification
  (MAW-Py/RDKit, SIRIUS, Tanimoto similarity), not spectral library dereplication.
- Input is not tandem MS2 data in .mzML format (e.g., already a feature table or MS1-only
  peaks).
- You need annotation against compound structure databases such as PubChem/COCONUT
  via MetFrag — MAW v1 integrates only SIRIUS for compound databases and spectral
  dereplication covers GNPS/HMDB/MassBank.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Spectra
  role: R package used to load and preprocess MS2 spectra and perform spectral matching
    against GNPS, HMDB, and MassBank
  repo: https://github.com/zmahnoor14/MAW
- name: MAW-R
  role: R segment of MAW executing the spectral database dereplication module with
    its scoring thresholds
  repo: https://github.com/zmahnoor14/MAW
- name: maw-r
  role: Docker image (zmahnoor/maw-r:1.0.8) in which the dereplication steps run reproducibly
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:235afcbf3c6ec4a01326b0eef6ebeced7aaeaed8e7b0377a3e8acdf459be2a7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
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
yes_rate: 0.7
n_claims: 10
---

# precursor-mass-ppm-filtering

## Summary

MAW's spectral database dereplication step: it matches MS2 query spectra (from .mzML) against GNPS, HMDB, and MassBank spectral libraries in R/Spectra, gating candidate matches by precursor m/z at 15 ppm and similarity thresholds (GNPS 0.85, HMDB/MassBank cosine 0.70, MAW metrics ≥ 0.50).

## When to use

Use when dereplicating untargeted LC-MS2 features (.mzML input) against public spectral libraries (GNPS, HMDB, MassBank) inside MAW-R and you need reproducible precursor-ppm and similarity-score filtering of candidates.

## When NOT to use

- You are doing compound-structure candidate selection or chemical classification (MAW-Py/RDKit, SIRIUS, Tanimoto similarity), not spectral library dereplication.
- Input is not tandem MS2 data in .mzML format (e.g., already a feature table or MS1-only peaks).
- You need annotation against compound structure databases such as PubChem/COCONUT via MetFrag — MAW v1 integrates only SIRIUS for compound databases and spectral dereplication covers GNPS/HMDB/MassBank.

## Inputs

- MS2 .mzML input data file(s)
- gnps.rda GNPS spectral database R-object (Zenodo record 7519270)
- hmdb.rda HMDB spectral database R-object (Zenodo record 7519270)
- mbankNIST.rda MassBank/NIST spectral database R-object (Zenodo record 7519270)

## Outputs

- spectral_results.csv with per-feature GNPS/HMDB/MassBank match entries
- per-database spectral match directories written to the spectral_dereplication folder

## How to apply

Run inside the maw-r Docker container (zmahnoor/maw-r:1.0.8): load the query .mzML plus the deposited database R-objects gnps.rda, hmdb.rda, and mbankNIST.rda (Zenodo record 7519270) via the R Spectra package. Preprocess every MS2 spectrum by normalizing peak intensities and removing peaks below 5% relative intensity. Match precursors at a 15 ppm tolerance, scoring candidates against GNPS with the GNPS altered matching-peak similarity score (threshold 0.85) and against HMDB and MassBank with the unaltered dot-product cosine similarity (threshold 0.70). Then compute MAW's mz_score and int_score metrics and retain only candidates with a matching score of at least 0.5 in addition to high cosine similarity — this dual gate reduces false positives but may discard candidates with low cosine similarity. Write per-feature results to spectral_results.csv in the spectral_dereplication folder.

## Related tools

- **Spectra** (R package used to load and preprocess MS2 spectra and perform spectral matching against GNPS, HMDB, and MassBank) — https://github.com/zmahnoor14/MAW
- **MAW-R** (R segment of MAW executing the spectral database dereplication module with its scoring thresholds) — https://github.com/zmahnoor14/MAW
- **maw-r** (Docker image (zmahnoor/maw-r:1.0.8) in which the dereplication steps run reproducibly) — https://github.com/zmahnoor14/MAW

## Examples

```
docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64
```

## Evaluation signals

- spectral_results.csv exists in the spectral_dereplication folder and contains per-feature GNPS/HMDB/MassBank match entries.
- Retained candidates satisfy the 0.50 MAW-metric (mz_score/int_score) threshold combined with high cosine similarity scores.
- Precursor masses of matched candidates agree within 15 ppm of the query feature precursor.
- GNPS hits meet the 0.85 altered-score threshold; HMDB and MassBank hits meet the 0.70 dot-product cosine threshold.
- Spectra were preprocessed (normalized intensities, peaks below 5% relative intensity removed) before matching.

## Limitations

- The MAW-defined 0.5 threshold may miss candidates with low cosine similarity as false negatives even when they are correct.
- Cosine similarity metrics fail under differing fragmentation patterns or peak shifts; the authors conclude spectral matching needs better metrics.
- Public spectral databases are not fully standardized and can only annotate spectra present in them (e.g., betaine annotated as reduced-state trimethylglycine).
- There is a large mismatch between spectra in spectral databases and structures in compound databases, limiting annotated features.

## Evidence

- [discussion] All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [discussion] Spectra uses the most common spectral matching metric which is the cosine similarity score: "Spectra uses the most common spectral matching metric which is the cosine similarity score"
- [methods] Download the databases gnps.rda, hmdb.rda, and mbankNIST.rda from Zenodo: "Download the databases gnps.rda, hmdb.rda, and mbankNIST.rda from https://zenodo.org/record/7519270"
- [readme] It performs spectral database dereplication using R Package Spectra: "It performs spectral database dereplication using R Package [Spectra]"
