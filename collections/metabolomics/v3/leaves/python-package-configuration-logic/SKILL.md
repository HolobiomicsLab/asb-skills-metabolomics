---
name: python-package-configuration-logic
description: Use when you need to modify or extend Rapid QC-MS's specimen-level QC
  designation logic in Python — e.g., reproducing the 'Warn' escalation rule based
  on IS intensity dropouts and per-IS warnings.
when_to_use_negative:
- Raw vendor (.raw) data has not yet been converted to mzML via MSConvert and processed
  by MS-DIAL — the warning logic operates on downstream QC results, not raw acquisitions
- The goal is only to run or view the existing dashboard (e.g., 'rapidqcms') rather
  than implement or alter QC escalation logic in the package
- The analysis targets file-level or run-level QC metrics rather than the specimen-level
  escalation rule in the Specimen Table
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Rapid QC-MS
  role: The Python package whose specimen-level warning evaluation routine is being
    implemented and configured
  repo: github.com/czbiohub-sf/Rapid-QC-MS
- name: Python
  role: Implementation language; Rapid QC-MS was written in Python compatible with
    versions 3.8–3.11
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:30c64bdc6d9d6389c3ef69c542dbbc48c4867aaad2f7a1eeffa49be41df50780
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/czbiohub-sf/Rapid-QC-MS
  tools:
  - Python
  - Rapid QC-MS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# python-package-configuration-logic

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement and configure QC escalation logic inside the Rapid QC-MS Python package, specifically the specimen-level 'Warning' rule that ORs together IS intensity-dropout and per-IS QC 'Warn' conditions for the dashboard Specimen Table.

## When to use

Use when you need to modify or extend Rapid QC-MS's specimen-level QC designation logic in Python — e.g., reproducing the 'Warn' escalation rule based on IS intensity dropouts and per-IS warnings.

## When NOT to use

- Raw vendor (.raw) data has not yet been converted to mzML via MSConvert and processed by MS-DIAL — the warning logic operates on downstream QC results, not raw acquisitions
- The goal is only to run or view the existing dashboard (e.g., 'rapidqcms') rather than implement or alter QC escalation logic in the package
- The analysis targets file-level or run-level QC metrics rather than the specimen-level escalation rule in the Specimen Table

## Inputs

- Rapid QC-MS source code from github.com/czbiohub-sf/Rapid-QC-MS (Python 3.8–3.11)
- User-defined IS intensity dropout threshold configuration value
- Per-internal-standard QC designations ('Warn'/'Pass') for the specimen's IS targets
- Counts of IS intensity dropouts per specimen

## Outputs

- Specimen-level 'Warning' label in the Rapid QC-MS dashboard Specimen Table
- Modified quality-control logic module implementing the OR-combined escalation rule

## How to apply

Load the Rapid QC-MS source code from github.com/czbiohub-sf/Rapid-QC-MS (installable via 'py -m pip install rapid-qc-ms'; the package is written in Python compatible with versions 3.8–3.11) and locate the specimen-level warning evaluation routine in the package's quality-control logic. Implement the first condition: flag a specimen 'Warning' when the number of IS intensity dropouts is >= 75% of the user-defined threshold. Implement the second condition: flag 'Warning' when >= 50% of the specimen's internal standards individually carry a 'Warn' QC designation. Combine both with OR logic so either trigger alone sets the specimen-level 'Warning' flag in the dashboard Specimen Table, keeping the IS dropout threshold user-configurable. Validate by running the module on boundary-crossing test inputs and confirming the flag toggles exactly at the 75%-of-threshold and 50%-of-IS-Warn boundaries.

## Related tools

- **Rapid QC-MS** (The Python package whose specimen-level warning evaluation routine is being implemented and configured) — github.com/czbiohub-sf/Rapid-QC-MS
- **Python** (Implementation language; Rapid QC-MS was written in Python compatible with versions 3.8–3.11)

## Examples

```
py -m pip install rapid-qc-ms
```

## Evaluation signals

- The modified QC module executes without error on the package's test inputs
- A test input with IS intensity dropouts >= 75% of the user-defined threshold produces a specimen 'Warning' designation
- A test input with >= 50% of internal standards individually carrying a 'Warn' designation produces a specimen 'Warning' designation
- Inputs below both boundaries (dropouts < 75% of threshold AND IS Warn fraction < 50%) produce no specimen 'Warning'
- OR logic is confirmed: either condition alone, in isolation, triggers the 'Warning' flag while the other is below its boundary

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have bugs
- The IS dropout threshold must remain user-configurable — hardcoding the 75%-of-threshold boundary would break configurability
- Dependencies such as MSConvert and MS-DIAL must be installed manually; the package was designed for Windows platforms

## Evidence
