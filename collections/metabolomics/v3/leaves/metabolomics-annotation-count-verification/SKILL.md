---
name: metabolomics-annotation-count-verification
description: Use when an untargeted metabolomics dataset of putative annotations must
  be classified by metabolite origin (peptides, halogen-containing, MDM, drugs, nutrients,
  plants, natural products) and the resulting category counts need to be checked against
  reported totals.
when_to_use_negative:
- The dataset is not a table of putative annotations from untargeted metabolomics
  (e.g., it is already a curated feature-intensity table — TPMetrics is the module
  that combines datasets with feature intensity and metadata)
- The goal is annotation cleanup rather than classification, e.g. recognizing isomers/equivalent
  compounds or removing redundant data — that is the REname or RowMerger step, not
  Tagger
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TurboPutative
  role: Web server and RESTful API whose Tagger module assigns each potential metabolite
    an origin classification used for the count verification
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:08b7479ee393e197417d9dc2b469826333e1cfb84e3f536bddb7416846fb1611
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolomics-annotation-count-verification

## Summary

Verifies metabolite classification counts from the TurboPutative Tagger module by re-running classification on a putative-annotation table and reconciling per-origin category counts against published reference totals.

## When to use

Use when an untargeted metabolomics dataset of putative annotations must be classified by metabolite origin (peptides, halogen-containing, MDM, drugs, nutrients, plants, natural products) and the resulting category counts need to be checked against reported totals.

## When NOT to use

- The dataset is not a table of putative annotations from untargeted metabolomics (e.g., it is already a curated feature-intensity table — TPMetrics is the module that combines datasets with feature intensity and metadata)
- The goal is annotation cleanup rather than classification, e.g. recognizing isomers/equivalent compounds or removing redundant data — that is the REname or RowMerger step, not Tagger

## Inputs

- Working-example input table of putative annotations (CSV) downloadable from https://proteomics.cnic.es/TurboPutative/
- Reference published classification totals (420 metabolites: 125 peptides, 32 halogen-containing, 2 MDM, 59 drugs, 84 nutrients, 34 plants, 148 natural products)

## Outputs

- tagger_results.csv with per-metabolite origin classification
- Per-origin-category count table reconciled against the published totals

## How to apply

Download the working-example input table of putative annotations from the TurboPutative web server (https://proteomics.cnic.es/TurboPutative/). Submit the table to the Tagger module, either through the web interface or programmatically via its RESTful API, so that each potential metabolite is assigned an origin classification. Parse the Tagger output and tabulate counts of metabolites per origin class — peptides, halogen-containing molecules, microbiota-dependent metabolites (MDM), drugs, nutrients, plants, and natural products. In the atherosclerosis working example this yields 420 classified metabolites: 125 peptides, 32 halogen-containing, 2 MDM, 59 drugs, 84 nutrients, 34 plants, and 148 natural products. Compare the tabulated counts against these reference totals and save the classification results as tagger_results.csv; any per-class mismatch signals an ingestion, API, or parsing error.

## Related tools

- **TurboPutative** (Web server and RESTful API whose Tagger module assigns each potential metabolite an origin classification used for the count verification) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- Total number of classified metabolites equals 420 in the atherosclerosis working example
- Per-class counts match the published breakdown exactly: 125 peptides, 32 halogen-containing, 2 MDM, 59 drugs, 84 nutrients, 34 plants, 148 natural products
- Sum of the seven per-class counts equals the reported total (125+32+2+59+84+34+148 = 420)
- tagger_results.csv is produced with one origin classification per metabolite from the Tagger output
- Counts are reproducible via both the web interface and the RESTful API submission path

## Limitations

- Tagger provides only preliminary insights into the biological system studied — classification is a first-pass categorization, not definitive metabolite identification
- The verified counts are specific to the atherosclerosis working example; other datasets will produce different per-class distributions
- The article provides no discussion-section caveats or changelog content; other failure modes are not documented in the available context

## Evidence

- [abstract] Tagger classifies the different compounds and provides preliminary insights into the biological system studied: "Tagger classifies the different compounds and provides preliminary insights into the biological system studied"
- [abstract] RESTful API automation: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
- [other] Tool availability and platform URL: "Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
