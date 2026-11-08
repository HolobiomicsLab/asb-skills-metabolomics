---
name: core-data-model-reconstruction
description: Use when you must recover the architecture of a modular data-processing
  framework from source code — specifically how modules declare parameters, depend
  only on core types, and sequence tasks.
when_to_use_negative:
- You need to run an LC-MS processing pipeline rather than reconstruct its software
  architecture — use MZmine's batch mode directly instead
- 'The target is mzmine 3/4 (repo mzmine/mzmine): mzmine2 is archived and its README
  states all development moved to the new repository, so core classes may differ'
- Input is already a feature table or peak list output — this skill operates on source
  code, not analysis results
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MZmine 2
  role: Source framework whose core-module separation, parameter previews, and batch
    queue are reconstructed
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: Implementation language; core data model classes and module interface contracts
    are Java packages to enumerate
- name: Java Development Kit (JDK)
  role: Required to compile/inspect the mzmine2 source tree while tracing module dependencies
  repo: https://github.com/mzmine/mzmine2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:48e5cd22f7ae4d01404297cb09661009d19d18a3b6ef241cd7cafc994a89a54a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - Java Development Kit (JDK)
  - MZmine 2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# core-data-model-reconstruction

## Summary

Reconstructs the core-module separation architecture of a pluggable scientific framework (here, MZmine 2) by mining its Java source tree for the core data model, module interface contracts, parameter-preview linkage, and batch-queue semantics, emitted as a machine-readable architecture spec.

## When to use

Use when you must recover the architecture of a modular data-processing framework from source code — specifically how modules declare parameters, depend only on core types, and sequence tasks.

## When NOT to use

- You need to run an LC-MS processing pipeline rather than reconstruct its software architecture — use MZmine's batch mode directly instead
- The target is mzmine 3/4 (repo mzmine/mzmine): mzmine2 is archived and its README states all development moved to the new repository, so core classes may differ
- Input is already a feature table or peak list output — this skill operates on source code, not analysis results

## Inputs

- Java source tree of mzmine/mzmine2 (application-core packages)
- Raw MS data files processed by the framework (to trace data model accessors)
- The MZmine 2 paper (DOI 10.1186/1471-2105-11-395) for architectural claims

## Outputs

- architecture_spec.json with top-level sections core_data_model, module_interfaces, parameter_previews, batch_queue
- Catalog of public accessors of core data model classes (raw data, peak lists, features)
- Module descriptor / parameter-set interface contracts mapped to named source classes

## How to apply

Obtain the source tree (github.com/mzmine/mzmine2, Java/JDK per the README) and locate the application-core packages holding the data model, module registration, and task control. Enumerate the compact core data model classes (raw data, peak lists, features) and record their public accessors as fixed contracts, confirming that original unprocessed data are never discarded — MZmine 2's data import and peak detection modules were deliberately designed to maintain precision without degradation due to inadequate resampling. Extract the pluggable module interface contracts (module descriptor, parameter set declaration) and verify strict core/module separation by checking that module code depends only on core data model types. For parameterized methods such as peak picking, record the configurable parameters and their linkage to embedded visualization modules that provide online previews during parameter setup. Finally reconstruct the batch-mode task queue that sequences processing steps (raw data import, optional filtering/smoothing, peak picking, deisotoping, alignment, gap filling, normalization) over lists of raw data files supporting gigabyte-scale high-resolution datasets, and emit everything as architecture_spec.json.

## Related tools

- **MZmine 2** (Source framework whose core-module separation, parameter previews, and batch queue are reconstructed) — https://github.com/mzmine/mzmine2
- **Java** (Implementation language; core data model classes and module interface contracts are Java packages to enumerate)
- **Java Development Kit (JDK)** (Required to compile/inspect the mzmine2 source tree while tracing module dependencies) — https://github.com/mzmine/mzmine2

## Examples

```
git clone https://github.com/mzmine/mzmine2; cd mzmine2; ./gradlew
```

## Evaluation signals

- architecture_spec.json parses as valid JSON
- The spec contains all four required top-level sections: core_data_model, module_interfaces, parameter_previews, batch_queue
- Every interface entry is traceable to a named source class in the mzmine2 tree
- No module entry references types outside the core data model (strict core/module separation holds)
- Peak-picking module entry lists its configurable parameters plus the embedded visualization module providing online previews

## Limitations

- The mzmine2 repository is archived (GPL-2.0); its README states all development moved to the new mzmine repository, so findings apply only to the MZmine 2 generation
- The paper's modules were tested mainly on LC-MS data; reconstructed contracts may not cover GC-MS or proteomics search-engine (MASCOT) paths that are only described as future expansion
- No specific framework version number is stated in the paper ('current version' is referenced without a version identifier), so the spec cannot be pinned to a release

## Evidence

- [methods] parameter previews grounded in paper: "critical data processing methods such as peak picking were linked to embedded visualization modules, providing online previews during parameter setup."
- [intro] modular redesign motivation: "For this reason, the new release, MZmine 2, was completely redesigned to support modularity."
- [readme] build prerequisite from README: "MZmine development requires Java Development Kit (JDK) version 12 or newer (http://jdk.java.net)."
- [methods] precision-preserving design of core processing: "We designed the data import and peak detection modules to maintain the precision of the imported data without any degradation due to inadequate resampling."
- [readme] project scope statement: "provide a user-friendly, flexible and easily extendable software with a complete set of modules covering the entire MS data analysis workflow."
