---
name: spectral-database-dereplication-ms2
description: Use when processing MS2 .mzML metabolomics data and you need precursor-anchored
  spectral matches to GNPS, HMDB, and MassBank to assign MSI-level 2 annotations (e.g.,
  bryophyte features like M287R318/Nicotiflorin).
when_to_use_negative:
- 'Input lacks MS2 spectra (MS1-only features): spectral matching cannot be performed
  and annotations will rely solely on compound-database dereplication'
- The target spectra are absent from GNPS/HMDB/MassBank — public spectral databases
  are not completely standardized and can only annotate spectra present in them
- 'You expect many candidates with low cosine similarity: MAW''s 0.5 metric threshold
  plus high cosine similarity can yield false negatives for such candidates'
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3272
tools:
- name: MAW-R
  role: R segment of MAW in which Spectra performs spectral database dereplication
    on .mzML input
  repo: https://github.com/zmahnoor14/MAW
- name: Spectra
  role: R package that integrates the spectral databases and computes cosine-similarity
    spectral matching
  repo: https://github.com/zmahnoor14/MAW
- name: maw-r
  role: Docker image (zmahnoor/maw-r:1.0.8) that executes the R dereplication script
    reproducibly
  repo: https://github.com/zmahnoor14/MAW
- name: MetFrag
  role: Downstream compound-database dereplication (COCONUT) whose candidates are
    merged with the spectral matches
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:5c50100f1631915e7fbfad88300ae2a40b7d8a3798610973e0576e6f051984cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - MAW-R
  - MetFrag
  - Spectra
  - maw-r
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# MS2 spectral database dereplication with the Spectra R package (MAW-R)

## Summary

Dereplication of untargeted tandem MS (MS2) spectra against public spectral databases (GNPS, HMDB, MassBank/MassBankEurope via mbankNIST.rda) using the R Spectra package inside the MAW workflow, producing MSI-level-annotated candidate lists per precursor feature.

## When to use

Use when processing MS2 .mzML metabolomics data and you need precursor-anchored spectral matches to GNPS, HMDB, and MassBank to assign MSI-level 2 annotations (e.g., bryophyte features like M287R318/Nicotiflorin).

## When NOT to use

- Input lacks MS2 spectra (MS1-only features): spectral matching cannot be performed and annotations will rely solely on compound-database dereplication
- The target spectra are absent from GNPS/HMDB/MassBank — public spectral databases are not completely standardized and can only annotate spectra present in them
- You expect many candidates with low cosine similarity: MAW's 0.5 metric threshold plus high cosine similarity can yield false negatives for such candidates

## Inputs

- MS2 .mzML file (e.g., bryophytes LC-MS2 dataset)
- gnps.rda spectral database
- hmdb.rda spectral database
- mbankNIST.rda spectral database (MassBank/MassBankEurope)

## Outputs

- spectral_results.csv with cosine-similarity spectral matches per precursor feature
- MSI-level annotations per feature (e.g., precursor m/z 287.065, RT 318.265 s, polarity, spectral match provenance)
- Candidate rows feeding MS1DATA.csv and the final mergedResults CSV

## How to apply

Load the MS2 .mzML input in the MAW-R container (zmahnoor/maw-r:1.0.8, linux/amd64) and run the spectral-database dereplication segment with the pre-built databases gnps.rda, hmdb.rda, and mbankNIST.rda (Zenodo record 7519270, GNPS snapshot dated 2023-01-09). Match spectra with a precursor-mass tolerance of 15 ppm, cosine-similarity spectral matching (with a MAW-altered metric for GNPS and the unaltered cosine metric for HMDB and MassBank), and collision-energy information where enabled. Accept candidates that meet MAW-defined metrics of at least 0.5 matching score in addition to a high cosine similarity score; spectral matches to a database correspond to MSI-level 2 confidence. Results feed the downstream MetFrag/SIRIUS compound-database dereplication and RDKit-based final candidate selection in MAW-Py (MetFrag score threshold 0.75), which merges everything into a mergedResults file per feature (e.g., file_1M287R318ID319 reporting Nicotiflorin, C27H30O15, class Flavonoids at MSI level 3).

## Related tools

- **MAW-R** (R segment of MAW in which Spectra performs spectral database dereplication on .mzML input) — https://github.com/zmahnoor14/MAW
- **Spectra** (R package that integrates the spectral databases and computes cosine-similarity spectral matching) — https://github.com/zmahnoor14/MAW
- **maw-r** (Docker image (zmahnoor/maw-r:1.0.8) that executes the R dereplication script reproducibly) — https://github.com/zmahnoor14/MAW
- **MetFrag** (Downstream compound-database dereplication (COCONUT) whose candidates are merged with the spectral matches) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
docker run --platform=linux/amd64 zmahnoor/maw-r:1.0.8 Rscript Workflow_R_Script_all_MetFrag.r bryophytes.mzML gnps.rda hmdb.rda mbankNIST.rda
```

## Evaluation signals

- Feature row for file_1M287R318ID319 (precursor m/z 287.065, RT 318.265 s) reports Nicotiflorin with formula C27H30O15 and class Flavonoids
- Nicotiflorin annotation is supported by spectral matches to all three databases (GNPS, HMDB, MassBank), corresponding to MSI-level 2
- MAW-defined metrics each reach at least a 0.5 matching score on top of a high cosine similarity score for retained candidates
- spectral_results.csv exists and is consumed by the MAW-Py step to produce a mergedResults CSV containing the feature row
- Database builds match the pinned versions: GNPS snapshot 2023-01-09 and gnps.rda/hmdb.rda/mbankNIST.rda from Zenodo record 7519270

## Limitations

- Cosine similarity metrics fail under differing fragmentation patterns or peak shifts; better spectral matching metrics are needed
- The MAW-defined 0.5 threshold plus high cosine similarity can miss candidates with low cosine similarity scores as false negatives
- Public spectral databases are not completely standardized, leading to misannotation (e.g., betaine annotated as reduced-state trimethylglycine)
- Mismatch between experimental spectra in spectral databases and structures in compound databases leads to a low number of annotated features

## Evidence

- [abstract] The databases are integrated via the R package Spectra: "The databases are integrated via the R package Spectra"
- [intro] MAW performs MS2 data preprocessing, spectral database dereplication, compound database dereplication, candidate selection, and chemical compound classification: "MAW performs (1) MS2 data preprocessing, (2) spectral database dereplication, (3) compound database dereplication, (4) candidate selection, and (5) chemical compound classification"
- [methods] Download the gnps/hmdb/mbankNIST databases from Zenodo record 7519270: "Download the databases gnps.rda, hmdb.rda, and mbankNIST.rda from https://zenodo.org/record/7519270"
- [discussion] MAW-defined metrics threshold of at least 0.5 matching score in addition to high cosine similarity: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [readme] README states spectral dereplication is performed with the Spectra R package on MS2 .mzML input: "The workflow takes MS2 .mzML format data files as an input in R. It performs spectral database dereplication using R Package Spectra"
