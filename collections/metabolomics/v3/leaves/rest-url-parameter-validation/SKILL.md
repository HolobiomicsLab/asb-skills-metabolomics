---
name: rest-url-parameter-validation
description: Use when you need to fetch Metabolomics Workbench study/analysis data
  programmatically via its REST interface and must construct valid REST request URLs
  from key-value parameters (e.g. study ID and analysis ID) rather than hard-coding
  endpoint strings.
when_to_use_negative:
- The data is already a local mwTab or JSONized file — use mwtab's read/convert functions
  instead of a REST fetch
- You need a REST interface for a repository other than Metabolomics Workbench — GenericMWURL
  validates MW-specific REST parameters only
- A Python 3.6+ mwtab installation is not available
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mwtab 1.0.1
  role: Provides the mwrest module (GenericMWURL, MWRESTFile, read_mwrest) that validates
    and constructs MW REST URLs and downloads/holds responses
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
  source_content_hash: sha256:2a35c4dfb0bd306d99518db62fcc24bf57c3ace66838e763614ea2fb242950f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - mwtab 1.0.1
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rest-url-parameter-validation

## Summary

Programmatic construction and validation of Metabolomics Workbench REST API URLs using the mwtab 1.0.1 mwrest module's GenericMWURL class, which validates REST parameter key-value pairs before combining them with the base REST URL.

## When to use

Use when you need to fetch Metabolomics Workbench study/analysis data programmatically via its REST interface and must construct valid REST request URLs from key-value parameters (e.g. study ID and analysis ID) rather than hard-coding endpoint strings.

## When NOT to use

- The data is already a local mwTab or JSONized file — use mwtab's read/convert functions instead of a REST fetch
- You need a REST interface for a repository other than Metabolomics Workbench — GenericMWURL validates MW-specific REST parameters only
- A Python 3.6+ mwtab installation is not available

## Inputs

- Base MW REST URL (default https://www.metabolomicsworkbench.org/rest/)
- Dictionary of REST parameter key-value pairs (e.g. study ID, analysis ID)
- Python 3.6+ environment with mwtab 1.0.1 installed

## Outputs

- Validated, specific MW REST analysis-request URL
- Downloaded response parsed as a MWTabFile (mwTab-format responses) or held as a MWRESTFile (raw csv/json/text responses)

## How to apply

Instantiate mwtab.mwrest.GenericMWURL with the MW REST base URL (default https://www.metabolomicsworkbench.org/rest/) and a dictionary of REST parameter key-value pairs such as study ID and analysis ID. The class validates that the supplied parameters are valid REST parameters and combines them with the base URL to yield a specific, valid analysis-request URL; run the resulting URL through read_mwrest() from the mwtab package (Python 3.6+), which downloads one response representable in mwTab format for parsing with the standard MWTabFile machinery, or holds non-mwTab responses (csv/json/text) as a MWRESTFile instance. Rely on the validation step rather than string concatenation so that invalid REST keys or values are rejected before any network request is made.

## Related tools

- **mwtab 1.0.1** (Provides the mwrest module (GenericMWURL, MWRESTFile, read_mwrest) that validates and constructs MW REST URLs and downloads/holds responses) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **Python** (Runtime for the mwtab package, which runs under Python 3.6+)

## Examples

```
from mwtab.mwrest import GenericMWURL; u = GenericMWURL('https://www.metabolomicsworkbench.org/rest/', {'study_id': 'ST000001', 'analysis_id': 'AN000001'}); print(u)
```

## Evaluation signals

- Each constructed URL passes GenericMWURL's key-value validation before any download is attempted
- The download over the MW REST interface succeeds and returns exactly one response representable in mwTab format (or is correctly retained as raw csv/json/text)
- mwTab-format payloads parse cleanly with the standard MWTabFile machinery; study_id and analysis_id match the requested parameters
- Non-mwTab payloads (csv/json/text) are retained intact as a MWRESTFile instance rather than being force-parsed

## Limitations

- read_mwrest() downloads one response that is representable in mwTab format; responses not representable in mwTab (csv/json/text) must be handled as MWRESTFile raw content
- Requires Python 3.6+ and the mwtab package installed from PyPI or GitHub
- Only MW REST parameter key-value pairs are considered valid; other key names will fail validation

## Evidence

- [abstract] REST interface enhancement: "many new enhancements including methods for interacting with the MW REST interface, enhanced format validation features, and advanced features for parsing and searching"
- [methods] Python version requirement: "The :mod:`mwtab` package runs under Python 3.6+."
- [readme] Quickstart fetching by analysis ID: ">>> # Here we use ANALYSIS_ID of file to fetch data from URL
>>> for mwfile in mwtab.read_files("1", "2"):"
