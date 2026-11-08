---
name: python-unit-test-execution
description: Use when reproducing or verifying the reported testing coverage of a
  Python package (e.g. MESSES) — specifically when the target is total code coverage
  above 90% exercised via the package's automated unit-test suite.
when_to_use_negative:
- The goal is validating experimental data files with the MESSES validate command
  rather than measuring test coverage of the package code itself.
- Only a quick single-platform smoke test is needed and no comparison against the
  reported >90% coverage figure is required.
- The target package is not MESSES and the reported coverage claim (above 90% via
  GitHub Actions CI) does not apply.
tools:
- name: Python 3
  role: Runtime language and environment in which MESSES is implemented and its test
    suite is executed
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: MESSES
  role: The package under test; its unit tests across messes.extract, messes.validate,
    and messes.convert modules are exercised to measure coverage
  repo: https://github.com/MoseleyBioinformaticsLab/messes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:7a02b3f986293e91ed5f5bb37fc5083cbf41e1f3d22e9d6c5405b125c75f98d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - Python
  - Python 3
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# python-unit-test-execution

## Summary

Runs the full MESSES package unit-test suite under coverage.py to reproduce the paper's reported total code coverage of above 90%, emitting a machine-readable coverage_report.json that records per-module and total coverage percentages.

## When to use

Use when reproducing or verifying the reported testing coverage of a Python package (e.g. MESSES) — specifically when the target is total code coverage above 90% exercised via the package's automated unit-test suite.

## When NOT to use

- The goal is validating experimental data files with the MESSES validate command rather than measuring test coverage of the package code itself.
- Only a quick single-platform smoke test is needed and no comparison against the reported >90% coverage figure is required.
- The target package is not MESSES and the reported coverage claim (above 90% via GitHub Actions CI) does not apply.

## Inputs

- MESSES package source (pip-installable from PyPI or cloned from MoseleyBioinformaticsLab/MESSES)
- Python 3 (3.10+) runtime environment
- The package's automated unit-test suite (tests wired into GitHub Actions CI, build.yml)
- coverage.py coverage-measurement tool

## Outputs

- coverage_report.json containing per-module line/statement coverage and total coverage percentage
- A pass/fail verdict on whether total coverage exceeds the reported 90% threshold

## How to apply

Install the MESSES package from MoseleyBioinformaticsLab/MESSES on GitHub or from PyPI under Python 3 (the README specifies Python 3.10+ via pip) in a Linux environment, which is one of the three platforms (Linux, Windows, MacOS) the paper reports testing automated on via GitHub Actions. Execute the full automated unit-test suite — the same suite wired into the package's continuous integration (build.yml workflow) — under the coverage.py tool so that every module is exercised: messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, and messes.convert.mwtab_functions. Emit the machine-readable coverage report as coverage_report.json, recording line/statement coverage per module plus the total coverage percentage. Finally, compare the measured total coverage percentage against the paper's reported threshold of above 90% and record whether the threshold is met. The rationale is that the codecov.io badge on the repository indicates coverage is a first-class CI signal for this project, so a faithful reproduction must match the CI-driven, all-module execution rather than a partial test run.

## Related tools

- **Python 3** (Runtime language and environment in which MESSES is implemented and its test suite is executed) — https://github.com/MoseleyBioinformaticsLab/messes
- **MESSES** (The package under test; its unit tests across messes.extract, messes.validate, and messes.convert modules are exercised to measure coverage) — https://github.com/MoseleyBioinformaticsLab/messes

## Examples

```
python3 -m pip install messes ; python3 -m coverage run -m pytest ; python3 -m coverage json -o coverage_report.json
```

## Evaluation signals

- coverage_report.json is valid JSON and contains per-module line/statement coverage for all five modules: messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, and messes.convert.mwtab_functions
- Measured total coverage percentage exceeds the paper's reported threshold of above 90%
- The test run exercises the same CI-wired suite that GitHub Actions runs across Linux, Windows, and MacOS
- The measurement is reproducible: re-running the suite under coverage.py yields a consistent total coverage percentage
- Coverage result is consistent with the repository's codecov.io coverage reporting

## Limitations

- The package requires Python 3.10+ and a current pip; older pip versions are known to have issues grabbing all dependencies.
- On Mac OS, installation may fail with an SSL 'certificate verify failed: unable to get local issuer certificate' error, requiring the Install Certificates.command fix.
- On Windows, the messes console script may not be found on the PATH; the CLI must be invoked via python3 -m messes or the full Scripts path.
- Coverage is only meaningful if all modules (extract, validate, convert including user_input_checking and mwtab_functions) are actually exercised; a partial run will understate coverage.

## Evidence

- [abstract] implemented in the Python 3 programming language and supported on Linux, Windows, and Mac operating systems: "implemented in the Python 3 programming language and supported on Linux, Windows, and Mac operating systems"
- [readme] The MESSES package runs under Python 3.10+. Use pip_ to install.: [evidence span withheld — anchor status: external_doc]
- [intro] MESSES (Metadata from Experimental SpreadSheets Extraction System) is a Python package that facilitates the conversion of tabular data into other formats.: "MESSES (Metadata from Experimental SpreadSheets Extraction System) is a Python package that facilitates the conversion of tabular data into other formats."
