---
name: instrumental-data-type-categorisation
description: Use when curating or classifying a metabolomics software catalogue and
  each tool must be labelled with its instrumental data type (LC-MS, GC-MS, NMR) and
  major functionality under a fixed controlled vocabulary.
when_to_use_negative:
- The tools of interest are pathway analysis tools — the review explicitly excludes
  them because most are not designed specifically for metabolomics
- A tool is not freely available (not open-source, not free for non-commercial, not
  free to use) — outside the review's scope
- You need a maintained, up-to-date classification — the README states the Metabolomics
  Tools Wiki 'is not being updated and will be archived shortly as it no longer reflected
  the current state of the field'
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetabolomicsTools catalogue
  role: The public, controlled-vocabulary-classified tool list that this labelling
    scheme populates and keeps searchable
  repo: https://github.com/RASpicer/MetabolomicsTools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-017-1242-7
    title: Navigating freely-available software tools for metabolomics analysis
  source_content_hash: sha256:60b448df58a2dac4c0bce8c91f5f836096e233574d20e9db74bf7662bc784557
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RASpicer/MetabolomicsTools
  tools:
  - MetabolomicsTools catalogue
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# instrumental-data-type-categorisation

## Summary

A controlled-vocabulary labelling scheme from Spicer et al. 2017 (Metabolomics) that assigns each freely available metabolomics software tool an instrumental data type (LC-MS, GC-MS, or NMR) for preprocessing tools and one major-functionality category, enabling a searchable, classifiable tool catalogue.

## When to use

Use when curating or classifying a metabolomics software catalogue and each tool must be labelled with its instrumental data type (LC-MS, GC-MS, NMR) and major functionality under a fixed controlled vocabulary.

## When NOT to use

- The tools of interest are pathway analysis tools — the review explicitly excludes them because most are not designed specifically for metabolomics
- A tool is not freely available (not open-source, not free for non-commercial, not free to use) — outside the review's scope
- You need a maintained, up-to-date classification — the README states the Metabolomics Tools Wiki 'is not being updated and will be archived shortly as it no longer reflected the current state of the field'

## Inputs

- Tool catalogue of freely available metabolomics software (extended list at https://github.com/RASpicer/MetabolomicsTools)
- Web of Science citation counts per tool as of 08/09/16
- Metabolomics Society survey tool-use reports (Weber et al. 2016)

## Outputs

- classified_tools.csv — a fully labelled table where every retained tool row carries exactly one data-type label and one functionality label drawn from the controlled vocabulary
- Per-tool inclusion-criterion flags (>=50 Web of Science citations vs. Metabolomics Society survey)

## How to apply

First restrict the catalogue to freely available tools (open-source, free for non-commercial, or free to use) meeting the 'most widely used' inclusion criterion: >=50 citations on Web of Science as of 08/09/16, or use reported in the Metabolomics Society survey (Weber et al. 2016). For each retained tool, assign exactly one instrumental data type label — LC-MS, GC-MS, or NMR — reflecting the type of instrumental data the tool takes as input; this subdivision applies to Preprocessing and Annotation tools. Assign exactly one major-functionality category from the fixed taxonomy: Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, or Other. For MS annotation tools, further partition by the Schymanski et al. (2014) annotation confidence level: Level 4 (unequivocal molecular formula), Level 3 (tentative candidates), or Level 2a (library spectrum match); label NMR identification tools separately. Exclude pathway analysis tools because most are not designed specifically for metabolomics, then write the fully labelled table with the controlled vocabulary to classified_tools.csv.

## Related tools

- **MetabolomicsTools catalogue** (The public, controlled-vocabulary-classified tool list that this labelling scheme populates and keeps searchable) — https://github.com/RASpicer/MetabolomicsTools

## Examples

```
df[["tool","data_type","functionality","inclusion_flag"]].to_csv("classified_tools.csv", index=False)
```

## Evaluation signals

- Every retained tool row in classified_tools.csv carries exactly one instrumental data-type label (LC-MS, GC-MS, or NMR) and one functionality label
- Every functionality and data-type label is drawn verbatim from the controlled vocabulary — no free-text categories
- Inclusion-criterion flags are recorded for each tool, with the >=50 Web of Science citations (as of 08/09/16) and Metabolomics Society survey criteria checkable per tool
- No pathway analysis tools appear in the labelled table
- MS annotation tools each carry one Schymanski et al. confidence-level sub-label (Level 4, Level 3, or Level 2a)

## Limitations

- The catalogue snapshot is frozen: the README states the project is not being updated and will be archived because it no longer reflected the current state of the field
- Each tool receives a single major-functionality label, which may under-represent multi-purpose tools that span several categories
- The citation threshold is date-anchored (>=50 Web of Science citations as of 08/09/16), so re-running the selection later will yield a different 'most widely used' set
- The review notes no single analytical technique captures the entire metabolome, so LC-MS, GC-MS, and NMR tools require distinct workflows and cannot be collapsed into one category

## Evidence

- [abstract] Tools were then categorised by the type of instrumental data (i.e. LC-MS, GC-MS or NMR) and the functionality (i.e. pre- and post-processing, statistical analysis, workflow and other functions): "Tools were then categorised by the type of instrumental data (i.e. LC-MS, GC-MS or NMR) and the functionality (i.e. pre- and post-processing, statistical analysis, workflow and other functions)"
- [intro] MS annotation confidence levels: "Mass spectrometry annotation tools are partitioned by level of annotation provided into: Level 4: unequivocal molecular formula, Level 3: tentative candidates and Level 2a: library spectrum match."
- [abstract] Controlled vocabulary catalogue: "An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary."
- [intro] Pathway analysis exclusion: "Despite accounting for an important section of data analysis, tools for pathway analysis will not be included in this review."
- [readme] README classification aim: "The Metabolomics Tools Wiki aims to classify metabolomics software tools by either the instrumental data type taken as input or major functionality."
