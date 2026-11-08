---
name: tandem-ms-data-preprocessing
description: Use when starting from raw untargeted LC-MS2 .mzML files (e.g. the bryophytes
  MTBLS709 dataset) that must be preprocessed and dereplicated against spectral databases
  before SIRIUS/MAW-Py annotation.
when_to_use_negative:
- Input is already a preprocessed feature table or MS1DATA_SiriusP.tsv output from
  a previous MAW-R run rather than raw .mzML files
- Data is MS1-only or not in .mzML format (the workflow takes MS2 .mzML files as input
  in R)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: MAW-R
  role: R segment of MAW that performs MS2 data preprocessing and spectral database
    dereplication via the Spectra package
  repo: https://github.com/zmahnoor14/MAW
- name: Spectra
  role: R package used for spectral database matching with cosine similarity plus
    MAW-defined metrics against gnps.rda, hmdb.rda, mbankNIST.rda
- name: maw-r
  role: Docker image (zmahnoor/maw-r:1.0.8) pulled with --platform linux/amd64 that
    packages the R preprocessing environment
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:55114ea395fda2dfc3914be450871614a8bff315ae57dcb9ab1b305e691b688b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - MAW-R
  - Spectra
  - maw-py
  - maw-r
  - maw-sirius5-old
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# tandem-ms-data-preprocessing

## Summary

The first MAW stage: pre-process untargeted tandem MS (MS2) .mzML input files in R before spectral and compound database dereplication, converting raw instrument output into per-feature data suitable for SIRIUS-based annotation.

## When to use

Use when starting from raw untargeted LC-MS2 .mzML files (e.g. the bryophytes MTBLS709 dataset) that must be preprocessed and dereplicated against spectral databases before SIRIUS/MAW-Py annotation.

## When NOT to use

- Input is already a preprocessed feature table or MS1DATA_SiriusP.tsv output from a previous MAW-R run rather than raw .mzML files
- Data is MS1-only or not in .mzML format (the workflow takes MS2 .mzML files as input in R)

## Inputs

- MS2 .mzML files (untargeted tandem mass spectrometry data, e.g. bryophytes MTBLS709)
- Spectral reference databases as .rda files: gnps.rda, hmdb.rda, mbankNIST.rda (Zenodo record 7519270)

## Outputs

- Preprocessed per-feature MS1/MS2 data file (MS1DATA_SiriusP.tsv) for downstream SIRIUS runs
- Spectral database dereplication results (matches against GNPS, HMDB, MassBank/NIST)

## How to apply

Run the MAW-R stage (Docker image zmahnoor/maw-r:1.0.8, pulled with --platform linux/amd64) on each MS2 .mzML file using Workflow_R_Script_all_SIRIUS.r. Spectral database dereplication uses the R package Spectra against gnps.rda, hmdb.rda and mbankNIST.rda (downloaded from Zenodo record 7519270), with a 15 ppm precursor tolerance for dereplication and TRUE for collision energy. Match quality is judged by cosine similarity plus MAW-defined metrics, each of which should have at least a 0.5 matching score. The R stage emits MS1DATA_SiriusP.tsv (precursor/MS2 peak lists) that is then consumed by the SIRIUS container (orbitrap profile, ppm_max=5, ppm_max_ms2=15), so correct preprocessing is what determines downstream feature coverage (881/937-style tallies validated at 881/933 annotated features).

## Related tools

- **MAW-R** (R segment of MAW that performs MS2 data preprocessing and spectral database dereplication via the Spectra package) — https://github.com/zmahnoor14/MAW
- **Spectra** (R package used for spectral database matching with cosine similarity plus MAW-defined metrics against gnps.rda, hmdb.rda, mbankNIST.rda)
- **maw-r** (Docker image (zmahnoor/maw-r:1.0.8) pulled with --platform linux/amd64 that packages the R preprocessing environment) — https://github.com/zmahnoor14/MAW

## Examples

```
Rscript Workflow_R_Script_all_SIRIUS.r bryophytes_sample.mzML
```

## Evaluation signals

- MS1DATA_SiriusP.tsv is produced for every input .mzML file and is consumable by the SIRIUS stage (Run_Sirius.r with the orbitrap profile)
- Dereplication used a 15 ppm precursor tolerance and collision energy set to TRUE, matching the validated bryophytes run parameters
- Spectral matches carry cosine similarity scores plus MAW-defined metrics each at least 0.5
- After downstream stages, per-feature annotation tallies reconcile to the published validation (881/933 features with chemical structures, 92 only class+formula, 18 only formula)
- Identical .rda database versions (gnps.rda, hmdb.rda, mbankNIST.rda from Zenodo record 7519270) are used so results are reproducible

## Limitations

- Cosine similarity metrics fail under differing fragmentation patterns or peak shifts; the paper concludes spectral matching needs better metrics
- The MAW-defined 0.5 threshold can miss candidates with low cosine similarity as false negatives
- Public spectral databases are not standardized, so features absent from the databases remain unannotated (e.g. betaine misannotation case)
- There is a large mismatch between experimental spectra in spectral databases and structures in compound databases, leading to a low number of annotated features

## Evidence

- [intro] MAW performs (1) MS2 data preprocessing, (2) spectral database dereplication, (3) compound database dereplication: "MAW performs (1) MS2 data preprocessing, (2) spectral database dereplication, (3) compound database dereplication"
- [readme] readme mzML input and Spectra dereplication: "The workflow takes MS2 .mzML format data files as an input in R. It performs spectral database dereplication using R Package"
- [methods] methods database download: "Download the databases gnps.rda, hmdb.rda, and mbankNIST.rda from https://zenodo.org/record/7519270"
- [discussion] discussion cosine similarity and MAW-defined metrics: [evidence span withheld — anchor status: not_found]
