---
name: spectral-library-matching
description: Use when comparing spectral library search methods (MS2Query, MS2Deepscore,
  cosine, modified cosine) on MS2 query spectra to establish which achieves the highest
  true-positive percentage across recall values for exact matches.
when_to_use_negative:
- The goal is analogue (non-identical molecule) search rather than exact-match benchmarking
  — analogue searches in the paper filter library spectra on a maximum precursor m/z
  difference of 100 Da, which is deliberately disabled here.
- Input spectra have not been cleaned (metadata formatting, peak filtering to ≥3 peaks,
  intensity normalization) — the benchmark assumes matchms-cleaned GNPS library spectra.
- The task requires assigning identities to experimental feature MS2 spectra rather
  than evaluating search-method ranking quality on a held-out test set.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2Query
  role: Library search tool under test; runs full-library ranking with MS2Deepscore
    embeddings plus a random forest re-ranking of the top 2000 candidates, with no
    precursor m/z pre-filtering
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Baseline similarity method; computes supervised spectral similarity scores
    between every test query and all library spectra for ranking
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Used to load and clean spectra and to compute cosine (greedy implementation,
    0.05 Da fragment mass tolerance) and modified cosine scores with 0.25 Da precursor
    mass tolerance preselection
  repo: https://github.com/iomega/ms2query
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:b602fcdaeaaab33a5d6dacc68ad4b5d15bbfc477be0727240152e7db665e13b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - matchms
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# spectral-library-matching

## Summary

Benchmark MS2Query's exact-match library search — with no precursor m/z pre-filtering — against MS2Deepscore, cosine (0.05 Da fragment tolerance), and modified cosine (0.25 Da precursor preselection) by sweeping top-N cutoffs and comparing recall versus true-positive percentage on the exact matches test set.

## When to use

Use when comparing spectral library search methods (MS2Query, MS2Deepscore, cosine, modified cosine) on MS2 query spectra to establish which achieves the highest true-positive percentage across recall values for exact matches.

## When NOT to use

- The goal is analogue (non-identical molecule) search rather than exact-match benchmarking — analogue searches in the paper filter library spectra on a maximum precursor m/z difference of 100 Da, which is deliberately disabled here.
- Input spectra have not been cleaned (metadata formatting, peak filtering to ≥3 peaks, intensity normalization) — the benchmark assumes matchms-cleaned GNPS library spectra.
- The task requires assigning identities to experimental feature MS2 spectra rather than evaluating search-method ranking quality on a held-out test set.

## Inputs

- Exact matches test set MS2 query spectra (from Zenodo 10.5281/zenodo.6811540, loaded with matchms)
- Cleaned GNPS library spectra (ALL_GNPS.mgf downloaded 15 Nov 2021, cleaned with matchms)
- Trained MS2Deepscore and Spec2Vec models trained so that no test spectra were used in training

## Outputs

- benchmark_exact_matches.csv with recall versus percentage of true positives for MS2Query, MS2Deepscore, cosine, and modified cosine
- Recall-vs-true-positive-rate comparison showing MS2Query outperforming all three baselines

## How to apply

Load the exact matches test set spectra and the cleaned GNPS library spectra from Zenodo deposit 10.5281/zenodo.6811540 using matchms, after cleaning metadata, normalizing intensities, removing peaks above 1000 Da and peaks below 0.1% of the highest peak, and dropping spectra with fewer than 3 peaks. Run MS2Query (version 0.3.2) over the test queries without any precursor m/z pre-filtering so the full library is ranked per query; internally MS2Query computes MS2Deepscore similarities against all library spectra, selects the top 2000, and re-ranks them with a random forest over 5 features. Compute baselines: MS2Deepscore rankings over all query–library pairs; cosine scores with matchms' greedy implementation at 0.05 Da fragment mass tolerance; and modified cosine scores with a 0.25 Da precursor mass tolerance preselection. Sweep the top-N ranking cutoff for each method, computing recall and the percentage of true positives (exact matches) at each cutoff, then assemble the four recall-versus-true-positive-rate curves into benchmark_exact_matches.csv.

## Related tools

- **MS2Query** (Library search tool under test; runs full-library ranking with MS2Deepscore embeddings plus a random forest re-ranking of the top 2000 candidates, with no precursor m/z pre-filtering) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Baseline similarity method; computes supervised spectral similarity scores between every test query and all library spectra for ranking) — https://github.com/iomega/ms2query
- **matchms** (Used to load and clean spectra and to compute cosine (greedy implementation, 0.05 Da fragment mass tolerance) and modified cosine scores with 0.25 Da precursor mass tolerance preselection) — https://github.com/iomega/ms2query

## Examples

```
ms2query --spectra ./exact_matches_test_set --library ./ms2query_library --ionmode positive
```

## Evaluation signals

- benchmark_exact_matches.csv exists and contains recall-versus-true-positive-rate values for all four methods (MS2Query, MS2Deepscore, cosine, modified cosine)
- MS2Query with no mass pre-filtering achieves a higher percentage of true positives across recall values than cosine, modified cosine, and MS2Deepscore, matching the reported benchmark finding
- Cosine scores were computed with the greedy matchms implementation using a fragment mass tolerance of 0.05 Da; modified cosine used a 0.25 Da precursor mass tolerance preselection
- Test spectra were excluded from training of the MS2Deepscore and Spec2Vec models used in the benchmark
- Results are reproducible from Zenodo deposit 10.5281/zenodo.6811540

## Limitations

- It is hard to objectively judge whether MS2Query performs similarly on newly generated data compared to the benchmarking test set.
- The exact-match benchmark considers only library spectra within a mass tolerance of 0.25 Da of the query precursor.
- Supplementary notes describing benchmarking details are referenced but their content is not included in the extracted article text.
- MS2Query does not perform peak picking or clustering of similar MS2 spectra; files with many MS2 spectra per feature should be preprocessed (e.g. with MZMine) first.

## Evidence

- [results] MS2Query using the same settings as for the analogue search (no mass pre-filtering) achieves a higher percentage of true positives across recall values than search methods relying on MS2Deepscore or the cosine score.: [evidence span withheld — anchor status: not_found]
- [discussion] matchms cosine greedy 0.05 Da: "the cosine greedy implementation of matchms33 was used with a fragment mass tolerance of 0.05 Da"
- [readme] no precursor m/z preselection, top 2000: [evidence span withheld — anchor status: external_doc]
- [discussion] raw data and results on Zenodo: "The raw data, intermediate files and raw results can be found on 10.5281/zenodo.6811540."
- [readme] random forest 5 features score 0-1: [evidence span withheld — anchor status: external_doc]
