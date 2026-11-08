---
name: rest-api-spectral-record-retrieval
description: Use when a recurrent LC-HRMS feature in a biological matrix (e.g., NIST
  plasma) needs its deposited spectrum/compound records retrieved programmatically
  from PeakForest rather than copied manually from an article.
when_to_use_negative:
- PeakForest database has not yet had a minimum number of compounds and spectra imported
  and is not fully functional locally
- Task requires storage or clean-up of raw spectral data — PeakForest stores metadata
  related to raw data rather than raw data itself and includes no data clean-up or
  extraction tools
- Input is already a curated feature/annotation table that does not need re-retrieval
  from the spectral database
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: PeakForest
  role: Hosts the spectral and compound database queried via REST protocol-based Web
    services (OpenAPI v3.0 compliant) to retrieve records for the target m/z feature
    and annotated metabolite
  repo: https://github.com/peakforest
- name: peakforest-webapp
  role: Java Web application exposing the Web services and providing spectrum viewers,
    chemical compound and spectra library managers, and a curation interface used
    to browse retrieved records
  repo: https://github.com/peakforest/peakforest-webapp
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:5b999617b3094033943bbbe045bcd08320572e384cfd339ca7066c1a39a8d3e1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PeakForest
  - peakforest-webapp
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# rest-api-spectral-record-retrieval

## Summary

Programmatically pull spectrum and compound records (e.g., an unassigned m/z 247.1441 signal and its hypaphorine annotation) from a PeakForest database via its REST protocol-based Web services, enabling reproducible cross-checking of annotations against published results.

## When to use

Use when a recurrent LC-HRMS feature in a biological matrix (e.g., NIST plasma) needs its deposited spectrum/compound records retrieved programmatically from PeakForest rather than copied manually from an article.

## When NOT to use

- PeakForest database has not yet had a minimum number of compounds and spectra imported and is not fully functional locally
- Task requires storage or clean-up of raw spectral data — PeakForest stores metadata related to raw data rather than raw data itself and includes no data clean-up or extraction tools
- Input is already a curated feature/annotation table that does not need re-retrieval from the spectral database

## Inputs

- Observed m/z value of the target LC-HRMS feature (e.g., 247.1441, positive ion mode)
- Candidate compound name or identifier (e.g., hypaphorine)
- Sample matrix and instrument context (NIST plasma; Orbitrap Fusion and Q-ToF Impact II)
- PeakForest REST Web service endpoint / OpenAPI v3.0 specification

## Outputs

- PeakForest spectrum record(s) for the target m/z feature in positive ion mode
- PeakForest compound record(s) for the annotated metabolite
- Structured annotation table (e.g., nist_plasma_lchrms_annotations.csv) with columns: sample matrix, LC-HRMS platform, ionization mode, observed m/z, annotation, MSI confidence level

## How to apply

Identify the PeakForest instance hosting the target records (e.g., the public demonstrator referenced at peakforest.org) and use its REST protocol-based Web services, documented as OpenAPI v3.0 compliant, to query the database. Query by observed m/z (e.g., 247.1441 in positive ion mode) and by compound name (e.g., hypaphorine), using the available filter system — which integrates basic logic gates ('AND', 'OR') and can filter on chromatography metadata such as column characteristics — to narrow results to the relevant instrument (Orbitrap Fusion or Q-ToF Impact II) and ionization mode. Retrieve the returned spectrum and compound records and cross-check each against the source article text before compiling results into a structured table (one row per platform × feature). The rationale is that PeakForest is designed to 'capture and store experimental spectral MS and NMR metadata as well as collect and display signal annotations', so records pulled via the API are the authoritative deposited versions of the annotations reported in the paper.

## Related tools

- **PeakForest** (Hosts the spectral and compound database queried via REST protocol-based Web services (OpenAPI v3.0 compliant) to retrieve records for the target m/z feature and annotated metabolite) — https://github.com/peakforest
- **peakforest-webapp** (Java Web application exposing the Web services and providing spectrum viewers, chemical compound and spectra library managers, and a curation interface used to browse retrieved records) — https://github.com/peakforest/peakforest-webapp

## Examples

```
curl -H "Accept: application/" "https://peakforest.org/rest/spectra/search?mz=247.1441&mode=positive"
```

## Evaluation signals

- Retrieved records resolve to the recurrent m/z 247.1441 signal observed in positive mode on both the Orbitrap Fusion and the Q-ToF Impact II
- Compound record corresponds to hypaphorine annotated at Metabolomics Standards Initiative confidence level 2
- Compiled CSV parses cleanly and contains one row per platform × feature (both instruments represented)
- Recorded CSV values match the article-reported results (m/z 247.1441; hypaphorine, MSI level 2)

## Limitations

- PeakForest does not include data clean-up or extraction tools and requires metadata to be manually entered in the database
- Users must supply completely-cleaned data following SOPs; raw spectral data is not stored, only metadata related to raw data
- Additional developments are needed to enhance the MS–MS and NMR-2D matching capabilities
- A minimum number of compounds and spectra must be imported before local PeakForest databases are fully functional

## Evidence

- [abstract] PeakForest is able to capture and store experimental spectral MS and NMR metadata as well as collect and display signal annotations.: "PeakForest is able to capture and store experimental spectral MS and NMR metadata as well as collect and display signal annotations."
- [results] A filter system, integrating basic logic gates ('AND', 'OR') allows users to query the database: [evidence span withheld — anchor status: not_found]
- [results] The column characteristics can be used as filter in Web service: [evidence span withheld — anchor status: not_found]
- [methods] PeakForest stores metadata related to raw data rather than raw data itself to minimize storage space usage.: "PeakForest stores metadata related to raw data rather than raw data itself to minimize storage space usage."
- [methods] Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
