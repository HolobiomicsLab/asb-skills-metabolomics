---
name: subject-sample-factor-consistency-checking
description: 'Use when validating mwTab-format files from Metabolomics Workbench studies:
  run this check on every parsed MWTabFile to detect blank/missing sample IDs or factors
  and sample IDs in SUBJECT_SAMPLE_FACTORS that do not appear in the _DATA blocks.'
when_to_use_negative:
- The file cannot be parsed into an MWTabFile at all (unparseable remainder of the
  corpus) — these are skipped by read_files and cannot be block-checked
- The file contains no SUBJECT_SAMPLE_FACTORS or _DATA blocks (e.g. metadata-only
  mwTab file), leaving nothing to cross-compare
- The goal is schema-structural validation rather than cross-block sample consistency
  — use MWTabFile.validate with ms_required_schema/nmr_required_schema instead
edam_operation: http://edamontology.org/operation_2428
tools:
- name: mwtab
  role: Provides validate_file() and its SUBJECT_SAMPLE_FACTORS consistency sub-validation
    over parsed MWTabFile objects (mwtab 1.0.1)
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: jsonschema
  role: Backend for the mwTab schema-validation leg wired into MWTabFile.validate()
    alongside this consistency check
- name: pandas
  role: Aggregates per-file validation results into category counts for the summary
    tallies
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:199fc357627d41b085a450381d35b9906578ca0b5753c9e005ff650e36b2b989
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - jsonschema
  - mwtab
  - pandas
  edam_operation: http://edamontology.org/operation_2428
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# subject-sample-factor-consistency-checking

## Summary

A sub-validation of mwTab-format study files that checks whether sample IDs and sample factors declared in the SUBJECT_SAMPLE_FACTORS block are consistent with the _DATA blocks — flagging blank or duplicate sample IDs and inconsistent factor headings. It was used as part of the mwtab validator that found validation errors in 1539 of 1818 parseable Metabolomics Workbench files.

## When to use

Use when validating mwTab-format files from Metabolomics Workbench studies: run this check on every parsed MWTabFile to detect blank/missing sample IDs or factors and sample IDs in SUBJECT_SAMPLE_FACTORS that do not appear in the _DATA blocks.

## When NOT to use

- The file cannot be parsed into an MWTabFile at all (unparseable remainder of the corpus) — these are skipped by read_files and cannot be block-checked
- The file contains no SUBJECT_SAMPLE_FACTORS or _DATA blocks (e.g. metadata-only mwTab file), leaving nothing to cross-compare
- The goal is schema-structural validation rather than cross-block sample consistency — use MWTabFile.validate with ms_required_schema/nmr_required_schema instead

## Inputs

- Parsed MWTabFile objects (mwTab-format study files from Metabolomics Workbench, loaded via mwtab.fileio.read_files)
- SUBJECT_SAMPLE_FACTORS block entries (sample IDs and factor values/headings)
- _DATA blocks containing per-sample data columns keyed by sample ID

## Outputs

- Per-file consistency warnings/errors (written to mwtab_validation_report.txt when verbose=True)
- Aggregated category counts of sample-ID/sample-factor inconsistencies (e.g. validation_summary.csv tallies)

## How to apply

After parsing each study file into an MWTabFile object with mwtab.fileio.read_files, invoke the consistency sub-validation inside mwtab.validator.validate_file(mwtabfile, ms_schema, nmr_schema, verbose=False), which checks that there are no blank sample ids or factors and that a subset of sample ids from the 'SUBJECT_SAMPLE_FACTORS' block is present in the '_DATA' blocks. Flag duplicate Sample IDs and inconsistent factor headings across SUBJECT_SAMPLE_FACTORS entries and _DATA blocks (per the reconstructed validator in Methods 4.5). Run with verbose=True to emit warnings/errors for the downloaded corpus rather than silently failing. Aggregate per-file results (alongside schema, null-value, and metabolite-name checks) into category counts with pandas, e.g. written to validation_summary.csv — in the reference run, 1539 of 1818 parseable files had some validation error. Note that files failing to parse at the tokenizer/read_files stage are skipped entirely and excluded from the denominator.

## Related tools

- **mwtab** (Provides validate_file() and its SUBJECT_SAMPLE_FACTORS consistency sub-validation over parsed MWTabFile objects (mwtab 1.0.1)) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **jsonschema** (Backend for the mwTab schema-validation leg wired into MWTabFile.validate() alongside this consistency check)
- **pandas** (Aggregates per-file validation results into category counts for the summary tallies)

## Examples

```
from mwtab.validator import validate_file; validate_file(mwtabfile, mwschema.ms_required_schema, mwschema.nmr_required_schema, verbose=True)
```

## Evaluation signals

- The reported tally reproduces: 1539 of 1818 parseable mwTab files contain at least one validation error, including subject/sample-factor inconsistencies
- No blank (empty) sample IDs or sample factors remain in any file the validator passes
- Every sample ID reported in SUBJECT_SAMPLE_FACTORS that is checked appears in the corresponding _DATA blocks; mismatches surface as warnings when verbose=True
- The set of sample IDs in SUBJECT_SAMPLE_FACTORS contains no duplicates and uses consistent factor headings matching the _DATA block columns
- Running validate_file with verbose=False vs verbose=True changes only output verbosity, not the pass/fail classification

## Limitations

- The check only requires that a subset of SUBJECT_SAMPLE_FACTORS sample IDs appear in _DATA blocks — not full one-to-one coverage, so some mismatches may go unflagged
- Files that fail to parse (excluded from the 1818-file parseable corpus) are never reached by this check, biasing error tallies toward parseable files
- It is one of four sub-validations; schema errors (1272 analysis files) and null-value errors (706 analysis files) must be checked separately

## Evidence

- [methods] validate_file signature: ".. autofunction:: validate_file(mwtabfile, ms_schema, nmr_schema, verbose = False)"
- [methods] schema validation dependency: "jsonschema_ for validating functionality of ``mwTab`` files based on ``JSON`` schema."
- [methods] validate purpose: "Validate data stored in ``mwTab`` file based on schema definition."
- [readme] library scope: "The ``mwtab`` package is a Python library that facilitates reading and writing files in ``mwTab`` format used by the `Metabolomics Workbench`_ for archival of Mass Spectrometry (MS) and Nuclear"
