---
name: massive-public-dataset-acquisition
description: Use when the analysis requires re-running public untargeted metabolomics
  data identified by MassIVE accessions (MSV IDs), such as reproducing reported drug
  detection frequencies from probe-drug studies.
when_to_use_negative:
- The data are already locally available as an extracted feature table or annotated
  MS/MS spectra, so re-downloading the raw public runs adds nothing
- The goal is immediate clinical decision-making — the article states the GNPS Drug
  Library workflow is 'not intended for immediate clinical decision-making'
- The study is a targeted clinical assay (e.g., LC-MRM) rather than untargeted metabolomics,
  since the downstream spectral library search expects untargeted MS/MS spectra of
  chromatographic peaks
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Public repository from which the untargeted metabolomics datasets (MSV accessions)
    are retrieved for retroactive drug-screening re-analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:dc4ea7b596add9bd3ba5b09d709658114fcd6985b2aeb1807d9b7cf61f74ac80
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# massive-public-dataset-acquisition

## Summary

Retrieve published untargeted metabolomics datasets from public GNPS/MassIVE repositories so they can be re-analyzed retroactively, e.g., for GNPS Drug Library-based drug exposure screening.

## When to use

Use when the analysis requires re-running public untargeted metabolomics data identified by MassIVE accessions (MSV IDs), such as reproducing reported drug detection frequencies from probe-drug studies.

## When NOT to use

- The data are already locally available as an extracted feature table or annotated MS/MS spectra, so re-downloading the raw public runs adds nothing
- The goal is immediate clinical decision-making — the article states the GNPS Drug Library workflow is 'not intended for immediate clinical decision-making'
- The study is a targeted clinical assay (e.g., LC-MRM) rather than untargeted metabolomics, since the downstream spectral library search expects untargeted MS/MS spectra of chromatographic peaks

## Inputs

- MassIVE dataset accession (MSV ID) for a published untargeted metabolomics study
- Sample metadata (biofluid, time point, cohort group) associated with the deposited runs

## Outputs

- Raw vendor or mzML MS/MS files for each sample in the public dataset
- Locally organized dataset directory keyed by accession and sample group (e.g., probe-drug cocktail n=14, diphenylamine/diphenhydramine n=10)

## How to apply

Identify the MassIVE accession (e.g., MSV000085944, MSV000084008, or MSV000082493) for the untargeted metabolomics runs of interest, which in this study contain the diphenhydramine (n=10) and caffeine/midazolam/omeprazole cocktail (n=14) probe-drug datasets. Download the raw vendor or mzML files from the MassIVE repository, preserving the per-sample metadata (biofluid, time point) needed to stratify later analyses. Because these are untargeted runs, the downloaded files then feed molecular feature extraction to yield MS/MS spectra of chromatographic peaks suitable for spectral library search against the GNPS Drug Library. Rationale: the paper's approach enables retroactive drug-exposure determination from public untargeted metabolomics data, so correct acquisition of the original raw data is the prerequisite for any re-analysis.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Public repository from which the untargeted metabolomics datasets (MSV accessions) are retrieved for retroactive drug-screening re-analysis)

## Evaluation signals

- All expected MSV accessions (MSV000085944, MSV000084008, MSV000082493) resolve and download without missing sample files
- Sample counts match the study design: n=10 for the diphenhydramine study and n=14 for the caffeine/midazolam/omeprazole cocktail study
- Downloaded files are loadable/convertible to mzML and yield MS/MS spectra amenable to spectral library search
- After downstream re-analysis, computed drug detection frequencies (e.g., midazolam 46% vs 69% in plasma, omeprazole 0% vs 21.4% in feces) reproduce the published values within reporting resolution
- Per-sample hit lists and computed frequencies are written to drug_detection_frequencies.csv with any deviant drug-matrix cell flagged

## Limitations

- The deposited data are untargeted metabolomics runs; they require conversion or feature extraction before library search, so acquisition alone does not yield drug annotations
- Retroactive analysis is limited to what was deposited; the paper notes the resource depends on 'the continued deposition of reference libraries and expansion of the public metabolomics datasets'
- Downstream drug readouts support data-driven exploration but are not validated for immediate clinical decision-making

## Evidence

- [abstract] the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [intro] This resource enables data science analysis to empirically - and retroactively - determine drug exposures using untargeted metabolomics, supporting research across disciplines.: "This resource enables data science analysis to empirically - and retroactively - determine drug exposures using untargeted metabolomics, supporting research across disciplines."
- [readme] This repository contains the code and data for the different domain-specific MASSTs currently under development in the Dorrestein Lab at UC San Diego.: [evidence span withheld — anchor status: external_doc]
