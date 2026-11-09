---
name: cli-command-test-verification
description: Use when a package or paper claims a quantitative test-coverage figure
  (e.g. 'total testing code coverage above 90%') and the claim must be reproduced
  by running the test suite and measuring coverage per module.
when_to_use_negative:
- The claim under test is a scientific result or data analysis outcome rather than
  a software-quality metric such as unit-test coverage.
- The package provides no test suite or continuous-integration coverage configuration,
  so no per-module line coverage can be measured against a reported figure.
- You only need functional verification of CLI behavior (e.g. that extract/validate/convert
  run end-to-end), not quantitative coverage measurement across modules.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: MESSES
  role: Package whose test suite is executed and whose modules' coverage is measured
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: Python 3
  role: Runtime (Python 3.10+) in which the package is installed and the test suite
    is executed
- name: jsonschema
  role: Dependency of the MESSES validate command exercised by the test suite
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:40c5c57ee1ac06c1d2c23ece4fb6af9b2d2d4d8e3567802a8237b4db79858e4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - Python 3
  - jsonschema
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cli-command-test-verification

## Summary

Verifies a software package's claimed test-suite quality by executing its automated unit-test suite under a coverage measurement tool and confirming that total line/coverage exceeds a stated threshold (here, above 90% for the MESSES package).

## When to use

Use when a package or paper claims a quantitative test-coverage figure (e.g. 'total testing code coverage above 90%') and the claim must be reproduced by running the test suite and measuring coverage per module.

## When NOT to use

- The claim under test is a scientific result or data analysis outcome rather than a software-quality metric such as unit-test coverage.
- The package provides no test suite or continuous-integration coverage configuration, so no per-module line coverage can be measured against a reported figure.
- You only need functional verification of CLI behavior (e.g. that extract/validate/convert run end-to-end), not quantitative coverage measurement across modules.

## Inputs

- MESSES package installed in a Python 3.10+ environment (from MoseleyBioinformaticsLab/MESSES GitHub source or PyPI)
- The package's unit-test suite as wired into its continuous integration (GitHub Actions workflow, e.g. build.yml)
- coverage.py coverage tooling for line/statement measurement

## Outputs

- coverage_report.json recording line/statement coverage per module (messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, messes.convert.mwtab_functions)
- Total coverage percentage compared against the reported above-90% threshold, with a pass/fail determination

## How to apply

Install the package in a clean environment (for MESSES: Python 3.10+ via pip from PyPI or the MoseleyBioinformaticsLab/MESSES GitHub source). Execute the full automated unit-test suite — the same suite wired into the project's continuous integration, which the paper reports runs via GitHub Actions on Linux, Windows, and MacOS — under the coverage.py tool so that every module is exercised, including messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, and messes.convert.mwtab_functions. Emit the machine-readable coverage report as coverage_report.json, recording line/statement coverage per module and the total coverage percentage. Compare the measured total percentage against the reported figure (above 90%) and record whether the threshold is met, noting any module that falls short. Rationale: the README's codecov badge links coverage to https://codecov.io/gh/MoseleyBioinformaticsLab/MESSES, so CI-integrated coverage is the package's own quality signal, and reproducing it confirms the claimed testing rigor rather than accepting it on trust.

## Related tools

- **MESSES** (Package whose test suite is executed and whose modules' coverage is measured) — https://github.com/MoseleyBioinformaticsLab/messes
- **Python 3** (Runtime (Python 3.10+) in which the package is installed and the test suite is executed)
- **jsonschema** (Dependency of the MESSES validate command exercised by the test suite)

## Examples

```
pip install messes; coverage run -m pytest tests/ && coverage json -o coverage_report.json
```

## Evaluation signals

- coverage_report.json exists and records per-module line/statement coverage for all five target modules (messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, messes.convert.mwtab_functions).
- Measured total coverage percentage is above 90%, matching the paper's reported figure.
- The test suite runs green (zero failures) across the platform under test, consistent with the paper's GitHub Actions testing on Linux, Windows, and MacOS.
- Per-module coverage values are internally consistent: total coverage is a statement-weighted aggregate of the recorded module values.

## Limitations

- Coverage above 90% measures exercised lines, not correctness of outputs; it does not guarantee the extract/validate/convert transformations are scientifically accurate.
- The reported figure is tied to the package's continuous-integration configuration; coverage may differ under a different Python version or platform than the CI matrix (Linux, Windows, MacOS).
- The README notes installation quirks (Mac OS SSL certificate errors, Windows console-script PATH issues) that can block a clean environment setup before tests can run.

## Evidence

- [readme] CI coverage badge: "https://codecov.io/gh/MoseleyBioinformaticsLab/MESSES/branch/main/graphs/badge.svg?branch=main"
- [abstract] cross-platform CI testing: "implemented in the Python 3 programming language and supported on Linux, Windows, and Mac operating systems"
- [methods] module coverage targets: ".. automodule:: messes.convert.mwtab_functions
    :members:"
- [abstract] package source availability: "The source code and extensive documentation is hosted on GitHub and is also available on the Python Package Index for easy installation"
