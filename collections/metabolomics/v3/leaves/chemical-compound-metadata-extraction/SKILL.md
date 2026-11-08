---
name: chemical-compound-metadata-extraction
description: Use when you need to enumerate the compound library of a PeakForest instance
  (e.g. a MetaboHUB demonstrator) and extract per-compound metadata into a flat table
  for inventory, curation, or downstream annotation reuse.
when_to_use_negative:
- You need raw spectral data storage or extraction — PeakForest stores metadata related
  to raw data rather than raw data itself and is designed for metabolite annotation
  report data
- 'Your input data is not cleaned: PeakForest expects users to supply completely-cleaned
  data following team-edited SOPs and does not include data clean-up or extraction
  tools'
- You are annotating spectra rather than inventorying the compound library — use external
  peak matching tools for annotation in biological matrices
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: PeakForest
  role: Source infrastructure providing REST Web services (OpenAPI v3.0) with a chemical
    compound library manager and query/filter system used to enumerate and extract
    compound metadata
  repo: https://github.com/peakforest
- name: Python
  role: Client language for generating an API client from the OpenAPI v3.0 specification
    and parsing returned compound records
- name: OpenAPI v3.0
  role: Specification used to generate the programmatic REST client for PeakForest
    Web services
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:854b3830fda432e12b53c0fb016d5814a45a1e750d28eb8199f7f8e807ca0022
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OpenAPI v3.0
  - PeakForest
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chemical-compound-metadata-extraction

## Summary

Extracts chemical compound records (identifiers, names, and structural/spectral metadata) from a PeakForest instance via its REST Web services and assembles them into a tab-separated, one-row-per-compound table. This mirrors the demonstrator workflow that reproduced the PeakForest compound collection of 96 biologically relevant compounds.

## When to use

Use when you need to enumerate the compound library of a PeakForest instance (e.g. a MetaboHUB demonstrator) and extract per-compound metadata into a flat table for inventory, curation, or downstream annotation reuse.

## When NOT to use

- You need raw spectral data storage or extraction — PeakForest stores metadata related to raw data rather than raw data itself and is designed for metabolite annotation report data
- Your input data is not cleaned: PeakForest expects users to supply completely-cleaned data following team-edited SOPs and does not include data clean-up or extraction tools
- You are annotating spectra rather than inventorying the compound library — use external peak matching tools for annotation in biological matrices

## Inputs

- PeakForest instance REST endpoint URL (OpenAPI v3.0 specification)
- Optional query/filter parameters (chromatography column characteristics; 'AND'/'OR' logic-gate filter expressions)

## Outputs

- Tab-separated table with one row per chemical compound, containing compound identifiers, names, and associated structural/spectral metadata fields

## How to apply

Access the PeakForest REST Web services, which are specified with OpenAPI v3.0 so a client can be generated in a common language such as Python. Call the chemical compound library endpoints and use the Web service's query/filter capabilities — column characteristics (brand, type, name, length, diameter for LC/GC chromatography data) and the logic-gate 'AND'/'OR' filter system — to list all compounds. Parse each returned compound record and extract key metadata fields: compound identifiers, names, and associated structural/spectral metadata. Assemble the records into a single tab-separated table with one row per compound. Rationale: PeakForest is a Java Web application (version 8) designed as an API's toolbox, so programmatic enumeration via REST is the intended access path rather than manual curation.

## Related tools

- **PeakForest** (Source infrastructure providing REST Web services (OpenAPI v3.0) with a chemical compound library manager and query/filter system used to enumerate and extract compound metadata) — https://github.com/peakforest
- **Python** (Client language for generating an API client from the OpenAPI v3.0 specification and parsing returned compound records)
- **OpenAPI v3.0** (Specification used to generate the programmatic REST client for PeakForest Web services)

## Examples

```
import requests; records = requests.get('https://<peakforest-host>/rest/v3/compounds').json(); rows = [f"{r['id']}\t{r['name']}\t{r.get('inChI','')}\t{r.get('smiles','')}" for r in records]; open('compounds.tsv','w').write('\n'.join(rows))
```

## Evaluation signals

- The output table contains exactly the expected number of compound rows (e.g. 96 rows matching the reported size of the PeakForest demonstrator compound collection)
- Every row contains the required metadata fields (compound identifier, name, structural/spectral metadata) with no missing values
- Returned record counts are consistent when re-queried with no filters versus an 'OR'-combined filter set
- Column-characteristic filters (e.g. LC vs GC column brand/type/name/length/diameter) return mutually consistent chromatography-restricted subsets
- Compound identifiers in the table resolve against the same instance's compound library via the Web interface

## Limitations

- PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database, so record completeness depends on prior curation
- A minimum number of compounds and spectra need to be imported before local PeakForest databases are fully functional
- Technical installation of PeakForest requires personnel with computing or bioinformatics skills, so a self-hosted instance may not be immediately queryable

## Evidence

- [methods] PeakForest is a Java Web application (version 8) and has been designed as an API's toolbox: "PeakForest is a Java Web application (version 8) and has been designed as an API’s toolbox (Online Resource 1)."
- [results] column characteristics can be used as filter in Web service: [evidence span withheld — anchor status: not_found]
- [results] filter system with logic gates AND/OR: "A filter system, integrating basic logic gates (‘AND’, ‘OR’) allows users to query the database, and a range of queries are possible depending on spectral type"
- [other] OpenAPI-based REST access enabling Python client generation: [evidence span withheld — anchor status: not_found]
- [readme] services provided include chemical compound library manager: "- chemical compound library manager
- spectra library manager
- spectrum viewers
- curation interface
- admin interface"
