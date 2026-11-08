---
name: software-quality-metric-comparison
description: Use when a paper claims a quantitative quality threshold for a package's
  test suite (e.g. 'code coverage above 90%') and the claim must be independently
  reproduced by executing the tests and measuring coverage.
when_to_use_negative:
- The paper reports no quantitative quality metric (e.g. only qualitative claims about
  functionality) — there is no threshold to reproduce.
- The quality metric concerns runtime performance or accuracy of data conversion results
  rather than test-suite code coverage.
- The package cannot be installed or its test suite is not shipped/published, making
  coverage measurement impossible.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3377
tools:
- name: MESSES
  role: Package under test whose unit-test suite is executed to measure code coverage
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: Python 3
  role: Runtime environment (Python 3.10+) in which the package, tests, and coverage
    tooling are executed
- name: coverage_report.json
  role: Machine-readable coverage output artifact capturing per-module and total coverage
    percentages
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:c55b542689b3a14a85d956fe7af4aa0d3622616fb48c94d446ab9921b5bed157
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - Python 3
  - coverage_report.json
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3377
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# software-quality-metric-comparison

## Summary

Verifies a software package's reported software-quality metric — here, unit-test code coverage above 90% — by running the full automated test suite under coverage.py and comparing the measured total against the published figure.

## When to use

Use when a paper claims a quantitative quality threshold for a package's test suite (e.g. 'code coverage above 90%') and the claim must be independently reproduced by executing the tests and measuring coverage.

## When NOT to use

- The paper reports no quantitative quality metric (e.g. only qualitative claims about functionality) — there is no threshold to reproduce.
- The quality metric concerns runtime performance or accuracy of data conversion results rather than test-suite code coverage.
- The package cannot be installed or its test suite is not shipped/published, making coverage measurement impossible.

## Inputs

- MESSES package installed from PyPI or github.com/MoseleyBioinformaticsLab/messes (Python 3.10+)
- The package's automated unit-test suite as wired into GitHub Actions CI
- Paper-reported quality metric: total code coverage above 90%

## Outputs

- coverage_report.json recording line/statement coverage per module (messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, messes.convert.mwtab_functions)
- Total measured coverage percentage and a verdict on whether the >90% threshold is met

## How to apply

Install the package under test (MESSES, Python 3.10+, via pip from PyPI or the GitHub source) in a Linux environment. Run the full automated unit-test suite — the same suite wired into the project's GitHub Actions CI across Linux, Windows, and MacOS — under the coverage.py tool so that every module is exercised, including messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, and messes.convert.mwtab_functions. Emit the machine-readable coverage report as coverage_report.json with line/statement coverage per module and the total percentage. Finally, compare the measured total against the reported threshold of above 90% and record whether the claim is met.

## Related tools

- **MESSES** (Package under test whose unit-test suite is executed to measure code coverage) — https://github.com/MoseleyBioinformaticsLab/messes
- **Python 3** (Runtime environment (Python 3.10+) in which the package, tests, and coverage tooling are executed)
- **coverage_report.json** (Machine-readable coverage output artifact capturing per-module and total coverage percentages)

## Examples

```
python3 -m pip install messes; coverage run -m pytest tests/ && coverage json -o coverage_report.json
```

## Evaluation signals

- coverage_report.json contains per-module coverage entries for all five target modules (messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, messes.convert.mwtab_functions)
- Total measured coverage percentage is compared against the reported 'above 90%' figure and the verdict is explicitly recorded
- The test run exercises the same suite used by the project's GitHub Actions continuous integration
- Coverage report is emitted in machine-readable JSON form rather than only human-readable console output

## Limitations

- The coverage measurement reflects only the modules exercised by the shipped test suite; untested code paths will lower the total below the reported figure.
- Reported coverage may differ across operating systems; the paper states testing is automated via GitHub Actions on Linux, Windows, and MacOS, so environment choice can affect results.
- MESSES requires Python 3.10+; older interpreters may fail to install or run the suite.

## Evidence

- [other] The MESSES package reports total testing code coverage above 90%, with testing automated via GitHub Actions across Linux, Windows, and MacOS environments.: [evidence span withheld — anchor status: not_found]
- [readme] The MESSES package runs under Python 3.10+. Use pip_ to install.: [evidence span withheld — anchor status: external_doc]
- [readme] Code coverage information: [evidence span withheld — anchor status: not_found]
