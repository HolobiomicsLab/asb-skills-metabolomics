---
name: module-parameter-specification
description: Use when implementing a pluggable processing module (e.g. peak picking,
  filtering, alignment) that must expose its configurable parameters to the user and
  support online visual preview of the parameter effect before committing the step.
when_to_use_negative:
- The module needs to touch or mutate core data model internals beyond the public
  accessors — MZmine 2 enforces a strict core/module separation, so parameter specification
  must not depend on unprocessed raw data internals.
- The workflow step is fixed and parameter-free (e.g. plain data export); a parameter-set
  declaration with preview linkage adds no value there.
- Working in the successor mzmine codebase (mzmine/mzmine), where the original MZmine
  2 parameter-preview architecture is a legacy reference rather than the current contract.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: The pluggable framework whose modules declare configurable parameter sets
    linked to online previews during parameter setup
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: Implementation language for the module descriptors, parameter sets, and embedded
    visualization linkage (platform independent)
  repo: https://github.com/mzmine/mzmine2
- name: JFreeChart
  role: Renders the online parameter previews (TIC, spectra, 2D visualizers) during
    module parameter setup
- name: VisAD
  role: Provides the 3D visualizer used for online parameter previews
- name: Java Development Kit (JDK) version 12
  role: Minimum JDK for building and developing the module parameter code
  repo: https://github.com/mzmine/mzmine2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:18ec3bbba5e8091e73a14c7fa90d6a9aaa6ea77deea996ebe0ea10b32a707059
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - JFreeChart
  - Java
  - Java Development Kit (JDK) version 12
  - MZmine 2
  - VisAD
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# module-parameter-specification

## Summary

Declares the configurable parameter sets that MZmine-style pluggable modules expose to the user, including linkage to embedded visualization modules that provide online previews during parameter setup for critical methods such as peak picking.

## When to use

Use when implementing a pluggable processing module (e.g. peak picking, filtering, alignment) that must expose its configurable parameters to the user and support online visual preview of the parameter effect before committing the step.

## When NOT to use

- The module needs to touch or mutate core data model internals beyond the public accessors — MZmine 2 enforces a strict core/module separation, so parameter specification must not depend on unprocessed raw data internals.
- The workflow step is fixed and parameter-free (e.g. plain data export); a parameter-set declaration with preview linkage adds no value there.
- Working in the successor mzmine codebase (mzmine/mzmine), where the original MZmine 2 parameter-preview architecture is a legacy reference rather than the current contract.

## Inputs

- Pluggable module descriptor with its parameter-set declaration (Java, MZmine 2 core/module interface)
- Raw high-resolution MS data files (e.g. LC-MS raw imports) over which preview tasks will run
- Visualization preview data from core data model types (peak lists, features)

## Outputs

- Declared parameter set exposed to the user during workflow configuration
- Online parameter previews rendered via embedded visualization modules (TIC, spectra, 2D via JFreeChart; 3D via VisAD)
- Configured module tasks queued in the batch-mode task queue

## How to apply

Define each module's parameter set as a discrete parameter-set declaration that the module descriptor carries, keeping parameter code strictly inside the module and depending only on the core data model types (raw data, peak lists, features). For parameterized methods such as peak picking, bind each parameter set to an embedded visualization module — JFreeChart for TIC, spectra, and 2D previews, VisAD for the 3D previewer — so the user can inspect the expected effect and choose optimal parameter values before running the task. Rationale: MZmine 2 was completely redesigned for modularity, and critical data processing methods were deliberately linked to embedded visualization modules providing online previews during parameter setup, while data import and peak detection modules were designed to preserve the precision of imported data without degradation from inadequate resampling.

## Related tools

- **MZmine 2** (The pluggable framework whose modules declare configurable parameter sets linked to online previews during parameter setup) — https://github.com/mzmine/mzmine2
- **Java** (Implementation language for the module descriptors, parameter sets, and embedded visualization linkage (platform independent)) — https://github.com/mzmine/mzmine2
- **JFreeChart** (Renders the online parameter previews (TIC, spectra, 2D visualizers) during module parameter setup)
- **VisAD** (Provides the 3D visualizer used for online parameter previews)
- **Java Development Kit (JDK) version 12** (Minimum JDK for building and developing the module parameter code) — https://github.com/mzmine/mzmine2

## Examples

```
git clone https://github.com/mzmine/mzmine2 && ./gradlew   # build the MZmine 2 distribution (build/MZmine-version-platform.zip) containing the parameter-preview modules
```

## Evaluation signals

- Each parameterized module exposes a declared parameter set that the user can configure during workflow/batch setup, without the module depending on anything beyond the core data model's public accessors.
- For a representative parameterized method (e.g. peak picking), launching the parameter dialog opens an embedded visualization preview (TIC, spectra, 2D via JFreeChart, or 3D via VisAD) showing the effect of the chosen parameter values.
- Previews remain available for optimal selection of parameter values, as stated for peak detection's several customizable steps.
- The parameter specification does not resample or degrade imported high-resolution data — precision of the imported data is maintained.
- Configured module tasks are accepted by the batch-mode task queue that sequences processing steps over lists of raw data files.

## Limitations

- The currently developed modules were tested mainly on LC-MS data, so parameter-preview behavior is validated for that data type rather than for other dataset types.
- Gas chromatography-MS support and interoperation with proteomics search engines such as MASCOT are described only as possible future expansion, not demonstrated.
- The MZmine 2 repository is archived as a reference; the original core/module and parameter-preview architecture has been succeeded by the mzmine codebase.

## Evidence

- [methods] peak picking parameter preview: "critical data processing methods such as peak picking were linked to embedded visualization modules, providing online previews during parameter setup."
- [results] previews for optimal parameter values: [evidence span withheld — anchor status: not_found]
- [methods] platform-independent Java framework: "MZmine 2 was developed using Java technology, and is therefore platform independent."
- [methods] precision-preserving import and peak detection design: "We designed the data import and peak detection modules to maintain the precision of the imported data without any degradation due to inadequate resampling."
- [methods] visualizer library roles: "JFreeChart (http://www.jfree.org/jfreechart/) for the TIC, spectra, 2D and other visualizers"
- [readme] archived successor repository: "All development happens in a new repository: [mzmine GitHub](https://github.com/mzmine/mzmine3)"
