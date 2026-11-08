---
name: isomer-redundancy-annotation-handling
description: Use when an untargeted metabolomics annotation table in CMM format contains
  multiple putative annotations per m/z feature (isomers, equivalent compound names,
  redundant rows) and must be reduced before manual curation.
when_to_use_negative:
- The annotation table is already curated and contains no redundant or isomer-equivalent
  putative entries, so RowMerger/REname would have nothing to merge
- The input is raw mass-spectrometry data (e.g., mzML) rather than an annotation table
  — TurboPutative operates on putative annotation tables, not spectra
- Only compound classification is needed without any redundancy handling — running
  the full four-module workflow would be excessive
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: TurboPutative
  role: Web server (https://proteomics.cnic.es/TurboPutative/) whose REname and RowMerger
    modules perform isomer/equivalent-compound recognition, redundant data removal,
    and row merging on the CMM annotation table
  repo: CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:11b28388d528a6f6b67a53d627c27e56d69638b793ed5a51eb5467d8060c35b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# isomer-redundancy-annotation-handling

## Summary

A curation skill for untargeted metabolomics putative-annotation tables that recognizes isomers and equivalent compounds, removes redundant annotation entries, and merges remaining rows to shrink massive datasets before manual inspection and metabolite prioritization.

## When to use

Use when an untargeted metabolomics annotation table in CMM format contains multiple putative annotations per m/z feature (isomers, equivalent compound names, redundant rows) and must be reduced before manual curation.

## When NOT to use

- The annotation table is already curated and contains no redundant or isomer-equivalent putative entries, so RowMerger/REname would have nothing to merge
- The input is raw mass-spectrometry data (e.g., mzML) rather than an annotation table — TurboPutative operates on putative annotation tables, not spectra
- Only compound classification is needed without any redundancy handling — running the full four-module workflow would be excessive

## Inputs

- CMM-format putative annotation table from untargeted metabolomics (e.g., the working-example table from the Lorenzo et al. 2021 atherosclerosis study, 2,275 entries)
- Feature intensity and metadata datasets for optional TPMetrics scoring

## Outputs

- curated_annotation_table.tsv with redundant/isomer-equivalent annotations removed (1,059 rows in the atherosclerosis case study, ~92% reduction)
- Tagger compound classifications and preliminary biological-system insights
- TPMetrics annotation scores based on adduct probability and feature correlations (optional final step)

## How to apply

Take the CMM-format annotation table produced by the annotation workflow (e.g., from the atherosclerosis case study) and run the TurboPutative pipeline modules in order: Tagger first to classify the putative compounds and generate preliminary biological-system insights, then REname to recognize isomers and equivalent compounds and remove redundant annotation entries, then RowMerger to merge remaining rows and reduce the dataset size for manual comparison among annotations. The rationale is that massive untargeted datasets carry multiple putative annotations per feature that inflate manual curation burden; the authors report an 80%-90% reduction overall, with the atherosclerosis working example reducing 2,275 entries to 1,059 (~92%, at the upper extreme). Optionally finish with TPMetrics, which combines feature intensity and metadata and calculates a score based on adduct probability and feature correlations to prioritize metabolites for pathway analysis. Save the curated output as curated_annotation_table.tsv and verify the row count against the expected reduction.

## Related tools

- **TurboPutative** (Web server (https://proteomics.cnic.es/TurboPutative/) whose REname and RowMerger modules perform isomer/equivalent-compound recognition, redundant data removal, and row merging on the CMM annotation table) — CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- Curated output file exists and its row count equals 1,059 for the atherosclerosis working example, consistent with the reported ~92% reduction from 2,275 entries
- Overall entry reduction falls within the 80%-90% range TurboPutative reports for massive datasets
- Isomers and equivalent compound names are grouped/merged rather than listed as separate rows after REname and RowMerger
- Redundant annotation entries present in the input CMM table are absent from the curated output table

## Limitations

- The ~92% atherosclerosis reduction is at the upper extreme of the 80%-90% range; typical massive datasets may show smaller reductions
- No discussion-section text or changelog is available in the source document, so failure modes and edge cases beyond the reported case study are not documented
- RowMerger output is intended to facilitate manual comparison among annotations — human review is still required after automated reduction

## Evidence

- [abstract] REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal: "REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal"
- [abstract] RowMerger reduces the dataset size, facilitating the manual comparison among annotations: "RowMerger reduces the dataset size, facilitating the manual comparison among annotations"
- [abstract] reduce the number of these entries by 80%-90%: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
