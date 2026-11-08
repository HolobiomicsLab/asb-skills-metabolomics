---
name: adduct-probability-feature-correlation-scoring
description: Use when an untargeted metabolomics dataset carries multiple putative
  annotations per feature (CMM-generated annotation tables) and annotations must be
  prioritized for pathway analysis using adduct probability and feature-correlation
  evidence.
when_to_use_negative:
- The dataset is already a curated feature table with one resolved annotation per
  feature, so no putative-annotation prioritization is needed
- The goal is compound classification or biological-system insight rather than annotation
  scoring — the Tagger module, not TPMetrics, provides that
- The input table still contains redundant entries and equivalent/isomer duplicates
  that have not yet passed through REname and RowMerger, since TPMetrics is designed
  to score the reduced annotation set
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0091
tools:
- name: TurboPutative
  role: Web platform whose TPMetrics module combines feature intensity and metadata
    with the curated annotation table and computes the adduct-probability and feature-correlation
    score; the preceding Tagger, REname, and RowMerger modules prepare its input
  repo: CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:8c73f28e5723edc7e49f1debca3e6e9c73e21e9b88893b438dc30458a0ab938c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# Adduct-probability and feature-correlation annotation scoring

## Summary

Scores putative metabolite annotations by combining feature intensity data and metadata into a ranking based on adduct probability and feature correlations, as implemented in the TPMetrics module of the TurboPutative web server for untargeted metabolomics.

## When to use

Use when an untargeted metabolomics dataset carries multiple putative annotations per feature (CMM-generated annotation tables) and annotations must be prioritized for pathway analysis using adduct probability and feature-correlation evidence.

## When NOT to use

- The dataset is already a curated feature table with one resolved annotation per feature, so no putative-annotation prioritization is needed
- The goal is compound classification or biological-system insight rather than annotation scoring — the Tagger module, not TPMetrics, provides that
- The input table still contains redundant entries and equivalent/isomer duplicates that have not yet passed through REname and RowMerger, since TPMetrics is designed to score the reduced annotation set

## Inputs

- CMM-format annotation table from untargeted metabolomics containing multiple putative annotations per feature
- Feature intensity table for the measured metabolomic features
- Researcher-relevant metadata table to be combined with annotations

## Outputs

- Scored/curated annotation table in which each putative annotation carries a TPMetrics score based on adduct probability and feature correlations
- Prioritized list of putative metabolite annotations suitable for downstream pathway analysis

## How to apply

Take the curated annotation table produced by the upstream TurboPutative modules (Tagger for compound classification, REname for isomer/equivalent-compound recognition and redundant data removal, and RowMerger for dataset-size reduction) as the input to TPMetrics. TPMetrics combines the annotation table with datasets of feature intensity and researcher-relevant metadata, then calculates a score per annotation based on adduct probability and feature correlations, so that putative entries can be ranked and prioritized for biological knowledge extrapolation and pathway analysis. The full Tagger→REname→RowMerger→TPMetrics sequence can be run interactively on the TurboPutative web server (https://proteomics.cnic.es/TurboPutative/) or scripted through its RESTful API services for automated pipelines. In the published atherosclerosis working example (Lorenzo et al. 2021), the upstream curation (through RowMerger) reduced the annotation table from 2,275 to 1,059 entries, and TPMetrics scoring operates on this reduced table rather than the full raw annotation set.

## Related tools

- **TurboPutative** (Web platform whose TPMetrics module combines feature intensity and metadata with the curated annotation table and computes the adduct-probability and feature-correlation score; the preceding Tagger, REname, and RowMerger modules prepare its input) — CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- The scored output table is built on the RowMerger-reduced annotation set; in the published working example this table has 1,059 rows (down from 2,275), an overall ~92% reduction from the original CMM annotation table
- The overall dataset reduction falls within the 80%-90% range TurboPutative reports for massive datasets (the atherosclerosis working example sits at the upper extreme)
- Every retained annotation entry carries a score derived from adduct probability and feature correlations, not an arbitrary ranking
- Feature intensity values and relevant metadata from the input datasets are joined onto each scored annotation row
- If the workflow is automated, the same result is reproducible through TurboPutative's RESTful API services

## Limitations

- The 80%-90% entry-reduction figure applies to massive datasets; the published working example represents the upper extreme of this range, so smaller or already-clean datasets may see smaller reductions
- Scoring quality depends on upstream curation: isomers/equivalent compounds and redundant entries must first be handled by REname and RowMerger
- TPMetrics requires feature intensity and metadata tables in addition to the annotation table; annotations lacking these inputs cannot be fully scored

## Evidence

- [abstract] TPMetrics scoring role: "TPMetrics combines different datasets with feature intensity and relevant information for the researcher and calculates a score based on adduct probability and feature correlations"
- [abstract] Reduction expectation for massive datasets: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
- [abstract] Upstream RowMerger step feeding TPMetrics: "RowMerger reduces the dataset size, facilitating the manual comparison among annotations"
- [abstract] Automation route for the scoring workflow: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
