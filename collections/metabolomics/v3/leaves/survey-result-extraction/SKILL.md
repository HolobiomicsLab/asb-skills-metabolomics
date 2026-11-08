---
name: survey-result-extraction
description: Use when a task asks to reproduce or verify specific tool-usage rankings
  or percentages reported from a community survey (e.g. the Metabolomics Society survey)
  within a full-text article.
when_to_use_negative:
- The question asks about a tool's features, parameters, or installation rather than
  its reported usage rank or percentage from a survey
- The usage statistics live in a raw survey dataset rather than being reported as
  claims inside a review article — extract directly from the survey data instead
- The article only lists tools without attributing usage rankings to a community survey;
  the review's inclusion criterion (≥50 Web of Science citations) alone is not a survey-percentage
  claim
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: XCMS
  role: 'Subject of the extracted claim: most widely used tool for LC-MS metabolomics
    analysis, reported by 70% of Metabolomics Society survey respondents'
- name: AMDIS
  role: 'Subject of an extracted claim: most used GC-MS processing tool in the survey
    results'
- name: TopSpin
  role: 'Subject of an extracted claim: most used NMR preprocessing tool in the survey
    results'
- name: MZmine2
  role: 'Subject of an extracted claim: second most used LC-MS preprocessing tool
    in the survey results'
- name: CAMERA
  role: 'Subject of an extracted claim: most used ionisation-product annotation tool
    in the survey results'
- name: MetabolomicsTools
  role: Extended, searchable companion list of the freely available metabolomics tools
    classified and described by the review
  repo: https://github.com/RASpicer/MetabolomicsTools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-017-1242-7
    title: Navigating freely-available software tools for metabolomics analysis
  source_content_hash: sha256:8e234dda3f4407be985b1b833f2ab257a9705a44107102b84972ad90d0eca891
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RASpicer/MetabolomicsTools
  tools:
  - AMDIS
  - CAMERA
  - MZmine2
  - MetabolomicsTools
  - TopSpin
  - XCMS
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# survey-result-extraction

## Summary

Extracts and serializes community-survey usage findings (tool name, data type, functionality, rank, reported percentage) from a review article, reproducing claims such as those reported from the Metabolomics Society survey (Weber et al. 2016) in Spicer et al. 2017.

## When to use

Use when a task asks to reproduce or verify specific tool-usage rankings or percentages reported from a community survey (e.g. the Metabolomics Society survey) within a full-text article.

## When NOT to use

- The question asks about a tool's features, parameters, or installation rather than its reported usage rank or percentage from a survey
- The usage statistics live in a raw survey dataset rather than being reported as claims inside a review article — extract directly from the survey data instead
- The article only lists tools without attributing usage rankings to a community survey; the review's inclusion criterion (≥50 Web of Science citations) alone is not a survey-percentage claim

## Inputs

- Full-text article (Spicer et al. 2017, DOI 10.1007/s11306-017-1242-7) containing the Metabolomics Society survey (Weber et al. 2016) usage results
- List of survey usage claims to reproduce (tool, data type, functionality, rank)
- Target output filename for the serialized claim records

## Outputs

- JSON array of six survey_usage_claims records, each with fields: tool, data_type, functionality, rank, percentage_if_reported, evidence_quote

## How to apply

Load the full-text article (Spicer et al. 2017, DOI 10.1007/s11306-017-1242-7) and locate the section reporting Metabolomics Society survey (Weber et al. 2016) usage rankings. For each claim, capture the tool name, the instrumental data type (LC-MS, GC-MS or NMR), the functionality category (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other), the rank (e.g. most used, second most used), any reported percentage (e.g. XCMS used by 70% of LC-MS respondents), and a verbatim evidence sentence from the article. Serialize all records with a single Python .dump call into one JSON array with a shared schema (tool, data_type, functionality, rank, percentage_if_reported, evidence_quote). Validate that the output file parses as JSON, contains the expected number of claim records, includes the expected tool-name substrings (e.g. XCMS, AMDIS, TopSpin, MZmine2, CAMERA), and that every record has a non-empty evidence_quote.

## Related tools

- **XCMS** (Subject of the extracted claim: most widely used tool for LC-MS metabolomics analysis, reported by 70% of Metabolomics Society survey respondents)
- **AMDIS** (Subject of an extracted claim: most used GC-MS processing tool in the survey results)
- **TopSpin** (Subject of an extracted claim: most used NMR preprocessing tool in the survey results)
- **MZmine2** (Subject of an extracted claim: second most used LC-MS preprocessing tool in the survey results)
- **CAMERA** (Subject of an extracted claim: most used ionisation-product annotation tool in the survey results)
- **MetabolomicsTools** (Extended, searchable companion list of the freely available metabolomics tools classified and described by the review) — https://github.com/RASpicer/MetabolomicsTools

## Examples

```
import ; .dump(survey_usage_claims, open('survey_usage_claims.','w'), indent=2)
```

## Evaluation signals

- Output file exists and parses as valid JSON
- Output contains exactly six claim records, one per expected survey usage ranking
- Every record includes the expected tool-name substring (XCMS, AMDIS, TopSpin, MZmine2, CAMERA)
- Every record has a non-empty evidence_quote field quoting the article verbatim
- Percentage fields are populated only where the article reports them (e.g. XCMS at 70% of LC-MS respondents); other percentage fields are null/empty

## Limitations

- The extracted claims reflect the Metabolomics Society survey reported in Weber et al. 2016 as of the review's 2017 publication; the companion GitHub list is archived and 'no longer reflected the current state of the field', so ranks and percentages may be outdated
- Survey claims are second-hand evidence quoted from the review, not the primary survey data; percentages may be rounded and ranks depend on the review's own categorization by data type and functionality
- Pathway analysis tools were excluded from the review, so survey results for such tools will not be recoverable from this source

## Evidence

- [abstract] The most widely used tools were selected for inclusion in the review by either ≥ 50 citations on Web of Science: "The most widely used tools were selected for inclusion in the review by either ≥ 50 citations on Web of Science"
- [intro] survey inclusion criterion: "the use of the tool being reported in the recent Metabolomics Society survey (Weber et al. 2016)"
- [abstract] categorisation scheme: "Tools were then categorised by the type of instrumental data (i.e. LC-MS, GC-MS or NMR) and the functionality (i.e. pre- and post-processing, statistical analysis, workflow and other functions)"
- [intro] functionality categories: "In this review software are classified into the following categories based on their major functionality: Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows and Other tools."
- [readme] companion tool list status: "This project is not being updated and will be archived shortly as it no longer reflected the current state of the field."
