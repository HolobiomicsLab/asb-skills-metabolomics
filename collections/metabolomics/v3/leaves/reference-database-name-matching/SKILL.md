---
name: reference-database-name-matching
description: Use when an untargeted metabolomics compound/annotation list needs biological-origin
  labels (nutrient, drug, microbiota-dependent metabolite, natural product/plant,
  halogenated, peptide-like) derived purely from compound names and synonyms rather
  than from spectra or structures.
when_to_use_negative:
- The input dataset already contains compound origin/class annotations and only needs
  reformatting or deduplication (use REname/RowMerger instead).
- The goal is structural or spectral metabolite confirmation (e.g., MS/MS-based identification)
  rather than name-based classification.
- The input is a feature-intensity/metadata table for scoring (TPMetrics handles adduct-probability
  scoring), not a compound name list.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TurboPutative
  role: Provides the Tagger module that implements this name/synonym matching against
    reference databases plus halogen/peptide regex classification; available as a
    web server and RESTful API.
  repo: CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:7616689586761f1b25b37a7fe6bc38daf1b1e050598abf7aa5bb11dd3b280d27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# reference-database-name-matching

## Summary

Classifies compounds from untargeted metabolomics annotation tables by matching compound names/synonyms against curated reference lists (HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, Metabolomics Data Explorer MDMs), as implemented in the Tagger module of TurboPutative to add biological-origin context to putative annotations.

## When to use

Use when an untargeted metabolomics compound/annotation list needs biological-origin labels (nutrient, drug, microbiota-dependent metabolite, natural product/plant, halogenated, peptide-like) derived purely from compound names and synonyms rather than from spectra or structures.

## When NOT to use

- The input dataset already contains compound origin/class annotations and only needs reformatting or deduplication (use REname/RowMerger instead).
- The goal is structural or spectral metabolite confirmation (e.g., MS/MS-based identification) rather than name-based classification.
- The input is a feature-intensity/metadata table for scoring (TPMetrics handles adduct-probability scoring), not a compound name list.

## Inputs

- Compound/annotation name table from untargeted metabolomics (e.g., TurboPutative demo dataset or putative annotation list)
- Reference compound lists: HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, Metabolomics Data Explorer MDMs

## Outputs

- Compound-to-origin-label table (e.g., tagger_output.tsv) with one origin label per compound

## How to apply

Parse the reference compound lists — HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, and Metabolomics Data Explorer MDMs — into searchable lookup structures keyed by compound name and synonym. For each compound in the input annotation table, first attempt a lookup match against the parsed reference lists; the database match takes precedence. For compounds not found, apply the halogen and peptide regular-expression classifiers to detect halogenated compounds and peptide-like names. Assign exactly one origin label per compound and write the compound-to-label table (e.g., tagger_output.tsv). The name-based strategy is what Tagger uses to 'classify the different compounds and provide preliminary insights into the biological system studied', so prioritize recall via synonyms and treat regex classifiers as fallback rather than replacement for database matches.

## Related tools

- **TurboPutative** (Provides the Tagger module that implements this name/synonym matching against reference databases plus halogen/peptide regex classification; available as a web server and RESTful API.) — CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- Every compound in the demo dataset receives a non-empty origin label.
- Label assignments are consistent with the classification behavior of the released Tagger module of TurboPutative.
- Database matches take precedence over regex classifiers: a compound present in HMDB nutrients, DrugBank/HMDB drugs, PlantCyc, LOTUS, or the MDM list is labeled from that list, not by regex.
- Exactly one origin label is assigned per compound row.
- Halogenated and peptide-like names not present in any reference list are still captured by the regex classifiers.

## Limitations

- Classification is purely name/synonym based; compounds absent from the reference lists and not matching the halogen/peptide regexes cannot be labeled.
- Coverage is bounded by the external databases (HMDB, DrugBank, PlantCyc, LOTUS, Metabolomics Data Explorer); database updates change classification results.
- The label set is limited to Tagger's categories (nutrients, drugs, MDM, natural products/plants, plus halogen/peptide detections) and does not capture finer biological roles.

## Evidence

- [abstract] Tagger classifies the different compounds and provides preliminary insights into the biological system studied: "Tagger classifies the different compounds and provides preliminary insights into the biological system studied"
- [abstract] we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics): "we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
