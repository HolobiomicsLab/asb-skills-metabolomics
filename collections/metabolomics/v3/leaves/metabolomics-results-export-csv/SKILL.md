---
name: metabolomics-results-export-csv
description: Use when MS-DIAL has finished annotating internal standard, reference
  standard, and external standard features for a Thermo raw file and the aligned results
  must be written to disk as CSV inputs for the Rapid QC-MS dashboard.
when_to_use_negative:
- Input is still vendor-proprietary raw data (à la Thermo .raw) that has not yet been
  converted to mzML by MSConvert — export happens downstream of conversion.
- MS-DIAL has not yet been run against the .msp library, so there are no aligned features
  to write; exporting an empty CSV would violate the validation check.
- You only need to view QC results interactively; the dashboard already reads the
  exported CSVs, so re-exporting adds no value.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Rapid QC-MS
  role: Python data-processing pipeline that exports MS-DIAL alignment results to
    CSV and serves the dashboard; provides the Watchdog/Pandas code that writes the
    export files
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MS-DIAL ver.4.9.221218
  role: Produces the aligned feature annotations that must be written to CSV; the
    source of match score, m/z error, and retention-time values
- name: MSConvert 3.0.23332-7faa7f422
  role: Converts raw vendor data into open mzML format before routing it to the data
    processing pipeline that ultimately exports CSV results
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS/issues
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:42967b4d0ff977140e5e847512d6d2871cdad36e53cc609e0f018f7e1ef3d7f6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/czbiohub-sf/Rapid-QC-MS
  repo_url: https://github.com/czbiohub-sf/Rapid-QC-MS
  tools:
  - MS-DIAL ver.4.9.221218
  - MSConvert 3.0.23332-7faa7f422
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metabolomics-results-export-csv

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Write the final Rapid QC-MS quality-control results (ES and RS feature annotations from MS-DIAL, internal-standard metrics, and QC checks) to CSV files that populate the browser-viewable dashboard and serve as the persistent record for downstream visualization.

## When to use

Use when MS-DIAL has finished annotating internal standard, reference standard, and external standard features for a Thermo raw file and the aligned results must be written to disk as CSV inputs for the Rapid QC-MS dashboard.

## When NOT to use

- Input is still vendor-proprietary raw data (à la Thermo .raw) that has not yet been converted to mzML by MSConvert — export happens downstream of conversion.
- MS-DIAL has not yet been run against the .msp library, so there are no aligned features to write; exporting an empty CSV would violate the validation check.
- You only need to view QC results interactively; the dashboard already reads the exported CSVs, so re-exporting adds no value.

## Inputs

- MS-DIAL (ver.4.9.221218) aligned feature-annotation output (m/z, MS/MS spectrum, match score, m/z error, retention time) for internal standard, reference standard, and external standard targets
- Predefined .msp spectral library matched to the analytical matrix
- mzML-converted Thermo raw files (produced by MSConvert 3.0.23332-7faa7f422)

## Outputs

- CSV results files (e.g. es_alignment_results.csv) containing feature-level match score, m/z error, and retention-time agreement for ES/RS alignments
- Structured CSV records that populate the Rapid QC-MS browser-viewable dashboard with instrument sensitivity, in-run retention time shifts, and mass accuracy drift metrics

## How to apply

After MS-DIAL (ver.4.9.221218) matches each feature's m/z and MS/MS spectrum against the predefined .msp spectral library, export the aligned annotation results to CSV with columns for match score, m/z error, and retention-time agreement, keeping the ES and RS libraries in separate modules because ES and RS are often not from the same biological matrix. The CSV files (e.g. es_alignment_results.csv) feed directly into Rapid QC-MS's Plotly Dash visualization, so they must exist and contain the alignment fields before the dashboard updates minutes after a new data file is written. Write these CSVs from the Python data-processing pipeline (Python 3.8–3.11, Pandas) rather than from MS-DIAL itself, so QC fail counts and Slack/email alerts can be computed against the exported values.

## Related tools

- **Rapid QC-MS** (Python data-processing pipeline that exports MS-DIAL alignment results to CSV and serves the dashboard; provides the Watchdog/Pandas code that writes the export files) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MS-DIAL ver.4.9.221218** (Produces the aligned feature annotations that must be written to CSV; the source of match score, m/z error, and retention-time values)
- **MSConvert 3.0.23332-7faa7f422** (Converts raw vendor data into open mzML format before routing it to the data processing pipeline that ultimately exports CSV results) — https://github.com/czbiohub-sf/Rapid-QC-MS/issues

## Examples

```
rapidqcms
```

## Evaluation signals

- Verify that the expected CSV file (e.g. es_alignment_results.csv) exists after the export step completes.
- Check that the CSV contains the required alignment fields: match score, m/z error, and retention-time agreement for every queried feature.
- Confirm that the ES results are kept in a separate table from the RS results, reflecting the separate molecular-target libraries.
- Compare the CSV values against the MS-DIAL output for at least one file to ensure the m/z error and match score ranges are sensible (e.g. m/z error within the MS-DIAL tolerance).
- Check that the dashboard updates within minutes of the new file being written, which confirms the CSV was written to the correct watched folder.

## Limitations

- Rapid QC-MS was designed to run on Windows platforms; on MacOS, monitoring/viewing works, but MSConvert and MS-DIAL processing and therefore CSV export cannot run natively.
- Support is only extensively tested on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files, so exports from other vendor formats may contain bugs or issues.
- Dependencies such as MSConvert and MS-DIAL must be installed manually; if either is missing, the alignment results (and thus the CSV export) cannot be produced.

## Evidence

- [methods] ES workflow separation rationale: [evidence span withheld — anchor status: not_found]
- [methods] MS-DIAL alignment fields: [evidence span withheld — anchor status: not_found]
- [methods] Exported file name and contents: [evidence span withheld — anchor status: not_found]
- [methods] CSV existence and field validation: [evidence span withheld — anchor status: not_found]
