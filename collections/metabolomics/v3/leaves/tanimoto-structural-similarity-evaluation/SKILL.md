---
name: tanimoto-structural-similarity-evaluation
description: Use when comparing MS2 spectral library search or analogue-search methods
  on spectra whose true structures are known, and you need a chemistry-aware accuracy
  metric (Tanimoto vs recall) rather than raw spectral scores.
when_to_use_negative:
- Input spectra lack known structure annotations (no InChIKeys), so Tanimoto ground
  truth cannot be computed — use annotation-rate metrics instead.
- You need raw spectral similarity scores (cosine/modified cosine) rather than chemistry-based
  correctness — this metric measures structural match quality, not score calibration.
- Benchmarking exact matches only under a tight mass tolerance (e.g., 0.25 Da) where
  identity, not analogue similarity, is the target.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: MS2Query
  role: Candidate analogue-search method being benchmarked; returns ranked library
    matches whose structures are compared to query structures by Tanimoto score
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Computes the modified cosine baseline scores, cleans spectra/InChIKeys, and
    supports PubChem lookup to fill missing structures before Tanimoto calculation
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Embedding-based spectral similarity predictor used to preselect the top 2000
    candidates before Tanimoto-based evaluation of final ranking
- name: Spec2Vec
  role: Unsupervised embedding model retrained per cross-validation fold to prevent
    test-spectrum leakage during Tanimoto benchmarking
- name: GNPS
  role: Source of the annotated spectral library providing query structures and ground-truth
    InChIKeys
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:306a8fab8b5a314ed747544e6695774f54702da63240fadfb7ac6516a1a0b4bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - GNPS
  - MS2Deepscore
  - MS2Query
  - Spec2Vec
  - matchms
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tanimoto-structural-similarity-evaluation

## Summary

Benchmark an MS2 analogue-search method by computing Tanimoto chemical-structure similarity between each query's true structure and the predicted top library hits, reported as average Tanimoto score as a function of recall (MS2Query paper uses average Tanimoto at 35% recall across 20 cross-validation folds).

## When to use

Use when comparing MS2 spectral library search or analogue-search methods on spectra whose true structures are known, and you need a chemistry-aware accuracy metric (Tanimoto vs recall) rather than raw spectral scores.

## When NOT to use

- Input spectra lack known structure annotations (no InChIKeys), so Tanimoto ground truth cannot be computed — use annotation-rate metrics instead.
- You need raw spectral similarity scores (cosine/modified cosine) rather than chemistry-based correctness — this metric measures structural match quality, not score calibration.
- Benchmarking exact matches only under a tight mass tolerance (e.g., 0.25 Da) where identity, not analogue similarity, is the target.

## Inputs

- Query MS2 spectra with known InChIKey structures (e.g., from the GNPS ALL_GNPS.mgf library)
- Predicted top library hits per query from each candidate method (MS2Query results or modified cosine matches)
- Cross-validation fold assignments (20 folds) ensuring test spectra excluded from model training

## Outputs

- recall_tanimoto_benchmark.csv — recall-versus-Tanimoto table per method
- recall_tanimoto_benchmark.png — Tanimoto vs recall curves per method
- Average Tanimoto score at 35% recall aggregated across 20 folds per method

## How to apply

For each query spectrum with a known structure, run the candidate search method (e.g., MS2Query, which preselects the top 2000 library spectra by MS2Deepscore and re-ranks them with a 5-feature random forest; or the modified cosine baseline filtered to a maximum precursor m/z difference of 100 Da using matchms). Compute the Tanimoto structural similarity between the true structure of the query (InChIKey-based, e.g., after adding missing InChIKeys via PubChem lookup in matchms) and each predicted top hit. Plot Tanimoto similarity as a function of recall for each method. Aggregate the average Tanimoto score at a fixed recall cutoff (35% in this benchmark) across the 20 cross-validation folds, retraining MS2Deepscore and Spec2Vec models per fold so no test-fold spectra leak into training. Reproduction targets: MS2Query average Tanimoto ~0.63 vs modified cosine ~0.45 at 35% recall.

## Related tools

- **MS2Query** (Candidate analogue-search method being benchmarked; returns ranked library matches whose structures are compared to query structures by Tanimoto score) — https://github.com/iomega/ms2query
- **matchms** (Computes the modified cosine baseline scores, cleans spectra/InChIKeys, and supports PubChem lookup to fill missing structures before Tanimoto calculation) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Embedding-based spectral similarity predictor used to preselect the top 2000 candidates before Tanimoto-based evaluation of final ranking)
- **Spec2Vec** (Unsupervised embedding model retrained per cross-validation fold to prevent test-spectrum leakage during Tanimoto benchmarking)
- **GNPS** (Source of the annotated spectral library providing query structures and ground-truth InChIKeys)

## Examples

```
ms2query --spectra ./query_folds --library ./gnps_library_folder --ionmode positive  # then compute Tanimoto vs recall per fold
```

## Evaluation signals

- Average Tanimoto score at 35% recall for MS2Query reproduces ~0.63 and modified cosine reproduces ~0.45 within reasonable tolerance
- Tanimoto-vs-recall curve shows MS2Query dominating the modified cosine baseline across recall values
- Aggregation across all 20 cross-validation folds completes without fold-wise model-training leakage (test spectra excluded from MS2Deepscore/Spec2Vec training)
- recall_tanimoto_benchmark.csv contains per-recall Tanimoto values for every method and recall_tanimoto_benchmark.png renders the corresponding curves

## Limitations

- MS2Deepscore and Spec2Vec models must be retrained per fold to avoid test-spectrum leakage, which is compute-intensive.
- Tanimoto evaluation depends on complete InChIKey annotations; missing structures must be filled via PubChem name/formula lookup, which can fail.
- It is hard to objectively judge whether MS2Query performs similarly on newly generated data compared to the benchmarking test set.

## Evidence

- [results] MS2Query achieves an average Tanimoto score of 0.63 (chemical similarity), a substantial improvement compared to the modified cosine score-based method: [evidence span withheld — anchor status: not_found]
- [methods] For each method and fold, compute the Tanimoto structural similarity between the true structure of each query and the predicted top hits as a function of recall.: [evidence span withheld — anchor status: not_found]
- [intro] First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model.: "First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model."
- [discussion] A method from matchms extras was used to add missing InChIKeys by searching the compound name and molecular formula on PubChem.: "A method from matchms extras was used to add missing InChIKeys by searching the compound name and molecular formula on PubChem."
- [discussion] for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models: "for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models"
- [readme] MS2Query is run on all MS2 spectra in a spectrum file.: [evidence span withheld — anchor status: external_doc]
