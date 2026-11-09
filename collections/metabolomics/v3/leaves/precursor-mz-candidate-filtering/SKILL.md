---
name: precursor-mz-candidate-filtering
description: 'Use when matching unknown query MS/MS spectra against a large spectral
  library (e.g., ~76,000–95,000 AllPositive/GNPS spectra) and full pairwise scoring
  would be too slow: filter candidates by precursor m/z before computing similarity
  scores.'
when_to_use_negative:
- When searching for structural analogues whose precursor masses differ from the query
  (analogues with different m/z are excluded by a tight precursor-m/z window)
- When query or library spectra lack reliable precursor m/z metadata (the paper relies
  on matchms metadata cleaning; unannotated or mislabeled precursor m/z will wrongly
  filter the true match)
- When the library is small enough to score exhaustively — then the 1 ppm pre-selection
  gate adds little value and may discard valid matches with slightly inaccurate precursor
  masses
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Imports, cleans, and validates spectra and their precursor-m/z metadata; supports
    fast similarity measures for initial pre-selection before heavier scoring
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Provides the spectrum-vector embedding and Spec2Vec cosine score applied to
    the pre-selected candidate subset
  repo: https://github.com/iomega/spec2vec
- name: gensim
  role: Trains the underlying Word2Vec model on documents of peak and loss words used
    to embed query and candidate spectra
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:10f2d7911c1e60ff90a90f2f4b54ad952502c87a08ce16179a0ef05e922d5ade
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - gensim
  - matchms
  - spec2vec
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# precursor-mz-candidate-filtering

## Summary

A candidate pre-selection step for library matching that restricts full spectral-similarity scoring (e.g., Spec2Vec cosine) to library spectra whose precursor m/z falls within a tight tolerance (1 ppm in the Spec2Vec paper) of the query, making structural-analogue searches over ~76,062 library spectra computationally tractable (~0.14 s per query).

## When to use

Use when matching unknown query MS/MS spectra against a large spectral library (e.g., ~76,000–95,000 AllPositive/GNPS spectra) and full pairwise scoring would be too slow: filter candidates by precursor m/z before computing similarity scores.

## When NOT to use

- When searching for structural analogues whose precursor masses differ from the query (analogues with different m/z are excluded by a tight precursor-m/z window)
- When query or library spectra lack reliable precursor m/z metadata (the paper relies on matchms metadata cleaning; unannotated or mislabeled precursor m/z will wrongly filter the true match)
- When the library is small enough to score exhaustively — then the 1 ppm pre-selection gate adds little value and may discard valid matches with slightly inaccurate precursor masses

## Inputs

- Processed query MS/MS spectra with precursor m/z metadata (e.g., the 1030 held-out spectra from 200 random InChIKeys, AllPositive/GNPS data)
- Processed library spectra with precursor m/z metadata (e.g., 76,062 training spectra from GNPS ALL_GNPS.json cleaned with matchms)
- Trained Spec2Vec/Word2Vec model (gensim) and spec2vec spectrum-vector vectors for library candidates

## Outputs

- Per-query candidate shortlist (library spectra within 1 ppm precursor-m/z tolerance)
- Top-10 matches per query with Spec2Vec cosine similarity scores and Tanimoto structural similarities
- Per-query runtime records (~0.14 s per query, ~140 s total reported in the paper)

## How to apply

For each query spectrum, first apply the matchms-based cleaning (keep peaks with m/z in [0, 1000], discard spectra with <10 peaks), then pre-select candidate library matches whose precursor m/z matches the query within 1 ppm tolerance — only these candidates are scored with the Spec2Vec spectrum-vector cosine, and the highest-scoring candidate above a similarity threshold is selected. This pre-selection is what makes the pipeline scale: the Spec2Vec score itself is computed as the cosine between intensity-weighted sums of Word2Vec (gensim) word vectors for peak and neutral-loss words (losses between 5.0 and 200.0 Da), which is only run on the small candidate subset rather than the full 76,062-spectrum library. The same pre-selection applies to the 1030-spectrum held-out query set (200 randomly chosen InChIKeys), where top-10 matches per query are then retrieved with their Tanimoto structural similarities. Record per-query runtimes as an explicit output to confirm the scalability claim.

## Related tools

- **matchms** (Imports, cleans, and validates spectra and their precursor-m/z metadata; supports fast similarity measures for initial pre-selection before heavier scoring) — https://github.com/matchms/matchms
- **spec2vec** (Provides the spectrum-vector embedding and Spec2Vec cosine score applied to the pre-selected candidate subset) — https://github.com/iomega/spec2vec
- **gensim** (Trains the underlying Word2Vec model on documents of peak and loss words used to embed query and candidate spectra)

## Evaluation signals

- Every returned candidate's precursor m/z differs from the query precursor by at most 1 ppm; candidates outside this window must be absent
- Per-query runtime should be ~0.14 s on average (~140 s total over 1030 queries), consistent with scoring only a pre-selected subset rather than all 76,062 library spectra
- For ~60% of the 1030 queries, the top-10 matches contain molecules with Tanimoto structural similarity > 0.6 (vs. random-draw p = 0.0103)
- No candidate should come from the held-out 200 InChIKeys — training/query molecule overlap must be zero after removing spectra without InChIKey annotations
- Accuracy and true/false-positive curves vs. similarity threshold should qualitatively reproduce Fig 4 (up to 88% correctly matched queries)

## Limitations

- Spec2Vec (the downstream scorer over candidates) requires training data and a training set covering a large fraction of the fragment/loss features seen in queries; missing words become problematic for smaller training sets or more m/z decimals
- Performance was only demonstrated on LC-MS positive-ionization data; GC-MS utility was not assessed
- A missing-fraction threshold (e.g., <0.05) is needed to avoid returning Spec2Vec scores for spectra far outside the learned peaks/losses; some remaining peaks/losses (~3%) may not be chemically meaningful

## Evidence

- [intro] 1 ppm tolerance pre-selection in library matching: "potentially matching spectra were pre-selected based on precursor-m/z matches (tolerance = 1ppm) before the highest scoring candidate above a similarity threshold was chosen."
- [abstract] Scalability rationale for pre-selection: "Spec2Vec is computationally more scalable allowing structural analogue searches in large databases within seconds."
- [discussion] Spec2Vec scoring applied to candidates: "we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi"
- [readme] matchms fast pre-selection support: [evidence span withheld — anchor status: external_doc]
