---
name: ms2deepscore-candidate-preselection-and-reranking
description: Use when annotating MS2 query spectra against a large library (e.g. GNPS)
  and both exact matches and analogues are needed, with a fast, scalable alternative
  to modified cosine score-based searching.
when_to_use_negative:
- Input spectra have not been peak-picked or deconvoluted — MS2Query does no peak
  picking or clustering, so feature-level preprocessing (e.g. with MZMine FBMN output
  MGF) must happen first
- The analysis requires a structurally annotated spectral library other than the pretrained
  GNPS-based one without training new models (see 'create your own library' workflow
  instead)
- Only a small library or a strict precursor-m/z-window search is needed — the article
  notes modified cosine with 100 Da precursor preselection is the baseline comparison,
  not the fast path for large libraries
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2Query
  role: Runs the full preselect-and-rerank search; computes the 5 features and random
    forest re-ranking of the top 2000 MS2Deepscore candidates
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Computes embedding-based spectral similarity between each query and all library
    spectra to preselect the top 2000 candidates
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Cleans spectra/metadata (peak filtering, intensity normalization, ion-mode
    splitting) before searching
- name: Spec2Vec
  role: Embedding-based chemical similarity predictor integrated in MS2Query alongside
    MS2Deepscore
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:d00f3571ca15288207c3f1f59225d1711b599e36f89b01593934cb0dff9cacdc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - Spec2Vec
  - matchms
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ms2deepscore-candidate-preselection-and-reranking

## Summary

Two-stage MS2 library matching used by MS2Query: MS2Deepscore embedding similarity preselects the top 2000 candidate library spectra (without precursor m/z preselection), then a random forest combining 5 features re-ranks candidates to identify exact matches and analogues in one run.

## When to use

Use when annotating MS2 query spectra against a large library (e.g. GNPS) and both exact matches and analogues are needed, with a fast, scalable alternative to modified cosine score-based searching.

## When NOT to use

- Input spectra have not been peak-picked or deconvoluted — MS2Query does no peak picking or clustering, so feature-level preprocessing (e.g. with MZMine FBMN output MGF) must happen first
- The analysis requires a structurally annotated spectral library other than the pretrained GNPS-based one without training new models (see 'create your own library' workflow instead)
- Only a small library or a strict precursor-m/z-window search is needed — the article notes modified cosine with 100 Da precursor preselection is the baseline comparison, not the fast path for large libraries

## Inputs

- MS2 query spectra in mzML, json, mgf, msp, mzxml, usi format or a pickled matchms object (e.g. the 139 positive-mode NIST 1950 plasma spectra from Zenodo 10.5281/zenodo.6811540)
- Pretrained MS2Query model files and preprocessed library spectra (sqlite library with pre-calculated MS2Deepscore embeddings, downloaded from Zenodo record 6124552 positive / 7104184 negative mode)
- Ionization mode specification (positive or negative)

## Outputs

- Per-spectrum results CSV (e.g. ms2query_results_nist1950_positive.csv) with predicted exact-match/analogue library hits and random-forest scores between 0 and 1
- Compound-class annotations of predicted matches (e.g. via LIPID MAPS / HMDB / GNPS mappings) for class-consistency checks

## How to apply

Clean query and library spectra with matchms (remove peaks above 1000 Da, peaks below 0.1% of the highest peak, spectra with fewer than 3 peaks; cap spectra at 500 peaks by removing lowest-intensity peaks; clean metadata and normalize intensities, splitting by ionization mode). Run MS2Query (version 0.3.2 in the case study), which first uses MS2Deepscore with pre-calculated library embeddings to score all library spectra against each query and selects the top 2000 by similarity — deliberately no preselection on precursor m/z difference. A random forest model then combines 5 features per candidate-query pair to produce a score between 0 and 1, and the highest-scoring library spectrum is selected; applying a minimum score threshold filters out unreliable matches. Library, models, and pre-calculated embeddings are downloaded from Zenodo (e.g. record 6124552 for positive mode, 7104184 for negative mode). Validate against the NIST 1950 plasma case study: 75 matches predicted from 139 positive-mode spectra, 72 of them lipid-class matches.

## Related tools

- **MS2Query** (Runs the full preselect-and-rerank search; computes the 5 features and random forest re-ranking of the top 2000 MS2Deepscore candidates) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Computes embedding-based spectral similarity between each query and all library spectra to preselect the top 2000 candidates) — https://github.com/iomega/ms2query
- **matchms** (Cleans spectra/metadata (peak filtering, intensity normalization, ion-mode splitting) before searching)
- **Spec2Vec** (Embedding-based chemical similarity predictor integrated in MS2Query alongside MS2Deepscore)

## Examples

```
ms2query --spectra ./nist1950_spectra --library ./library_folder --ionmode positive
```

## Evaluation signals

- NIST 1950 plasma case-study reproduction: 75 matches predicted from 139 positive-mode query spectra, of which 72 belong to lipid compound classes
- Dummy-data test: running the downloaded library on dummy_spectra.mgf reproduces expected_results_dummy_data.csv in under half a minute
- Random forest scores fall in the 0–1 range per candidate-query pair, with the top-scoring library spectrum selected and unreliable matches removed by the minimum score threshold
- Preselection step yields exactly 2000 library spectra per query with the highest MS2Deepscore, with no precursor m/z preselection applied
- Benchmark comparison: MS2Query outperforms cosine, modified cosine (0.05 Da fragment tolerance, 0.25 Da mass tolerance for exact matches), and MS2Deepscore alone on exact matches and analogues

## Limitations

- It is hard to objectively judge whether MS2Query performs similarly on newly generated data compared to the benchmarking test set, given the challenge of manually validating results
- MS2Query does not perform peak picking or clustering of similar MS2 spectra; files with many MS2 spectra per feature must be preprocessed (e.g. MZMine FBMN)
- The pretrained library is GNPS-based (e.g. ALL_GNPS.mgf downloaded 15 Nov 2021); results depend on library coverage, and GNPS libraries contain only about 2.5% of known natural products

## Evidence

- [discussion] Two-stage preselection/reranking grounded in article workflow: "The workflow for running MS2Query first uses MS2Deepscore to calculate spectral similarity scores between all library spectra and a query spectrum."
- [discussion] Top-2000 selection without precursor m/z preselection: "The top 2000 spectra with the highest MS2Deepscore are selected."
- [discussion] Random forest reranking with 5 features and score threshold: [evidence span withheld — anchor status: external_doc]
- [discussion] matchms peak filtering thresholds: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [results] Case-study metric for validation: [evidence span withheld — anchor status: not_found]
- [readme] README: command-line invocation and library download: [evidence span withheld — anchor status: external_doc]
