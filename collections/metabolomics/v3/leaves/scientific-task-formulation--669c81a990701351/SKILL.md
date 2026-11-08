---
name: scientific-task-formulation
description: Use when a research question must be converted into a concrete computational
  task with named tools, input formats (e.g. mzML, isotope-labeled LC-MS data), and
  verifiable outputs before any analysis is attempted.
when_to_use_negative:
- When the input is already a fully specified pipeline command or workflow file —
  formulation is complete and this skill would only re-describe it.
- When the research question can be answered by reading a single paper finding without
  running any tool; no bounded operation is needed.
- When the only available article content describes repository-contribution mechanics
  (opening issues, pull requests) rather than a scientific analysis — these are excluded
  as non-scientific setup per the source card.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: OpenMS
  role: Provides the tool-chain and API (C++ and Python) that the formulated task
    must scope to; its 185 ready-made tools and workflow steps define what a bounded
    operation can look like.
  repo: https://github.com/OpenMS/OpenMS
- name: pyOpenMS
  role: Python bindings to the OpenMS API; enables writing the concrete script a formulated
    task would invoke for feature detection, retention-time alignment, or database
    searching.
  repo: https://github.com/OpenMS/OpenMS
- name: TOPPTools
  role: The ready-to-use tool set whose individual TOPP tools (e.g. feature detection,
    database search) a formulated task must name as the executable component.
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:68625023186860c35c767fcd34abc55e26d9fa80137d73f91fe550dc6336846e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - OpenMS
  - TOPPTools
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# scientific_task_formulation

## Summary

Formulates a bounded, executable scientific task from a research question by scoping it to the tools, input data formats, and measurable outputs described in a source article, so a downstream agent can run the analysis rather than only describe it.

## When to use

Use when a research question must be converted into a concrete computational task with named tools, input formats (e.g. mzML, isotope-labeled LC-MS data), and verifiable outputs before any analysis is attempted.

## When NOT to use

- When the input is already a fully specified pipeline command or workflow file — formulation is complete and this skill would only re-describe it.
- When the research question can be answered by reading a single paper finding without running any tool; no bounded operation is needed.
- When the only available article content describes repository-contribution mechanics (opening issues, pull requests) rather than a scientific analysis — these are excluded as non-scientific setup per the source card.

## Inputs

- A research question expressed in natural language
- Article metadata: DOI, authors, journal (e.g. Nature Methods 2016, DOI 10.1038/nmeth.3959)
- Extracted tool names (e.g. OpenMS, pyOpenMS, TOPPTools)
- Extracted workflow steps (e.g. file conversion, centroiding, database searching, feature detection, retention time alignment)
- Extracted findings or source cards (e.g. '23 novel cathepsin L substrates' from task_006)

## Outputs

- A bounded scientific-task specification: one tool, one input data type, one parameter scope, one expected output object
- An explicit success criterion grounded in the article's findings (e.g. number of N-termini quantified, list of novel protease substrates)
- A short rationale explaining why the chosen tool and inputs can answer the research question
- Exclusion notes for non-scientific setup mechanics (GitHub issues, pull requests) that must not be treated as task content

## How to apply

1) Read the article's abstract and tool descriptions to identify which named tools (e.g. OpenMS 2.0, pyOpenMS, TOPPTools) and which ready-made workflow steps (file conversion, centroiding, database searching, feature detection, retention time alignment, protein inference) can address the research question. 2) Locate the concrete input data referenced in the article — for example, the FFPE degradomics specimens with stable-isotope-labeled N-termini quantified in the Lai et al. card — and record their file format and experimental design. 3) Define a bounded operation: choose a specific tool or tool-chain (such as an OpenMS TOPP tool for feature detection with knock-out correction, or a pyOpenMS script using the C++/Python API), specify its input file type, parameter scope, and the expected output object (e.g. a peptide feature table, protein N-termini list, or protease substrate set). 4) State success criteria grounded in the article's own findings — e.g. 'resolved thousands of protein N-termini' or 'identified 23 novel cathepsin L substrates' — so the agent can verify the task was executed rather than merely described. 5) Reject formulations that rely on non-scientific setup mechanics (opening GitHub issues, writing unit tests, pull-request review) because those do not constitute a bounded computational operation, as the source card for task_006 explicitly excluded them.

## Related tools

- **OpenMS** (Provides the tool-chain and API (C++ and Python) that the formulated task must scope to; its 185 ready-made tools and workflow steps define what a bounded operation can look like.) — https://github.com/OpenMS/OpenMS
- **pyOpenMS** (Python bindings to the OpenMS API; enables writing the concrete script a formulated task would invoke for feature detection, retention-time alignment, or database searching.) — https://github.com/OpenMS/OpenMS
- **TOPPTools** (The ready-to-use tool set whose individual TOPP tools (e.g. feature detection, database search) a formulated task must name as the executable component.) — https://github.com/OpenMS/OpenMS

## Examples

```
from pyopenms import MSExperiment, MzMLFile, FeatureFinderAlgorithmPicked; exp = MSExperiment(); MzMLFile().load('sample.mzML', exp); FeatureFinderAlgorithmPicked().run(exp, ...)
```

## Evaluation signals

- The formulation names at least one concrete tool from the article (e.g. OpenMS, pyOpenMS) and at least one input file type (e.g. mzML, isotope-labeled LC-MS data).
- The formulation states a measurable output — e.g. 'a table of protein N-termini' or 'a set of 23 novel cathepsin L substrates' — that maps to a finding in the article or source card.
- The formulation contains no setup-only content: GitHub issues, pull requests, unit tests, or documentation edits are absent from the task body.
- A downstream agent could, from the formulation alone, identify the command to run (e.g. a TOPP tool name with its CTD parameters) without needing to re-read the article.
- The success criterion is phrased in the article's own terminology (e.g. 'knock-out situations', 'feature detection correction', 'thousands of protein N-termini').

## Limitations

- If the source card explicitly states 'No workflow emitted' and the article text does not ground a concrete input dataset or parameter setting, no executable task can be formulated; the skill must return an honest exclusion rather than invent one.
- Formulation depends entirely on what the article exposes: OpenMS 2.0 describes 185 tools but does not list every parameter, so a formulated task may still require consulting external documentation for exact thresholds.
- Repository mechanics (issue filing, PR review) are never valid task content; mixing them into a formulation invalidates the bounded operation.
- When the article only names a tool family (e.g. '185 tools') without a specific tool for the research question, the formulation must remain at the tool-chain level and cannot specify a single command.

## Evidence

- [readme] pyOpenMS documentation: [evidence span withheld — anchor status: external_doc]
- [readme] Over 150+ individual analysis tools (TOPP Tools): [evidence span withheld — anchor status: external_doc]
- [readme] Support for most MS identification and quantification workflows: [evidence span withheld — anchor status: external_doc]
