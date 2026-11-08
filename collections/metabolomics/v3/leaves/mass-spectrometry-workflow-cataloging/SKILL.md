---
name: mass-spectrometry-workflow-cataloging
description: Use when you need to inventory a mass spectrometry software release's
  tool/workflow set — e.g. counting TOPP tools and workflow entries from OpenMS release
  configuration files to reproduce or audit a published tool count.
when_to_use_negative:
- The goal is running an actual proteomics/metabolomics analysis (e.g. label-free
  quantitation, SILAC, iTRAQ, TMT, SRM, SWATH) rather than auditing which tools exist
- The platform's tools are not declared in parsable registration/config files — manual
  documentation scraping would be needed instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: OpenMS
  role: Source of the tool registration/configuration files being cataloged; the release
    platform whose 185 tools and workflows are enumerated
  repo: https://github.com/OpenMS/OpenMS
- name: Python
  role: Used to parse registration/config files and compile the classified tool inventory
    into CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:20f7b8651460e02471a090606146ac6f8414d2fc752a9dbcea868f1df4ad8b43
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - OpenMS
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# mass-spectrometry-workflow-cataloging

## Summary

Systematically enumerates and classifies the pre-built tools and ready-made workflows shipped with an MS data-analysis platform (here OpenMS 2.0's set of 185 tools), separating native TOPP tools from adaptors that wrap external software.

## When to use

Use when you need to inventory a mass spectrometry software release's tool/workflow set — e.g. counting TOPP tools and workflow entries from OpenMS release configuration files to reproduce or audit a published tool count.

## When NOT to use

- The goal is running an actual proteomics/metabolomics analysis (e.g. label-free quantitation, SILAC, iTRAQ, TMT, SRM, SWATH) rather than auditing which tools exist
- The platform's tools are not declared in parsable registration/config files — manual documentation scraping would be needed instead

## Inputs

- OpenMS release source tree from github.com/OpenMS/OpenMS (tool registration and configuration files)
- Tool registration/config files declaring tool name, class, and category

## Outputs

- CSV table with columns: tool name, type, category
- Validated count of tool/workflow entries (185 for OpenMS 2.0)

## How to apply

Obtain the release source (github.com/OpenMS/OpenMS at the 2.0 tag) and locate the tool registration and configuration files that declare each TOPP tool. Parse those files programmatically with Python to extract each entry's tool name, class, and declared category. Classify each entry as a native TOPP tool versus an adaptor/utility wrapping external software (e.g. adapters to tools like Comet), noting that TOPPTools support integration into workflow engines such as nextflow, KNIME, Galaxy, and TOPPAS via the unified 'common tool description' (CTD) parameter scheme. Compile the classified entries into a structured CSV (columns: tool name, type, category) and validate that the inventory contains 185 entries, matching the count reported in the OpenMS 2.0 abstract. These tools carry out common MS data processing tasks: file conversion, centroiding, database searching, feature detection, retention time alignment, cross-run feature linking, protein inference, and statistical error control.

## Related tools

- **OpenMS** (Source of the tool registration/configuration files being cataloged; the release platform whose 185 tools and workflows are enumerated) — https://github.com/OpenMS/OpenMS
- **Python** (Used to parse registration/config files and compile the classified tool inventory into CSV)

## Examples

```
python parse_tool_configs.py --repo-path OpenMS --release 2.0 --out tools_inventory.csv --expect-count 185
```

## Evaluation signals

- Assembled inventory contains exactly 185 tool/workflow entries, matching the count in the OpenMS 2.0 abstract
- Every CSV row has non-empty values for all three columns (tool name, type, category)
- Each entry is assigned exactly one class: native TOPP tool vs adaptor/utility wrapping external software
- Covered task categories include file conversion, centroiding, database searching, feature detection, retention time alignment, cross-run feature linking, protein inference, and statistical error control

## Limitations

- The count is release-specific: the OpenMS 2.0 abstract reports 185 tools, while the current README describes 'Over 150+ individual analysis tools (TOPP Tools)', so the inventory must be pinned to a release tag
- Adaptor entries wrap external software (e.g. Comet), so a raw count conflates native algorithms with third-party tool wrappers
- Tool registration/config file locations and formats may change between releases, requiring re-verification of the parsing step

## Evidence

- [readme] README TOPPTools integration: [evidence span withheld — anchor status: external_doc]
- [readme] README adapters: [evidence span withheld — anchor status: external_doc]
- [readme] README tool count: [evidence span withheld — anchor status: external_doc]
