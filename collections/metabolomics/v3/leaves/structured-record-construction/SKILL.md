---
name: structured-record-construction
description: Use when a task requires turning qualitative findings from a review article
  (e.g. tool usage rankings from the Weber et al. 2016 Metabolomics Society survey)
  into a machine-readable record set with a uniform schema.
when_to_use_negative:
- The source text does not contain per-tool usage rankings or survey-reported percentages,
  so no claim records can be grounded
- The goal is to enumerate or classify tools themselves (e.g. building a searchable
  tool catalogue) rather than to extract specific usage-ranking claims
- The output must be a relational database or spreadsheet rather than a single JSON
  file of uniform records
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetabolomicsTools (RASpicer/MetabolomicsTools)
  role: Provides the controlled vocabulary used to classify tools by instrumental
    data type and major functionality, informing the data_type and functionality fields
    of each record
  repo: https://github.com/RASpicer/MetabolomicsTools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-017-1242-7
    title: Navigating freely-available software tools for metabolomics analysis
  source_content_hash: sha256:4e277482f18af005f85d755ea9dad0c7dce8fd2c8a17cb650736b568c7a5b2ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RASpicer/MetabolomicsTools
  tools:
  - MetabolomicsTools (RASpicer/MetabolomicsTools)
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# structured-record-construction

## Summary

Extract a fixed set of structured claim records (here, six software-usage rankings from the Spicer et al. 2017 metabolomics software review) and serialize them into a single JSON file with a shared schema, then validate the file programmatically.

## When to use

Use when a task requires turning qualitative findings from a review article (e.g. tool usage rankings from the Weber et al. 2016 Metabolomics Society survey) into a machine-readable record set with a uniform schema.

## When NOT to use

- The source text does not contain per-tool usage rankings or survey-reported percentages, so no claim records can be grounded
- The goal is to enumerate or classify tools themselves (e.g. building a searchable tool catalogue) rather than to extract specific usage-ranking claims
- The output must be a relational database or spreadsheet rather than a single JSON file of uniform records

## Inputs

- Full-text article: Spicer et al. 2017 'Navigating freely-available software tools for metabolomics analysis' (DOI 10.1007/s11306-017-1242-7)
- Survey usage-ranking statements from the Metabolomics Society survey (Weber et al. 2016) as reported in the article

## Outputs

- survey_usage_claims: a JSON file containing exactly six claim records with fields tool, data_type, functionality, rank, percentage_if_reported, evidence_quote

## How to apply

Load the full-text article (Spicer et al. 2017, DOI 10.1007/s11306-017-1242-7) and locate the survey results reporting Metabolomics Society (Weber et al. 2016) usage rankings. Enumerate the six usage-ranking claims: XCMS most used LC-MS tool (70% of respondents), XCMS second most used GC-MS tool, AMDIS most used GC-MS processing tool, TopSpin most used NMR preprocessing tool, MZmine2 second most used LC-MS preprocessing tool, and CAMERA most used ionisation-product annotation tool. For each claim, capture the tool name, instrumental data type (LC-MS, GC-MS, NMR), functionality category (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other), rank, percentage where reported, and the quoted evidence sentence supporting it. Serialize the records with a Python .dump call into a file named survey_usage_claims, one object per claim sharing the schema (tool, data_type, functionality, rank, percentage_if_reported, evidence_quote). The review's own classification scheme — tools categorised by instrumental data type (LC-MS, GC-MS or NMR) and functionality — provides the controlled vocabulary for the data_type and functionality fields.

## Related tools

- **MetabolomicsTools (RASpicer/MetabolomicsTools)** (Provides the controlled vocabulary used to classify tools by instrumental data type and major functionality, informing the data_type and functionality fields of each record) — https://github.com/RASpicer/MetabolomicsTools

## Examples

```
import ; .dump(survey_usage_claims, open('survey_usage_claims.', 'w'), indent=2)
```

## Evaluation signals

- The output file exists and parses as valid JSON
- The file contains exactly six claim records
- Every record includes the expected tool-name substring (XCMS, AMDIS, TopSpin, MZmine2, CAMERA)
- Every record has a non-empty evidence_quote field
- Each record's data_type is one of LC-MS, GC-MS, or NMR, consistent with the review's instrumental categorisation

## Limitations

- Survey-based rankings reflect self-reported tool use by Metabolomics Society respondents and may not generalise to the wider community
- The GitHub MetabolomicsTools wiki 'is not being updated and will be archived shortly as it no longer reflected the current state of the field', so its controlled vocabulary is frozen as of the review's publication
- Percentages are only reported for some claims (e.g. XCMS at 70% of LC-MS respondents); percentage_if_reported must be null for claims lacking a figure

## Evidence

- [abstract] Tools categorised by instrumental data type and functionality: "Tools were then categorised by the type of instrumental data (i.e. LC-MS, GC-MS or NMR) and the functionality (i.e. pre- and post-processing, statistical analysis, workflow and other functions)"
- [intro] Survey use as inclusion criterion: "the use of the tool being reported in the recent Metabolomics Society survey (Weber et al. 2016)"
- [abstract] GitHub list classified via controlled vocabulary: "An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary."
- [readme] README on tool classification scope: "The Metabolomics Tools Wiki aims to classify metabolomics software tools by either the instrumental data type taken as input or major functionality."
