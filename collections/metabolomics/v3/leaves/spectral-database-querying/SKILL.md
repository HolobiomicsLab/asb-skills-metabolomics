---
name: spectral-database-querying
description: Use when you need to programmatically list or filter compounds, NMR or
  mass spectra, and their metadata from a PeakForest database, e.g. to reproduce a
  demonstrator compound collection such as the 96-compound set.
when_to_use_negative:
- 'You need to store raw spectral data: PeakForest stores metadata related to raw
  data rather than raw data itself and is designed for metabolite annotation report
  data rather than storage of raw spectral data'
- 'You need data clean-up or extraction of spectra from instrument files: PeakForest
  does not include data clean-up or extraction tools and requires metadata to be manually
  entered'
- The local PeakForest database has not had a minimum number of compounds and spectra
  imported and is not yet fully functional
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PeakForest
  role: Infrastructure providing the REST Web services, OpenAPI v3.0 specification,
    compound library manager, and 'AND'/'OR' filter system used to query compound
    and spectral records
  repo: https://github.com/peakforest
- name: peakforest-webapp
  role: Java Web application hosting the Web service endpoints and the chemical compound
    library / spectra library managers being queried
  repo: https://services.pfem.clermont.inrae.fr/gitlab/peakforest/peakforest-webapp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:80dc17e7df0a109181ce88beec54cfaeb58ae3566051bda414064b7c236ca9c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PeakForest
  - peakforest-webapp
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# spectral-database-querying

## Summary

Querying a PeakForest instance's compound and spectral libraries through its REST Web services to retrieve metabolite records and associated NMR/MS metadata, using the query/filter capabilities (column characteristics as filters, 'AND'/'OR' logic-gate filter system) exposed by the service.

## When to use

Use when you need to programmatically list or filter compounds, NMR or mass spectra, and their metadata from a PeakForest database, e.g. to reproduce a demonstrator compound collection such as the 96-compound set.

## When NOT to use

- You need to store raw spectral data: PeakForest stores metadata related to raw data rather than raw data itself and is designed for metabolite annotation report data rather than storage of raw spectral data
- You need data clean-up or extraction of spectra from instrument files: PeakForest does not include data clean-up or extraction tools and requires metadata to be manually entered
- The local PeakForest database has not had a minimum number of compounds and spectra imported and is not yet fully functional

## Inputs

- PeakForest REST Web service endpoint of a local or demonstrator instance
- OpenAPI (v3.0) specification of the PeakForest Web services
- Query/filter parameters (e.g. LC, GC or ion chromatography column characteristics; 'AND'/'OR' logic-gate conditions)

## Outputs

- List of compound records with identifiers, names, and structural/spectral metadata
- Tab-separated table with one row per compound (e.g. 96 rows for the PeakForest demonstrator dataset)

## How to apply

Access the PeakForest REST Web services of the target instance; programmatic access is based on the REST protocol with an OpenAPI (v3.0) specification, so generate a client in a common language such as Python from that specification. Call the chemical compound library and spectra library manager endpoints, applying the available filter capabilities — e.g. chromatography column characteristics (brand, type, name, length, diameter) as filters — to narrow records. Use the advanced filter system integrating basic logic gates ('AND', 'OR') to combine query conditions, noting that the range of possible queries depends on spectral type. Parse the returned records and extract key metadata fields (compound identifiers, names, structural/spectral metadata), assembling them into a tabular structure with one row per record.

## Related tools

- **PeakForest** (Infrastructure providing the REST Web services, OpenAPI v3.0 specification, compound library manager, and 'AND'/'OR' filter system used to query compound and spectral records) — https://github.com/peakforest
- **peakforest-webapp** (Java Web application hosting the Web service endpoints and the chemical compound library / spectra library managers being queried) — https://services.pfem.clermont.inrae.fr/gitlab/peakforest/peakforest-webapp

## Evaluation signals

- Returned table contains exactly the expected number of compound rows (e.g. 96 for the PeakForest demonstrator compound collection)
- Every returned record includes required metadata fields (compound identifiers, names, associated structural/spectral metadata)
- Filter combinations using logic gates ('AND', 'OR') return record sets consistent with the applied conditions
- Column-characteristic filters (brand, type, name, length, diameter) restrict results to chromatography data matching the specified column

## Limitations

- Query capabilities depend on spectral type: 'a range of queries are possible depending on spectral type'
- PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data
- PeakForest does not include data clean-up or extraction tools and requires metadata to be manually entered in the database

## Evidence

- [results] A filter system, integrating basic logic gates ('AND', 'OR') allows users to query the database, and a range of queries are possible depending on spectral type: "A filter system, integrating basic logic gates (‘AND’, ‘OR’) allows users to query the database, and a range of queries are possible depending on spectral type"
- [intro] programmatic data access based on REST protocol with OpenAPI v3.0 and Python client generation: "In this paper we present PeakForest, an open-source and open access infrastructure which hosts for the first time both NMR and mass spectra, with an ergonomic Web interface and Web services to"
- [results] Column characteristics can be used as filter in Web service: [evidence span withheld — anchor status: not_found]
- [readme] PeakForest webapp provides a chemical compound library manager among its services: "## Services provided


- chemical compound library manager
- spectra library manager
- spectrum viewers
- curation interface"
