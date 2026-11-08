---
name: web-server-restful-api-workflow-execution
description: Use when a massive untargeted metabolomics table contains multiple putative
  annotations per feature and needs classification (e.g. origin labels from Tagger)
  and deduplication via an automatable web workflow.
when_to_use_negative:
- The goal is raw spectral processing (e.g. feature detection on mzML/NetCDF files)
  rather than handling and classifying already-putative annotation entries
- Only a handful of annotations need manual curation, where scripting a RESTful API
  call adds overhead without benefit
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: TurboPutative
  role: Web server whose four modules (Tagger, REname, RowMerger, TPMetrics) are invoked
    here via web interface or RESTful API to classify and handle putative metabolite
    annotations
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:288ca213d968507b4917d76672f4afd28960bb284b0dce92b4fa1c81d352496a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# web-server-restful-api-workflow-execution

## Summary

Executes TurboPutative's annotation-handling and metabolite-classification workflow (Tagger, REname, RowMerger, TPMetrics) through the web server at https://proteomics.cnic.es/TurboPutative/ or programmatically via its RESTful API, so results are reproducible in scripted pipelines.

## When to use

Use when a massive untargeted metabolomics table contains multiple putative annotations per feature and needs classification (e.g. origin labels from Tagger) and deduplication via an automatable web workflow.

## When NOT to use

- The goal is raw spectral processing (e.g. feature detection on mzML/NetCDF files) rather than handling and classifying already-putative annotation entries
- Only a handful of annotations need manual curation, where scripting a RESTful API call adds overhead without benefit

## Inputs

- Table of putative annotations from an untargeted metabolomics dataset (working-example input table from the TurboPutative web server)
- Feature intensity and relevant metadata (for TPMetrics)

## Outputs

- Classification table assigning each compound an origin category (e.g. tagger_results.csv with peptides, halogen-containing, MDM, drugs, nutrients, plants, natural products counts)
- Reduced annotation table with redundant data removed (REname/RowMerger) and score based on adduct probability and feature correlations (TPMetrics)

## How to apply

Download the working-example input table of putative annotations from the TurboPutative web server, then submit it to the relevant module (e.g. Tagger for origin classification) either through the web interface or programmatically via the RESTful API, which the paper offers for inclusion in programmed bioinformatics pipelines. Parse the module output and tabulate results per class — for the atherosclerosis case study, Tagger classifies 420 potential metabolites into peptides (125), halogen-containing molecules (32), microbiota-dependent metabolites (2), drugs (59), nutrients (84), plants (34), and natural products (148) — and save them as tagger_results.csv. When reducing massive datasets, expect the full TurboPutative workflow (REname redundant-data removal plus RowMerger dataset-size reduction) to shrink putative annotation entries by 80%-90% before downstream biological knowledge extrapolation and pathway prioritization.

## Related tools

- **TurboPutative** (Web server whose four modules (Tagger, REname, RowMerger, TPMetrics) are invoked here via web interface or RESTful API to classify and handle putative metabolite annotations) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- Sum of the seven origin-class counts equals the total classified metabolites (125+32+2+59+84+34+148 = 420 for the atherosclerosis working example)
- Every compound in the input table receives exactly one Tagger origin classification from {peptides, halogen-containing, MDM, drugs, nutrients, plants, natural products}
- Results are persisted with the expected schema (tagger_results.csv) matching the class-by-count breakdown
- When the full module chain is run, putative annotation entries are reduced by roughly 80%-90% versus the input table

## Limitations

- TPMetrics scoring depends on feature intensity and metadata being supplied alongside the annotation table; without them no adduct-probability/correlation score can be computed
- The workflow is designed for massive datasets with multiple putative annotations; small, already-curated tables will not benefit from the 80%-90% entry reduction
- The article's discussion section is not available, so edge-case behaviors and changelog details cannot be verified from the source

## Evidence

- [abstract] TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
- [abstract] Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics): "Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger,"
- [abstract] Tagger classifies the different compounds and provides preliminary insights into the biological system studied: "Tagger classifies the different compounds and provides preliminary insights into the biological system studied"
- [results] In the atherosclerosis working example, Tagger classified a total of 420 potential metabolites as peptides (125), halogen-containing molecules (32), microbiota-dependent metabolites (2), drugs (59), nutrients (84), plants (34), and natural products (148).: [evidence span withheld — anchor status: not_found]
