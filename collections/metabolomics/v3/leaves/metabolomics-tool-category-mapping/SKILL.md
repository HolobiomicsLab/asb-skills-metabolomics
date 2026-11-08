---
name: metabolomics-tool-category-mapping
description: 'Use when curating or ranking metabolomics software for a workflow: you
  need to place a tool into a data-type/functionality category, check whether it counts
  as ''most widely used'' (≥50 Web of Science citations or Metabolomics Society survey
  use), or build structured claim records.'
when_to_use_negative:
- The input is a pathway analysis tool — these are explicitly excluded because most
  are not designed specifically for metabolomics
- The tool is not freely available (open-source, free for non-commercial, or free
  to use) — the review scope covers only freely-available software
- You need processing of raw LC-MS/GC-MS/NMR instrument files rather than categorisation
  of software tools
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: RASpicer/MetabolomicsTools
  role: Extended searchable list that classifies tools by instrumental data type or
    major functionality using a simple controlled vocabulary; used to check and extend
    category assignments
  repo: https://github.com/RASpicer/MetabolomicsTools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-017-1242-7
    title: Navigating freely-available software tools for metabolomics analysis
  source_content_hash: sha256:15bcb43dfd2c767330d78fc59c805510ba47d1ac652a4a68777be41f334441a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RASpicer/MetabolomicsTools
  tools:
  - RASpicer/MetabolomicsTools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolomics-tool-category-mapping

## Summary

Classifies freely-available metabolomics software tools by instrumental data type (LC-MS, GC-MS or NMR) and major functionality (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other tools), following the Spicer et al. 2017 review scheme and the RASpicer/MetabolomicsTools controlled vocabulary.

## When to use

Use when curating or ranking metabolomics software for a workflow: you need to place a tool into a data-type/functionality category, check whether it counts as 'most widely used' (≥50 Web of Science citations or Metabolomics Society survey use), or build structured claim records.

## When NOT to use

- The input is a pathway analysis tool — these are explicitly excluded because most are not designed specifically for metabolomics
- The tool is not freely available (open-source, free for non-commercial, or free to use) — the review scope covers only freely-available software
- You need processing of raw LC-MS/GC-MS/NMR instrument files rather than categorisation of software tools

## Inputs

- Tool metadata (name, publication, citation counts) from the freely-available tool list (Supplementary Table 1 or https://github.com/RASpicer/MetabolomicsTools)
- Metabolomics Society survey (Weber et al. 2016) usage-ranking results
- Review full text (Spicer et al. 2017, DOI 10.1007/s11306-017-1242-7)

## Outputs

- Categorised tool records with data_type (LC-MS/GC-MS/NMR), functionality (Preprocessing/Annotation/Post-processing/Statistical analysis/Workflows/Other) and, for MS annotation, Schymanski level (2a/3/4)
- survey_usage_claims JSON file with exactly six records sharing the schema (tool, data_type, functionality, rank, percentage_if_reported, evidence_quote)

## How to apply

First check the tool's inclusion status: it is 'most widely used' if it has ≥50 citations on Web of Science (as of 08/09/16) or its use is reported in the Metabolomics Society survey (Weber et al. 2016). Then assign the tool to one instrumental data type — LC-MS, GC-MS or NMR — since no single technique captures the entire metabolome and each requires distinct analysis tools. Assign exactly one major functionality category: Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows or Other tools; note that pathway analysis tools are deliberately excluded because most are not metabolomics-specific. For MS annotation tools, additionally record the Schymanski et al. annotation level: Level 4 (unequivocal molecular formula), Level 3 (tentative candidates) or Level 2a (library spectrum match). When capturing survey usage claims, serialize one record per claim with a shared schema (tool, data_type, functionality, rank, percentage_if_reported, evidence_quote) — e.g. XCMS = most used LC-MS tool (70% of survey respondents), AMDIS = most used GC-MS processing tool, TopSpin = most used NMR preprocessing tool — via a Python .dump call into a survey_usage_claims file.

## Related tools

- **RASpicer/MetabolomicsTools** (Extended searchable list that classifies tools by instrumental data type or major functionality using a simple controlled vocabulary; used to check and extend category assignments) — https://github.com/RASpicer/MetabolomicsTools

## Examples

```
import ; .dump({"tool":"XCMS","data_type":"LC-MS","functionality":"Preprocessing","rank":"most used","percentage_if_reported":70,"evidence_quote":"XCMS was the most widely used software for LC-MS"}, open("survey_usage_claims.","w"), indent=2)
```

## Evaluation signals

- Output file exists, parses as valid JSON, and contains exactly six claim records
- Every record has the shared schema fields: tool, data_type, functionality, rank, percentage_if_reported, evidence_quote
- Every record includes the expected tool-name substring (XCMS, AMDIS, TopSpin, MZmine2, CAMERA) and a non-empty evidence_quote
- Each tool is assigned to exactly one instrumental data type (LC-MS, GC-MS or NMR) and one functionality category
- Survey ranks match the review's claims: XCMS most used LC-MS tool (70% of respondents) and second most used GC-MS tool, AMDIS most used GC-MS processing tool, TopSpin most used NMR preprocessing tool, MZmine2 second most used LC-MS preprocessing tool, CAMERA most used ionisation-product annotation tool

## Limitations

- The GitHub MetabolomicsTools list is no longer updated and was archived shortly after publication because it no longer reflected the current state of the field
- The Web of Science citation threshold (≥50) was assessed as of 08/09/16, so rankings are a snapshot of 2016 usage
- MATLAB-based tools may require MATLAB or GNU Octave to run; testing GNU Octave compatibility was beyond the scope of the review

## Evidence

- [abstract] Tools were then categorised by the type of instrumental data (i.e. LC-MS, GC-MS or NMR) and the functionality (i.e. pre- and post-processing, statistical analysis, workflow and other functions): "Tools were then categorised by the type of instrumental data (i.e. LC-MS, GC-MS or NMR) and the functionality (i.e. pre- and post-processing, statistical analysis, workflow and other functions)"
- [intro] In this review software are classified into the following categories based on their major functionality: Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows and Other tools.: "In this review software are classified into the following categories based on their major functionality: Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows and Other tools."
- [intro] Mass spectrometry annotation tools are partitioned by level of annotation provided into: Level 4: unequivocal molecular formula, Level 3: tentative candidates and Level 2a: library spectrum match.: "Mass spectrometry annotation tools are partitioned by level of annotation provided into: Level 4: unequivocal molecular formula, Level 3: tentative candidates and Level 2a: library spectrum match."
- [abstract] The most widely used tools were selected for inclusion in the review by either ≥ 50 citations on Web of Science: "The most widely used tools were selected for inclusion in the review by either ≥ 50 citations on Web of Science"
- [intro] Despite accounting for an important section of data analysis, tools for pathway analysis will not be included in this review.: "Despite accounting for an important section of data analysis, tools for pathway analysis will not be included in this review."
- [readme] The Metabolomics Tools Wiki aims to classify metabolomics software tools by either the instrumental data type taken as input or major functionality.: "The Metabolomics Tools Wiki aims to classify metabolomics software tools by either the instrumental data type taken as input or major functionality."
