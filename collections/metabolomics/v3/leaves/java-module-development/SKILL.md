---
name: java-module-development
description: Use when a new MS data-processing step (e.g., the RANSAC peak list alignment)
  must be added to MZmine 2 as a Java module, or when building the framework from
  source is required.
when_to_use_negative:
- The goal is to develop against the current mzmine (mzmine3) codebase — that repository
  requires JDK 23 or newer and is MIT-licensed, so the mzmine2-specific instructions
  here do not apply
- The task only requires running MZmine 2 as an end user — no module development or
  source build is needed since mzmine packages its own runtime
- The new method cannot be expressed within the existing core/module separation of
  MZmine and would require modifying the application core
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MZmine 2
  role: Framework being extended; the new module plugs into its core and follows its
    modular architecture
  repo: https://github.com/mzmine/mzmine2
- name: Java Development Kit (JDK)
  role: Toolchain requirement; MZmine development requires JDK version 12 or newer
- name: google/styleguide
  role: Code formatting convention (Google Java Style Guide) that contributed module
    code must adhere to
  repo: https://github.com/google/styleguide
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:15020216ab63bdc0021143a13cc517a1c35dc28ded6ff8041b44938828ad7741
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - Java Development Kit (JDK)
  - MZmine 2
  - google/styleguide
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# Java module development

## Summary

Develop and build a new data-processing module for the MZmine 2 framework in Java, respecting the strict separation between the application core and individual modules, and compiling the framework from source with Gradle.

## When to use

Use when a new MS data-processing step (e.g., the RANSAC peak list alignment) must be added to MZmine 2 as a Java module, or when building the framework from source is required.

## When NOT to use

- The goal is to develop against the current mzmine (mzmine3) codebase — that repository requires JDK 23 or newer and is MIT-licensed, so the mzmine2-specific instructions here do not apply
- The task only requires running MZmine 2 as an end user — no module development or source build is needed since mzmine packages its own runtime
- The new method cannot be expressed within the existing core/module separation of MZmine and would require modifying the application core

## Inputs

- mzmine2 source tree from github.com/mzmine/mzmine2
- Java Development Kit (JDK) version 12 or newer
- Module design specification (e.g., alignment-window and RANSAC-window parameter definitions for the RANSAC aligner)

## Outputs

- Compiled MZmine distribution in build/MZmine-version-platform.zip
- A new Java module integrated with the MZmine 2 core that exposes its own parameter setup
- Module outputs such as an aligned peak list (e.g., RANSAC-aligned peak lists for precision/recall evaluation)

## How to apply

Obtain the mzmine2 source tree (https://github.com/mzmine/mzmine2, GPL-2.0, now archived — new development happens in https://github.com/mzmine/mzmine3) and install JDK version 12 or newer. Build the package from the sources with ./gradlew or gradlew.bat; the final distribution is placed in build/MZmine-version-platform.zip. Implement the new module in Java so it plugs into the core without modifying it, exposing its user-facing parameters through a parameter setup component — for example the RANSAC aligner exposes the alignment-window (AW: m/z threshold rm0 and RT threshold ar0) and RANSAC-window (RW: rm0 and rr0) settings, with embedded visualizers providing online previews during parameter setup. Follow the Google Java Style Guide (google/styleguide) and write JavaDoc comments as full sentences starting with a capital letter and ending with a period; report problems at https://github.com/mzmine/mzmine2/issues.

## Related tools

- **MZmine 2** (Framework being extended; the new module plugs into its core and follows its modular architecture) — https://github.com/mzmine/mzmine2
- **Java Development Kit (JDK)** (Toolchain requirement; MZmine development requires JDK version 12 or newer)
- **google/styleguide** (Code formatting convention (Google Java Style Guide) that contributed module code must adhere to) — https://github.com/google/styleguide

## Examples

```
git clone https://github.com/mzmine/mzmine2 && cd mzmine2 && ./gradlew
```

## Evaluation signals

- ./gradlew (or gradlew.bat) completes successfully and the distribution appears in build/MZmine-version-platform.zip
- The new module compiles without modifying the application core, honoring the core/module separation
- The module's parameter setup exposes all user-set parameters (e.g., AW with rm0/ar0 and RW with rm0/rr0 for the RANSAC aligner) with online preview during setup
- Running the module produces the expected output artifact (e.g., an aligned peak list) that can be scored against ground-truth alignment tables
- Code passes style checks: Google Java Style formatting and full-sentence JavaDoc comments

## Limitations

- The mzmine2 repository is archived — 'All development happens in a new repository' (mzmine3) — so mzmine2-specific module work is legacy-only
- JDK requirements differ between versions: mzmine2 needs JDK 12+, while the current mzmine needs JDK 23 or newer
- Newly developed modules were tested mainly on LC-MS data, so validation on other data types is not demonstrated
- If problems are encountered during building, the documented recourse is posting an issue to the project's GitHub issues tracker

## Evidence

- [methods] MZmine 2 was developed using Java technology, and is therefore platform independent.: "MZmine 2 was developed using Java technology, and is therefore platform independent."
- [methods] critical data processing methods such as peak picking were linked to embedded visualization modules: "critical data processing methods such as peak picking were linked to embedded visualization modules, providing online previews during parameter setup."
- [readme] To build the MZmine package from the sources, run the following command: ./gradlew or gradlew.bat: "To build the MZmine package from the sources, run the following command:

    ./gradlew"
- [readme] MZmine development requires Java Development Kit (JDK) version 12 or newer (http://jdk.java.net).: "MZmine development requires Java Development Kit (JDK) version 12 or newer (http://jdk.java.net)."
- [readme] Please write JavaDoc comments as full sentences, starting with a capital letter and ending with a period.: "Please write JavaDoc comments as full sentences, starting with a capital letter and ending with a period."
- [readme] All development happens in a new repository: "All development happens in a new repository: [mzmine GitHub](https://github.com/mzmine/mzmine3)"
