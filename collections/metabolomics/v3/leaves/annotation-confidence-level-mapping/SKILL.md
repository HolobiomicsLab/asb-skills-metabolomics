---
name: annotation-confidence-level-mapping
description: Use when curating or transcribing a metabolomics software catalog where
  LC-MS/GC-MS annotation tools must be tagged with Schymanski confidence levels (Level
  4, Level 3, Level 2a) alongside functional category and instrumental data type.
when_to_use_negative:
- The tools under curation are pathway analysis tools — the review explicitly excludes
  them because most are not designed specifically for metabolomics
- The goal is a live, maintained tool directory — the MetabolomicsTools project is
  stated to be no longer updated and archived shortly as it no longer reflects the
  current state of the field
- The input records already carry normalized category, data-type and confidence-level
  fields and only need format conversion rather than level mapping
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetabolomicsTools Wiki
  role: Source repository whose classified tool records (category, instrument data
    type, annotation level) are transcribed into the single machine-readable table
  repo: https://github.com/RASpicer/MetabolomicsTools
- name: FingerID
  role: Example MS/MS annotation tool (fingerprint prediction via multiple kernel
    learning on fragmentation trees) that would receive a Schymanski confidence-level
    assignment in the Annotation category
  repo: huibinshen/fingerid
- name: Galaxy-M
  role: Metabolomics workflow tool for Galaxy whose components (including XCMS-based
    processing and MI-Pack identification) fall under the Workflows category rather
    than receiving a standalone annotation level
  repo: Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-017-1242-7
    title: Navigating freely-available software tools for metabolomics analysis
  source_content_hash: sha256:f6e25ffd56988651c95a82c70db5811093768126dfb4b2df9da5537c00a01ba3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RASpicer/MetabolomicsTools
  tools:
  - FingerID
  - Galaxy-M
  - MetabolomicsTools Wiki
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# annotation-confidence-level-mapping

## Summary

Assigns mass spectrometry annotation tools from the Spicer et al. 2017 metabolomics software review to their reported Schymanski et al. annotation confidence level (Level 4: unequivocal molecular formula, Level 3: tentative candidates, Level 2a: library spectrum match) while assembling the review's tool tables into one machine-readable table.

## When to use

Use when curating or transcribing a metabolomics software catalog where LC-MS/GC-MS annotation tools must be tagged with Schymanski confidence levels (Level 4, Level 3, Level 2a) alongside functional category and instrumental data type.

## When NOT to use

- The tools under curation are pathway analysis tools — the review explicitly excludes them because most are not designed specifically for metabolomics
- The goal is a live, maintained tool directory — the MetabolomicsTools project is stated to be no longer updated and archived shortly as it no longer reflects the current state of the field
- The input records already carry normalized category, data-type and confidence-level fields and only need format conversion rather than level mapping

## Inputs

- Tool classification records from the RASpicer/MetabolomicsTools GitHub repository (mirrors the review's supplementary material)
- Review Tables 1-5 covering Preprocessing, Annotation, Post-processing, Statistical analysis and Workflows categories
- Review's controlled vocabulary of categories, instrumental data types (LC-MS, GC-MS, NMR) and software types (open-source, free for non-commercial, free to use)

## Outputs

- Single CSV table with per-tool rows: name, functional category, instrumental data type, software type, Schymanski annotation confidence level (Level 2a/3/4, empty for non-annotation tools), and citation/reference
- Subset of annotation-tool rows partitioned by annotation confidence level

## How to apply

Retrieve the tool classification records mirrored at github.com/RASpicer/MetabolomicsTools (which contains all information from the review's supplementary material) and load them with pandas. Normalize each entry to the review's controlled vocabulary: functional category (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other), instrumental data type (LC-MS, GC-MS, NMR), and software type (open-source, free for non-commercial, free to use). For tools whose major functionality is Annotation and which take mass spectrometry input, transcribe the reported Schymanski et al. confidence level — Level 4 (unequivocal molecular formula), Level 3 (tentative candidates), or Level 2a (library spectrum match); leave the level field empty for non-annotation tools. Carry over each tool's citation/reference and assemble all rows into a single CSV with consistent column headers. Validate that the assembled table covers the tools in the review's five categories (Tables 1-5) and that every annotation tool has a populated level drawn from the three reported confidence levels.

## Related tools

- **MetabolomicsTools Wiki** (Source repository whose classified tool records (category, instrument data type, annotation level) are transcribed into the single machine-readable table) — https://github.com/RASpicer/MetabolomicsTools
- **FingerID** (Example MS/MS annotation tool (fingerprint prediction via multiple kernel learning on fragmentation trees) that would receive a Schymanski confidence-level assignment in the Annotation category) — huibinshen/fingerid
- **Galaxy-M** (Metabolomics workflow tool for Galaxy whose components (including XCMS-based processing and MI-Pack identification) fall under the Workflows category rather than receiving a standalone annotation level) — Viant-Metabolomics/Galaxy-M

## Examples

```
import pandas as pd; df = pd.read_csv('metabolomics_tools.csv'); annot = df[(df['category']=='Annotation') & (df['annotation_level'].isin(['Level 4','Level 3','Level 2a']))][['tool','instrument','annotation_level','reference']]
```

## Evaluation signals

- Every row whose functional category is 'Annotation' with MS input carries a confidence level in exactly {Level 4, Level 3, Level 2a}; no other values appear
- All non-annotation tool rows have an empty annotation-level field
- Each tool row has a populated category and instrumental data type drawn from the review's controlled vocabulary (Preprocessing/Annotation/Post-processing/Statistical analysis/Workflows/Other; LC-MS/GC-MS/NMR)
- The assembled CSV parses cleanly and covers the tools reported in the review's five category tables (Tables 1-5)
- Row counts per category and per confidence level match the tool listings in the review's supplementary material / GitHub mirror

## Limitations

- The MetabolomicsTools GitHub project is not being updated and will be archived shortly, so the level assignments reflect the 2017 state of the field
- Tool selection was restricted to the most widely used tools (≥50 Web of Science citations as of 08/09/16, or reported use in the Metabolomics Society survey), so newer annotation tools and levels are absent
- The huibinshen/fingerid repository is FingerID 1.4, not the CSI:FingerID code; it is unmaintained and has not been tested on Windows, limiting reuse as a modern annotation reference

## Evidence

- [intro] MS annotation tools partitioned by Schymanski confidence level: "Mass spectrometry annotation tools are partitioned by level of annotation provided into: Level 4: unequivocal molecular formula, Level 3: tentative candidates and Level 2a: library spectrum match."
- [intro] Controlled-vocabulary categories of the review: "In this review software are classified into the following categories based on their major functionality: Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows and Other tools."
- [intro] GitHub mirror of supplementary tool tables: "All information in the supplementary material is further available at https://github.com/RASpicer/MetabolomicsTools."
- [readme] Purpose of the MetabolomicsTools repository: "The Metabolomics Tools Wiki aims to classify metabolomics software tools by either the instrumental data type taken as input or major functionality."
- [readme] Archival status of the tool wiki: "This project is not being updated and will be archived shortly as it no longer reflected the current state of the field."
