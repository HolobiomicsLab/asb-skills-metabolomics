---
name: metabolomics-workbench-data-retrieval
description: Use when a workflow needs to download specific study/analysis data from
  the Metabolomics Workbench REST interface (e.g. paired mwTab and JSON files for
  cross-format comparison) rather than manually browsing the site.
when_to_use_negative:
- The data files are already available locally in mwTab or JSON form and only need
  parsing/conversion — use mwtab's read_files/MWTabFile directly instead of the REST
  retrieval path.
- The target repository is not Metabolomics Workbench; GenericMWURL parameter validation
  is specific to MW's REST interface conventions.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: mwtab
  role: Python library whose mwrest module (GenericMWURL, MWRESTFile, read_mwrest)
    provides programmatic access to the MW REST interface and whose MWTabFile/read_files/tokenizer
    components parse the retrieved mwTab files
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: Python
  role: Runtime for the mwtab package, which runs under Python 3.6+
merged_aliases:
- metabolomics-dataset-retrieval
merged_alias_records:
- alias: metabolomics-workbench-dataset-retrieval
  slug: metabolomics-workbench-dataset-retrieval
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
- alias: metabolomics-workbench-rest-data-retrieval
  slug: metabolomics-workbench-rest-data-retrieval
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_004
  - task_002
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:6c1f838bdbe325bec7f1ba78aeac98e9cad9f0c2d061cfaf3ac5a8c27085753f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - mwtab
  - pandas
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolomics-workbench-data-retrieval

## Summary

Programmatically retrieve Metabolomics Workbench (MW) analysis data — mwTab-format and JSONized representations — using the mwtab package's mwrest module (GenericMWURL for constructing/validating REST URLs, MWRESTFile for non-mwTab responses) and the read_files/MWTabFile machinery for parsing.

## When to use

Use when a workflow needs to download specific study/analysis data from the Metabolomics Workbench REST interface (e.g. paired mwTab and JSON files for cross-format comparison) rather than manually browsing the site.

## When NOT to use

- The data files are already available locally in mwTab or JSON form and only need parsing/conversion — use mwtab's read_files/MWTabFile directly instead of the REST retrieval path.
- The target repository is not Metabolomics Workbench; GenericMWURL parameter validation is specific to MW's REST interface conventions.

## Inputs

- REST parameter key-value pairs (e.g. study ID, analysis ID) for Metabolomics Workbench queries
- Metabolomics Workbench REST base URL (default https://www.metabolomicsworkbench.org/rest/)
- Metabolomics Workbench analysis identifiers (e.g. AN001492, AN001493)

## Outputs

- mwTab-format responses parsed into MWTabFile instances
- Non-mwTab responses (csv/json/text) held as MWRESTFile instances
- Constructed, validated MW REST request URLs

## How to apply

Instantiate mwtab.mwrest.GenericMWURL with the base URL (default https://www.metabolomicsworkbench.org/rest/) and a dictionary of REST parameter key-value pairs (e.g. study ID and analysis ID); GenericMWURL validates that the parameters are valid before combining them with the base URL into a specific request URL. Invoke read_mwrest() to download a response representable in mwTab format and parse it with the standard MWTabFile machinery (mwtab.fileio.read_files, mwtab.tokenizer), or invoke read_mwrest()/MWRESTFile to hold responses not representable in mwTab format (csv/json/text) as a MWRESTFile instance. This same retrieval path is what enabled the paper's cross-format consistency comparison across the 1655 analyses present in both mwTab and JSONized formats. Validation consists of confirming each constructed URL passes GenericMWURL's key-value validation, downloads succeed over the MW REST interface, and mwTab-format payloads parse cleanly while non-mwTab payloads are retained as raw csv/json/text.

## Related tools

- **mwtab** (Python library whose mwrest module (GenericMWURL, MWRESTFile, read_mwrest) provides programmatic access to the MW REST interface and whose MWTabFile/read_files/tokenizer components parse the retrieved mwTab files) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **Python** (Runtime for the mwtab package, which runs under Python 3.6+)

## Examples

```
import mwtab; [print(mwfile.study_id, mwfile.analysis_id) for mwfile in mwtab.read_files("1", "2")]
```

## Evaluation signals

- Each constructed URL passes GenericMWURL's key-value validation of the REST parameters.
- Both download types succeed over the MW REST interface (mwTab-format and csv/json/text).
- The mwTab-format payload parses cleanly with the standard MWTabFile machinery.
- The non-mwTab payload is retained as raw csv/json/text content in a MWRESTFile instance.
- Retrieved analysis IDs match the expected target set (e.g. the 1655 analyses available in both formats for cross-format comparison).

## Limitations

- Only some REST responses are representable in mwTab format; responses in csv/json/text must be handled separately as MWRESTFile instances rather than parsed as MWTabFile.
- REST parameter key-value pairs must be valid for MW's interface; GenericMWURL rejects invalid parameter combinations.
- The mwTab package requires Python 3.6+.

## Evidence

- [other] GenericMWURL provides a programmatic representation of MW REST API URLs: [evidence span withheld — anchor status: not_found]
- [other] mwrest download of both representable and non-representable responses: [evidence span withheld — anchor status: not_found]
- [methods] package runs under Python 3.6+: "The :mod:`mwtab` package runs under Python 3.6+."
- [readme] package facilitates reading and writing mwTab files: "The ``mwtab`` package is a Python library that facilitates reading and writing files in ``mwTab`` format used by the `Metabolomics Workbench`_"
- [readme] quickstart fetch by analysis ID: ">>> # Here we use ANALYSIS_ID of file to fetch data from URL
   >>> for mwfile in mwtab.read_files("1", "2"):"
