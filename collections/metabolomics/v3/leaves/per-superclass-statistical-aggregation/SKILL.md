---
name: per-superclass-statistical-aggregation
description: Use when you have per-compound cosine-similarity scores from spectral
  prediction evaluation and need to check whether prediction quality is uniform across
  compound classes rather than only reporting a global median.
when_to_use_negative:
- Input is already aggregated (e.g., a single global cosine similarity) and per-compound
  scores are unavailable
- Compounds lack SMILES or cannot be mapped to ClassyFire superclasses, so no class
  grouping is possible
- The goal is benchmarking against other predictors (CFM-ID, ICEBERG) rather than
  stratifying one predictor's quality by chemical class
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: FIORA
  role: Predicts MS/MS spectra (fiora-predict) and computes per-compound spectral_sqrt_cosine
    scores (fiora-eval); its per-molecule graph node embeddings are mean-pooled for
    the UMAP superclass visualization
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:f3550a0f349d4c679f9c47eaaf5fee2a5f6226146d8146aee0a76e48de6be330
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# per-superclass-statistical-aggregation

## Summary

Groups per-compound spectral prediction quality metrics (spectral_sqrt_cosine similarity) by ClassyFire superclass to test whether a predictor's performance is consistent across chemical classes, and optionally visualizes learned graph embeddings by superclass with mean-pooled UMAP projection.

## When to use

Use when you have per-compound cosine-similarity scores from spectral prediction evaluation and need to check whether prediction quality is uniform across compound classes rather than only reporting a global median.

## When NOT to use

- Input is already aggregated (e.g., a single global cosine similarity) and per-compound scores are unavailable
- Compounds lack SMILES or cannot be mapped to ClassyFire superclasses, so no class grouping is possible
- The goal is benchmarking against other predictors (CFM-ID, ICEBERG) rather than stratifying one predictor's quality by chemical class

## Inputs

- Test compound CSV with header columns: Name, SMILES, Precursor_type, CE, Instrument_type
- Per-compound evaluation CSV (validation_eval.csv, test_eval.csv) containing spectral_sqrt_cosine similarity scores from fiora-eval
- SMILES structures for ClassyFire superclass assignment
- FIORA graph node embeddings per molecule (for UMAP panel)

## Outputs

- Per-superclass median spectral_sqrt_cosine table
- Superclass-labeled per-compound score assignments
- 2-D UMAP projection of mean-pooled node embeddings colored by superclass (Fig. 4-style panel)

## How to apply

After generating predicted spectra with fiora-predict (input CSV with columns Name, SMILES, Precursor_type, CE, Instrument_type; output in mgf format), run fiora-eval with the trained checkpoint to obtain per-compound spectral_sqrt_cosine scores (validation_eval.csv, test_eval.csv). Assign a ClassyFire superclass to each compound from its SMILES structure, then group scores by superclass and compute the median per group. Compare group medians against reference values from the paper: organohalogens 0.63, hydrocarbon derivatives 0.66, nucleosides 0.86, with all other superclasses well above 0.7. Rationale: a global median can hide class-specific weaknesses; per-superclass aggregation plus mean-pooled per-molecule graph node embeddings projected to 2-D with UMAP (colored by superclass, as in Fig. 4 panels a-c) reveals whether failures concentrate in chemically coherent clusters.

## Related tools

- **FIORA** (Predicts MS/MS spectra (fiora-predict) and computes per-compound spectral_sqrt_cosine scores (fiora-eval); its per-molecule graph node embeddings are mean-pooled for the UMAP superclass visualization) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-predict -i test_compounds.csv -o predicted_spectra.mgf && fiora-eval -m checkpoints/fiora.pt -i test_compounds.csv --output-dir results/
```

## Evaluation signals

- Per-superclass median table reproduces reported values: organohalogens 0.63, hydrocarbon derivatives 0.66, nucleosides 0.86
- All superclasses except Organohalogen compounds and Hydrocarbon derivatives show median cosine similarity well above 0.7
- Validation set (validation_eval.csv) medians agree with test set (test_eval.csv) stratification patterns
- UMAP plot shows superclass-colored embedding clusters rather than randomly intermixed points
- fiora-predict output mgf matches the expected reference output shipped in the repo (verifiable via pytest)

## Limitations

- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, so rare superclasses (e.g., organohalogens) have limited training coverage and lower prediction quality
- Median per superclass on the ~20% test split means small classes may contain few compounds, making medians sensitive to outliers
- Exact model specification is only in the Supplementary Information

## Evidence

- [methods] Assign ClassyFire superclass from SMILES and group per-compound spectral_sqrt_cosine scores by superclass, computing the median per superclass: [evidence span withheld — anchor status: not_found]
- [results] Median cosine similarity exceeds 0.7 for all superclasses except Organohalogen compounds (0.63) and Hydrocarbon derivatives (0.66): [evidence span withheld — anchor status: not_found]
- [other] The rendered UMAP must show superclass-colored embedding clusters as validation of the figure reproduction: [evidence span withheld — anchor status: not_found]
- [readme] fiora-predict consumes a CSV and emits mgf/msp, the prerequisite artifact for the per-compound evaluation scores: "An input csv file must be provided and an output file specified (`mgf` or `msp` format)."
