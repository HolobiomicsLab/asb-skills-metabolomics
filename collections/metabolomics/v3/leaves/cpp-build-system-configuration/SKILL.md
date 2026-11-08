---
name: cpp-build-system-configuration
description: Use when pyOpenMS or TOPP tools must be compiled from source, e.g. no
  pre-built wheel exists for the platform, a develop-branch feature is needed, or
  Python bindings to MSExperiment/FeatureMap must be built against a locally compiled
  OpenMS core.
when_to_use_negative:
- A pre-built pyOpenMS wheel or conda package from bioconda suffices and no develop-branch
  code is needed
- The goal is only to run existing TOPP tools or KNIME/Galaxy workflows without compiling
  anything
- The task targets another C++ project whose build system is unrelated to OpenMS's
  CMake/toolchain setup
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3520
tools:
- name: OpenMS
  role: The C++ core library being configured and built from source
  repo: github.com/OpenMS/OpenMS
- name: nanobind
  role: Binding generator for the pyOpenMS extension module (bindings in src/pyOpenMS/bindings/)
  repo: github.com/OpenMS/OpenMS
- name: Python
  role: Target language for the nanobind-compiled bindings; used for import smoke
    tests and scripted data-processing validation
- name: Doxygen
  role: Generates the advanced developer API documentation expected for contributions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:3f166d18618677a6eb2b710512dcb04ab6e75d829dc90a6675b96f722de711cf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C++
  - Doxygen
  - OpenMS
  - Python
  - nanobind
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cpp-build-system-configuration

## Summary

Configures and builds the OpenMS C++ core library (BSD 3-Clause, modern C++23) from source with a native toolchain so downstream applications — including the nanobind-based pyOpenMS bindings and TOPP tools — can be compiled and validated. It covers platform-specific build setup on Windows, macOS, and Linux.

## When to use

Use when pyOpenMS or TOPP tools must be compiled from source, e.g. no pre-built wheel exists for the platform, a develop-branch feature is needed, or Python bindings to MSExperiment/FeatureMap must be built against a locally compiled OpenMS core.

## When NOT to use

- A pre-built pyOpenMS wheel or conda package from bioconda suffices and no develop-branch code is needed
- The goal is only to run existing TOPP tools or KNIME/Galaxy workflows without compiling anything
- The task targets another C++ project whose build system is unrelated to OpenMS's CMake/toolchain setup

## Inputs

- OpenMS source checkout (github.com/OpenMS/OpenMS, develop branch)
- Native C++ toolchain per platform (Linux/Windows/macOS build instructions)
- Python interpreter with nanobind for binding generation
- src/pyOpenMS/CLAUDE.md wrapping instructions

## Outputs

- Compiled OpenMS C++ core library
- pyOpenMS Python extension module (importable as pyopenms)
- Passing unit and functional test suites
- Doxygen API documentation

## How to apply

Check out the OpenMS repository (develop branch for bindings work) and consult the platform-specific build pages: install_linux.html, install_win.html, and install_mac.html on the abibuilder.cs.uni-tuebingen.de documentation archive. Configure the C++ core with a native toolchain under the project's three-clause BSD licence using modern C++23, then generate and compile the nanobind-based Python extension for pyOpenMS from src/pyOpenMS/bindings/, following the wrapping instructions in src/pyOpenMS/CLAUDE.md. Run an import smoke test that instantiates core data structures (MSExperiment, PeakMap, FeatureMap) and exercise a minimal scripted call (loading a small mzML file and iterating spectra) to confirm the build. Finally run the unit and functional test suites — following the pattern of class tests such as the MSNumpressCoder test — and validate the build with Doxygen documentation if contributing.

## Related tools

- **OpenMS** (The C++ core library being configured and built from source) — github.com/OpenMS/OpenMS
- **nanobind** (Binding generator for the pyOpenMS extension module (bindings in src/pyOpenMS/bindings/)) — github.com/OpenMS/OpenMS
- **Python** (Target language for the nanobind-compiled bindings; used for import smoke tests and scripted data-processing validation)
- **Doxygen** (Generates the advanced developer API documentation expected for contributions)

## Examples

```
cmake -DCMAKE_BUILD_TYPE=Release ../OpenMS && make -j$(nproc) && python -c "import pyopenms; exp = pyopenms.MSExperiment(); pyopenms.MzMLFile().load('small.mzML', exp); print(len(exp))"
```

## Evaluation signals

- pyopenms module imports cleanly in a fresh Python session
- Instantiation of MSExperiment, PeakMap, and FeatureMap succeeds without import or ABI errors
- A minimal mzML load-and-iterate call through the Python API completes and yields expected spectrum counts
- Unit and functional tests pass (e.g. the MSNumpressCoder_test.cpp pattern)
- Doxygen documentation builds without warnings for new code

## Limitations

- Build instructions differ per platform (Windows, macOS, Linux) and must be followed from the platform-specific pages
- Python bindings cover 'a large part' of the OpenMS API, not necessarily every class or algorithm
- The develop branch carries binding changes; release-branch builds may lack current nanobind bindings files
- Building from source is only recommended 'for developers who want to build OpenMS from source' — end-users should prefer conda/bioconda or pre-built releases

## Evidence

- [readme] Building OpenMS from source: "For developers who want to build OpenMS from source:

- [Build on Linux](https://abibuilder.cs.uni-tuebingen.de/archive/openms/Documentation/release/latest/html/install_linux.html)"
- [readme] pyOpenMS bindings location: "With pyOpenMS, OpenMS offers Python bindings to a large part of the OpenMS API
to enable rapid algorithm development."
- [methods] Python bindings build files: "have Python bindings — nanobind binding files are in `src/pyOpenMS/bindings/`; see `src/pyOpenMS/CLAUDE.md` for wrapping instructions"
- [methods] unit and functional tests: "have [unit tests and functional tests](https://openms.readthedocs.io/en/latest/manual/develop.html#automated-unit-tests)"
- [readme] OpenMS core language and licence: "OpenMS
is an open-source software C++ library for LC-MS data management and
analyses."
- [readme] Doxygen documentation requirement: [evidence span withheld — anchor status: external_doc]
