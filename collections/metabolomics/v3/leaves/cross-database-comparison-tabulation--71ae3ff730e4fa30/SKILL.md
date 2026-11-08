---
name: cross-database-comparison-tabulation
description: Use when a source article discusses multiple tools or databases side
  by side with respect to one capability (e.g., programmatic sample-level meta-analysis)
  and the design choices behind each must be compared and valenced in a tabular output.
when_to_use_negative:
- The article discusses only one tool with no side-by-side comparison, so no cross-tool
  valence contrast is possible.
- The authors describe tools only generically without attributing specific design
  choices (e.g., vocabulary constraints, API availability) that can be individually
  classified.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMetaS
  role: Source article whose discussion of Metabolomics Workbench, ReDU, and MetaboLights
    supplies the design choices being tabulated; its own vocabulary-constraint design
    parallels ReDU's favorable string-equality pattern.
  repo: https://github.com/metabolomics-us/metadatastandardizer
- name: manual expert review
  role: Method used to locate the relevant passages, extract individual design choices
    per tool, and assign favorable/hindering valence.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:2f9a838ce0c62c11afe8a3580046260125f4ba5f58a6ffce269ae809ba1fc3c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-us/metadatastandardizer
  tools:
  - SMetaS
  - manual expert review
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-database-comparison-tabulation

## Summary

Extracts and tabulates individual design choices that authors attribute to each of several related tools/databases (here Metabolomics Workbench, ReDU, and MetaboLights), classifying each choice as favorable or hindering for a stated purpose such as programmatic sample-level meta-analysis.

## When to use

Use when a source article discusses multiple tools or databases side by side with respect to one capability (e.g., programmatic sample-level meta-analysis) and the design choices behind each must be compared and valenced in a tabular output.

## When NOT to use

- The article discusses only one tool with no side-by-side comparison, so no cross-tool valence contrast is possible.
- The authors describe tools only generically without attributing specific design choices (e.g., vocabulary constraints, API availability) that can be individually classified.

## Inputs

- Full-text or section text of the source article discussing the compared tools (e.g., the SMetaS article's passages on MS Workbench, ReDU, and MetaboLights)
- Target capability statement (e.g., programmatic sample-level meta-analysis)

## Outputs

- tool_design_choices_comparison.csv with one row per tool-design-choice pair and columns: tool name, design choice description, valence (favorable/hindering)

## How to apply

Locate the passages of the source article (e.g., SMetaS, DOI 10.3390/metabo13080941) that discuss each target tool — here MS Workbench, ReDU, and MetaboLights — in the context of the capability of interest (programmatic sample-level meta-analysis). For each tool, extract the individual design choices the authors mention and classify each as favorable or hindering for that purpose, preserving the authors' wording where possible. Build a tabular structure with one row per tool–design-choice pair and columns for tool name, design choice description, and valence. Ground the valence assignment in the authors' own rationale, e.g., ReDU's constrained vocabularies are favorable because they let programmatic meta-analysis proceed via string equality rather than entity recognition. Write the table to tool_design_choices_comparison.csv and verify coverage of every tool before finalizing.

## Related tools

- **SMetaS** (Source article whose discussion of Metabolomics Workbench, ReDU, and MetaboLights supplies the design choices being tabulated; its own vocabulary-constraint design parallels ReDU's favorable string-equality pattern.) — https://github.com/metabolomics-us/metadatastandardizer
- **manual expert review** (Method used to locate the relevant passages, extract individual design choices per tool, and assign favorable/hindering valence.)

## Examples

```
python -c "import pandas as pd; pd.DataFrame(rows, columns=['tool','design_choice','valence']).to_csv('tool_design_choices_comparison.csv', index=False)"
```

## Evaluation signals

- Each of the three compared tools (Metabolomics Workbench, ReDU, MetaboLights) appears at least once in tool_design_choices_comparison.csv.
- Every row carries a non-empty valence of exactly 'favorable' or 'hindering' tied to the stated purpose (programmatic sample-level meta-analysis).
- Design-choice descriptions preserve the authors' wording from the article rather than paraphrase.
- The favorable classification for ReDU's constrained vocabularies is justified via the string-equality-vs-entity-recognition rationale.
- Row count equals the number of distinct tool-design-choice pairs identified in the located passages.

## Limitations

- Coverage is limited to design choices the authors explicitly document; undocumented design choices in the compared databases cannot be tabulated.
- Valence judgments depend on correctly locating and reading the relevant passages; a missed passage leads to an incomplete table for that tool.
- The classification reflects the article authors' opinion for the stated purpose and may not transfer to other analysis goals.

## Evidence

- [full_text] ReDU constrained vocabularies favorable via string equality: [evidence span withheld — anchor status: not_found]
- [abstract] SMetaS architecture quote: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [abstract] FAIRness motivation for standardized metadata across databases: "Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness (Findability, Accessibility, Interoperability, and Reusability) of data"
- [readme] README documentation pointer: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
