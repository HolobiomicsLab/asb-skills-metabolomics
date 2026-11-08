---
name: cmm-format-annotation-table-processing
description: Use when a CMM-format annotation table from untargeted metabolomics carries
  multiple putative annotations per feature and must be curated to a smaller, score-ranked
  entry set for manual inspection or pathway analysis.
when_to_use_negative:
- The annotation table has already been curated, i.e. redundant isomer/equivalent
  entries were removed and rows merged — re-running RowMerger and REname would offer
  no further 80%-90% reduction
- The input is a raw feature-intensity table without putative metabolite annotations,
  since Tagger, REname, and RowMerger operate on annotation entries rather than raw
  features
- The goal is small-scale manual annotation review where the number of putative entries
  is already low enough that no massive-dataset reduction is needed
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TurboPutative
  role: Web platform whose four modules (Tagger, REname, RowMerger, TPMetrics) are
    executed sequentially on the CMM-format annotation table to classify, de-redundify,
    merge, and score putative annotations
  repo: CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:93a8abaec3daa3d5262682a3ccc2f133561c0a898a4f63a7865479e61b75189d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cmm-format-annotation-table-processing

## Summary

Curation of a CMM-format putative annotation table from untargeted metabolomics by running the four TurboPutative modules (Tagger, REname, RowMerger, TPMetrics) sequentially, reducing redundant annotation entries by 80%-90% before manual comparison and pathway analysis.

## When to use

Use when a CMM-format annotation table from untargeted metabolomics carries multiple putative annotations per feature and must be curated to a smaller, score-ranked entry set for manual inspection or pathway analysis.

## When NOT to use

- The annotation table has already been curated, i.e. redundant isomer/equivalent entries were removed and rows merged — re-running RowMerger and REname would offer no further 80%-90% reduction
- The input is a raw feature-intensity table without putative metabolite annotations, since Tagger, REname, and RowMerger operate on annotation entries rather than raw features
- The goal is small-scale manual annotation review where the number of putative entries is already low enough that no massive-dataset reduction is needed

## Inputs

- CMM-format annotation table of putative metabolite annotations from untargeted metabolomics (working example: CMM-generated annotation table from the Lorenzo et al. 2021 atherosclerosis study)
- Dataset with feature intensities and relevant metadata, required as input to the TPMetrics module

## Outputs

- curated_annotation_table.tsv with a reduced row count (benchmark: 1,059 rows from 2,275, ~92% reduction)
- Compound classification and preliminary biological-system insights from Tagger
- Redundancy-collapsed annotation table from REname (isomers/equivalent compounds recognized)
- Merged row table from RowMerger sized for manual comparison among annotations
- Scored table from TPMetrics with scores based on adduct probability and feature correlations

## How to apply

Obtain the working-example CMM-format annotation table from the TurboPutative web server (https://proteomics.cnic.es/TurboPutative/) as the workflow input. Run Tagger first to classify the putative compounds and provide preliminary insights into the biological system studied. Run REname to recognize isomers and equivalent compounds and remove redundant annotation entries, improving annotation handling and visualization. Run RowMerger to merge remaining rows and reduce dataset size for manual comparison among annotations, then run TPMetrics to combine feature intensities and metadata and calculate a score based on adduct probability and feature correlations. Benchmark against the article's executed case study on the Lorenzo et al. 2021 atherosclerosis dataset, where RowMerger reduced the table from 2,275 to 1,059 entries, representing an overall ~92% reduction from the original annotation table. Save the final curated table as curated_annotation_table.tsv; the workflow can alternatively be automated through TurboPutative's RESTful API services in programmed bioinformatics pipelines.

## Related tools

- **TurboPutative** (Web platform whose four modules (Tagger, REname, RowMerger, TPMetrics) are executed sequentially on the CMM-format annotation table to classify, de-redundify, merge, and score putative annotations) — CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- curated_annotation_table.tsv exists and its row count equals 1,059 for the Lorenzo et al. 2021 atherosclerosis working example
- Overall reduction is consistent with the reported ~92% for this working example, at the upper extreme of the 80%-90% range TurboPutative reports for massive datasets
- REname output no longer contains redundant entries for recognized isomers and equivalent compounds
- TPMetrics output includes a score computed from adduct probability and feature correlations alongside combined feature intensity and metadata
- Module order is preserved: Tagger → REname → RowMerger (third module) → TPMetrics

## Limitations

- The reported 80%-90% entry reduction applies to massive datasets; the atherosclerosis working example sits at the upper extreme (~92%), so smaller datasets may reduce less
- RowMerger's output is sized to facilitate manual comparison among annotations — manual inspection is still expected after the automated curation
- The source document provides no discussion-section text (no changelog found), so caveats beyond the abstract-level scope statements are not documented here

## Evidence

- [abstract] massive-dataset entry reduction range: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
- [abstract] RowMerger purpose: "RowMerger reduces the dataset size, facilitating the manual comparison among annotations"
- [abstract] TPMetrics scoring method: "TPMetrics combines different datasets with feature intensity and relevant information for the researcher and calculates a score based on adduct probability and feature correlations"
- [abstract] REname de-redundification: "REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal"
- [abstract] automation path: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
