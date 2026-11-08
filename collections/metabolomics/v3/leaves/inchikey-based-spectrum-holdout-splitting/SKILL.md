---
name: inchikey-based-spectrum-holdout-splitting
description: Use when building or benchmarking a spectral similarity model (Spec2Vec,
  cosine) on GNPS-style MS/MS libraries and you must guarantee that query compounds
  never appear in the training library, i.e. unknown-compound (structural-analogue)
  search evaluation.
when_to_use_negative:
- When the goal is to evaluate library matching of spectra whose exact duplicates
  exist in the reference library (identity matching) rather than unknown-compound
  analogue search — InChIKey holdout is designed to exclude that overlap.
- When spectra lack InChIKey annotations and cannot be linked to compounds via lookup
  (e.g. automated PubChem lookup via pubchempy); the method cannot guarantee molecule-level
  separation without these identifiers.
- When evaluating a model that is retrained per query or used transductively, where
  the train/query split concept does not apply.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Used to clean and correct metadata (including InChIKey annotations) before
    the holdout split; also handles spectrum filtering such as removing spectra with
    fewer than 10 peaks
  repo: https://github.com/matchms/matchms
- name: Spec2Vec
  role: Downstream model trained on the InChIKey-split training library and evaluated
    on the held-out query spectra
  repo: https://github.com/iomega/spec2vec
- name: spec2vec_gnps_data_analysis
  role: Notebook repository implementing the analysis, including the AllPositive dataset
    split and unknown-compound matching benchmark
  repo: https://github.com/iomega/spec2vec_gnps_data_analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:a2a74fcc74fd168e6abf91b928abff3bda37e0055150fcae2129b3cbf1e451fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Spec2Vec
  - matchms
  - spec2vec_gnps_data_analysis
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# InChIKey-based spectrum holdout splitting

## Summary

A leakage-free train/query split for MS/MS spectral ML: group spectra by their InChIKey compound identifier, randomly hold out a set of InChIKeys (e.g. 200 InChIKeys, 1030 spectra from the AllPositive GNPS dataset), and drop all spectra lacking InChIKey annotations so no molecule appears in both training and query data.

## When to use

Use when building or benchmarking a spectral similarity model (Spec2Vec, cosine) on GNPS-style MS/MS libraries and you must guarantee that query compounds never appear in the training library, i.e. unknown-compound (structural-analogue) search evaluation.

## When NOT to use

- When the goal is to evaluate library matching of spectra whose exact duplicates exist in the reference library (identity matching) rather than unknown-compound analogue search — InChIKey holdout is designed to exclude that overlap.
- When spectra lack InChIKey annotations and cannot be linked to compounds via lookup (e.g. automated PubChem lookup via pubchempy); the method cannot guarantee molecule-level separation without these identifiers.
- When evaluating a model that is retrained per query or used transductively, where the train/query split concept does not apply.

## Inputs

- Processed MS/MS spectral library with per-spectrum InChIKey annotations (e.g. AllPositive dataset: 95,320 spectra, 77,092 with InChIKey, derived from GNPS ALL_GNPS.json)
- Number of unique InChIKeys to hold out (paper used 200)

## Outputs

- Held-out query set of spectra (1030 spectra from 200 randomly chosen InChIKeys in the paper)
- Training library with no molecule overlap to queries (76,062 spectra after also removing unannotated spectra)

## How to apply

Start from a cleaned library set (e.g. the AllPositive dataset of 95,320 positive-ionization spectra from GNPS ALL_GNPS.json, of which 77,092 have InChIKey annotations, obtained after matchms metadata cleaning and removal of spectra with <10 fragment peaks). Randomly select a fixed number of unique InChIKeys — in the paper 200 InChIKeys corresponding to 1030 spectra — and move all spectra carrying those InChIKeys into the held-out query set. Critically, also remove every spectrum without an InChIKey annotation from the training side, because unannotated spectra could contain the same molecules as the query set and silently break the holdout. The remaining training set in the paper contained 76,062 spectra with no molecule overlap between training and query data. Rationale: splitting by spectrum rather than by molecule lets near-duplicate spectra of the same compound inflate apparent retrieval performance; splitting by InChIKey makes the task a true unknown-compound analogue search.

## Related tools

- **matchms** (Used to clean and correct metadata (including InChIKey annotations) before the holdout split; also handles spectrum filtering such as removing spectra with fewer than 10 peaks) — https://github.com/matchms/matchms
- **Spec2Vec** (Downstream model trained on the InChIKey-split training library and evaluated on the held-out query spectra) — https://github.com/iomega/spec2vec
- **spec2vec_gnps_data_analysis** (Notebook repository implementing the analysis, including the AllPositive dataset split and unknown-compound matching benchmark) — https://github.com/iomega/spec2vec_gnps_data_analysis

## Evaluation signals

- Zero molecule overlap: no InChIKey in the training library matches any InChIKey in the held-out query set (verify programmatically after the split).
- Counts match expectations: paper reference points are 200 held-out InChIKeys -> 1030 query spectra and 76,062 training spectra from the 95,320-spectrum AllPositive dataset.
- All spectra lacking InChIKey annotations must be absent from the final training set (they were explicitly removed to avoid overlap).
- Downstream sanity check: on a Spec2Vec analogue-search task, ~60% of the 1030 queries should have a top-10 match with Tanimoto structural similarity > 0.6, versus random-draw p = 0.0103; a much higher rate would suggest leakage from an improper split.

## Limitations

- Requires reliable InChIKey annotations; the paper obtained them only after matchms cleaning plus an automated PubChem lookup (via pubchempy), leaving spectra without annotations unusable for this splitting scheme.
- Removing all unannotated spectra shrinks the usable training library (95,320 -> 76,062 spectra here), which may hurt model training on sparser datasets.
- Only demonstrated on positive-ionization LC-MS/MS data from GNPS; generalization to other acquisition types was not assessed.

## Evidence

- [intro] To make sure that there is no overlap regarding molecules between training and query data, we also removed all spectra without InChIKey annotation.: "To make sure that there is no overlap regarding molecules between training and query data, we also removed all spectra without InChIKey annotation."
- [intro] we moved all spectra belonging to 200 randomly chosen InChIKey (1030 spectra in total): "To test whether Spec2Vec similarity scores can be used to help detect highly related molecules, we moved all spectra belonging to 200 randomly chosen InChIKey (1030 spectra in total) from the"
- [discussion] This left us with 95,320 spectra (out of which 77,092 with InChIKey) for the AllPositive dataset: "This left us with 95,320 spectra (out of which 77,092 with InChIKey) for the AllPositive dataset"
- [discussion] The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations.: "The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations."
- [readme] Cleaned and processed GNPS dataset for positive mode: [evidence span withheld — anchor status: external_doc]
