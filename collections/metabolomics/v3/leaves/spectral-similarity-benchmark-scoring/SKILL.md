---
name: spectral-similarity-benchmark-scoring
description: Use when comparing MS2 library-matching or analogue-search methods on
  a spectral library benchmark where ground-truth structures (InChIKeys) are known
  and you need a recall-versus-Tanimoto curve as the evaluation metric.
when_to_use_negative:
- The goal is running MS2Query on new experimental spectra to annotate features, not
  benchmarking search methods against known structures
- The query spectra have no ground-truth structural annotations (InChIKeys), so Tanimoto
  scores cannot be computed
- Exact-match library search is the target — exact match benchmarking in this article
  used a 0.25 Da mass tolerance rather than the analogue-search Tanimoto/recall protocol
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2Query
  role: Analogue-search method under benchmark (version 0.3.2); preselects top 2000
    library spectra by MS2Deepscore then re-ranks with a 5-feature random forest
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Embedding-based spectral similarity predictor; run alone as a baseline and
    used inside MS2Query for candidate preselection
  repo: https://github.com/iomega/ms2query
- name: Spec2Vec
  role: Unsupervised spectral embedding model retrained per cross-validation fold
    alongside MS2Deepscore
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Library cleaning, modified cosine score computation for the baseline, and
    precursor m/z difference filtering
  repo: https://github.com/iomega/ms2query
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:991ea40e88a65e13431822761d920cd69812bd671ad9f85b2314e9616a5e974d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - Spec2Vec
  - matchms
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# spectral-similarity-benchmark-scoring

## Summary

Benchmark analogue-search methods (MS2Query, MS2Deepscore, modified cosine) by computing Tanimoto chemical similarity between true query structures and predicted top hits as a function of recall, aggregated over 20 cross-validation folds on a GNPS analogue test set.

## When to use

Use when comparing MS2 library-matching or analogue-search methods on a spectral library benchmark where ground-truth structures (InChIKeys) are known and you need a recall-versus-Tanimoto curve as the evaluation metric.

## When NOT to use

- The goal is running MS2Query on new experimental spectra to annotate features, not benchmarking search methods against known structures
- The query spectra have no ground-truth structural annotations (InChIKeys), so Tanimoto scores cannot be computed
- Exact-match library search is the target — exact match benchmarking in this article used a 0.25 Da mass tolerance rather than the analogue-search Tanimoto/recall protocol

## Inputs

- GNPS ALL_GNPS.mgf mass spectral library (downloaded 15 Nov 2021) with structural annotations (InChIKeys)
- Query spectra from the analogues test set, split into 20 cross-validation folds
- Trained MS2Deepscore and Spec2Vec models per fold (test-fold spectra excluded from training)

## Outputs

- recall_tanimoto_benchmark.csv: recall-versus-Tanimoto table per method and fold
- recall_tanimoto_benchmark.png: recall-versus-Tanimoto curves
- Average Tanimoto score at 35% recall aggregated across the 20 folds per method

## How to apply

Clean the GNPS library with matchms (remove peaks above 1000 Da, peaks below 0.1% of the highest peak, trim spectra to at most 500 peaks, drop spectra with fewer than 3 peaks), then split the analogues test set into 20 cross-validation folds, retraining MS2Deepscore and Spec2Vec models per fold so no test-fold spectra leak into training. Run each method on the same folds: MS2Query 0.3.2 (top 2000 candidates by MS2Deepscore, re-ranked by a 5-feature random forest), MS2Deepscore alone, and modified cosine after filtering the library to a maximum precursor m/z difference of 100 Da. For each method and fold, compute the Tanimoto structural similarity between each query's true structure and the predicted top hits as a function of recall, aggregate the average Tanimoto score at 35% recall across folds, and write recall_tanimoto_benchmark.csv and recall_tanimoto_benchmark.png. Correct application reproduces the reported values (MS2Query 0.63, modified cosine 0.45 average Tanimoto at 35% recall) within reasonable tolerance, with MS2Query outperforming the modified cosine baseline.

## Related tools

- **MS2Query** (Analogue-search method under benchmark (version 0.3.2); preselects top 2000 library spectra by MS2Deepscore then re-ranks with a 5-feature random forest) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Embedding-based spectral similarity predictor; run alone as a baseline and used inside MS2Query for candidate preselection) — https://github.com/iomega/ms2query
- **Spec2Vec** (Unsupervised spectral embedding model retrained per cross-validation fold alongside MS2Deepscore) — https://github.com/iomega/ms2query
- **matchms** (Library cleaning, modified cosine score computation for the baseline, and precursor m/z difference filtering) — https://github.com/iomega/ms2query

## Examples

```
ms2query --spectra ./query_folds --library ./library_folder --ionmode positive --results ./benchmark_results
```

## Evaluation signals

- Reproduced average Tanimoto score at 35% recall matches reported values: MS2Query 0.63 and modified cosine 0.45, within reasonable tolerance
- MS2Query outperforms the modified cosine baseline on the same analogues test set
- Per-fold models are retrained such that no test-fold spectra were used in training MS2Deepscore or Spec2Vec
- Library cleaning invariants hold: no peaks above 1000 Da, no peaks below 0.1% of the highest peak, at most 500 peaks per spectrum, no spectra with fewer than 3 peaks
- recall_tanimoto_benchmark.csv contains recall-versus-Tanimoto values for all 20 folds for every benchmarked method

## Limitations

- It is hard to objectively judge whether MS2Query performs similarly on newly generated data compared to the benchmarking test set
- MS2Query performs no peak picking or clustering of similar MS2 spectra, so heavily redundant query files should be preprocessed first
- Software versions for rdkit, matchms, and scikit-learn are not stated in the article, which may hinder exact reproduction

## Evidence

- [results] MS2Query achieves an average Tanimoto score of 0.63 (chemical similarity), a substantial improvement compared to the modified cosine score-based method, which on the same test set resulted in an average Tanimoto score of 0.45.: [evidence span withheld — anchor status: not_found]
- [discussion] For all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models.: "for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models"
- [discussion] Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed.: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [readme] The top 2000 spectra with the highest MS2Deepscore are selected. In contrast to other analogue search methods, no preselection on precursor m/z is performed.: [evidence span withheld — anchor status: external_doc]
- [readme] MS2Query is run on all MS2 spectra in a spectrum file. MS2Query does not do any peak picking or clustering of similar MS2 spectra.: [evidence span withheld — anchor status: external_doc]
