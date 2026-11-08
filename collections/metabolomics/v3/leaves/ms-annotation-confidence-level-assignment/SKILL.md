---
name: ms-annotation-confidence-level-assignment
description: Use when labelling MS-based annotation tools or annotation results by
  confidence level (Schymanski Level 4/3/2a) during curation of a metabolomics tool
  catalogue or annotation pipeline.
when_to_use_negative:
- The tool is a pathway analysis tool — these are excluded from the review because
  most are not designed specifically for metabolomics
- The tool is an NMR identification tool — label it separately rather than forcing
  it into the MS Level 4/3/2a partition
- The input is raw spectrometric data or peak tables rather than a tool/annotation
  catalogue entry to be classified
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetabolomicsTools catalogue
  role: Source of the tool list to which Schymanski confidence-level labels are assigned
  repo: https://github.com/RASpicer/MetabolomicsTools
- name: FingerID
  role: Example MS/MS annotation tool producing predicted molecular fingerprints,
    i.e. tentative-candidate-style output classifiable under the scheme
  repo: https://github.com/huibinshen/fingerid
- name: Galaxy-M
  role: Galaxy workflow bundling metabolomics analysis tools (XCMS, MI-Pack) whose
    annotation components can be level-labelled
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-017-1242-7
    title: Navigating freely-available software tools for metabolomics analysis
  source_content_hash: sha256:4f4e9809c2ff25212db26301e105a54d5dbb6f8065847131a6353fbe69641e33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RASpicer/MetabolomicsTools
  tools:
  - FingerID
  - Galaxy-M
  - MetabolomicsTools catalogue
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ms-annotation-confidence-level-assignment

## Summary

Assigns MS metabolite-annotation tools and annotations to Schymanski et al. confidence levels (Level 4: unequivocal molecular formula; Level 3: tentative candidates; Level 2a: library spectrum match) as used in the Spicer et al. 2017 metabolomics software review. Enables consistent labelling of annotation depth across a tool catalogue using a controlled vocabulary.

## When to use

Use when labelling MS-based annotation tools or annotation results by confidence level (Schymanski Level 4/3/2a) during curation of a metabolomics tool catalogue or annotation pipeline.

## When NOT to use

- The tool is a pathway analysis tool — these are excluded from the review because most are not designed specifically for metabolomics
- The tool is an NMR identification tool — label it separately rather than forcing it into the MS Level 4/3/2a partition
- The input is raw spectrometric data or peak tables rather than a tool/annotation catalogue entry to be classified

## Inputs

- Tool catalogue entries from the MetabolomicsTools list (github.com/RASpicer/MetabolomicsTools)
- Tool descriptions specifying the annotation output type (molecular formula, candidate list, or spectral library match)
- Inclusion-criterion metadata (Web of Science citation counts as of 08/09/16; Metabolomics Society survey usage reports)

## Outputs

- Fully labelled tool table (classified_tools.csv) with one Schymanski confidence level per MS annotation tool
- Controlled-vocabulary labels for functionality (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other) and data type (LC-MS, GC-MS, NMR)

## How to apply

For each MS annotation tool, determine the highest level of annotation it provides and partition it accordingly: Level 4 for tools yielding an unequivocal molecular formula, Level 3 for tools returning tentative candidates, and Level 2a for tools performing library spectrum matches. Retain the label as a single value drawn from the fixed controlled vocabulary so each tool row carries exactly one functionality label alongside its instrumental data-type label (LC-MS, GC-MS or NMR). Label NMR identification tools separately, since the Level 4/3/2a partition applies only to mass spectrometry annotation tools. Persist the fully labelled table (e.g. classified_tools.csv) and record the inclusion-criterion flags (>=50 Web of Science citations as of 08/09/16, or use reported in the Weber et al. 2016 Metabolomics Society survey) for each retained tool.

## Related tools

- **MetabolomicsTools catalogue** (Source of the tool list to which Schymanski confidence-level labels are assigned) — https://github.com/RASpicer/MetabolomicsTools
- **FingerID** (Example MS/MS annotation tool producing predicted molecular fingerprints, i.e. tentative-candidate-style output classifiable under the scheme) — https://github.com/huibinshen/fingerid
- **Galaxy-M** (Galaxy workflow bundling metabolomics analysis tools (XCMS, MI-Pack) whose annotation components can be level-labelled) — https://github.com/Viant-Metabolomics/Galaxy-M

## Examples

```
python label_annotation_levels.py --catalogue https://github.com/RASpicer/MetabolomicsTools --levels Level4,Level3,Level2a --output classified_tools.csv
```

## Evaluation signals

- Every retained tool row carries exactly one data-type label (LC-MS, GC-MS, NMR) and one functionality label from the controlled vocabulary
- Each MS annotation tool is assigned exactly one of Level 4 (unequivocal molecular formula), Level 3 (tentative candidates), or Level 2a (library spectrum match)
- NMR identification tools are labelled separately and never receive a Schymanski MS level
- Inclusion-criterion flags (>=50 Web of Science citations as of 08/09/16 OR Metabolomics Society survey use) are recorded for every tool row
- Pathway analysis tools are absent from the output table

## Limitations

- The MetabolomicsTools catalogue is no longer being updated and was archived as it no longer reflected the current state of the field
- The Schymanski partition is defined only for MS annotation tools; it does not transfer to NMR identification tools
- MATLAB- or Mathematica-based tools may be freely available but were not all tested in the review (GNU Octave compatibility was out of scope)

## Evidence

- [intro] Mass spectrometry annotation tools are partitioned by level of annotation provided into: Level 4: unequivocal molecular formula, Level 3: tentative candidates and Level 2a: library spectrum match.: "Mass spectrometry annotation tools are partitioned by level of annotation provided into: Level 4: unequivocal molecular formula, Level 3: tentative candidates and Level 2a: library spectrum match."
- [abstract] An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary.: "An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary."
- [abstract] Tools were then categorised by the type of instrumental data (i.e. LC-MS, GC-MS or NMR) and the functionality (i.e. pre- and post-processing, statistical analysis, workflow and other functions): "Tools were then categorised by the type of instrumental data (i.e. LC-MS, GC-MS or NMR) and the functionality (i.e. pre- and post-processing, statistical analysis, workflow and other functions)"
- [readme] This project is not being updated and will be archived shortly as it no longer reflected the current state of the field.: "This project is not being updated and will be archived shortly as it no longer reflected the current state of the field."
