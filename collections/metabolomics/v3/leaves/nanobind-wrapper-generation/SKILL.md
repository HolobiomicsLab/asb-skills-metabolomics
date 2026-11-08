---
name: nanobind-wrapper-generation
description: Use when a C++ library needs Python scripting access to its core data
  structures and algorithms for rapid prototyping, and the binding layer is nanobind.
when_to_use_negative:
- The library already ships working Python bindings that can be installed via conda/bioconda
  (e.g. pyopenms on bioconda) and no wrapper extension is needed
- The goal is standalone tool interoperability via CTD/parameter files or workflow
  engines rather than in-process scripting of the C++ API
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: nanobind
  role: Binding generator used to create the Python extension module from the C++
    API
- name: OpenMS
  role: The C++ core library being wrapped; provides MSExperiment, PeakMap, FeatureMap
    and the algorithms exposed through pyOpenMS
  repo: https://github.com/OpenMS/OpenMS
- name: Python
  role: Target scripting language; validated by import smoke tests and data-processing
    calls through the compiled module
- name: C++
  role: Source language of the core library whose classes and functions are exposed
    to Python via the generated bindings
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:bf0af4d3385d9712a27356825e803848a3068fea82cf8ef2d126d3d9de609617
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - C++
  - OpenMS
  - Python
  - nanobind
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# nanobind-wrapper-generation

## Summary

Expose an existing C++ core library (e.g. OpenMS) to Python by writing and building nanobind binding files, so that Python users get direct access to the same data structures and algorithms as C++ developers.

## When to use

Use when a C++ library needs Python scripting access to its core data structures and algorithms for rapid prototyping, and the binding layer is nanobind.

## When NOT to use

- The library already ships working Python bindings that can be installed via conda/bioconda (e.g. pyopenms on bioconda) and no wrapper extension is needed
- The goal is standalone tool interoperability via CTD/parameter files or workflow engines rather than in-process scripting of the C++ API

## Inputs

- C++ source tree containing nanobind binding files (e.g. src/pyOpenMS/bindings/)
- Wrapping instructions file (e.g. src/pyOpenMS/CLAUDE.md)
- Native build toolchain configuration for the C++ core
- Small test data file such as an mzML file for the smoke test

## Outputs

- Compiled Python extension module (e.g. pyopenms) importable from Python
- Verified binding layer exposing core data structures (MSExperiment, PeakMap, FeatureMap) and algorithms
- Passing unit and functional test results for the built module

## How to apply

Check out the project source (OpenMS develop branch) and locate the binding templates in src/pyOpenMS/bindings/, following the wrapping instructions in src/pyOpenMS/CLAUDE.md. Configure and build the C++ core with a native toolchain, then generate and compile the nanobind-based Python extension module. Validate with an import smoke test that instantiates core structures such as MSExperiment, PeakMap and FeatureMap, then exercise a minimal scripted call — loading a small mzML file and iterating its spectra through the Python API — to confirm workflow scripting support, respecting the project's documented coding conventions. Finish by running the unit and functional tests against the built module, following the pattern of existing class tests such as the MSNumpressCoder test.

## Related tools

- **nanobind** (Binding generator used to create the Python extension module from the C++ API)
- **OpenMS** (The C++ core library being wrapped; provides MSExperiment, PeakMap, FeatureMap and the algorithms exposed through pyOpenMS) — https://github.com/OpenMS/OpenMS
- **Python** (Target scripting language; validated by import smoke tests and data-processing calls through the compiled module)
- **C++** (Source language of the core library whose classes and functions are exposed to Python via the generated bindings)

## Examples

```
import pyopenms as ms; exp = ms.MSExperiment(); ms.MzMLFile().load('small.mzML', exp); [s for s in exp.getSpectra()]
```

## Evaluation signals

- The compiled pyopenms module imports cleanly from Python without errors
- Core data structures MSExperiment, PeakMap and FeatureMap can be instantiated from the compiled module
- A small mzML file can be loaded and its spectra iterated through the Python API
- The unit and functional test suite for the built module passes without errors (following the pattern of existing class tests such as MSNumpressCoder_test.cpp)

## Limitations

- The binding layer must follow the project's own wrapping instructions (e.g. src/pyOpenMS/CLAUDE.md) and coding conventions, so the skill is project-specific rather than fully generic
- Building the C++ core natively is required before the Python extension can be generated and compiled, so a working native toolchain is a prerequisite

## Evidence

- [readme] pyOpenMS bindings: [evidence span withheld — anchor status: external_doc]
