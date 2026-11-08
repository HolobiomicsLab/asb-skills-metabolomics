---
name: cross-platform-annotation-table-compilation
description: Use when an unassigned LC-HRMS signal recurs across two or more instruments/modes
  and you need to pull its spectral and compound records from a spectral database
  and tabulate platform-by-platform annotations with MSI confidence levels.
when_to_use_negative:
- The target signal was observed on only a single instrument/platform and no cross-platform
  comparison is required
- You need to store or clean raw spectral data — PeakForest stores metadata related
  to raw data rather than raw data itself and requires users to supply completely-cleaned
  data
- Annotations are already available in a compiled feature table; re-querying PeakForest
  records would duplicate existing work
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: PeakForest
  role: Spectral/compound database queried via REST Web services to retrieve spectrum
    and compound records for the unassigned m/z 247.1441 signal and hypaphorine in
    positive ion mode
  repo: https://github.com/peakforest/peakforest-webapp
- name: Galaxy
  role: Optional environment in which PeakForest databases can be used with bespoke
    data analysis pipelines
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:a0cea305e3972e28b0a69775182d36d36c9fe0cc08ed7193e0f0d0c8e3ae46f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/peakforest/peakforest-webapp
  tools:
  - Galaxy
  - PeakForest
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# cross-platform-annotation-table-compilation

## Summary

Compiles metabolite annotations observed across multiple LC-HRMS platforms (e.g. Orbitrap Fusion and Q-ToF Impact II) into a single structured table, using PeakForest REST/Web services to retrieve spectrum and compound records for cross-platform signals such as the recurrent m/z 247.1441 in positive-mode NIST plasma annotated as hypaphorine (MSI level 2).

## When to use

Use when an unassigned LC-HRMS signal recurs across two or more instruments/modes and you need to pull its spectral and compound records from a spectral database and tabulate platform-by-platform annotations with MSI confidence levels.

## When NOT to use

- The target signal was observed on only a single instrument/platform and no cross-platform comparison is required
- You need to store or clean raw spectral data — PeakForest stores metadata related to raw data rather than raw data itself and requires users to supply completely-cleaned data
- Annotations are already available in a compiled feature table; re-querying PeakForest records would duplicate existing work

## Inputs

- Article reporting LC-HRMS(+MS/MS) positive-mode annotation of NIST plasma on Orbitrap Fusion and Q-ToF Impact II (DOI 10.1007/s11306-022-01899-3)
- Deposited demonstrator/supplementary data accompanying the article
- PeakForest public demonstrator database records (spectrum and compound entries for m/z 247.1441 and hypaphorine, positive ion mode)
- PeakForest REST/Web services (OpenAPI v3.0 compliant) endpoints

## Outputs

- nist_plasma_lchrms_annotations.csv with one row per platform × feature and columns: sample matrix, LC-HRMS platform, ionization mode, observed m/z, annotation, MSI confidence level
- Rows for both Orbitrap Fusion and Q-ToF Impact II recording m/z 247.1441 annotated as hypaphorine at MSI confidence level 2

## How to apply

Locate the LC-HRMS/MS results reporting positive-mode annotation of the matrix of interest (here NIST plasma) acquired on each platform (Orbitrap Fusion and Q-ToF Impact II). Open the deposited demonstrator/supplementary data and the public PeakForest web application, then use PeakForest's REST protocol-based Web services (OpenAPI v3.0 compliant) to programmatically pull the spectrum and compound records for the unassigned feature (m/z 247.1441) and the candidate metabolite (hypaphorine) in positive ion mode. Cross-check every retrieved record against the article text before tabulation. Compile one row per platform × feature with columns for sample matrix, LC-HRMS platform, ionization mode, observed m/z, annotation, and MSI confidence level, writing the result as a CSV (nist_plasma_lchrms_annotations.csv). The rationale is that PeakForest stores both NMR and MS spectral metadata with a Web interface and Web services, so a single API query path covers records generated on different instruments.

## Related tools

- **PeakForest** (Spectral/compound database queried via REST Web services to retrieve spectrum and compound records for the unassigned m/z 247.1441 signal and hypaphorine in positive ion mode) — https://github.com/peakforest/peakforest-webapp
- **Galaxy** (Optional environment in which PeakForest databases can be used with bespoke data analysis pipelines)

## Examples

```
curl -X GET "https://peakforest.org/rest/spectra/lcms/search/compoundName?name=hypaphorine&mode=positive"
```

## Evaluation signals

- nist_plasma_lchrms_annotations.csv parses cleanly as valid CSV
- CSV contains rows for both the Orbitrap Fusion and the Q-ToF Impact II platforms
- Recorded values match the reported results: observed m/z 247.1441 in positive ionization mode
- Annotation column reads hypaphorine with Metabolomics Standards Initiative confidence level 2 on every row
- Each tabulated record is consistent with both the PeakForest retrieved record and the article text (cross-check step)

## Limitations

- PeakForest does not include data clean-up or extraction tools and requires metadata to be manually entered; users must supply completely-cleaned data following SOPs
- A minimum number of compounds and spectra need to be imported by users in order to make local PeakForest databases fully-functional
- Additional developments are needed to enhance the MS–MS and NMR-2D matching capabilities
- PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data

## Evidence

- [abstract] PeakForest is able to capture and store experimental spectral MS and NMR metadata as well as collect and display signal annotations.: "PeakForest is able to capture and store experimental spectral MS and NMR metadata as well as collect and display signal annotations."
- [results] PeakForest REST services are the programmatic retrieval path: [evidence span withheld — anchor status: not_found]
- [methods] Import and exploit data via Web services: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [discussion] PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data: "PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data"
- [readme] PeakForest is freely available on GitHub as open-source under CECILL-2.16 license: "main usage:  PeakForest web-application; for further informations, please refer to [peakforest.org](https://peakforest.org/)"
