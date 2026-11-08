---
name: regular-expression-compound-classification
description: Use when a putative-annotation compound list from untargeted metabolomics
  needs origin labels (nutrient, drug, MDM, natural product/plant) via reference-list
  matching plus regex rules, as in Tagger.
when_to_use_negative:
- The dataset is already a reduced feature/annotation table downstream of REname and
  RowMerger, where compound origin labels are not the analytical goal
- The task is scoring features by adduct probability and correlations (TPMetrics)
  rather than labeling compound origin
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TurboPutative
  role: Web platform whose Tagger module performs the compound origin classification;
    demo dataset is obtained from its web server
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:4c89cfcffe667c9de3e04f7d62e47a1e633dea16b7b094ac10debf1d84bf9ab8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# regular-expression-compound-classification

## Summary

Reconstructs the Tagger module of TurboPutative, which classifies each compound in an untargeted metabolomics dataset by origin (nutrients, drugs, microbiota-dependent metabolites, or natural products/plants) by matching parsed reference lists and applying halogen/peptide regular-expression classifiers. It provides preliminary biological context for the system under study before deeper prioritization.

## When to use

Use when a putative-annotation compound list from untargeted metabolomics needs origin labels (nutrient, drug, MDM, natural product/plant) via reference-list matching plus regex rules, as in Tagger.

## When NOT to use

- The dataset is already a reduced feature/annotation table downstream of REname and RowMerger, where compound origin labels are not the analytical goal
- The task is scoring features by adduct probability and correlations (TPMetrics) rather than labeling compound origin

## Inputs

- Demo dataset of putative compound annotations from the TurboPutative web server
- Reference compound lists: HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, and Metabolomics Data Explorer MDMs

## Outputs

- tagger_output.tsv — compound-to-origin-label table with one origin label per compound (nutrients, drugs, MDM, or natural products/plants)

## How to apply

Download the demo dataset from the TurboPutative web server (https://proteomics.cnic.es/TurboPutative/) and parse the reference compound lists — HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, and Metabolomics Data Explorer MDMs — into searchable lookup structures keyed by compound name and synonym. Implement the halogen regular-expression classifier to flag halogenated compound names and the peptide regular-expression classifier to flag peptide-like names. For each compound in the demo dataset, match against the parsed reference lists first and then apply the regex classifiers, assigning one origin label per compound. Write the resulting compound-to-label table to tagger_output.tsv; the classification replicates the Tagger module of the four-module TurboPutative platform, which can also be driven via its RESTful API services for pipeline automation.

## Related tools

- **TurboPutative** (Web platform whose Tagger module performs the compound origin classification; demo dataset is obtained from its web server) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Examples

```
python tagger.py --input demo_dataset.tsv --references hmdb_nutrients.tsv,drugbank_hmdb_drugs.tsv,plantcyc.tsv,lotus.tsv,mde_mdms.tsv --output tagger_output.tsv
```

## Evaluation signals

- Every demo-dataset compound receives a non-empty origin label in tagger_output.tsv
- Label assignments are consistent with the classification behavior of the released Tagger module (expert review)
- Reference lookups cover compound synonyms, since matching is keyed by compound name/synonym before regex classification
- Halogenated and peptide-like names are captured by the regex classifiers only after reference-list matching fails to assign a label

## Limitations

- Origin labels are preliminary insights into the biological system, not definitive biological annotations
- Coverage depends on the reference databases parsed (HMDB, DrugBank, PlantCyc, LOTUS, MDE); compounds absent from all lists fall back to regex-based halogen/peptide heuristics

## Evidence

- [abstract] Tagger classifies compounds and provides preliminary insights: "Tagger classifies the different compounds and provides preliminary insights into the biological system studied"
- [abstract] TurboPutative four-module platform: "Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
- [other] Tagger origin labels and databases: [evidence span withheld — anchor status: not_found]
