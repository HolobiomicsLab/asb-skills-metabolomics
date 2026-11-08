---
name: tabular-metadata-standardization
description: Reference for converting a curated software/tool listing (with per-tool
  name, category, instrument data type, software type, reference) into one consistently-headered
  CSV governed by a fixed controlled vocabulary.
when_to_use_negative:
- The goal is pathway analysis tooling — the review explicitly excludes pathway analysis
  tools because most are not designed specifically for metabolomics.
- The records include tools outside the review's scope (e.g. MATLAB/Mathematica/GNU
  Octave language environments or ISA-Tab experimental metadata management) that do
  not fit the metabolomics-analysis categories and data-type vocabulary.
- You need a current, maintained tool landscape — the MetabolomicsTools repository
  is archived and 'no longer reflected the current state of the field'.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Loads the retrieved tool records and normalizes each entry to the controlled
    vocabulary before assembling the CSV table
- name: RASpicer/MetabolomicsTools
  role: Source repository holding the classified, searchable tool list that is transcribed
    into the machine-readable table
  repo: https://github.com/RASpicer/MetabolomicsTools
- name: XCMS
  role: 'Example tool record to transcribe: listed as an R package handling LC-MS
    and GC-MS under Preprocessing, with website and reference columns'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-017-1242-7
    title: Navigating freely-available software tools for metabolomics analysis
  source_content_hash: sha256:68d3cf37a6824c97b4e359fc3a67600da45e8b6d86874b11432bd9eac2796095
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RASpicer/MetabolomicsTools
  tools:
  - RASpicer/MetabolomicsTools
  - XCMS
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# tabular-metadata-standardization

## Summary

Normalizes heterogeneous tool records (e.g. metabolomics software listings) into a single machine-readable table using a controlled vocabulary for functional category, instrumental data type, software type, and annotation confidence level, with per-tool citations carried over.

## When to use

Reference for converting a curated software/tool listing (with per-tool name, category, instrument data type, software type, reference) into one consistently-headered CSV governed by a fixed controlled vocabulary.

## When NOT to use

- The goal is pathway analysis tooling — the review explicitly excludes pathway analysis tools because most are not designed specifically for metabolomics.
- The records include tools outside the review's scope (e.g. MATLAB/Mathematica/GNU Octave language environments or ISA-Tab experimental metadata management) that do not fit the metabolomics-analysis categories and data-type vocabulary.
- You need a current, maintained tool landscape — the MetabolomicsTools repository is archived and 'no longer reflected the current state of the field'.

## Inputs

- Tool classification records from the RASpicer/MetabolomicsTools GitHub repository (mirror of the review's supplementary material)
- Review's Supplementary Table 1 (per-tool name, website, reference, instrument data type, software type)
- Controlled vocabulary: categories (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other); data types (LC-MS, GC-MS, NMR); software types (open-source, free for non-commercial, free to use)

## Outputs

- Single machine-readable CSV table with consistent column headers covering tools from all five reported categories (Tables 1-5)
- Per-row fields: tool name, functional category, instrumental data type, software type, Schymanski annotation level (annotation/MS tools only, else empty), citation/reference

## How to apply

Retrieve the tool classification records from the GitHub repository RASpicer/MetabolomicsTools, which mirrors all information in the review's supplementary material, and load them with pandas. Normalize each entry to the review's controlled vocabulary: functional category (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other), instrumental data type (LC-MS, GC-MS, NMR), and software type (open-source, free for non-commercial, free to use). For mass spectrometry annotation tools only, transcribe the Schymanski et al. annotation confidence level (Level 4: unequivocal molecular formula; Level 3: tentative candidates; Level 2a: library spectrum match), leaving this field empty for non-annotation tools. Carry over the citation/reference for each tool and assemble all rows into a single CSV with consistent column headers (tool name, instrument data type, software type, category, annotation level, reference). Validate that every tool has a populated category and data-type field drawn from the reported classification and that the CSV parses cleanly.

## Related tools

- **pandas** (Loads the retrieved tool records and normalizes each entry to the controlled vocabulary before assembling the CSV table)
- **RASpicer/MetabolomicsTools** (Source repository holding the classified, searchable tool list that is transcribed into the machine-readable table) — https://github.com/RASpicer/MetabolomicsTools
- **XCMS** (Example tool record to transcribe: listed as an R package handling LC-MS and GC-MS under Preprocessing, with website and reference columns)

## Examples

```
import pandas as pd; df = pd.read_csv('metabolomics_tools.csv'); df = df[df['category'].isin(['Preprocessing','Annotation','Post-processing','Statistical analysis','Workflows','Other'])]
```

## Evaluation signals

- Assembled table covers tools from all five reported categories (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows) matching Tables 1-5.
- Every row has a populated functional-category and instrumental-data-type field whose values are drawn exactly from the controlled vocabulary (Preprocessing/Annotation/Post-processing/Statistical analysis/Workflows/Other; LC-MS/GC-MS/NMR).
- Annotation confidence levels appear only for mass-spectrometry annotation tools and only take the values Level 4, Level 3, or Level 2a; the field is empty for all other tools.
- Each tool row carries a citation/reference, mirroring the review's website and reference columns (e.g. XCMS as an R package handling LC-MS and GC-MS).
- The output CSV parses cleanly with pandas and has the expected consistent column headers.

## Limitations

- The RASpicer/MetabolomicsTools repository is no longer updated and is being archived, so the standardized table reflects the 2017 review snapshot only.
- Inclusion was restricted to widely used tools (≥50 Web of Science citations as of 08/09/16 or reported in the Weber et al. 2016 Metabolomics Society survey), so the table is not an exhaustive software catalogue.
- The controlled vocabulary is simple and coarse; tool records lacking an instrument data type or category value cannot be validated under the required populated-field checks.

## Evidence

- [intro] In this review software are classified into the following categories based on their major functionality: Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows and Other tools.: "In this review software are classified into the following categories based on their major functionality: Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows and Other tools."
- [intro] Mass spectrometry annotation tools are partitioned by level of annotation provided into: Level 4: unequivocal molecular formula, Level 3: tentative candidates and Level 2a: library spectrum match.: "Mass spectrometry annotation tools are partitioned by level of annotation provided into: Level 4: unequivocal molecular formula, Level 3: tentative candidates and Level 2a: library spectrum match."
- [abstract] An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary.: "An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary."
- [intro] All information in the supplementary material is further available at https://github.com/RASpicer/MetabolomicsTools.: "All information in the supplementary material is further available at https://github.com/RASpicer/MetabolomicsTools."
- [readme] The Metabolomics Tools Wiki aims to classify metabolomics software tools by either the instrumental data type taken as input or major functionality.: "The Metabolomics Tools Wiki aims to classify metabolomics software tools by either the instrumental data type taken as input or major functionality."
