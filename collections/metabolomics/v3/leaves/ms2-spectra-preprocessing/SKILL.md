---
name: ms2-spectra-preprocessing
description: Use when starting the MAW workflow on raw untargeted tandem MS data in
  .mzML format and you need MS2 pre-processing plus spectral dereplication outputs
  that feed SIRIUS or MetFrag candidate generation.
when_to_use_negative:
- Input is not raw MS2 .mzML data (e.g. an already-annotated feature table or MS1-only
  data) — MAW expects tandem mass spectrometry input files in R
- Compounds of interest are absent from GNPS/HMDB/MassBank, since spectral dereplication
  can only annotate spectra present in these public databases
- You need deterministic ranking thresholds on SIRIUS output — the SIRIUS CLI does
  not provide a deterministic probability score, so thresholding was discarded in
  MAW
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0153
tools:
- name: Spectra
  role: R package used inside MAW-R for spectral database dereplication (cosine similarity
    matching against GNPS/HMDB/MassBank)
  repo: https://rformassspectrometry.github.io/Spectra/
- name: MAW-R
  role: R segment of MAW that performs MS2 preprocessing and spectral dereplication,
    emitting spectral_results.csv and the SIRIUS input table
  repo: https://github.com/zmahnoor14/MAW
- name: maw-r
  role: Docker image (zmahnoor/maw-r:1.0.8, linux/amd64) providing the containerized
    environment to run MAW-R
- name: SIRIUS
  role: Downstream consumer of the MS1DATA_SiriusP.tsv / .ms preprocessing outputs
    for in silico annotation
  repo: https://bio.informatik.uni-jena.de/software/sirius/
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:c43d347f3842089f679492f511cdadaade4a628460f243de3d9b3940474158ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAW-R
  - MetFrag
  - SIRIUS
  - Spectra
  - maw-py
  - maw-r
  - maw-sirius5-old
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# ms2-spectra-preprocessing

## Summary

MAW's MS2 input data pre-processing and spectral database dereplication stage: untargeted LC-MS/MS .mzML files are processed in R (MAW-R) using the Spectra package against GNPS, HMDB, and MassBank to produce dereplication results and SIRIUS input files that seed downstream in silico annotation.

## When to use

Use when starting the MAW workflow on raw untargeted tandem MS data in .mzML format and you need MS2 pre-processing plus spectral dereplication outputs that feed SIRIUS or MetFrag candidate generation.

## When NOT to use

- Input is not raw MS2 .mzML data (e.g. an already-annotated feature table or MS1-only data) — MAW expects tandem mass spectrometry input files in R
- Compounds of interest are absent from GNPS/HMDB/MassBank, since spectral dereplication can only annotate spectra present in these public databases
- You need deterministic ranking thresholds on SIRIUS output — the SIRIUS CLI does not provide a deterministic probability score, so thresholding was discarded in MAW

## Inputs

- Untargeted LC-MS2 data in .mzML format (e.g. diatom standards or bryophytes datasets)
- Spectral database objects gnps.rda, hmdb.rda, mbankNIST.rda (Zenodo record 7519270; GNPS snapshot from 2023-01-09)

## Outputs

- spectral_results.csv (spectral dereplication results per feature)
- MS1DATA_SiriusP.tsv (SIRIUS input table) with .ms files under insilico/SIRIUS/no_isotope
- MS1DATA.csv consumed by the MAW-Py post-processing script

## How to apply

Pull the maw-r Docker image (e.g. zmahnoor/maw-r:1.0.8 with --platform=linux/amd64) and mount a working directory containing the input .mzML files plus the spectral database objects gnps.rda, hmdb.rda, and mbankNIST.rda downloaded from Zenodo record 7519270. Inside the container, run the MAW-R R script with the .mzML file, the three .rda databases, a 15 ppm tolerance for spectral database dereplication precursor matches, and TRUE for collision energy. This performs the workflow's first two stages — MS2 data preprocessing and spectral database dereplication — using cosine similarity (unaltered for HMDB and MassBank, altered for GNPS, where matching gives better results). The run outputs spectral_results.csv and the SIRIUS input table MS1DATA_SiriusP.tsv with .ms files under insilico/SIRIUS/no_isotope, which are consumed by the SIRIUS5 container (Run_Sirius.r with the 'orbitrap' profile) and the MAW-Py post-processing script. Rationale: spectral dereplication first ranks compounds already present in public spectral databases, so only features without spectral hits need computationally expensive in silico annotation.

## Related tools

- **Spectra** (R package used inside MAW-R for spectral database dereplication (cosine similarity matching against GNPS/HMDB/MassBank)) — https://rformassspectrometry.github.io/Spectra/
- **MAW-R** (R segment of MAW that performs MS2 preprocessing and spectral dereplication, emitting spectral_results.csv and the SIRIUS input table) — https://github.com/zmahnoor14/MAW
- **maw-r** (Docker image (zmahnoor/maw-r:1.0.8, linux/amd64) providing the containerized environment to run MAW-R)
- **SIRIUS** (Downstream consumer of the MS1DATA_SiriusP.tsv / .ms preprocessing outputs for in silico annotation) — https://bio.informatik.uni-jena.de/software/sirius/

## Examples

```
docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64
```

## Evaluation signals

- spectral_results.csv exists and contains per-feature spectral matches within the 15 ppm precursor-mass tolerance
- MS1DATA_SiriusP.tsv plus .ms files are generated under insilico/SIRIUS/no_isotope and are parseable by Run_Sirius.r
- On the nine diatom standards, the dereplication-driven pipeline reproduces the benchmark: 6 of 9 standards rank-1 (8 of 9 identified overall), isovaleryl-l-carnitine ranked 2, cysteinolic acid ranked 3
- MAW-defined matching metrics meet at least a 0.5 matching score in addition to a high cosine similarity score
- For annotated features, spectral matches to GNPS, HMDB, and MassBank support MSI-level 2 evidence in the merged results

## Limitations

- Cosine similarity fails under differing fragmentation patterns or peak shifts; spectral matching needs better metrics
- The MAW-defined 0.5 threshold reduces false positives but can miss candidates with low cosine similarity scores as false negatives
- Public spectral databases are not completely standardized — e.g. betaine was only annotated as its reduced form trimethylglycine
- A large mismatch between experimental spectra in spectral databases and structures in compound databases leads to a low number of annotated features

## Evidence

- [abstract] The databases are integrated via the R package Spectra: "The databases are integrated via the R package Spectra"
- [intro] workflow stages: "MAW performs (1) MS2 data preprocessing, (2) spectral database dereplication, (3) compound database dereplication, (4) candidate selection, and (5) chemical compound classification"
- [methods] database download: "Download the databases gnps.rda, hmdb.rda, and mbankNIST.rda from https://zenodo.org/record/7519270"
- [methods] container pull: "docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64"
- [readme] README input/output: "The workflow takes MS2 .mzML format data files as an input in R. It performs spectral database dereplication using R Package"
- [discussion] GNPS metric: "For GNPS, this similarity score is altered as described in the method section and the spectral matching with GNPS gives better results"
