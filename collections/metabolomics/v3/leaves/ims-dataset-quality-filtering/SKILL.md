---
name: ims-dataset-quality-filtering
description: Use when curating a large batch of public METASPACE IMS datasets (e.g.,
  before training or benchmarking a metabolite annotation model) and you need to exclude
  non-centroided or low-quality profiles.
when_to_use_negative:
- Input is a single, locally acquired, already centroided and curated imaging MS run
  rather than a batch of public repository datasets
- The goal is FDR estimation or metabolite scoring for one dataset — that is handled
  by the METASPACE-ML model itself, not this dataset-level curation filter
- Datasets are not represented in the METASPACE knowledge base (e.g., QTOF data with
  no public availability), so the annotation-based filters cannot be computed
edam_operation: http://edamontology.org/operation_3695
edam_topics: []
tools:
- name: METASPACE
  role: Source of the public dataset metadata and annotation records that the filters
    are applied to
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE-ML
  role: 'Downstream consumer: filtered datasets form the training/evaluation corpus
    for the METASPACE-ML model'
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:10784d6a8da1ce6b3dfa51bf4d4f32b0add05e44e1ffcbadd09e1ec00a44255d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ims-dataset-quality-filtering

## Summary

A quality-control workflow for selecting public METASPACE imaging mass spectrometry datasets suitable for METASPACE-ML training/evaluation, using a centroid-overlap (proportion_overlap) heuristic plus annotation-count and peak-count thresholds. It reduces the 9251 downloaded public datasets to 7713 curated ones.

## When to use

Use when curating a large batch of public METASPACE IMS datasets (e.g., before training or benchmarking a metabolite annotation model) and you need to exclude non-centroided or low-quality profiles.

## When NOT to use

- Input is a single, locally acquired, already centroided and curated imaging MS run rather than a batch of public repository datasets
- The goal is FDR estimation or metabolite scoring for one dataset — that is handled by the METASPACE-ML model itself, not this dataset-level curation filter
- Datasets are not represented in the METASPACE knowledge base (e.g., QTOF data with no public availability), so the annotation-based filters cannot be computed

## Inputs

- Metadata and annotation records for ~9251 public METASPACE datasets (from the METASPACE knowledge base)
- Per-dataset ion intensity pixel data (to select top 50 highest-intensity pixels)
- Theoretical/annotated m/z values used with a 3 ppm tolerance for overlap scoring

## Outputs

- retained_datasets.csv containing the identifiers of datasets passing all filters (expected: 7713 identifiers)
- Per-dataset quality metrics: proportion_overlap score, annotation count at FDR 20%, median peak count

## How to apply

Retrieve metadata and annotation records for all public datasets in the METASPACE knowledge base (9251 datasets downloaded in September 2023). For each dataset, take the top 50 highest-intensity pixels and compute a proportion_overlap score using a 3 ppm m/z tolerance; this detects profile-mode data whose observed peaks do not match theoretical centroided annotations. Exclude datasets whose proportion_overlap exceeds 0.5, that have fewer than 10 annotations at FDR 20%, or whose median peak count is greater than 50,000. Write the identifiers of datasets passing all filters to retained_datasets.csv and check that the retained count matches the expected 7713 of 9251.

## Related tools

- **METASPACE** (Source of the public dataset metadata and annotation records that the filters are applied to) — https://github.com/metaspace2020/metaspace
- **METASPACE-ML** (Downstream consumer: filtered datasets form the training/evaluation corpus for the METASPACE-ML model) — https://github.com/metaspace2020/metaspace

## Evaluation signals

- retained_datasets.csv exists and contains exactly 7713 dataset identifiers out of the 9251 candidates
- No retained dataset has a proportion_overlap score above 0.5 (computed on top 50 highest-intensity pixels at 3 ppm tolerance)
- Every retained dataset has at least 10 annotations at FDR 20%
- Every retained dataset has a median peak count of at most 50,000
- Composition of retained datasets is consistent with the reported split (84% animal, 14.3% plant datasets)

## Limitations

- Selection is biased towards public datasets represented in METASPACE (contributed from ~47 labs), so the filtered corpus inherits that bias
- QTOF-based datasets are underrepresented because of the lack of publicly available data needed for statistically reliable model evaluation
- Filters are tuned to the September 2023 METASPACE snapshot; a different download date may yield different candidate and retained counts

## Evidence

- [other] Exclude any dataset whose proportion_overlap score exceeds 0.5.: [evidence span withheld — anchor status: not_found]
- [other] For each dataset, select the top 50 highest-intensity pixels and compute the proportion_overlap score using a 3 ppm m/z tolerance.: [evidence span withheld — anchor status: not_found]
- [readme] spatial metabolite knowledgebase of the metabolites from thousands of public datasets provided by the community.: "spatial metabolite knowledgebase of the metabolites from thousands of public datasets provided by the community"
- [discussion] this selection is biased towards public datasets represented in METASPACE: "this selection is biased towards public datasets represented in METASPACE"
- [intro] This opened an opportunity to train a machine-learning model on this big data.: "This opened an opportunity to train a machine-learning model on this big data."
