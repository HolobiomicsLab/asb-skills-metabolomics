---
name: controlled-vocabulary-classification
description: Use when a curated list of metabolomics software tools must be normalized
  to a controlled vocabulary of functionality category, instrumental data type, software
  type and MS annotation confidence level.
when_to_use_negative:
- Pathway analysis tools are the target — the review explicitly excludes them because
  most are not designed specifically for metabolomics
- Tools that are not freely available (the review covers only open-source, free for
  non-commercial or free to use tools)
- A current, actively maintained tool directory is needed — the MetabolomicsTools
  repository is archived and no longer reflects the state of the field
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: RASpicer/MetabolomicsTools
  role: Source of the classified, searchable tool records that mirror the review's
    supplementary material
  repo: https://github.com/RASpicer/MetabolomicsTools
- name: XCMS
  role: 'Example row transcription target: listed in Table 1 as an R package handling
    LC-MS and GC-MS under the Preprocessing category'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-017-1242-7
    title: Navigating freely-available software tools for metabolomics analysis
  source_content_hash: sha256:a7d68cef0f7880201e2c19b369e28d37584e6fc779114058b4bbda79547bd443
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RASpicer/MetabolomicsTools
  tools:
  - RASpicer/MetabolomicsTools
  - XCMS
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# controlled-vocabulary-classification

## Summary

Classifies metabolomics software tools into functionality categories (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other) subdivided by instrumental data type (LC-MS, GC-MS, NMR) and Schymanski annotation confidence levels, so reported tables can be transcribed into one machine-readable CSV.

## When to use

Use when a curated list of metabolomics software tools must be normalized to a controlled vocabulary of functionality category, instrumental data type, software type and MS annotation confidence level.

## When NOT to use

- Pathway analysis tools are the target — the review explicitly excludes them because most are not designed specifically for metabolomics
- Tools that are not freely available (the review covers only open-source, free for non-commercial or free to use tools)
- A current, actively maintained tool directory is needed — the MetabolomicsTools repository is archived and no longer reflects the state of the field

## Inputs

- Tool classification records from the RASpicer/MetabolomicsTools GitHub repository (mirrors the review's supplementary material)
- Review's supplementary tables 1-5 (preprocessing, annotation, post-processing, statistics, workflows tool tables) with tool name, instrument data type, software type, website and reference columns
- Schymanski et al. annotation confidence levels for MS annotation tools

## Outputs

- Single CSV table with consistent column headers: tool name, functional category, instrumental data type, software type, Schymanski annotation confidence level, website, reference
- Validation report confirming coverage of all five reported categories and populated category/data-type fields

## How to apply

Retrieve tool classification records from the GitHub repository RASpicer/MetabolomicsTools, which mirrors all information in the review's supplementary material. Load the records with pandas and normalize each entry to the review's controlled vocabulary: functional category (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other), instrumental data type (LC-MS, GC-MS, NMR), and software type (open-source, free for non-commercial, free to use). For mass spectrometry annotation tools, transcribe the reported Schymanski et al. annotation confidence level (Level 4: unequivocal molecular formula, Level 3: tentative candidates, Level 2a: library spectrum match); leave this field empty for non-annotation tools. Carry over each tool's citation/reference and website, then assemble all rows into a single CSV table with consistent column headers. Validate that the assembled table covers the tools reported in the review's five categories and that each tool has a populated category and data-type field.

## Related tools

- **RASpicer/MetabolomicsTools** (Source of the classified, searchable tool records that mirror the review's supplementary material) — https://github.com/RASpicer/MetabolomicsTools
- **XCMS** (Example row transcription target: listed in Table 1 as an R package handling LC-MS and GC-MS under the Preprocessing category)

## Examples

```
import pandas as pd; df = pd.read_csv('MetabolomicsTools.csv'); df['category'] = df['category'].str.strip().replace({'Statistical analysis':'Statistical analysis'}); df.to_csv('tools_table.csv', index=False)
```

## Evaluation signals

- The assembled table covers the tools reported in the review's five categories (Tables 1-5)
- Every tool row has a populated functional category drawn from the controlled vocabulary (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other)
- Every tool row has a populated instrumental data type from {LC-MS, GC-MS, NMR}
- Annotation confidence level (Level 4, Level 3, Level 2a) is populated only for MS annotation tools and empty otherwise
- The CSV parses cleanly with expected fields present in every row

## Limitations

- The MetabolomicsTools repository is not being updated and will be archived shortly as it no longer reflected the current state of the field
- Annotation confidence levels apply only to mass spectrometry annotation tools; NMR tools lack this field
- Inclusion is limited to the most widely used freely available tools (>=50 Web of Science citations as of 08/09/16 or reported use in the Metabolomics Society survey)

## Evidence

- [intro] In this review software are classified into the following categories based on their major functionality: Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows and Other tools.: "In this review software are classified into the following categories based on their major functionality: Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows and Other tools."
- [intro] Mass spectrometry annotation tools are partitioned by level of annotation provided into: Level 4: unequivocal molecular formula, Level 3: tentative candidates and Level 2a: library spectrum match.: "Mass spectrometry annotation tools are partitioned by level of annotation provided into: Level 4: unequivocal molecular formula, Level 3: tentative candidates and Level 2a: library spectrum match."
- [abstract] An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary.: "An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary."
- [readme] The Metabolomics Tools Wiki aims to classify metabolomics software tools by either the instrumental data type taken as input or major functionality.: "The Metabolomics Tools Wiki aims to classify metabolomics software tools by either the instrumental data type taken as input or major functionality."
- [readme] This project is not being updated and will be archived shortly as it no longer reflected the current state of the field.: "This project is not being updated and will be archived shortly as it no longer reflected the current state of the field."
