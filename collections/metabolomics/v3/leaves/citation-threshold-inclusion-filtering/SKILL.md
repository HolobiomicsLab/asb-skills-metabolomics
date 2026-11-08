---
name: citation-threshold-inclusion-filtering
description: Use when compiling a curated catalogue of metabolomics analysis tools
  from a broader candidate list and an objective, reproducible popularity criterion
  is needed to decide which tools qualify as 'most widely used'.
when_to_use_negative:
- The goal is to catalogue pathway analysis tools, since most are not designed specifically
  for metabolomics and are explicitly excluded by the review.
- The aim is an exhaustive inventory of all metabolomics software rather than a selection
  of the most widely used tools — the extended GitHub list serves that broader purpose.
- Tools are not freely available to use (open-source, free for non-commercial, or
  free to use), so they fall outside the review's scope regardless of citation count.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetabolomicsTools catalogue
  role: Public, classified and searchable tool list from which candidates are drawn
    and flagged against the inclusion criteria
  repo: https://github.com/RASpicer/MetabolomicsTools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-017-1242-7
    title: Navigating freely-available software tools for metabolomics analysis
  source_content_hash: sha256:9cacf60a0d9019aacc145c87d5abc5650e3b134055702f5dfa462f79272883fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RASpicer/MetabolomicsTools
  tools:
  - MetabolomicsTools catalogue
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# citation-threshold-inclusion-filtering

## Summary

A bibliometric inclusion filter that selects the 'most widely used' freely available metabolomics software tools by requiring either ≥50 Web of Science citations (as of 08/09/16) or reported use in the recent Metabolomics Society survey (Weber et al. 2016).

## When to use

Use when compiling a curated catalogue of metabolomics analysis tools from a broader candidate list and an objective, reproducible popularity criterion is needed to decide which tools qualify as 'most widely used'.

## When NOT to use

- The goal is to catalogue pathway analysis tools, since most are not designed specifically for metabolomics and are explicitly excluded by the review.
- The aim is an exhaustive inventory of all metabolomics software rather than a selection of the most widely used tools — the extended GitHub list serves that broader purpose.
- Tools are not freely available to use (open-source, free for non-commercial, or free to use), so they fall outside the review's scope regardless of citation count.

## Inputs

- Candidate list of freely available metabolomics software tools (e.g. from the MetabolomicsTools catalogue)
- Web of Science citation counts per tool as of 08/09/16
- Metabolomics Society survey results (Weber et al. 2016) reporting tool usage

## Outputs

- Flagged tool table with per-tool inclusion-criterion indicators (citations threshold vs. survey)
- classified_tools.csv with each retained tool carrying exactly one data-type label (LC-MS, GC-MS, NMR) and one functionality label from the controlled vocabulary

## How to apply

For each candidate tool from the freely-available metabolomics software survey (open-source, free for non-commercial, or free to use), first look up its citation count on Web of Science and flag it as included if it reaches the threshold of ≥50 citations as of 08/09/16. If the citation threshold is not met, alternatively include the tool when its use is reported in the recent Metabolomics Society survey (Weber et al. 2016); a tool needs to satisfy only one of the two criteria (OR logic). Record which criterion triggered inclusion for every tool so the filter is auditable. After filtering, classify each retained tool by instrumental data type (LC-MS, GC-MS, NMR) and functionality (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other), while excluding pathway analysis tools. Write the labelled table with inclusion flags to classified_tools.csv.

## Related tools

- **MetabolomicsTools catalogue** (Public, classified and searchable tool list from which candidates are drawn and flagged against the inclusion criteria) — https://github.com/RASpicer/MetabolomicsTools

## Evaluation signals

- Every retained tool row satisfies at least one inclusion criterion: ≥50 Web of Science citations as of 08/09/16 OR use reported in the Metabolomics Society survey (Weber et al. 2016).
- Each tool's triggering inclusion-criterion flag is explicitly recorded, making the filter auditable.
- Every retained tool carries exactly one data-type label (LC-MS, GC-MS, or NMR) and one functionality label drawn from the controlled vocabulary (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other).
- No pathway analysis tools remain in the filtered catalogue.
- Boundary checks: tools with exactly 50 citations are included, tools with 49 or fewer are included only if covered by the survey.

## Limitations

- The citation threshold is date-stamped (as of 08/09/16), so the filter is not reproducible against current Web of Science counts without using historical citation data.
- The MetabolomicsTools GitHub project is no longer updated and was archived because it no longer reflected the current state of the field.
- A tool meeting only one criterion is included, so the filter reflects popularity proxies (citations, self-reported survey usage) rather than tool quality; future work should directly compare tools' abilities to perform specific tasks such as peak picking.

## Evidence

- [intro] Inclusion criterion for 'most widely used' tools is either >=50 citations on Web of Science (as of 08/09/16) or reported use in the recent Metabolomics Society survey.: "the use of the tool being reported in the recent Metabolomics Society survey (Weber et al. 2016)"
- [abstract] The most widely used tools were selected for inclusion in the review by either ≥ 50 citations on Web of Science: "The most widely used tools were selected for inclusion in the review by either ≥ 50 citations on Web of Science"
- [abstract] An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary.: "An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary."
- [intro] This review will therefore focus on tools that are freely available to use: open-source, free for non-commercial or free to use.: "This review will therefore focus on tools that are freely available to use: open-source, free for non-commercial or free to use."
- [readme] The GitHub project README confirms the classification scheme by instrumental data type or major functionality and its archived status.: "The Metabolomics Tools Wiki aims to classify metabolomics software tools by either the instrumental data type taken as input or major functionality."
