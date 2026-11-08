---
name: analogues-test-set-cross-validation-design
description: Use when benchmarking MS2 library-matching or analogue-search methods
  on an annotated spectral library and you need a leakage-free test set with structure-based
  (Tanimoto vs recall) evaluation rather than only spectral scores.
when_to_use_negative:
- The query spectra are experimental/unannotated data without known structures — Tanimoto-vs-recall
  evaluation cannot be computed
- You only need to evaluate exact-match retrieval within a 0.25 Da mass tolerance
  rather than analogue search quality
- Input spectra are already split into predefined train/test sets; reusing this fold
  design would ignore the existing split
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MS2Query
  role: Method under benchmark; run per fold with MS2Deepscore preselection of top
    2000 library spectra and 5-feature random-forest re-ranking
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Used to clean library metadata/intensities and to compute the modified cosine
    baseline after filtering to a maximum precursor m/z difference of 100 Da
- name: MS2Deepscore
  role: Spectral similarity model retrained per CV fold for candidate preselection
    and as a standalone baseline
- name: Spec2Vec
  role: Unsupervised embedding model retrained per CV fold to avoid test-spectrum
    leakage
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:b39d18f07b02b153ca949794b5a761c5c42688515d4c2ad5f6b516d2dcc7fd20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - Spec2Vec
  - matchms
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# analogues-test-set-cross-validation-design

## Summary

Design a leakage-free cross-validation benchmark for MS2 analogue search by splitting an annotated GNPS library into 20 folds, retraining embedding models (MS2Deepscore, Spec2Vec) per fold so no test spectra are seen in training, and evaluating methods by average Tanimoto chemical similarity versus recall.

## When to use

Use when benchmarking MS2 library-matching or analogue-search methods on an annotated spectral library and you need a leakage-free test set with structure-based (Tanimoto vs recall) evaluation rather than only spectral scores.

## When NOT to use

- The query spectra are experimental/unannotated data without known structures — Tanimoto-vs-recall evaluation cannot be computed
- You only need to evaluate exact-match retrieval within a 0.25 Da mass tolerance rather than analogue search quality
- Input spectra are already split into predefined train/test sets; reusing this fold design would ignore the existing split

## Inputs

- Annotated GNPS mass spectral library (MGF, e.g. ALL_GNPS.mgf) with structures/InChIKeys
- Candidate search methods (e.g. MS2Query 0.3.2, MS2Deepscore, modified cosine via matchms)
- Query spectra per fold derived from the library

## Outputs

- recall_tanimoto_benchmark.csv (recall-versus-Tanimoto table per method)
- recall_tanimoto_benchmark.png (recall-versus-Tanimoto curves)
- Aggregated average Tanimoto score at 35% recall across the 20 folds per method

## How to apply

Load and clean the GNPS ALL_GNPS.mgf library with matchms: clean metadata to uniform format, remove peaks above 1000 Da, drop peaks below 0.1% of the highest peak, trim spectra with more than 500 peaks by removing lowest-intensity peaks, and discard spectra with fewer than 3 peaks. Construct the analogues test set from the cleaned library and split it into 20 cross-validation folds; critically, retrain the MS2Deepscore and Spec2Vec models for each fold so none of the test-fold spectra were used in training (this prevents optimistic bias in the embeddings). Run each candidate method (e.g. MS2Query with top-2000 MS2Deepscore preselection and random-forest re-ranking; modified cosine with matchms filtered to a maximum precursor m/z difference of 100 Da) on the same folds. For each method and fold, compute Tanimoto structural similarity between the true query structure and predicted top hits as a function of recall, then aggregate the average Tanimoto at a fixed recall point (e.g. 35%) across folds.

## Related tools

- **MS2Query** (Method under benchmark; run per fold with MS2Deepscore preselection of top 2000 library spectra and 5-feature random-forest re-ranking) — https://github.com/iomega/ms2query
- **matchms** (Used to clean library metadata/intensities and to compute the modified cosine baseline after filtering to a maximum precursor m/z difference of 100 Da)
- **MS2Deepscore** (Spectral similarity model retrained per CV fold for candidate preselection and as a standalone baseline)
- **Spec2Vec** (Unsupervised embedding model retrained per CV fold to avoid test-spectrum leakage)

## Examples

```
ms2query --spectra ./query_fold_01 --library ./library_folder --ionmode positive
```

## Evaluation signals

- Average Tanimoto score at 35% recall for MS2Query should reproduce the reported 0.63 (and 0.45 for modified cosine) within reasonable tolerance across the 20 folds
- MS2Query curves should dominate the modified cosine baseline (higher Tanimoto at equal recall) on the same folds
- Leakage check: verify that for every fold, neither MS2Deepscore nor Spec2Vec models were trained on that fold's test spectra
- Cleaning invariants: no library spectrum with fewer than 3 peaks; spectra with >500 peaks trimmed by removing lowest-intensity peaks; peaks above 1000 Da removed
- Per-fold results should be aggregated over all 20 folds, not a single split, before comparing methods

## Limitations

- MS2Query 0.3.2 benchmark results may not generalize to newly generated data; the authors note it is hard to objectively judge if performance transfers beyond the benchmarking test set
- Results depend on the GNPS library snapshot and date (ALL_GNPS.mgf downloaded 15 November 2021); different snapshots will change fold composition
- Software versions for rdkit, matchms, and scikit-learn are not stated in the article, complicating exact reproduction

## Evidence

- [discussion] for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models: "for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models"
- [discussion] Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed.: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [discussion] Spectra with less than 3 peaks were completely removed from the library.: "Spectra with less than 3 peaks were completely removed from the library."
- [discussion] library spectra were first filtered on a maximum precursor m/z difference of 100 Da: "library spectra were first filtered on a maximum precursor m/z difference of 100 Da"
- [discussion] The GNPS dataset used was downloaded from GNPS (https://gnps-external.ucsd.edu/gnpslibrary/ALL_GNPS.mgf) on the 15th of November 2021: [evidence span withheld — anchor status: not_found]
- [intro] First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model.: "First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model."
