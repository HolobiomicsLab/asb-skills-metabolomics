---
name: exact-match-benchmark-evaluation
description: Use when you need to quantitatively compare MS/MS library-search methods
  at finding exact library matches, by sweeping top-N cutoffs and measuring true-positive
  percentage across recall values.
when_to_use_negative:
- When the goal is analogue search (non-identical matches) rather than exact library
  matches — the analogue benchmark applies different settings and a precursor m/z
  difference filter of 100 Da.
- When the library or models have not been checked for training/test contamination
  — models must be trained so that none of the test spectra were used for training
  MS2Deepscore or Spec2Vec.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2Query
  role: Primary method under evaluation; ranks full library per query (no precursor
    m/z pre-filtering) using MS2Deepscore scores plus random-forest re-ranking of
    the top 2000 spectra with 5 features
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Baseline spectral similarity method used to rank library spectra for every
    test query
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Used for loading/cleaning spectra and computing cosine (greedy implementation,
    0.05 Da fragment mass tolerance) and modified cosine (0.25 Da precursor preselection)
    baseline scores
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:88fae80e95f5320a873e0046b6888eac87d073ba65c4622e8c753bea9a5036a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - matchms
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# exact-match-benchmark-evaluation

## Summary

Benchmarks MS2Query (with no precursor m/z pre-filtering) against MS2Deepscore, cosine, and modified cosine (0.25 Da preselection) baselines on an exact matches test set, by sweeping top-N ranking cutoffs and comparing recall versus percentage of true positives.

## When to use

Use when you need to quantitatively compare MS/MS library-search methods at finding exact library matches, by sweeping top-N cutoffs and measuring true-positive percentage across recall values.

## When NOT to use

- When the goal is analogue search (non-identical matches) rather than exact library matches — the analogue benchmark applies different settings and a precursor m/z difference filter of 100 Da.
- When the library or models have not been checked for training/test contamination — models must be trained so that none of the test spectra were used for training MS2Deepscore or Spec2Vec.

## Inputs

- Exact matches test set MS2 spectra (from Zenodo 10.5281/zenodo.6811540, loaded with matchms)
- Cleaned GNPS library spectra (GNPS ALL_GNPS.mgf downloaded 15 Nov 2021; cleaned metadata, normalized intensities, peaks >1000 Da and <0.1% of highest peak removed, spectra with <3 peaks removed)

## Outputs

- benchmark_exact_matches.csv — recall-versus-true-positive-percentage values for MS2Query, MS2Deepscore, cosine, and modified cosine across top-N cutoffs

## How to apply

Load the exact matches test set spectra and cleaned GNPS library spectra from Zenodo deposit 10.5281/zenodo.6811540 using matchms, having cleaned metadata and normalized peak intensities. Run MS2Query (version 0.3.2) over the test queries without any precursor m/z pre-filtering so the full library is ranked. Run MS2Deepscore to compute spectral similarity scores between every query and all library spectra and rank the library accordingly; compute cosine scores with the greedy matchms implementation using a fragment mass tolerance of 0.05 Da; and compute modified cosine scores with matchms, preselecting only library spectra within a 0.25 Da precursor mass tolerance of each query. For each of the four methods, sweep the top-N ranking cutoff, computing recall and the percentage of true positives (exact matches) at each cutoff, and assemble the values into a recall-versus-true-positive-rate table (benchmark_exact_matches.csv). Validation: MS2Query should achieve a higher percentage of true positives across recall values than cosine, modified cosine, and MS2Deepscore, consistent with the published benchmark finding.

## Related tools

- **MS2Query** (Primary method under evaluation; ranks full library per query (no precursor m/z pre-filtering) using MS2Deepscore scores plus random-forest re-ranking of the top 2000 spectra with 5 features) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Baseline spectral similarity method used to rank library spectra for every test query) — https://github.com/iomega/ms2query
- **matchms** (Used for loading/cleaning spectra and computing cosine (greedy implementation, 0.05 Da fragment mass tolerance) and modified cosine (0.25 Da precursor preselection) baseline scores)

## Examples

```
ms2query --spectra ./exact_matches_test_set --library ./library_folder --ionmode positive --filter_ionmode
```

## Evaluation signals

- Across swept top-N cutoffs, MS2Query's true-positive percentage curve lies above those of cosine, modified cosine, and MS2Deepscore on the exact matches test set.
- The output table benchmark_exact_matches.csv contains recall and true-positive percentage columns for all four methods at each top-N cutoff.
- Modified cosine baseline is restricted to library spectra within 0.25 Da precursor mass tolerance of each query, and cosine uses a 0.05 Da fragment mass tolerance, matching the published settings.
- MS2Query is run with no precursor m/z pre-filtering, i.e., the full library is ranked for each query.

## Limitations

- It is hard to objectively judge whether MS2Query performs similarly on newly generated data compared to the benchmarking test set.
- Benchmark performance depends on models being retrained so none of the test spectra were used for training MS2Deepscore or Spec2Vec.
- The cosine baseline is not intended for analogue search and the modified cosine requires precursor preselection, so the comparison reflects exact-match retrieval rather than analogue discovery.

## Evidence

- [results] MS2Query using the same settings as for the analogue search (no mass pre-filtering) achieves a higher percentage of true positives across recall values than search methods relying on MS2Deepscore or the cosine score.: [evidence span withheld — anchor status: not_found]
- [discussion] the cosine greedy implementation of matchms33 was used with a fragment mass tolerance of 0.05 Da: "the cosine greedy implementation of matchms33 was used with a fragment mass tolerance of 0.05 Da"
- [discussion] only library spectra were considered within a mass tolerance of 0.25 Da: "only library spectra were considered within a mass tolerance of 0.25 Da"
- [discussion] The raw data, intermediate files and raw results can be found on 10.5281/zenodo.6811540.: "The raw data, intermediate files and raw results can be found on 10.5281/zenodo.6811540."
- [readme] In contrast to other analogue search methods, no preselection on precursor m/z is performed.: [evidence span withheld — anchor status: external_doc]
