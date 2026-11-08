---
name: metabolite-origin-classification
description: Use when an untargeted metabolomics feature/annotation table needs compound-origin
  labels (e.g., nutrient, drug, MDM, plant, peptide, halogenated) to prioritize metabolites
  for pathway analysis.
when_to_use_negative:
- The task is reducing redundant putative annotations rather than assigning origin
  labels — use REname or RowMerger modules instead
- The input already contains curated compound identities with known origin classes,
  so database matching would add no information
- Compounds are outside the scope of the parsed reference lists (HMDB nutrients, DrugBank/HMDB
  drugs, PlantCyc, LOTUS, Metabolomics Data Explorer MDMs) and fail both regex classifiers,
  so no label can be assigned
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TurboPutative
  role: Tagger module classifies compounds by origin (nutrients, drugs, MDM, plants/natural
    products, peptides, halogenated) against parsed reference databases and regex
    classifiers
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:e2f5d84292acf8388e0f4190fd14a180a44fe7825e8d23c230ea1091ddf52d48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metabolite-origin-classification

## Summary

Classifies compounds from untargeted metabolomics putative-annotation tables by biological/chemical origin (nutrients, drugs, microbiota-dependent metabolites, plants/natural products, peptides, halogenated molecules) using the TurboPutative Tagger module, which parses reference databases and applies halogen/peptide regex classifiers.

## When to use

Use when an untargeted metabolomics feature/annotation table needs compound-origin labels (e.g., nutrient, drug, MDM, plant, peptide, halogenated) to prioritize metabolites for pathway analysis.

## When NOT to use

- The task is reducing redundant putative annotations rather than assigning origin labels — use REname or RowMerger modules instead
- The input already contains curated compound identities with known origin classes, so database matching would add no information
- Compounds are outside the scope of the parsed reference lists (HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, Metabolomics Data Explorer MDMs) and fail both regex classifiers, so no label can be assigned

## Inputs

- Table of putative metabolite annotations (compound names/synonyms) from an untargeted metabolomics dataset
- Demo dataset or working-example input table from the TurboPutative web server (https://proteomics.cnic.es/TurboPutative/)

## Outputs

- Compound-to-origin-label table (e.g., tagger_output.tsv or tagger_results.csv) with one origin label per compound
- Per-category classification counts (peptides, halogen-containing, MDM, drugs, nutrients, plants, natural products)

## How to apply

Submit the table of putative annotations to the Tagger module via the TurboPutative web interface or its RESTful API. Tagger first matches compound names/synonyms against parsed reference lists (HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, and Metabolomics Data Explorer MDMs) to assign origin labels, then applies halogen and peptide regular-expression classifiers to detect halogenated compounds and peptide-like names. Each compound receives one origin label; results are written as a compound-to-label table (e.g., tagger_output.tsv or tagger_results.csv). Validate by confirming every input compound receives a non-empty label and by comparing per-category counts against known classification behavior — e.g., in the atherosclerosis working example Tagger classified 420 potential metabolites: 125 peptides, 32 halogen-containing, 2 MDM, 59 drugs, 84 nutrients, 34 plants, and 148 natural products.

## Related tools

- **TurboPutative** (Tagger module classifies compounds by origin (nutrients, drugs, MDM, plants/natural products, peptides, halogenated) against parsed reference databases and regex classifiers) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- Every demo-dataset compound receives a non-empty origin label
- Per-category counts match known reference behavior — e.g., 420 metabolites split as 125 peptides / 32 halogen-containing / 2 MDM / 59 drugs / 84 nutrients / 34 plants / 148 natural products in the atherosclerosis working example
- Output table (tagger_output.tsv / tagger_results.csv) contains one origin label per input compound with no unmatched rows
- Halogen and peptide regex classifiers correctly flag halogenated compounds and peptide-like names in a held-out sanity check

## Limitations

- Compounds absent from the reference databases (HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, Metabolomics Data Explorer MDMs) and not matching the halogen/peptide regexes remain unclassified
- Each compound receives a single origin label, which may be ambiguous for compounds spanning multiple categories (e.g., a plant-derived drug)
- Classification quality depends on synonym coverage of the parsed lookup structures; naming variants in the input annotation table may cause missed matches

## Evidence

- [other] Tagger classifies the compounds in the dataset according to their origin by parsing several databases, labeling them as nutrients, drugs, microbiota-dependent metabolites (MDM), or natural products/plants.: [evidence span withheld — anchor status: not_found]
- [abstract] Tagger classifies the different compounds and provides preliminary insights into the biological system studied: "Tagger classifies the different compounds and provides preliminary insights into the biological system studied"
- [other] In the atherosclerosis working example, Tagger classified a total of 420 potential metabolites as peptides (125), halogen-containing molecules (32), microbiota-dependent metabolites (2), drugs (59), nutrients (84), plants (34), and natural products (148).: [evidence span withheld — anchor status: not_found]
