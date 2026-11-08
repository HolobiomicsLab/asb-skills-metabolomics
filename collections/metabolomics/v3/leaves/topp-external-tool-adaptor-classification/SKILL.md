---
name: topp-external-tool-adaptor-classification
description: Use when you need to enumerate and classify OpenMS TOPP tools from a
  pinned release — e.g. to verify the '185 tools and ready-made workflows' claim or
  to identify which entries are external-tool adapters versus native implementations.
when_to_use_negative:
- The target is a different OpenMS version — the current README states 'Over 150+
  individual analysis tools (TOPP Tools)', so the 185 count is specific to the 2.0
  release configuration and will not match
- You want to execute a mass-spectrometry analysis (e.g. database search with an adapter
  like Comet) rather than inventory and classify the tool set
- Input is already a processed tool list or feature table rather than raw tool registration/config
  files from the release source
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: OpenMS 2.0
  role: Source platform whose release registration/configuration files are inventoried
    and whose TOPPTools/CTD scheme defines the tool classes being classified
  repo: https://github.com/OpenMS/OpenMS
- name: Python
  role: Parsing the release's tool registration/config files and emitting the classified
    CSV inventory
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:460cd47b614bbb46f8041f12a5b6c75d8760d38070b683107dc7d4f1ddf27d55
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - OpenMS 2.0
  - Python
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# topp-external-tool-adaptor-classification

## Summary

Reconstructs and classifies the full TOPP tool/workflow inventory of a specific OpenMS release (2.0: 185 tools and ready-made workflows), separating native TOPP tools from adaptors that wrap external software such as Comet.

## When to use

Use when you need to enumerate and classify OpenMS TOPP tools from a pinned release — e.g. to verify the '185 tools and ready-made workflows' claim or to identify which entries are external-tool adapters versus native implementations.

## When NOT to use

- The target is a different OpenMS version — the current README states 'Over 150+ individual analysis tools (TOPP Tools)', so the 185 count is specific to the 2.0 release configuration and will not match
- You want to execute a mass-spectrometry analysis (e.g. database search with an adapter like Comet) rather than inventory and classify the tool set
- Input is already a processed tool list or feature table rather than raw tool registration/config files from the release source

## Inputs

- OpenMS 2.0 release source tree from github.com/OpenMS/OpenMS (tool registration and configuration files)
- TOPP tool CTD ('common tool description') parameter files declaring tool name, class, and category
- Reference claim to validate: 185 tools and ready-made workflows reported in the OpenMS 2.0 abstract

## Outputs

- Structured CSV inventory with columns: tool name, type (native TOPP tool vs adaptor/utility wrapping external software), category
- Row count of 185 tool/workflow entries covering file conversion through statistical error control

## How to apply

Check out the OpenMS 2.0 release source from github.com/OpenMS/OpenMS and locate the tool registration/configuration files of that release, which include the 'common tool description' (CTD) scheme that TOPPTools uses for unified parameter handling. Parse these files with Python to extract each entry's tool name, class, and declared category. Classify each entry as a native TOPP tool versus an adaptor or utility that wraps external state-of-the-art software (the README names Comet as an example of an adapted tool), covering the stages the abstract enumerates: file conversion, centroiding, database searching, feature detection, retention time alignment, cross-run feature linking, protein inference, and statistical error control. Compile the classified entries into a structured CSV with columns (tool name, type, category) and validate that the inventory contains exactly 185 entries, matching the OpenMS 2.0 abstract.

## Related tools

- **OpenMS 2.0** (Source platform whose release registration/configuration files are inventoried and whose TOPPTools/CTD scheme defines the tool classes being classified) — https://github.com/OpenMS/OpenMS
- **Python** (Parsing the release's tool registration/config files and emitting the classified CSV inventory)

## Examples

```
python parse_topp_tools.py --src OpenMS-2.0 --csv topp_tools.csv --expect-count 185
```

## Evaluation signals

- Row count of the compiled CSV equals exactly 185 entries, matching the count reported in the OpenMS 2.0 abstract
- Every CSV row has non-empty tool name, type, and category fields extracted from the registration/config files
- Each row's type is one of the two classes: native TOPP tool vs adaptor/utility wrapping external software (e.g. Comet adapter tagged as external)
- The inventory spans all abstract-declared stages: file conversion, centroiding, database searching, feature detection, retention time alignment, cross-run feature linking, protein inference, and statistical error control
- Classification agrees with the README's TOPPTools concept: tools integrating into workflow engines (nextflow, KNIME, Galaxy, TOPPAS) via CTD appear as first-class entries

## Limitations

- The 185-entry count is version-specific to OpenMS 2.0; the current repository README reports 'Over 150+ individual analysis tools (TOPP Tools)', so re-running on another release yields a different inventory
- Native-vs-adaptor classification depends on the declared class/category in the release's registration and config files; wrappers of external state-of-the-art tools (like Comet) must be recognized as such, and mislabeled categories propagate into the CSV
- The inventory enumerates tools and ready-made workflows at the configuration level; it does not validate that each tool executes correctly on real MS data

## Evidence

- [readme] External-tool adapters exist as a distinct entry class (Comet example): [evidence span withheld — anchor status: external_doc]
- [readme] TOPPTools concept and CTD scheme define the tool registration being parsed: [evidence span withheld — anchor status: external_doc]
