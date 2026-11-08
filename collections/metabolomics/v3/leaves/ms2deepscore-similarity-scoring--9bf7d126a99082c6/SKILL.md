---
name: ms2deepscore-similarity-scoring
description: Use when ranking GNPS-style MS2 library spectra against query spectra
  without precursor m/z preselection, e.g. selecting top-2000 candidates for random-forest
  re-ranking or comparing spectral similarity methods for exact-match benchmarking.
when_to_use_negative:
- When a strict exact-match identity search with precursor m/z tolerance filtering
  is required — modified cosine with 0.25 Da preselection is the appropriate baseline
  in this paper's design
- When spectra have not been cleaned/normalized with matchms (misplaced metadata,
  unnormalized intensities, spectra with fewer than three peaks are out of scope)
- When a small library makes full spectral-similarity scoring unnecessary — the embedding
  precalculation overhead targets large libraries like GNPS
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2Deepscore
  role: Supervised embedding model computing spectral similarity scores between query
    and all library spectra
  repo: https://github.com/iomega/ms2query
- name: MS2Query
  role: Wrapper pipeline that runs MS2Deepscore, selects the top 2000 spectra, and
    re-ranks with a random forest over 5 features
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Library cleaning, normalization, and baseline cosine / modified cosine score
    computation (fragment mass tolerance 0.05 Da)
- name: Spec2Vec
  role: Unsupervised spectral embedding combined with MS2Deepscore inside MS2Query's
    candidate scoring
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:518fc538d2668780d5790b7cc3e100d68a63198ff95954aa67e62ece2ac19ff9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - Spec2Vec
  - matchms
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# ms2deepscore-similarity-scoring

## Summary

Computes supervised embedding-based spectral similarity scores (MS2Deepscore) between MS2 query spectra and library spectra, serving as a chemical-similarity proxy for analogue search and candidate preselection in MS2Query. It benchmarks favorably against cosine and modified cosine scores for exact-match and analogue retrieval.

## When to use

Use when ranking GNPS-style MS2 library spectra against query spectra without precursor m/z preselection, e.g. selecting top-2000 candidates for random-forest re-ranking or comparing spectral similarity methods for exact-match benchmarking.

## When NOT to use

- When a strict exact-match identity search with precursor m/z tolerance filtering is required — modified cosine with 0.25 Da preselection is the appropriate baseline in this paper's design
- When spectra have not been cleaned/normalized with matchms (misplaced metadata, unnormalized intensities, spectra with fewer than three peaks are out of scope)
- When a small library makes full spectral-similarity scoring unnecessary — the embedding precalculation overhead targets large libraries like GNPS

## Inputs

- Query MS2 spectra (mzML, mgf, msp, mzxml, json, usi, or pickled matchms object)
- GNPS library spectra (e.g. ALL_GNPS.mgf / ALL_GNPS_NO_PROPOGATED.mgf) cleaned with matchms
- Pre-trained MS2Deepscore model and pre-calculated library embeddings from Zenodo (e.g. 10.5281/zenodo.6811540; positive-mode models at zenodo record 6124552, negative-mode at 7104184)

## Outputs

- Ranked list of library spectra per query by MS2Deepscore similarity (top 2000 for MS2Query)
- Per-pair MS2Deepscore similarity scores used as features for random-forest re-ranking
- Recall-versus-true-positive-percentage benchmark tables (e.g. benchmark_exact_matches.csv)

## How to apply

For each query spectrum, MS2Query uses MS2Deepscore to calculate spectral similarity scores against all library spectra, made fast by pre-calculating MS2Deepscore embeddings for the library. Library spectra are ranked by score and the top 2000 are retained (no precursor m/z preselection, unlike modified cosine with 0.25 Da tolerance). Scores from the 10 most similar library molecules are aggregated as a weighted average MS2Deepscore feature for random-forest re-ranking. For benchmarking, sweep the top-N cutoff and compute recall versus true-positive percentage, with cosine (fragment mass tolerance 0.05 Da) and modified cosine (0.25 Da precursor preselection) as baselines.

## Related tools

- **MS2Deepscore** (Supervised embedding model computing spectral similarity scores between query and all library spectra) — https://github.com/iomega/ms2query
- **MS2Query** (Wrapper pipeline that runs MS2Deepscore, selects the top 2000 spectra, and re-ranks with a random forest over 5 features) — https://github.com/iomega/ms2query
- **matchms** (Library cleaning, normalization, and baseline cosine / modified cosine score computation (fragment mass tolerance 0.05 Da))
- **Spec2Vec** (Unsupervised spectral embedding combined with MS2Deepscore inside MS2Query's candidate scoring)

## Examples

```
ms2query --spectra ./location_of_spectra --library ./library_folder --ionmode positive
```

## Evaluation signals

- On the exact-matches test set, MS2Query/MS2Deepscore ranking should yield a higher percentage of true positives across recall values than cosine (0.05 Da tolerance) and modified cosine (0.25 Da preselection) baselines
- The top-2000 preselection step must run over the full library with no precursor m/z filtering for the exact-match benchmark setting
- For benchmarking, new MS2Deepscore and Spec2Vec models must be trained such that no test spectra were used for training
- In random-forest feature-importance analysis, the weighted average MS2Deepscore over 10 similar library molecules should have the highest Gini importance of the five features, with all features nonzero (Supplementary Tables 1 and 2)
- A dummy-data test run after downloading the pretrained library should reproduce the expected_results_dummy_data.csv output in under half a minute

## Limitations

- MS2Deepscore/MS2Query performs better for positive mode MS2 spectra; negative-mode relies on separately trained models (Zenodo record 7104184)
- It is hard to objectively judge whether performance on newly generated data matches the curated benchmarking test set, since manual validation is challenging
- MS2Query does no peak picking or MS2-spectrum clustering; files with many MS2 spectra per feature should be preprocessed (e.g. MZMine FBMN) first

## Evidence

- [intro] MS2Query first uses MS2Deepscore32 to calculate spectral similarity scores between all library spectra and a query spectrum: "MS2Query first uses MS2Deepscore32 to calculate spectral similarity scores between all library spectra and a query spectrum"
- [discussion] Pre-calculated embeddings enable full-library comparison: "The biggest speed improvement was achieved by pre-calculating mass spectral embeddings for Spec2Vec and MS2Deepscore."
- [readme] Top-2000 selection without precursor preselection: [evidence span withheld — anchor status: external_doc]
- [discussion] Benchmark models exclude test spectra: "for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models"
- [discussion] Comparison result versus spectral similarity baselines: "Comparison with cosine score, modified cosine score, and MS2Deepscore shows that MS2Query performs better both at finding exact matches as well as finding analogues"
