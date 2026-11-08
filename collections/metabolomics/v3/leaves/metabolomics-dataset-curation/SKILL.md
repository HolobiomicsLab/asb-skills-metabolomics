---
name: metabolomics-dataset-curation
description: Use when curating large batches of public METASPACE imaging MS datasets
  for downstream model training or evaluation, especially when non-centroided, low-annotation,
  or oversaturated datasets must be excluded systematically.
when_to_use_negative:
- The dataset collection is already centroided, high-quality, and small enough to
  inspect manually — the batch heuristic adds no value.
- The goal is annotating a single user-uploaded imzML file through METASPACE rather
  than curating a training corpus.
- Input is already a curated feature table or annotation list rather than raw METASPACE
  dataset records.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: METASPACE
  role: Source of the public spatial metabolomics knowledge base (dataset metadata
    and annotation records) being curated
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE-ML
  role: 'Downstream consumer: the filtered dataset set is used to train and evaluate
    the machine-learning annotation model'
  repo: https://github.com/metaspace2020/metaspace
- name: python-client
  role: Python library and example Jupyter notebooks for programmatic retrieval of
    annotations from the METASPACE database
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:51747d201bbbc56a4eb0ee119e287a423294a3dc3d2b07e63f11c7ec42564e47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  - python-client
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metabolomics-dataset-curation

## Summary

Quality-filter public METASPACE spatial metabolomics datasets (e.g., 9251 downloads down to 7713 retained) using the proportion_overlap heuristic and annotation/peak-count criteria, to assemble a reliable training corpus for METASPACE-ML.

## When to use

Use when curating large batches of public METASPACE imaging MS datasets for downstream model training or evaluation, especially when non-centroided, low-annotation, or oversaturated datasets must be excluded systematically.

## When NOT to use

- The dataset collection is already centroided, high-quality, and small enough to inspect manually — the batch heuristic adds no value.
- The goal is annotating a single user-uploaded imzML file through METASPACE rather than curating a training corpus.
- Input is already a curated feature table or annotation list rather than raw METASPACE dataset records.

## Inputs

- Metadata and annotation records for ~9251 public METASPACE datasets (via METASPACE GraphQL API / python-client)
- Per-dataset ion intensity matrices with m/z and pixel coordinates from METASPACE imzML-derived records

## Outputs

- retained_datasets.csv containing exactly 7713 dataset identifiers passing all filters
- Per-dataset quality metrics: proportion_overlap score (top 50 pixels, 3 ppm tolerance), annotation count at FDR 20%, median peak count

## How to apply

Retrieve metadata and annotation records for the full public METASPACE corpus (9251 datasets as of September 2023) from the METASPACE knowledge base, e.g. via the GraphQL API or python-client. For each dataset, take the top 50 highest-intensity pixels and compute a proportion_overlap score using a 3 ppm m/z tolerance; exclude datasets with proportion_overlap > 0.5, since high overlap indicates non-centroided or oversaturated data. Apply two additional quality gates: require at least 10 annotations at FDR 20%, and require a median peak count of at most 50,000. Write the surviving dataset identifiers to retained_datasets.csv. The rationale is that METASPACE-ML's training on this corpus depends on clean, centroided, spatially informative inputs, and the resulting 7713-dataset set (84% animal, 14.3% plant) is the curated base for model training and evaluation.

## Related tools

- **METASPACE** (Source of the public spatial metabolomics knowledge base (dataset metadata and annotation records) being curated) — https://github.com/metaspace2020/metaspace
- **METASPACE-ML** (Downstream consumer: the filtered dataset set is used to train and evaluate the machine-learning annotation model) — https://github.com/metaspace2020/metaspace
- **python-client** (Python library and example Jupyter notebooks for programmatic retrieval of annotations from the METASPACE database) — https://github.com/metaspace2020/metaspace

## Evaluation signals

- retained_datasets.csv exists and contains exactly 7713 dataset identifiers out of the 9251 downloaded (≈83% retention).
- No retained dataset has a proportion_overlap score above 0.5 (computed on top 50 highest-intensity pixels at 3 ppm m/z tolerance).
- Every retained dataset has at least 10 annotations at FDR 20% and a median peak count of at most 50,000.
- Composition of retained set is consistent with the reported split: ~84% animal and ~14.3% plant datasets.
- Identifier schema matches METASPACE dataset UUIDs as returned by the GraphQL/python-client interface.

## Limitations

- Training data selection is biased towards public datasets represented in METASPACE (originating from 47 labs), limiting generalization.
- QTOF-based datasets are underrepresented in the resulting model due to lack of publicly available data for statistically reliable evaluation.
- Less common data types, e.g. single-cell metabolomics from cultured cells or industrial lab data, are underrepresented and not validated by this curation.

## Evidence

- [results] From 9251 public datasets downloaded from METASPACE in September 2023, 7713 datasets passed the applied quality criteria, with 84% corresponding to animal datasets and 14.3% to plant datasets.: [evidence span withheld — anchor status: not_found]
- [methods] For each dataset, select the top 50 highest-intensity pixels and compute the proportion_overlap score using a 3 ppm m/z tolerance.: [evidence span withheld — anchor status: not_found]
- [intro] This opened an opportunity to train a machine-learning model on this big data.: [evidence span withheld — anchor status: not_found]
- [readme] hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase: "hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase of the metabolites from thousands of public datasets provided by the"
- [discussion] this selection is biased towards public datasets represented in METASPACE: "this selection is biased towards public datasets represented in METASPACE"
