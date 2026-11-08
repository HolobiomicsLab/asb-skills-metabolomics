---
name: metabolomics-software-usage-ranking-extraction
description: Use when a task asks which freely available metabolomics software tools
  were most widely used per data type/functionality, or when reproducing the survey-based
  usage claims (e.g. XCMS 70% of LC-MS respondents) from the Spicer et al. 2017 review.
when_to_use_negative:
- The goal is a current ranking of tool popularity — the review reflects 2016 survey
  data and the companion GitHub list is archived and 'no longer reflected the current
  state of the field'.
- Pathway analysis tools are of interest — the review explicitly excludes them because
  most are not designed specifically for metabolomics.
- The target is the complete tool catalogue rather than the survey usage rankings
  — use the GitHub controlled-vocabulary list or Supplementary Table 1 instead of
  extracting rank claims.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetabolomicsTools (RASpicer/MetabolomicsTools)
  role: Extended, searchable companion list of freely available metabolomics tools
    classified by instrumental data type and functionality, used to cross-check extracted
    tool categories
  repo: https://github.com/RASpicer/MetabolomicsTools
- name: manual expert review
  role: Method used to read the article full text, locate the survey results section,
    and quote the evidence sentences supporting each of the six usage-ranking claims
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-017-1242-7
    title: Navigating freely-available software tools for metabolomics analysis
  source_content_hash: sha256:cacc223545d6fda1bcaa375661d834f6334f52212cf1388ee5c107cc02821f55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RASpicer/MetabolomicsTools
  tools:
  - MetabolomicsTools (RASpicer/MetabolomicsTools)
  - manual expert review
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolomics-software-usage-ranking-extraction

## Summary

Extracts and structures the Metabolomics Society (Weber et al. 2016) software usage rankings reported in Spicer et al. 2017, capturing each tool's instrumental data type (LC-MS, GC-MS, NMR), functionality category (preprocessing, annotation, post-processing, statistical analysis, workflows), rank, and reported usage percentage.

## When to use

Use when a task asks which freely available metabolomics software tools were most widely used per data type/functionality, or when reproducing the survey-based usage claims (e.g. XCMS 70% of LC-MS respondents) from the Spicer et al. 2017 review.

## When NOT to use

- The goal is a current ranking of tool popularity — the review reflects 2016 survey data and the companion GitHub list is archived and 'no longer reflected the current state of the field'.
- Pathway analysis tools are of interest — the review explicitly excludes them because most are not designed specifically for metabolomics.
- The target is the complete tool catalogue rather than the survey usage rankings — use the GitHub controlled-vocabulary list or Supplementary Table 1 instead of extracting rank claims.

## Inputs

- Full-text article Spicer et al. 2017, 'Navigating freely-available software tools for metabolomics analysis' (DOI 10.1007/s11306-017-1242-7)
- Metabolomics Society survey reference (Weber et al. 2016) as cited in the review
- Extended tool list at github.com/RASpicer/MetabolomicsTools for cross-checking tool classification

## Outputs

- survey_usage_claims JSON file containing exactly six claim records with fields: tool, data_type, functionality, rank, percentage_if_reported, evidence_quote

## How to apply

Load the full text of Spicer et al. 2017 (DOI 10.1007/s11306-017-1242-7) and locate the section reporting the Metabolomics Society survey (Weber et al. 2016) usage rankings, which the review used as an inclusion criterion alongside >=50 Web of Science citations. Extract the six usage-ranking claims: XCMS most used LC-MS tool (70% of respondents), XCMS second most used GC-MS tool, AMDIS most used GC-MS processing tool, TopSpin most used NMR preprocessing tool, MZmine2 second most used LC-MS preprocessing tool, and CAMERA most used ionisation-product annotation tool. For each claim record the tool name, instrumental data type, functionality category, rank, reported percentage if given, and a verbatim evidence sentence. Serialize the six records with a Python .dump call into a survey_usage_claims object using one shared schema (tool, data_type, functionality, rank, percentage_if_reported, evidence_quote).

## Related tools

- **MetabolomicsTools (RASpicer/MetabolomicsTools)** (Extended, searchable companion list of freely available metabolomics tools classified by instrumental data type and functionality, used to cross-check extracted tool categories) — https://github.com/RASpicer/MetabolomicsTools
- **manual expert review** (Method used to read the article full text, locate the survey results section, and quote the evidence sentences supporting each of the six usage-ranking claims)

## Examples

```
import ; .dump(survey_usage_claims, open('survey_usage_claims.','w'), indent=2)
```

## Evaluation signals

- The output file exists and parses as valid JSON.
- The output contains exactly six claim records, one per extracted usage-ranking claim.
- Every record's tool field contains the expected tool-name substring (XCMS, AMDIS, TopSpin, MZmine2, CAMERA).
- Every record has a non-empty evidence_quote field.
- The XCMS LC-MS record carries percentage_if_reported = 70%, consistent with the reported Metabolomics Society survey figure.

## Limitations

- Usage percentages are only reported for some claims (e.g. XCMS LC-MS at 70%); other rankings carry no percentage and must be recorded with a null/empty percentage_if_reported value.
- The rankings reflect the 2016 Metabolomics Society survey (Weber et al. 2016); the companion GitHub repository README warns the project 'is not being updated and will be archived shortly'.
- Extraction is limited to tools meeting the review's inclusion criteria (>=50 Web of Science citations as of 08/09/16, or reported use in the survey), so niche tools are absent by design.

## Evidence

- [intro] Inclusion criterion for 'most widely used' tools is either >=50 citations on Web of Science (as of 08/09/16) or reported use in the recent Metabolomics Society survey.: "the use of the tool being reported in the recent Metabolomics Society survey (Weber et al. 2016)"
- [abstract] Tools are classified by functionality and instrumental data type, which defines the schema fields for each extracted claim.: "Tools were then categorised by the type of instrumental data (i.e. LC-MS, GC-MS or NMR) and the functionality (i.e. pre- and post-processing, statistical analysis, workflow and other functions)"
- [abstract] The review selects the most widely used tools by citation count or survey use, bounding which tools can appear in extracted claims.: "The most widely used tools were selected for inclusion in the review by either ≥ 50 citations on Web of Science"
- [abstract] The companion GitHub list provides the searchable controlled-vocabulary classification used to validate data_type and functionality fields.: "An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary."
