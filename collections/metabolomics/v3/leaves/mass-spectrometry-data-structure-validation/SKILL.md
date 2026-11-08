---
name: mass-spectrometry-data-structure-validation
description: Use when scripting against OpenMS from Python — after building/installing
  pyOpenMS and before writing mzML-loading or quantitation workflows, to confirm MSExperiment/FeatureMap
  objects work in Python.
when_to_use_negative:
- Input data is already loaded and validated (e.g. you have a working MSExperiment
  object) — this skill is an install/build smoke test, not per-file validation.
- You are using OpenMS via TOPPTools/workflow engines (KNIME, Galaxy, nextflow) rather
  than the Python bindings — there is no pyOpenMS module to instantiate.
- Working from prebuilt bioconda binaries without source changes — use the packaged
  module's own import check rather than rebuilding bindings.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pyOpenMS
  role: Python bindings module that must be imported and used to instantiate MSExperiment,
    PeakMap, FeatureMap
  repo: https://github.com/OpenMS/OpenMS
- name: nanobind
  role: binding generator used to compile the Python extension from src/pyOpenMS/bindings/
- name: OpenMS
  role: C++ core library whose data structures and algorithms the Python bindings
    expose; built natively before generating the extension
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:4227c0c11acc9d408aaa67ed77ba9de6a1574cd1960f5ee831a8efa2ffc91d71
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - C++
  - OpenMS
  - nanobind
  - pyOpenMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mass-spectrometry-data-structure-validation

## Summary

Validate that core OpenMS/pyOpenMS data structures (MSExperiment, PeakMap, FeatureMap) are instantiated and interoperable with C++-level algorithms before building scripted MS workflows. This is the smoke-test step that confirms a pyOpenMS installation actually exposes the C++ core library's data structures and algorithms to Python.

## When to use

Use when scripting against OpenMS from Python — after building/installing pyOpenMS and before writing mzML-loading or quantitation workflows, to confirm MSExperiment/FeatureMap objects work in Python.

## When NOT to use

- Input data is already loaded and validated (e.g. you have a working MSExperiment object) — this skill is an install/build smoke test, not per-file validation.
- You are using OpenMS via TOPPTools/workflow engines (KNIME, Galaxy, nextflow) rather than the Python bindings — there is no pyOpenMS module to instantiate.
- Working from prebuilt bioconda binaries without source changes — use the packaged module's own import check rather than rebuilding bindings.

## Inputs

- pyopenms Python module compiled from the OpenMS develop branch with nanobind bindings (src/pyOpenMS/bindings/)
- a small mzML file for the scripted data-processing smoke test
- unit/functional test suite from src/tests/class_tests/

## Outputs

- confirmation that MSExperiment, PeakMap and FeatureMap instantiate from pyopenms in Python
- a scripted mzML-loading/spectra-iterating call demonstrating algorithm access through Python
- a clean run of OpenMS unit and functional tests against the built module

## How to apply

Build the OpenMS C++ core and its nanobind-based Python extension (bindings live in src/pyOpenMS/bindings/ on the develop branch; wrapping instructions are in src/pyOpenMS/CLAUDE.md). Once compiled, run an import smoke test that instantiates the core data structures — MSExperiment, PeakMap, FeatureMap — directly from the pyopenms module, verifying that Python sees the same efficient structures used in C++. Then exercise a minimal scripted data-processing call, such as loading a small mzML file and iterating its spectra, to confirm workflow scripting support end to end. Finally run the unit and functional tests against the built module, following the pattern of existing class tests like the MSNumpressCoder test (src/tests/class_tests/openms/source/MSNumpressCoder_test.cpp). Validation passes only if the module imports cleanly, all core data-structure instantiations succeed, and the test suite runs without errors.

## Related tools

- **pyOpenMS** (Python bindings module that must be imported and used to instantiate MSExperiment, PeakMap, FeatureMap) — https://github.com/OpenMS/OpenMS
- **nanobind** (binding generator used to compile the Python extension from src/pyOpenMS/bindings/)
- **OpenMS** (C++ core library whose data structures and algorithms the Python bindings expose; built natively before generating the extension) — https://github.com/OpenMS/OpenMS

## Examples

```
import pyopenms; exp = pyopenms.MSExperiment(); pyopenms.MzMLFile().load('small.mzML', exp); [s for s in exp.getSpectra()]
```

## Evaluation signals

- `import pyopenms` succeeds without errors
- MSExperiment(), PeakMap() and FeatureMap() construct successfully from the compiled module
- loading a small mzML file and iterating its spectra returns spectra through the Python API
- unit and functional tests (e.g. MSNumpressCoder_test.cpp pattern) pass with no errors

## Limitations

- Validation is tied to a source build of the develop branch; prebuilt installations may not have reproducible binding behavior.
- Core data-structure instantiation only proves API exposure, not numerical correctness of downstream algorithms.
- Requires a native C++ toolchain; build failures upstream of the binding step block the entire validation.
- OpenMS licensing is three-clause BSD per the README; runtime dependencies may carry different licenses.

## Evidence

- [methods] pyOpenMS interface exposes core data structures to Python: [evidence span withheld — anchor status: not_found]
- [readme] README on Python bindings and file formats: [evidence span withheld — anchor status: external_doc]
- [readme] README feature list on Python bindings: [evidence span withheld — anchor status: external_doc]
- [readme] README on top-level documentation: [evidence span withheld — anchor status: external_doc]
- [readme] pyOpenMS documentation pointer: [evidence span withheld — anchor status: external_doc]
