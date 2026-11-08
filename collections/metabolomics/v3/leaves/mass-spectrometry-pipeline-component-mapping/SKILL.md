---
name: mass-spectrometry-pipeline-component-mapping
description: Use when you need to decompose an MS analysis framework (e.g. OpenMS)
  into its constituent layers and components — such as deciding whether to extend
  the C++ API, wrap a TOPP tool, or reuse a pre-built workflow.
when_to_use_negative:
- You need to actually run proteomics analyses — this skill inventories the architecture,
  it does not execute workflows or process mzML data.
- The artifact under study is not a layered software framework (e.g. a single standalone
  script or a command-line one-tool pipeline) — layer decomposition would be forced.
- 'Tool counts are needed for a specific release without checking the corresponding
  version: counts differ across releases (185 tools in the OpenMS 2.0 paper vs. 150+
  TOPP Tools in the current README).'
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: OpenMS
  role: The framework being decomposed; its core library, TOPPTools, TOPPView, and
    pyOpenMS bindings constitute the layers being mapped
  repo: https://github.com/OpenMS/OpenMS
- name: C++
  role: Language of the core OpenMS API layer (1,300+ classes); layer boundaries follow
    its API surface
- name: Python
  role: pyOpenMS bindings expose the C++ API as the Python-facing layer; nanobind
    files under src/pyOpenMS/bindings/ enumerate the bound surface
- name: nanobind
  role: Binding generator whose files under src/pyOpenMS/bindings/ provide the inventory
    of the Python binding layer
- name: Doxygen
  role: API documentation source for verifying the extent of the core C++ layer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:e080ab2309578dc59fb077101e98bb3723c72dbda2879e8b19699321674062e6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - C++
  - Doxygen
  - OpenMS
  - Python
  - nanobind
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mass-spectrometry-pipeline-component-mapping

## Summary

Maps the layered architecture of a mass spectrometry analysis framework (data/signal libraries, core C++ API, TOPP tool executables, ready-made workflows) by combining article text with repository module layout. Produces a dependency-ordered inventory so future agents know which layer to extend or invoke.

## When to use

Use when you need to decompose an MS analysis framework (e.g. OpenMS) into its constituent layers and components — such as deciding whether to extend the C++ API, wrap a TOPP tool, or reuse a pre-built workflow.

## When NOT to use

- You need to actually run proteomics analyses — this skill inventories the architecture, it does not execute workflows or process mzML data.
- The artifact under study is not a layered software framework (e.g. a single standalone script or a command-line one-tool pipeline) — layer decomposition would be forced.
- Tool counts are needed for a specific release without checking the corresponding version: counts differ across releases (185 tools in the OpenMS 2.0 paper vs. 150+ TOPP Tools in the current README).

## Inputs

- Article text (abstract and methods sections describing OpenMS 2.0 software design)
- Repository module layout from github.com/OpenMS/OpenMS
- Project README with the Features section
- Python bindings inventory (nanobind files under src/pyOpenMS/bindings/)

## Outputs

- openms_architecture_layers.csv with columns rank, layer name, constituent components, description (one row per documented layer in dependency order)

## How to apply

Load the article's design sections (abstract and methods describing OpenMS 2.0's software design) together with the repository module layout from github.com/OpenMS/OpenMS. Identify the documented constituents: standardized open data format support (mzML, mzXML, mzIdentXML, pepXML, mzTab) and the numeric/XML/sequence/statistics utility libraries; the well-defined core C++ API (over 1,300 classes built on modern C++); the TOPP executable layer (185 tools covering file conversion, centroiding, database searching, feature detection, retention time alignment, cross-run feature linking, protein inference, and statistical error control); and the pre-assembled workflow layer. Cross-check tool counts and layer descriptions against the README's Features section, noting that the current README reports 150+ TOPP tools for the OpenMS 3 series. Order layers bottom-up by dependency (data/signal processing libraries → core C++ library → TOPP executables → workflows) and write one row per layer with rank, layer name, constituents, and description. Validate that each documented layer has a row and that counts are attributed to the correct software version.

## Related tools

- **OpenMS** (The framework being decomposed; its core library, TOPPTools, TOPPView, and pyOpenMS bindings constitute the layers being mapped) — https://github.com/OpenMS/OpenMS
- **C++** (Language of the core OpenMS API layer (1,300+ classes); layer boundaries follow its API surface)
- **Python** (pyOpenMS bindings expose the C++ API as the Python-facing layer; nanobind files under src/pyOpenMS/bindings/ enumerate the bound surface)
- **nanobind** (Binding generator whose files under src/pyOpenMS/bindings/ provide the inventory of the Python binding layer)
- **Doxygen** (API documentation source for verifying the extent of the core C++ layer)

## Examples

```
python scripts/build_layer_inventory.py --article-text openms_2.0.txt --repo github.com/OpenMS/OpenMS --out openms_architecture_layers.csv
```

## Evaluation signals

- The CSV contains one row for each of the four documented layers (data/signal libraries, core C++ library, TOPP executables, workflows) in bottom-up dependency order.
- The workflow-layer row is consistent with the reported tool count (185 tools in the OpenMS 2.0 paper; 150+ TOPP Tools in the current README) and the C++/Python API description.
- Tool-count figures are attributed to the correct software version rather than mixed across the OpenMS 2.0 paper and the OpenMS 3 README.
- Each layer row's constituent components are grounded in article text or README Features (e.g. mzML/mzXML/mzIdentXML/pepXML/mzTab format support, TOPPView visualization).
- The CSV parses as valid with the expected rank/name/components/description columns and no empty layer names.

## Limitations

- Tool counts are version-dependent: the paper reports 185 tools for OpenMS 2.0, while the current README reports 150+ TOPP Tools for the OpenMS 3 series.
- The paper text and the README describe different releases (OpenMS 2.0 vs. OpenMS 3, which cites Pfeuffer et al. 2024 and modern C++23), so cross-source layer details must be attributed carefully.
- The README is current-project documentation that may drift from the 2016 article's described architecture; statements should cite their source.

## Evidence

- [abstract] OpenMS is a flexible, user-friendly, open-source software platform for the biological analysis of mass spectrometry proteomics and metabolomics data.: "OpenMS is a flexible, user-friendly, open-source software platform for the biological analysis of mass spectrometry proteomics and metabolomics data."
- [abstract] 185 tools and ready-made workflow: "OpenMS additionally provides a set of 185 tools and ready-made workflow to carry out common mass spectrometric data processing tasks"
- [readme] Over 150+ individual analysis tools (TOPP Tools): [evidence span withheld — anchor status: external_doc]
- [readme] Core C++ library under three-clause BSD licence using modern C++23: [evidence span withheld — anchor status: external_doc]
