---
name: random-forest-spectrum-pair-reranking
description: Use when building or reproducing an MS2Query-style analogue search pipeline
  where MS2Deepscore preselection must be re-ranked to place the analogue or exact
  match at the top.
when_to_use_negative:
- When only an exact spectral library match is needed — plain cosine or modified cosine
  score matching (matchms, 0.05 Da fragment tolerance) is sufficient and the re-ranker
  adds unneeded complexity
- When the query set has not been reduced to representative MS2 spectra — MS2Query
  performs no peak picking or clustering, so running the re-ranker on many near-duplicate
  MS2 spectra per feature wastes computation; cluster or preprocess with MZMine/FBMN
  first
- When evaluating raw spectral similarity rather than analogue-search ranking — MS2Deepscore
  or Spec2Vec scores alone are the right outputs, not the re-ranked 0–1 match score
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MS2Query
  role: 'Runs the full pipeline: computes MS2Deepscore similarities, selects top 2000
    candidates, applies the 5-feature random forest re-ranker, and applies the score
    threshold'
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Supervised siamese similarity model whose scores define the top-2000 candidate
    set fed to the re-ranker
  repo: https://github.com/iomega/ms2query
- name: scikit-learn
  role: Implementation of the random forest re-ranker (250 estimators, max depth 5,
    MSE loss)
- name: matchms
  role: Used to clean library spectra and compute baseline similarity scores (cosine,
    modified cosine) that the re-ranking approach is benchmarked against
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:2517a4808cd66d833e2dcf6f2de759e730fc53d610e92a0dad89f079b9b0aec9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - matchms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# random-forest-spectrum-pair-reranking

## Summary

Train and apply a random forest re-ranker that combines 5 spectrum-pair features to rank the top 2000 MS2Deepscore-preselected library spectra, outputting a calibrated 0–1 match score for analogue and exact-match search in MS2Query.

## When to use

Use when building or reproducing an MS2Query-style analogue search pipeline where MS2Deepscore preselection must be re-ranked to place the best analogue or exact match at the top.

## When NOT to use

- When only an exact spectral library match is needed — plain cosine or modified cosine score matching (matchms, 0.05 Da fragment tolerance) is sufficient and the re-ranker adds unneeded complexity
- When the query set has not been reduced to representative MS2 spectra — MS2Query performs no peak picking or clustering, so running the re-ranker on many near-duplicate MS2 spectra per feature wastes computation; cluster or preprocess with MZMine/FBMN first
- When evaluating raw spectral similarity rather than analogue-search ranking — MS2Deepscore or Spec2Vec scores alone are the right outputs, not the re-ranked 0–1 match score

## Inputs

- Top 2000 library spectra preselected by MS2Deepscore for each query MS2 spectrum
- 5 spectrum-pair features per query–library candidate (precursor m/z difference, MS2Deepscore, Spec2Vec, and related similarity features)
- Trained random forest model file (scikit-learn) plus library spectra in sqlite and pre-calculated MS2Deepscore/Spec2Vec embeddings
- For training: cleaned GNPS library spectra (MGF, e.g. ALL_GNPS.mgf) with InChIKey-annotated structures and a structure-disjoint train/test split manifest

## Outputs

- Random forest score between 0 and 1 for each of the 2000 candidate library spectra per query
- Single best library match (analogue or exact) per query spectrum with its score
- Filtered results CSV where candidates below the score threshold are removed as unreliable matches
- Trained random forest model file stored alongside MS2Deepscore/Spec2Vec models and the split manifest

## How to apply

First run MS2Deepscore between the query spectrum and all library spectra and select the top 2000 spectra by MS2Deepscore score — unlike modified cosine analogue search, no precursor m/z preselection (e.g. the 100 Da window) is applied. Compute the 5 spectrum-pair features (including precursor m/z difference and spectral similarity scores) for each of the 2000 query–library pairs and feed them to the random forest, implemented with scikit-learn, which was trained with 250 estimators, max depth 5, and MSE loss. The forest predicts a score between 0 and 1 per library–query pair; the highest-scoring library spectrum is selected as the match, and a minimum-score threshold filters out unreliable matches. When training a new re-ranker, use structure-disjoint splits (InChIKey-based) so no test spectrum's structure appears in training, and train on pairs formed from the top-scoring candidates per query; save the model file together with the MS2Deepscore and Spec2Vec models and the data-split manifest.

## Related tools

- **MS2Query** (Runs the full pipeline: computes MS2Deepscore similarities, selects top 2000 candidates, applies the 5-feature random forest re-ranker, and applies the score threshold) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Supervised siamese similarity model whose scores define the top-2000 candidate set fed to the re-ranker) — https://github.com/iomega/ms2query
- **scikit-learn** (Implementation of the random forest re-ranker (250 estimators, max depth 5, MSE loss))
- **matchms** (Used to clean library spectra and compute baseline similarity scores (cosine, modified cosine) that the re-ranking approach is benchmarked against)

## Examples

```
pip install ms2query; ms2query --spectra ./query_mgf/ --library ./library_folder --ionmode positive --filter_ionmode
```

## Evaluation signals

- The random forest outputs scores bounded in [0, 1] for every candidate pair, and the single top-scoring library spectrum is selected per query
- Exactly 2000 candidates are re-ranked per query (the MS2Deepscore preselection size), with no precursor m/z filter applied before re-ranking
- On the GNPS benchmark, re-ranked analogue search should outperform cosine score, modified cosine score, and MS2Deepscore alone at finding exact matches and analogues
- Training data must be structure-disjoint: no InChIKey structure from the test set appears in training spectra for MS2Deepscore, Spec2Vec, or the random forest
- Model artifacts are complete: random forest file saved together with MS2Deepscore and Spec2Vec models, library sqlite, pre-calculated embeddings, and the split manifest

## Limitations

- Reliability on newly generated data is hard to judge objectively, since it is difficult to verify whether MS2Query performs as well on new data as on the benchmarking test set
- The score threshold trades recall for reliability: setting it too high filters out correct analogues; the article notes a minimum threshold is used to filter out unreliable matches
- Details of other tested features and the motivation for training on the top 100 highest-scoring spectra are only in Supplementary Notes 3 and 5, which are not included here
- Benchmarked library versions are time-stamped downloads (GNPS ALL_GNPS.mgf from 15 Nov 2021; ALL_GNPS_NO_PROPOGATED.mgf from 1 Nov 2022); results on other library snapshots may differ

## Evidence

- [intro] First select candidate library spectra with MS2Deepscore, then re-rank with a random forest: "First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model."
- [discussion] MS2Query calculates 5 features combined by random forest: "To optimally rank these 2000 spectra, MS2Query calculates 5 features which are combined by a random forest model."
- [readme] Random forest predicts 0-1 score; threshold filters unreliable matches: [evidence span withheld — anchor status: external_doc]
- [discussion] Training used structure-disjoint splits to avoid data leakage: "for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models"
- [readme] No precursor m/z preselection before re-ranking: [evidence span withheld — anchor status: external_doc]
