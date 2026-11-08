---
name: code-coverage-measurement
description: Use when reproducing the reported >90% test-suite code-coverage figure
  for the MESSES package, or when verifying that a Python package's automated unit
  tests exercise its extract, validate, and convert command modules to a stated coverage
  threshold.
when_to_use_negative:
- You need to benchmark runtime performance or conversion accuracy of MESSES rather
  than verify its test-suite line coverage
- Input is not the MESSES package and its own test suite (this skill verifies the
  package's coverage claim, not coverage of arbitrary third-party code)
- You want to validate scientific data against JSON schemas — that is the separate
  'messes validate' command workflow, not a coverage-measurement task
tools:
- name: Python
  role: Runtime and testing environment in which the MESSES package and its unit-test
    suite are executed
- name: MESSES
  role: Package under test; coverage is measured across its extract, validate, and
    convert command modules
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: jsonschema
  role: Dependency whose import and use by the validate code path must be exercised
    to count toward module coverage
  repo: https://pypi.org/project/jsonschema/
- name: mwtab
  role: Dependency exercised by the messes.convert.mwtab_functions module during the
    test run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:a050a45f94a6ea94b5f49abd6ed81bc7a620defc433d4b1876b3d84b40a1ceba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - Python
  - jsonschema
  - mwtab
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# code-coverage-measurement

## Summary

Reproduce the paper's testing-coverage claim for the MESSES Python package by running its full automated unit-test suite under coverage.py, recording per-module line/statement coverage for messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, and messes.convert.mwtab_functions, and confirming total coverage exceeds 90%.

## When to use

Use when reproducing the reported >90% test-suite code-coverage figure for the MESSES package, or when verifying that a Python package's automated unit tests exercise its extract, validate, and convert command modules to a stated coverage threshold.

## When NOT to use

- You need to benchmark runtime performance or conversion accuracy of MESSES rather than verify its test-suite line coverage
- Input is not the MESSES package and its own test suite (this skill verifies the package's coverage claim, not coverage of arbitrary third-party code)
- You want to validate scientific data against JSON schemas — that is the separate 'messes validate' command workflow, not a coverage-measurement task

## Inputs

- MESSES Python package installed under Python 3 (3.10+), from the MoseleyBioinformaticsLab/MESSES GitHub source or PyPI
- The package's automated unit-test suite (as wired into the GitHub Actions CI)
- coverage.py coverage-measurement tool
- Package dependencies jsonschema and mwtab (exercised by the validate and mwTab conversion code paths)

## Outputs

- coverage_report.json recording line/statement coverage per module (messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, messes.convert.mwtab_functions) and the total coverage percentage
- A pass/fail verdict comparing the measured total coverage against the paper's reported >90% threshold

## How to apply

Install the MESSES package (Python 3, 3.10+ per the README, via pip from PyPI or from the MoseleyBioinformaticsLab/MESSES GitHub source) in a Linux environment. Execute the full automated unit-test suite — the same suite wired into the package's continuous integration, which runs via GitHub Actions across Linux, Windows, and MacOS — under the coverage.py tool so that every module (messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, and messes.convert.mwtab_functions) is exercised; the suite's jsonschema and mwtab dependencies must be importable since the validate and mwTab conversion code paths depend on them. Emit the machine-readable coverage report as coverage_report.json, recording line/statement coverage per module and the total coverage percentage. Finally, compare the measured total coverage percentage against the paper's reported figure of above 90% and record whether the threshold is met.

## Related tools

- **Python** (Runtime and testing environment in which the MESSES package and its unit-test suite are executed)
- **MESSES** (Package under test; coverage is measured across its extract, validate, and convert command modules) — https://github.com/MoseleyBioinformaticsLab/messes
- **jsonschema** (Dependency whose import and use by the validate code path must be exercised to count toward module coverage) — https://pypi.org/project/jsonschema/
- **mwtab** (Dependency exercised by the messes.convert.mwtab_functions module during the test run)

## Examples

```
python -m coverage run -m pytest && python -m coverage json -o coverage_report.json
```

## Evaluation signals

- coverage_report.json exists and contains per-module coverage entries for all five target modules (messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, messes.convert.mwtab_functions)
- Measured total coverage percentage is above 90%, matching the paper's reported figure
- All per-module coverage values are in the valid 0–100 range and consistent with the recorded statement counts
- The run reproduces under the same OS matrix the paper claims (GitHub Actions across Linux, Windows, and MacOS); a local Linux run should at minimum match the >90% total
- The measured total does not contradict the codecov.io tracking for the MESSES main branch referenced in the repository README

## Limitations

- The article reports only that coverage is above 90%; exact per-module percentages are not published, so the comparison is a threshold check rather than an exact-value match
- Coverage was reported under the package's CI matrix (Linux, Windows, MacOS via GitHub Actions); local environment differences in Python version or OS can shift the measured percentage
- The coverage claim covers the MESSES package modules only, not the correctness of the converted mwTab output itself

## Evidence

- [abstract] Python 3 supported on all target platforms: "implemented in the Python 3 programming language and supported on Linux, Windows, and Mac operating systems"
- [readme] Package installation prerequisite for running the test suite: [evidence span withheld — anchor status: external_doc]
- [readme] Codecov tracking of the MESSES main branch corroborates continuous coverage monitoring: ":target: https://codecov.io/gh/MoseleyBioinformaticsLab/MESSES"
