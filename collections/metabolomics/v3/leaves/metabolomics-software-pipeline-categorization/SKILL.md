---
name: metabolomics-software-pipeline-categorization
description: Use when auditing a metabolomics software bundle (e.g. MetumpX-bin) to
  enumerate its packaged tools, categorize them by pipeline stage, and verify the
  count against claims in a paper abstract or README.
when_to_use_negative:
- The goal is running the metabolomics analysis itself (peak conversion, MSEA) rather
  than auditing which tools are packaged
- The repository ships no tool manifest/setup script from which the packaged tool
  inventory can be enumerated
- Input is already a tool-by-pipeline-stage table that only needs reformatting
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetumpX
  role: Ubuntu-based software package whose manifest/setup script is enumerated and
    categorized into pipeline stages
  repo: https://github.com/hasaniqbal777/MetumpX-bin
- name: Ubuntu
  role: Operating system on which MetumpX and its packaged untargeted metabolomics
    MS tools are installed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btz765
    title: MetumpX—a metabolomics support package for untargeted mass spectrometry
  source_content_hash: sha256:287386f53af9211e61aa00b57c707eeb655a643a39feed21d9c51765bcbfc520
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hasaniqbal777/MetumpX-bin
  tools:
  - MetumpX
  - Ubuntu
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabolomics-software-pipeline-categorization

## Summary

Enumerates the software tools packaged for an untargeted metabolomics MS pipeline from a repository's manifest/setup scripts, assigns each tool to its pipeline-stage category, and reconciles the extracted count against counts reported in the paper (103) and README revision history (about 89).

## When to use

Use when auditing a metabolomics software bundle (e.g. MetumpX-bin) to enumerate its packaged tools, categorize them by pipeline stage, and verify the count against claims in a paper abstract or README.

## When NOT to use

- The goal is running the metabolomics analysis itself (peak conversion, MSEA) rather than auditing which tools are packaged
- The repository ships no tool manifest/setup script from which the packaged tool inventory can be enumerated
- Input is already a tool-by-pipeline-stage table that only needs reformatting

## Inputs

- Git clone of the MetumpX-bin repository (https://github.com/hasaniqbal777/MetumpX-bin)
- Tool manifest/configuration files (e.g. MetumpX_setup_enUS setup script, tool lists)
- Paper abstract and README claims about tool counts (103 vs about 89)

## Outputs

- Structured table of extracted tool names with assigned pipeline-stage categories
- Explicit comparison of extracted tool count vs the abstract's count (103) and the README revision-history count (about 89), with any discrepancy recorded

## How to apply

Clone the target repository (https://github.com/hasaniqbal777/MetumpX-bin) and inspect the tool manifest/configuration files, such as the MetumpX_setup_enUS setup script and any tool lists, to enumerate the packaged metabolomics software. Assign each extracted tool to its pipeline-stage category based on how the manifest structures the standard Untargeted Metabolomics Mass Spectrometry pipeline (e.g. conversion of MS peak lists to compound concentrations, MSEA via MetaboAnalystR module 14). Tabulate the tool names and categories in a structured table. Finally, compare the extracted tool count against the two reported counts: 103 tools stated in the abstract and 'about 89 tools' in the README/intro, and explicitly record any discrepancy (the README revision history shows v1.0=50 tools, v2.0=+39 (89 total), v2.1=+15 (103 total)).

## Related tools

- **MetumpX** (Ubuntu-based software package whose manifest/setup script is enumerated and categorized into pipeline stages) — https://github.com/hasaniqbal777/MetumpX-bin
- **Ubuntu** (Operating system on which MetumpX and its packaged untargeted metabolomics MS tools are installed)

## Examples

```
git clone https://github.com/hasaniqbal777/MetumpX-bin
```

## Evaluation signals

- Output table file exists and contains one row per extracted tool with an assigned pipeline-stage category
- Extracted tool count is explicitly compared with the two reported counts (103 from the abstract; about 89 from the README)
- Any discrepancy between extracted count and reported counts is recorded rather than silently dropped
- Category assignments are traceable to the manifest structure of the setup script/tool lists
- README revision-history counts (v1.0: 50; v2.0: 89; v2.1: 103) are consistent with the extracted inventory given the repository version audited

## Limitations

- The packaged tool count (about 89) is fewer than the 103 tools claimed in the paper's abstract, so counts depend on which repository version is audited
- The exact MetaboAnalystR 4.0 version number is not provided in the article
- No changelog is found in the repository discussion, limiting version reconciliation

## Evidence

- [abstract] Abstract claim of 103 packaged tools: "this paper aims to provide MetumpX, a software package that eases in the installation of 103 software"
- [readme] README claim of about 89 tools: "MetumpX is a Ubuntu based software package that facilitate easy download and installation of about 89 tools related to standard Untargeted Metabolomics Mass Spectrometry pipeline."
- [readme] README revision history showing per-version tool counts: "**Version 2.1**
 - 15 Tools added (Total 103 Tools)
 - User Guide released"
- [methods] Pipeline-stage example: concentration conversion: [evidence span withheld — anchor status: not_found]
