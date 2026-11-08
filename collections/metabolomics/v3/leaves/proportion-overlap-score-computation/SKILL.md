---
name: proportion-overlap-score-computation
description: Use when curating large batches of public METASPACE imaging mass spectrometry
  datasets and you must exclude non-centroided or noisy datasets via a pixel-overlap
  heuristic before model training or benchmarking.
when_to_use_negative:
- Dataset is a small, manually curated set already verified as centroided and high
  quality — the heuristic targets bulk public data curation
- The analysis requires QTOF-based datasets that are underrepresented in METASPACE;
  the article notes these lack publicly available data for reliable evaluation
- Input is a private or locally processed dataset not represented in the METASPACE
  knowledge base, where annotation records and FDR 20% counts are unavailable
edam_operation: http://edamontology.org/operation_3695
tools:
- name: METASPACE
  role: Source of the public dataset metadata and annotation records queried for the
    9251 datasets and used to compute proportion_overlap scores at 3 ppm tolerance
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE-ML
  role: Downstream consumer of the 7713 retained datasets; the filtering step defines
    the representative contexts on which the machine-learning model is trained and
    evaluated
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:2a96bfc10ccef659674e3b12c0e87d764b7780756f12554425bbad8bc0b39cc7
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# proportion-overlap-score-computation

## Summary

Computes the proportion_overlap heuristic used to exclude non-centroided and low-quality public METASPACE imaging MS datasets prior to METASPACE-ML training and evaluation, reducing 9251 public datasets (September 2023 download) to 7713 quality-passing datasets.

## When to use

Use when curating large batches of public METASPACE imaging mass spectrometry datasets and you must exclude non-centroided or noisy datasets via a pixel-overlap heuristic before model training or benchmarking.

## When NOT to use

- Dataset is a small, manually curated set already verified as centroided and high quality — the heuristic targets bulk public data curation
- The analysis requires QTOF-based datasets that are underrepresented in METASPACE; the article notes these lack publicly available data for reliable evaluation
- Input is a private or locally processed dataset not represented in the METASPACE knowledge base, where annotation records and FDR 20% counts are unavailable

## Inputs

- Metadata and annotation records for 9251 public METASPACE datasets from the METASPACE knowledge base
- Per-dataset ion intensity images (imzML-derived) for selecting the top 50 highest-intensity pixels
- Per-dataset annotation counts at FDR 20% and median peak counts

## Outputs

- retained_datasets.csv with exactly 7713 dataset identifiers passing all quality criteria
- Per-dataset proportion_overlap scores (excluded if > 0.5)

## How to apply

Retrieve metadata and annotation records for all public METASPACE datasets from the METASPACE knowledge base (9251 datasets in the September 2023 snapshot). For each dataset, select the top 50 highest-intensity pixels and compute the proportion_overlap score using a 3 ppm m/z tolerance to assess whether raw/uncorrected data behaves like centroided data. Exclude any dataset whose proportion_overlap score exceeds 0.5, then apply two additional quality filters: drop datasets with fewer than 10 annotations at FDR 20%, and drop datasets with a median peak count greater than 50,000. Write the surviving dataset identifiers to retained_datasets.csv; the expected yield for this curation was 7713 of 9251 datasets (84% animal, 14.3% plant), which feeds downstream METASPACE-ML training contexts.

## Related tools

- **METASPACE** (Source of the public dataset metadata and annotation records queried for the 9251 datasets and used to compute proportion_overlap scores at 3 ppm tolerance) — https://github.com/metaspace2020/metaspace
- **METASPACE-ML** (Downstream consumer of the 7713 retained datasets; the filtering step defines the representative contexts on which the machine-learning model is trained and evaluated) — https://github.com/metaspace2020/metaspace

## Examples

```
df[(df.proportion_overlap <= 0.5) & (df.n_annotations_fdr20 >= 10) & (df.median_peak_count <= 50000)].dataset_id.to_csv('retained_datasets.csv', index=False)
```

## Evaluation signals

- retained_datasets.csv exists and contains exactly 7713 dataset identifiers out of the 9251 downloaded datasets
- No retained dataset has a proportion_overlap score greater than 0.5 when recomputed from the top 50 highest-intensity pixels at 3 ppm m/z tolerance
- No retained dataset has fewer than 10 annotations at FDR 20%
- No retained dataset has a median peak count greater than 50,000
- Composition of retained datasets roughly matches the reported split of 84% animal and 14.3% plant datasets

## Limitations

- The filtering is biased towards public datasets represented in METASPACE (from 47 labs), limiting diversity of training contexts
- QTOF-based datasets are underrepresented in the resulting model due to lack of publicly available data
- Less common data types, e.g. single-cell metabolomics from cultured cells or industrial lab data, remain underrepresented after this curation

## Evidence

- [methods] workflow: [evidence span withheld — anchor status: not_found]
- [full_text] finding: [evidence span withheld — anchor status: not_found]
- [abstract] tool: "METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores"
