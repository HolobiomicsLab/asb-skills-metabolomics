---
name: cross-tool-dataframe-equality-checking
description: Use when an mzML file has been converted to Parquet by arcMS (convert_mzml_to_parquet())
  and you must verify that pandas read_parquet() reproduces exactly the DEIMoS deimos.load()
  DataFrame from the same mzML.
when_to_use_negative:
- The two DataFrames come from different samples or different mzML files rather than
  the same shared source file — equality would reflect identical inputs, not conversion
  fidelity
- The Parquet file was not produced by arcMS convert_mzml_to_parquet() (e.g. it is
  an HDF5 file or a UNIFI-API-derived Parquet), so the mzML-vs-Parquet fidelity question
  does not apply
- Tolerance-based numerical comparison is desired rather than strict identity — this
  skill's criterion is exact match on rows, columns, dtypes and values
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DEIMoS
  role: Loads the shared mzML file via deimos.load() to build the reference DataFrame
    for comparison
- name: arcMS
  role: Converts the same mzML to Parquet with convert_mzml_to_parquet(), the transformation
    whose fidelity is being validated
  repo: https://github.com/leesulab/arcMS
- name: pandas
  role: Reads the Parquet file with read_parquet() and performs element-wise DataFrame
    equality checks (DataFrame.equals / assert_frame_equal)
- name: Apache Arrow
  role: Columnar Parquet format in which the converted data is stored and read back
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:069012a419e34433e48d3b8a1b651d5a623c522feb92359b09ecd407db17c6e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Apache Arrow
  - DEIMoS
  - arcMS
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cross-tool-dataframe-equality-checking

## Summary

Validates lossless data fidelity between two scientific data-loading stacks by element-wise comparing the Python DataFrame loaded from an mzML file with DEIMoS deimos.load() against the DataFrame read from the arcMS-converted Parquet file with pandas read_parquet().

## When to use

Use when an mzML file has been converted to Parquet by arcMS (convert_mzml_to_parquet()) and you must verify that pandas read_parquet() reproduces exactly the DEIMoS deimos.load() DataFrame from the same mzML.

## When NOT to use

- The two DataFrames come from different samples or different mzML files rather than the same shared source file — equality would reflect identical inputs, not conversion fidelity
- The Parquet file was not produced by arcMS convert_mzml_to_parquet() (e.g. it is an HDF5 file or a UNIFI-API-derived Parquet), so the mzML-vs-Parquet fidelity question does not apply
- Tolerance-based numerical comparison is desired rather than strict identity — this skill's criterion is exact match on rows, columns, dtypes and values

## Inputs

- mzML mass spectrometry data file (e.g. Waters (HD)MSE export)
- arcMS-converted Parquet file (converted_file.parquet) produced by convert_mzml_to_parquet()
- Reference DataFrame from DEIMoS deimos.load() on the same mzML
- DataFrame loaded from the Parquet file with pandas read_parquet()

## Outputs

- fidelity_check.csv recording per-attribute comparison outcomes (identical rows, columns, dtypes, values)
- Boolean equality verdict per attribute (rows, columns, dtypes, values)

## How to apply

Load the shared mzML file directly with the DEIMoS library's deimos.load() to obtain the reference DataFrame. Convert the same mzML to Parquet with the arcMS function convert_mzml_to_parquet(), producing converted_file.parquet, then read it back with pandas read_parquet(). Compare the two DataFrames element-wise on row count, column names, dtypes and values, e.g. with DataFrame.equals or pandas assert_frame_equal. Decision point: any mismatch in rows, columns, dtypes or values fails the fidelity check; exact identity on all four attributes passes it. Record the per-attribute comparison outcome (identical rows, columns, dtypes, values) as fidelity_check.csv. The rationale is that identical DataFrames confirm the arcMS conversion of Waters (HD)MS/E mzML data to the Apache Parquet columnar format is lossless.

## Related tools

- **DEIMoS** (Loads the shared mzML file via deimos.load() to build the reference DataFrame for comparison)
- **arcMS** (Converts the same mzML to Parquet with convert_mzml_to_parquet(), the transformation whose fidelity is being validated) — https://github.com/leesulab/arcMS
- **pandas** (Reads the Parquet file with read_parquet() and performs element-wise DataFrame equality checks (DataFrame.equals / assert_frame_equal))
- **Apache Arrow** (Columnar Parquet format in which the converted data is stored and read back)

## Examples

```
import pandas as pd; df_deimos = deimos.load('sample.mzML'); df_parquet = pd.read_parquet('converted_file.parquet'); pd.testing.assert_frame_equal(df_deimos, df_parquet); df_deimos.equals(df_parquet)
```

## Evaluation signals

- Row counts of the DEIMoS-loaded DataFrame and the Parquet-loaded DataFrame are identical
- Column names and dtypes match exactly between the two DataFrames
- pandas.testing.assert_frame_equal (or DataFrame.equals) raises no AssertionError, i.e. all values match element-wise
- fidelity_check.csv records 'identical' for rows, columns, dtypes and values, satisfying the validation criterion of exact match

## Limitations

- The check is strict identity: any dtype or floating-point representation difference between DEIMoS and the Parquet round-trip will fail even if scientifically negligible
- arcMS is an R package (installable via pak::pak("leesulab/arcMS")); DEIMoS and pandas are Python, so the workflow spans both languages and requires the mzML conversion step to have succeeded
- Validation applies to mzML conversion; arcMS primarily collects MSE data from Waters UNIFI, and data handling for ion mobility (HDMSE) dimensions may add columns not covered by the simple equality check

## Evidence

- [readme] Conversion of mzML files is also supported: "It is compatible with data containing ion mobility
(HDMS<sup>E</sup>) or not (MS<sup>E</sup>). Conversion of mzML files is
also supported"
- [methods] data = read_parquet("converted_file.parquet"): "data = read_parquet("converted_file.parquet")"
- [results] DataFrames found to be identical, confirming lossless conversion: [evidence span withheld — anchor status: not_found]
