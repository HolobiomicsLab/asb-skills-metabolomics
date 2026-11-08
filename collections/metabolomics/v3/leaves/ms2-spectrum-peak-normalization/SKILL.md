---
name: ms2-spectrum-peak-normalization
description: Use when MS2 spectra in .mzML format must be normalized and low-intensity
  peaks filtered prior to cosine-similarity dereplication against spectral databases
  like GNPS, HMDB, or MassBank.
when_to_use_negative:
- The input is a processed MS1 feature table with no MS2 peak lists — peak-level normalization
  of tandem spectra cannot apply
- The goal is compound database dereplication or structure annotation (SIRIUS/MetFrag/CANOPUS)
  rather than spectral-library matching
- Spectra have already been normalized and peak-filtered to the same 5% relative-intensity
  convention by an upstream step
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Spectra
  role: R package used to load mzML query spectra and the database R-objects, and
    to preprocess MS2 peaks (normalize intensities, remove low-intensity peaks) before
    cosine/dot-product similarity scoring
  repo: https://rformassspectrometry.github.io/Spectra/
- name: MAW-R
  role: R segment of MAW hosting the spectral database dereplication module that applies
    the normalization and scoring thresholds
  repo: https://github.com/zmahnoor14/MAW
- name: maw-r
  role: Docker image in which the Spectra-based preprocessing and dereplication are
    executed for reproducibility
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:4c1ff507d3b7122c3ff449a662e7e8beb2a8ffaf796e25dd3a9e6278db4d77d6
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
yes_rate: 0.8461538461538461
n_claims: 13
---

# ms2-spectrum-peak-normalization

## Summary

MS2 spectrum preprocessing from MAW: normalize peak intensities and remove peaks below 5% relative intensity with the R Spectra package before spectral similarity matching. This conditioning makes cosine/dot-product dereplication against GNPS, HMDB, and MassBank comparable between query and library spectra.

## When to use

Use when MS2 spectra in .mzML format must be normalized and low-intensity peaks filtered prior to cosine-similarity dereplication against spectral databases like GNPS, HMDB, or MassBank.

## When NOT to use

- The input is a processed MS1 feature table with no MS2 peak lists — peak-level normalization of tandem spectra cannot apply
- The goal is compound database dereplication or structure annotation (SIRIUS/MetFrag/CANOPUS) rather than spectral-library matching
- Spectra have already been normalized and peak-filtered to the same 5% relative-intensity convention by an upstream step

## Inputs

- Query MS2 spectra in .mzML format
- Prebuilt spectral database R-objects: gnps.rda, hmdb.rda, mbankNIST.rda (Zenodo record 7519270)
- maw-r Docker container environment with the R Spectra package

## Outputs

- Normalized, peak-filtered MS2 Spectra objects ready for similarity matching
- Per-feature dereplication results written as spectral_results.csv in the spectral_dereplication folder, with GNPS/HMDB/MassBank match directories

## How to apply

Load the query .mzML spectra and the deposited database R-objects (gnps.rda, hmdb.rda, mbankNIST.rda from Zenodo record 7519270) into R using the Spectra package, inside the maw-r Docker container. Normalize peak intensities of each MS2 spectrum and remove peaks whose relative intensity falls below 5% so that query and library spectra share a comparable peak representation. Match the preprocessed spectra with precursor matching at 15 ppm: GNPS uses an altered GNPS matching-peak similarity score accepted at a 0.85 threshold, while HMDB and MassBank use the unaltered dot-product cosine similarity at a 0.70 default threshold. Compute the MAW-defined mz_score and int_score for matched candidates and retain only candidates with a matching score of at least 0.5 in addition to high cosine similarity, then write per-feature results to spectral_dereplication/spectral_results.csv.

## Related tools

- **Spectra** (R package used to load mzML query spectra and the database R-objects, and to preprocess MS2 peaks (normalize intensities, remove low-intensity peaks) before cosine/dot-product similarity scoring) — https://rformassspectrometry.github.io/Spectra/
- **MAW-R** (R segment of MAW hosting the spectral database dereplication module that applies the normalization and scoring thresholds) — https://github.com/zmahnoor14/MAW
- **maw-r** (Docker image in which the Spectra-based preprocessing and dereplication are executed for reproducibility)

## Examples

```
docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64
```

## Evaluation signals

- Retained candidates satisfy the MAW-defined mz_score/int_score matching score of at least 0.5 combined with a high cosine similarity score
- GNPS matches are accepted only at the 0.85 threshold; HMDB and MassBank dot-product matches at the 0.70 threshold
- Precursor matching respects the 15 ppm tolerance
- spectral_results.csv exists in the spectral_dereplication folder and contains per-feature GNPS/HMDB/MassBank match entries
- Normalized spectra no longer contain peaks with relative intensity below 5%

## Limitations

- The MAW-defined 0.5 threshold may miss candidates with low cosine similarity scores as false negatives
- Cosine similarity metrics fail under differing fragmentation patterns or peak shifts, motivating better spectral matching metrics
- Public spectral databases are not completely standardized and can only annotate spectra present in the databases (e.g. betaine annotated as reduced-state trimethylglycine)

## Evidence

- [abstract] Spectra integrates the query and database spectra: "The databases are integrated via the R package Spectra"
- [readme] README input format: "The workflow takes MS2 .mzML format data files as an input in R."
- [discussion] Cosine similarity as the spectral matching metric: "Spectra uses the most common spectral matching metric which is the cosine similarity score"
- [discussion] MAW metric retention threshold: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
