---
name: non-mwtab-response-handling-csv-json-text
description: Use when downloading Metabolomics Workbench REST responses (csv, json,
  or text) that cannot be parsed into mwTab format and need to be captured as raw
  payload via MWRESTFile.
when_to_use_negative:
- The REST response is representable in mwTab format — use the MWTabFile parsing path
  via read_mwrest() instead of retaining raw content.
- The input is a local mwTab-format file rather than a downloaded REST response —
  use the standard mwtab file reading/conversion machinery.
- You need schema validation of mwTab data (validate/validate_file with ms/nmr required
  schemas) rather than raw non-mwTab payload retrieval.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: mwtab 1.0.1
  role: Provides GenericMWURL, MWRESTFile, and read_mwrest for constructing validated
    REST URLs and capturing non-mwTab csv/json/text responses
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: Python
  role: Runtime for the mwtab package, which runs under Python 3.6+
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:1f77118bd0403d62471d21c5ab2f2b3ac037a7b220644e58716ef77bf903a275
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - mwtab
  - mwtab 1.0.1
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# non-mwtab-response-handling-csv-json-text

## Summary

Handles MW REST interface responses that are not representable in mwTab format by retaining csv/json/text payloads as MWRESTFile instances, complementing mwTab-format parsing. Introduced in mwtab 1.0.1 alongside GenericMWURL for constructing validated REST request URLs.

## When to use

Use when downloading Metabolomics Workbench REST responses (csv, json, or text) that cannot be parsed into mwTab format and need to be captured as raw payload via MWRESTFile.

## When NOT to use

- The REST response is representable in mwTab format — use the MWTabFile parsing path via read_mwrest() instead of retaining raw content.
- The input is a local mwTab-format file rather than a downloaded REST response — use the standard mwtab file reading/conversion machinery.
- You need schema validation of mwTab data (validate/validate_file with ms/nmr required schemas) rather than raw non-mwTab payload retrieval.

## Inputs

- REST parameter key-value pairs (e.g. study ID, analysis ID) for GenericMWURL validation
- MW REST base URL (default https://www.metabolomicsworkbench.org/rest/)
- MW REST interface responses in csv, json, or text format

## Outputs

- MWRESTFile instance holding raw csv/json/text response content
- Validated MW REST request URL strings
- Results file containing constructed URLs plus both mwTab-format and non-mwTab response payloads

## How to apply

First construct a specific MW REST request URL using the mwtab.mwrest.GenericMWURL class, supplying the base URL (default https://www.metabolomicsworkbench.org/rest/) plus a dictionary of REST parameter key-value pairs (e.g. study ID and analysis ID) that GenericMWURL validates. Then invoke read_mwrest(): for responses representable in mwTab format, the payload parses through the standard MWTabFile machinery; for responses NOT representable in mwTab format (csv/json/text), read_mwrest()/MWRESTFile downloads and holds the content as a MWRESTFile instance without attempting mwTab parsing. Finally, write both the constructed URLs and the downloaded payloads (mwTab-format and non-mwTab) to a single results file. The rationale is that the MW REST interface serves many response types beyond mwTab, so the 1.0.1 release added this branch to preserve raw csv/json/text content rather than failing on unparseable formats.

## Related tools

- **mwtab 1.0.1** (Provides GenericMWURL, MWRESTFile, and read_mwrest for constructing validated REST URLs and capturing non-mwTab csv/json/text responses) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **Python** (Runtime for the mwtab package, which runs under Python 3.6+)

## Examples

```
from mwtab.mwrest import GenericMWURL, read_mwrest; url = GenericMWURL('https://www.metabolomicsworkbench.org/rest/', {'study_id': 'ST000001', 'analysis_id': 'AN000001'}); payload = read_mwrest(url)
```

## Evaluation signals

- Each constructed URL passes GenericMWURL's key-value parameter validation before any download is attempted.
- The non-mwTab download succeeds over the MW REST interface and is retained as a MWRESTFile instance with raw csv/json/text content.
- The mwTab-format payload, if also fetched, parses cleanly through the standard MWTabFile machinery.
- The single results file contains both the constructed URLs and the downloaded payloads (both mwTab-format and non-mwTab).

## Limitations

- Requires Python 3.6+ as the mwtab package runs under Python 3.6+.
- Only responses NOT representable in mwTab format (csv/json/text) are held raw; mwTab-representable responses follow the separate MWTabFile parsing path.
- No changelog was found in the article for tracking version-specific behavior changes.

## Evidence

- [results] GenericMWURL validation and URL construction: [evidence span withheld — anchor status: not_found]
- [methods] non-mwTab response handling: "The :mod:`mwtab` package runs under Python 3.6+."
- [abstract] REST interface methods as new enhancement: "many new enhancements including methods for interacting with the MW REST interface, enhanced format validation features"
- [readme] package purpose from README: "The ``mwtab`` package is a Python library that facilitates reading and writing files in ``mwTab`` format"
