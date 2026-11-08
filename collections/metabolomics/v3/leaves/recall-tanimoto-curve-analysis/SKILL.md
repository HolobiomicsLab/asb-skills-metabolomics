---
name: recall-tanimoto-curve-analysis
description: Use when comparing MS2 spectral library-search or analogue-search methods
  and you need a recall-versus-structural-similarity benchmark on a GNPS-derived analogues
  test set.
when_to_use_negative:
- The query spectra have already been preprocessed/annotated and you only need exact-match
  benchmarking under a 0.25 Da precursor mass tolerance rather than analogue-search
  benchmarking.
- You lack structural annotations (InChIKeys) for the query spectra — Tanimoto similarity
  cannot be computed without known true structures.
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2Query
  role: 'Method under benchmarking: MS2Deepscore-based top-2000 preselection plus
    random-forest re-ranking to predict analogues and exact matches; version 0.3.2
    used for the benchmark'
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Supervised spectral-embedding similarity model used to score and preselect
    candidate library spectra; retrained per CV fold
  repo: https://github.com/iomega/ms2query
- name: Spec2Vec
  role: Unsupervised spectral-embedding model contributing features for re-ranking;
    retrained per CV fold
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Library cleaning, modified cosine score computation, and InChIKey supplementation
    via PubChem lookup
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:7ff30ad1717fdf5f7303a8f9ded84ec7658757a362a9650c8f8a9a7e2cccad3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - Spec2Vec
  - matchms
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# recall-tanimoto-curve-analysis

## Summary

Benchmarks analogue-search methods (MS2Query vs modified cosine score) by computing Tanimoto structural similarity of predicted hits as a function of recall under 20-fold cross-validation, aggregating the average Tanimoto score at 35% recall.

## When to use

Use when comparing MS2 spectral library-search or analogue-search methods and you need a recall-versus-structural-similarity benchmark on a GNPS-derived analogues test set.

## When NOT to use

- The query spectra have already been preprocessed/annotated and you only need exact-match benchmarking under a 0.25 Da precursor mass tolerance rather than analogue-search benchmarking.
- You lack structural annotations (InChIKeys) for the query spectra — Tanimoto similarity cannot be computed without known true structures.

## Inputs

- GNPS ALL_GNPS.mgf mass spectral library (downloaded 15 November 2021; ALL_GNPS_NO_PROPOGATED.mgf downloaded 1 November 2022 is an alternative)
- Structural annotations (InChIKeys) for library and query spectra, supplemented via matchms extras PubChem lookup by compound name and molecular formula
- MS2 query spectra (MGF) grouped into 20 cross-validation folds
- Trained MS2Deepscore and Spec2Vec models per fold, plus the MS2Query random-forest re-ranking model

## Outputs

- recall_tanimoto_benchmark.csv — recall-versus-Tanimoto similarity table per method across 20 folds
- recall_tanimoto_benchmark.png — recall-versus-Tanimoto curve plot
- Average Tanimoto score at 35% recall per method (expected: MS2Query 0.63, modified cosine 0.45)

## How to apply

Load and clean the GNPS ALL_GNPS.mgf library with matchms (remove peaks above 1000 Da, peaks below 0.1% of the highest peak, trim spectra with >500 peaks to the 500 most intense, and discard spectra with fewer than 3 peaks). Split the library into 20 cross-validation folds, retraining MS2Deepscore and Spec2Vec models per fold so no test-fold spectra leak into training. For each fold, run MS2Query 0.3.2 (MS2Deepscore preselection of the top 2000 library spectra, then random-forest re-ranking on 5 features) and the modified cosine baseline filtered to a maximum precursor m/z difference of 100 Da. For each method and fold, compute the Tanimoto similarity between the true query structure (InChIKey-derived) and the predicted top hits as a function of recall, then average the Tanimoto score at 35% recall across the 20 folds. Validation: the reproduced average Tanimoto at 35% recall should match the reported values (MS2Query 0.63, modified cosine 0.45) within reasonable tolerance, with MS2Query clearly outperforming the baseline.

## Related tools

- **MS2Query** (Method under benchmarking: MS2Deepscore-based top-2000 preselection plus random-forest re-ranking to predict analogues and exact matches; version 0.3.2 used for the benchmark) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Supervised spectral-embedding similarity model used to score and preselect candidate library spectra; retrained per CV fold) — https://github.com/iomega/ms2query
- **Spec2Vec** (Unsupervised spectral-embedding model contributing features for re-ranking; retrained per CV fold) — https://github.com/iomega/ms2query
- **matchms** (Library cleaning, modified cosine score computation, and InChIKey supplementation via PubChem lookup)

## Examples

```
ms2query --spectra ./test_fold_spectra --library ./library_folder --ionmode positive
```

## Evaluation signals

- Reproduced average Tanimoto score at 35% recall for MS2Query is ~0.63 and for the modified cosine baseline ~0.45 within reasonable tolerance.
- The recall-versus-Tanimoto curve shows MS2Query outperforming the modified cosine score, MS2Deepscore alone, and cosine score across recall values.
- Per-fold model retraining verified: no test-fold spectra were used in training MS2Deepscore or Spec2Vec for any of the 20 folds.
- Library cleaning invariants hold: no spectra with fewer than 3 peaks remain, no peaks above 1000 Da, and spectra with >500 peaks were trimmed to their 500 most intense peaks.
- Output files recall_tanimoto_benchmark.csv and recall_tanimoto_benchmark.png exist with 20-fold aggregated values for both compared methods.

## Limitations

- It is hard to objectively judge whether MS2Query performs similarly on newly generated data compared to the benchmarking test set.
- Results depend on the GNPS library snapshot and version (MS2Query 0.3.2); software versions for rdkit, matchms, and scikit-learn are not stated in the article.
- Supplementary Notes (e.g. other tested features, GNPS analogue search comparison) are referenced but not included, limiting full reproduction detail.

## Evidence

- [results] MS2Query achieves an average Tanimoto score of 0.63 (chemical similarity), a substantial improvement compared to the modified cosine score-based method, which on the same test set resulted in an average Tanimoto score of 0.45.: [evidence span withheld — anchor status: not_found]
- [intro] First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model.: "First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model."
- [intro] Preselect library spectra by maximum precursor m/z difference (e.g. 100 Da for modified cosine analogue search): "a preselection on a maximum precursor m/z difference of 100 Da took 9 hours and 24 minutes"
- [discussion] Remove peaks above 1000 Da and peaks with an intensity of less than 0,1 % of the highest peak were removed.: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [discussion] For all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models.: "for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models"
- [readme] The workflow for running MS2Query first uses MS2Deepscore to calculate spectral similarity scores between all library spectra and a query spectrum.: "The workflow for running MS2Query first uses MS2Deepscore to calculate spectral similarity scores between all library spectra and a query spectrum."
