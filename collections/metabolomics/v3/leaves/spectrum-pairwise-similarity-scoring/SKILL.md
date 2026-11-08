---
name: spectrum-pairwise-similarity-scoring
description: Use when you have filtered/normalized spectra and need all-pairs spectral
  similarity scores (e.g. cosine scores with matching-peak counts) to identify related
  spectra or build network edges.
when_to_use_negative:
- The spectra have not yet been cleaned and normalized (metadata cleaning, default_filters,
  normalize_intensities) — run the filtering pipeline first since inconsistent metadata
  and unnormalized intensities distort cosine scores.
- You need structure-aware or precursor-aware scores such as the modified cosine score
  or fingerprint-based comparisons — use those similarity functions instead of plain
  CosineGreedy, or ecosystem scores like Spec2Vec/MS2DeepScore.
- You only need a single reference-vs-query comparison rather than all-pairs scoring
  — a targeted comparison avoids computing the full N×N score matrix.
edam_operation: http://edamontology.org/operation_3840
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Provides calculate_scores, the Scores object, and CosineGreedy similarity
    function used to compute pairwise spectral similarity over filtered spectra
  repo: https://github.com/matchms/matchms
- name: NumPy
  role: Score implementations (e.g. cosine) are based on NumPy, enabling fast similarity
    derivation over large numbers of spectra
- name: pyteomics
  role: 'File import layer: matchms mzML, mzXML and MGF importers are built on top
    of pyteomics'
- name: Numba
  role: Underlying acceleration backend for matchms score implementations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:7c6faf2942b07bfbc0178d1fd5eada218ceb937de92f0ededdaef9fe9da2f54c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - NumPy
  - Numba
  - SciPy
  - matchms
  - pyteomics
  edam_operation: http://edamontology.org/operation_3840
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spectrum-pairwise-similarity-scoring

## Summary

Computes pairwise similarity scores between all reference and query MS/MS spectra using matchms.calculate_scores with a similarity function such as CosineGreedy, yielding per-pair score and matched-peak-count records ready for thresholding and export.

## When to use

Use when you have filtered/normalized spectra and need all-pairs spectral similarity scores (e.g. cosine scores with matching-peak counts) to identify related spectra or build network edges.

## When NOT to use

- The spectra have not yet been cleaned and normalized (metadata cleaning, default_filters, normalize_intensities) — run the filtering pipeline first since inconsistent metadata and unnormalized intensities distort cosine scores.
- You need structure-aware or precursor-aware scores such as the modified cosine score or fingerprint-based comparisons — use those similarity functions instead of plain CosineGreedy, or ecosystem scores like Spec2Vec/MS2DeepScore.
- You only need a single reference-vs-query comparison rather than all-pairs scoring — a targeted comparison avoids computing the full N×N score matrix.

## Inputs

- Pre-processed MS/MS spectra in MGF format (or mzML/mzXML/msp/JSON/USI) loaded via matchms.load_from_mgf
- A similarity function instance, e.g. CosineGreedy() from matchms.similarity
- A minimum matching-peak threshold (e.g. n_matching >= 20)
- Output column schema: reference_scan_id, query_scan_id, score, n_matching

## Outputs

- matchms.Scores object containing one entry per reference/query pair with (reference, query, score, n_matching)
- CSV file of retained spectrum pairs with columns reference_scan_id, query_scan_id, score, n_matching

## How to apply

Load pre-processed spectra (already passed through default_filters and normalize_intensities) from the MGF file with matchms.load_from_mgf, then call matchms.calculate_scores with references=spectrums, queries=spectrums and similarity_function=CosineGreedy() so every reference is compared to every query via the matchms.Scores object. Iterate over the returned Scores object, unpack each entry as (reference, query, score, n_matching), and skip self-comparisons where the reference is the query. Apply the paper's example retention threshold of keeping only pairs with at least 20 matching peaks (n_matching >= 20); CosineGreedy scores fall in [0, 1]. Collect reference_scan_id and query_scan_id from spectrum.metadata['scans'] and write records to CSV with columns reference_scan_id, query_scan_id, score, n_matching. For very large collections, prefer the sparse-scores handling in matchms >= 0.18.0, where only actually computed non-null values are stored, or the Pipeline class configured via score_computations=[['cosinegreedy', {'tolerance': 1.0}]] to express the same workflow in a YAML-configurable pipeline.

## Related tools

- **matchms** (Provides calculate_scores, the Scores object, and CosineGreedy similarity function used to compute pairwise spectral similarity over filtered spectra) — https://github.com/matchms/matchms
- **NumPy** (Score implementations (e.g. cosine) are based on NumPy, enabling fast similarity derivation over large numbers of spectra)
- **pyteomics** (File import layer: matchms mzML, mzXML and MGF importers are built on top of pyteomics)
- **Numba** (Underlying acceleration backend for matchms score implementations)

## Examples

```
spectrums = matchms.load_from_mgf('pesticides.mgf'); scores = matchms.calculate_scores(references=spectrums, queries=spectrums, similarity_function=CosineGreedy())
```

## Evaluation signals

- CSV output must contain rows only for distinct spectrum pairs meeting the retention threshold (n_matching >= 20), with self-comparisons excluded.
- All score values must lie in [0, 1] as produced by CosineGreedy.
- Re-read the CSV and verify the exact column schema: reference_scan_id, query_scan_id, score, n_matching.
- Every retained row's scan ids must be resolvable in the input spectra metadata (spectrum.metadata['scans']).
- If using the Pipeline workflow, confirm the YAML config file was generated and rerunning pipeline.run() on the same MGF reproduces the same score set.

## Limitations

- Cosine-based scores are spectra-based measures; they do not account for precursor mass differences — for that use the modified cosine score, also provided by matchms.
- Not all pairs and scores are equally important in large many-to-many comparisons; since matchms >= 0.18.0 scores are handled sparsely, so only computed non-null values are stored.
- Very large spectrum collections can be memory-intensive; matchms supports faster similarity measures for initial pre-selection and sparse storage formats, enabling comparison of several hundred thousands of spectra.
- Metadata in large collected datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect entries, which degrades score quality if not filtered first.

## Evidence

- [abstract] Pairwise scoring over filtered spectra: "matchms facilitates easily deriving similarity measures between large number of spectra at comparably fast speed due to score implementations based on NumPy"
- [abstract] Cosine score availability: "matchms provides functions to derive different similarity scores between spectra. Those include the established spectra-based measures of the cosine score or modified cosine score"
- [readme] Sparse scores since 0.18.0: [evidence span withheld — anchor status: external_doc]
- [readme] Pipeline invocation via score_computations: [evidence span withheld — anchor status: external_doc]
