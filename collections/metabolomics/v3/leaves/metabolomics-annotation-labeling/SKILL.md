---
name: metabolomics-annotation-labeling
description: Use when an untargeted metabolomics putative-annotation table needs each
  compound assigned a biological-origin label to prioritize metabolites and gain preliminary
  insight into the studied biological system.
when_to_use_negative:
- The goal is deduplication or merging of redundant annotation rows rather than origin
  labeling — use REname or RowMerger instead
- Scoring features by adduct probability and intensity correlations is needed — that
  is TPMetrics' role, not Tagger's
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TurboPutative
  role: Web platform whose Tagger module performs the compound-origin classification;
    also accessible via its RESTful API
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:cde4af72634d7e749922fb6fbc8ad9ecc4fbc8d2b18063e7794714982c5666be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolomics-annotation-labeling

## Summary

Implements the Tagger module of TurboPutative, which classifies putative annotation entries from untargeted metabolomics datasets by compound origin (nutrient, drug, microbiota-dependent metabolite, or natural product/plant) by parsing reference databases and applying halogen/peptide regex classifiers.

## When to use

Use when an untargeted metabolomics putative-annotation table needs each compound assigned a biological-origin label to prioritize metabolites and gain preliminary insight into the studied biological system.

## When NOT to use

- The goal is deduplication or merging of redundant annotation rows rather than origin labeling — use REname or RowMerger instead
- Scoring features by adduct probability and intensity correlations is needed — that is TPMetrics' role, not Tagger's

## Inputs

- Untargeted metabolomics putative-annotation dataset (compound/feature list, e.g. the TurboPutative demo dataset)
- Reference compound lists: HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, Metabolomics Data Explorer MDMs

## Outputs

- tagger_output.tsv — compound-to-origin-label table (nutrient, drug, MDM, natural product/plant, or regex-derived halogen/peptide label)

## How to apply

Download the demo dataset from the TurboPutative web server (https://proteomics.cnic.es/TurboPutative/) and parse the reference compound lists — HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, and Metabolomics Data Explorer MDMs — into searchable lookup structures keyed by compound name/synonym. Implement the halogen and peptide regular-expression classifiers to detect halogenated compounds and peptide-like names. For each compound in the dataset, match against the parsed reference lists first, then apply the regex classifiers, assigning one origin label per compound. Write the compound-to-label table to tagger_output.tsv, mirroring the released Tagger module's classification behavior.

## Related tools

- **TurboPutative** (Web platform whose Tagger module performs the compound-origin classification; also accessible via its RESTful API) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Examples

```
python tagger.py --input demo_dataset.tsv --references hmdb_nutrients.tsv drugbank_hmdb_drugs.tsv plantcyc.tsv lotus.tsv mde_mdm.tsv --output tagger_output.tsv
```

## Evaluation signals

- Every demo-dataset compound receives a non-empty origin label in tagger_output.tsv
- Label assignments are consistent with the classification behavior of the released Tagger module (expert review)
- Each compound is matched against reference lists (HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, MDE MDMs) before regex classifiers are applied
- Labels fall within the Tagger ontology: nutrients, drugs, microbiota-dependent metabolites (MDM), or natural products/plants

## Limitations

- Classification depends on coverage and synonym completeness of the parsed external databases; compounds absent from all references rely solely on the halogen/peptide regex classifiers
- Tagger assigns one origin label per compound and provides only preliminary biological insight, not pathway-level scoring (that requires TPMetrics)

## Evidence

- [other] Tagger classifies the compounds in the dataset according to their origin by parsing several databases, labeling them as nutrients, drugs, microbiota-dependent metabolites (MDM), or natural products/plants.: [evidence span withheld — anchor status: not_found]
- [other] TurboPutative web server at https://proteomics.cnic.es/TurboPutative/ with HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, and Metabolomics Data Explorer MDMs as reference lists.: [evidence span withheld — anchor status: not_found]
- [abstract] Tagger classifies the different compounds and provides preliminary insights into the biological system studied.: "Tagger classifies the different compounds and provides preliminary insights into the biological system studied"
