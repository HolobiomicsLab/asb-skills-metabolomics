---
name: metabolite-compound-classification-tagger
description: Use when an untargeted metabolomics putative annotation table (CMM-generated
  format) needs its compounds classified and preliminary biological-system insights
  generated as the first step before REname/RowMerger curation.
when_to_use_negative:
- When redundant annotations, isomers, or equivalent compounds must be removed or
  rows merged — that is the role of the downstream REname and RowMerger modules, not
  Tagger
- When annotation-level scoring based on adduct probability and feature correlations
  is needed — that is TPMetrics, the fourth module
- When the input is already a fully curated annotation table rather than a raw putative
  annotation table needing classification
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: TurboPutative
  role: Web-server platform whose Tagger module classifies putative compounds in the
    CMM annotation table as the first workflow step; also exposes RESTful API services
    for automation
  repo: CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:b6f545ac6987c6ba040b815666a9295c5d68b5e16c4807a0ac285df6a1891f09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# metabolite-compound-classification-tagger

## Summary

The Tagger module of the TurboPutative web server (https://proteomics.cnic.es/TurboPutative/), the first of its four modules, classifies the different putative compounds in an untargeted metabolomics annotation table and provides preliminary insights into the biological system studied before further curation.

## When to use

Use when an untargeted metabolomics putative annotation table (CMM-generated format) needs its compounds classified and preliminary biological-system insights generated as the first step before REname/RowMerger curation.

## When NOT to use

- When redundant annotations, isomers, or equivalent compounds must be removed or rows merged — that is the role of the downstream REname and RowMerger modules, not Tagger
- When annotation-level scoring based on adduct probability and feature correlations is needed — that is TPMetrics, the fourth module
- When the input is already a fully curated annotation table rather than a raw putative annotation table needing classification

## Inputs

- CMM-format putative annotation table from untargeted metabolomics (e.g., the working-example atherosclerosis annotation table from the TurboPutative web server)

## Outputs

- Classified annotation table in which each putative compound is assigned a compound class
- Preliminary biological-system insights derived from the compound classification, feeding the subsequent REname module

## How to apply

Obtain the CMM-format annotation table produced for the untargeted metabolomics experiment (e.g., the working-example table from the Lorenzo et al. 2021 atherosclerosis study) and load it into the TurboPutative web server. Run Tagger as the first module of the sequential workflow: it classifies the different putative compounds in the table and yields preliminary insights into the biological system under study. Tagger's classified output is the input for the next module, REname, which recognizes isomers/equivalent compounds and removes redundant entries — so Tagger output must remain in the CMM-compatible table format. On the atherosclerosis case study, the full four-module sequence (Tagger, REname, RowMerger, TPMetrics) reduced 2,275 to 1,059 entries (~92% reduction), validating that Tagger's classified table feeds correctly into the downstream curation. The workflow can also be automated through TurboPutative's RESTful API services for inclusion in programmed bioinformatics pipelines.

## Related tools

- **TurboPutative** (Web-server platform whose Tagger module classifies putative compounds in the CMM annotation table as the first workflow step; also exposes RESTful API services for automation) — CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- Tagger output table retains the same row count as the input CMM annotation table — classification alone does not reduce entries; reduction to e.g. 2,275 → 1,059 rows happens only after REname and RowMerger run
- Every putative compound in the output table carries a compound class assignment enabling preliminary biological-system insight
- Output remains a CMM-compatible annotation table that the REname module can consume without reformatting
- Running Tagger first in the module sequence (Tagger → REname → RowMerger → TPMetrics) reproduces the reported ~92% overall reduction on the atherosclerosis working example
- If invoked via the RESTful API, the classification completes within the automated pipeline without manual intervention

## Limitations

- Tagger provides only preliminary insights into the biological system studied; definitive biological interpretation requires the full four-module pipeline plus manual comparison among annotations
- The reported 80%-90% reduction of putative annotation entries is achieved by the complete TurboPutative workflow, not by Tagger alone
- No discussion/changelog text is available in the source document, so failure modes and edge cases of Tagger are not documented here

## Evidence

- [abstract] Tagger classifies the different compounds and provides preliminary insights into the biological system studied: "Tagger classifies the different compounds and provides preliminary insights into the biological system studied"
- [abstract] TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics): "TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
- [full_text] Run the Tagger module on the input table to classify the putative compounds and generate preliminary biological-system insights.: [evidence span withheld — anchor status: not_found]
- [abstract] TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
