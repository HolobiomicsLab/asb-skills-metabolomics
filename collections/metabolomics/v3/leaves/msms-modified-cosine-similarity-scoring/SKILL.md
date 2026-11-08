---
name: msms-modified-cosine-similarity-scoring
description: Use when building the testing matrix of MS/MS fingerprints for NPOmix
  KNN classification — i.e., when you have query reference spectra (e.g., 15 GNPS
  reference spectra) and paired samples with merged experimental spectra, and need
  per-sample maximum modified cosine similarity scores.
when_to_use_negative:
- When the goal is full spectral library annotation or molecular networking — the
  article explicitly states full classical GNPS molecular networking was not used,
  only pairwise modified cosine scoring
- When the query organism has no BGC similar to any BGC in the NPOmix training dataset
  (e.g., a fungal species dissimilar to PoDP-derived training BGCs) — the output will
  contain no answer
- When distinguishing isomers or regioselectivity of homologs — the correlation-based
  approach lacks that resolution
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: GNPS
  role: Provides the pairwise modified cosine similarity score computation between
    query and experimental MS/MS spectra (without molecular networking)
- name: NPOmix
  role: Consumes the MS/MS fingerprint testing matrix and projects it into the KNN
    space trained on BGC fingerprints to assign GCF labels
  repo: https://github.com/tiagolbiotech/NPOmix_python
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:720d0e78cdf87cf69804fda6f7621d1e99353d351dcd6fb7b2c9dd2c369dd0a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - MIBiG database
  - NPOmix
  - antiSMASH
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# msms-modified-cosine-similarity-scoring

## Summary

Computes MS/MS fingerprint matrices for NPOmix by scoring query reference spectra against merged experimental spectra of each paired genome–metabolome sample using pairwise GNPS modified cosine scores, deliberately avoiding full GNPS molecular networking.

## When to use

Use when building the testing matrix of MS/MS fingerprints for NPOmix KNN classification — i.e., when you have query reference spectra (e.g., 15 GNPS reference spectra) and paired samples with merged experimental spectra, and need per-sample maximum modified cosine similarity scores.

## When NOT to use

- When the goal is full spectral library annotation or molecular networking — the article explicitly states full classical GNPS molecular networking was not used, only pairwise modified cosine scoring
- When the query organism has no BGC similar to any BGC in the NPOmix training dataset (e.g., a fungal species dissimilar to PoDP-derived training BGCs) — the output will contain no answer
- When distinguishing isomers or regioselectivity of homologs — the correlation-based approach lacks that resolution

## Inputs

- Query MS/MS reference spectra from the GNPS database (e.g., 15 reference spectra for the NPOmix release)
- Merged experimental MS/MS spectra per paired sample (multiple LC-MS/MS files per genome merged into one spectral set; e.g., 1,040 paired genome/metagenome–metabolome samples from PoDP)
- GNPS job with metabolomes for the training-set strains (e.g., task b915d1f78c364bf4be9900047760d95c)

## Outputs

- MS/MS fingerprint testing matrix: rows = query spectra, columns = paired samples, values = maximum GNPS modified cosine similarity score per sample
- KNN-space label assignments per query spectrum (GCF labels of the k=3 nearest BGC fingerprint neighbors)

## How to apply

Take each query MS/MS spectrum (a GNPS reference spectrum or a cryptic spectrum from a new sample) and compute the pairwise GNPS modified cosine score against every merged experimental spectrum of each of the paired samples (e.g., the 1,040 genome/metagenome–MS/MS pairs in the NPOmix training set); multiple LC-MS/MS files for the same genome are merged into a single set of experimental spectra first. For each sample, retain only the maximum modified cosine similarity of the query spectrum to any spectrum in that sample, yielding one row per query spectrum with one column per sample. Do NOT run full GNPS molecular networking — only the pairwise modified cosine function is needed. Scores conceptually mirror the BGC fingerprint side (BiG-SCAPE similarity = 1 minus raw distance, with a 0.7 similarity cutoff), so the resulting testing matrix has the same sample columns as the BGC fingerprint training matrix and can be projected into the same KNN space (k=3 neighbors, GCF labels assigned with possible repeats, Jaccard co-occurrence threshold 0.7).

## Related tools

- **GNPS** (Provides the pairwise modified cosine similarity score computation between query and experimental MS/MS spectra (without molecular networking))
- **NPOmix** (Consumes the MS/MS fingerprint testing matrix and projects it into the KNN space trained on BGC fingerprints to assign GCF labels) — https://github.com/tiagolbiotech/NPOmix_python

## Evaluation signals

- Testing matrix dimensions match expectations: rows = query reference spectra (15 in the reference release), columns = paired samples (1,040 in the training set)
- Each row contains the maximum (not average or sum) modified cosine score per sample column, mirroring the max-similarity construction of the BGC fingerprint training matrix
- Downstream evaluation against curated MIBiG/PoDP links yields top-1 precision ≈ 81.8% and top-10 precision ≈ 92.9% at k=3 with the Jaccard co-occurrence threshold of 0.7 (13 of 14 reference spectra correctly labeled)
- No full GNPS molecular networking output (network edges/components) is required or produced — only pairwise spectrum-to-spectrum scores
- Randomness of KNN label assignment equals 0 when using similarity plus biosynthetic class features with the co-occurrence threshold

## Limitations

- NPOmix precision may not be reproducible for every dataset, especially if the dataset is not similar to the current training set
- Submitting a species with no BGC similar to the training set produces no answer; metagenome quality can hinder BGC assembly and fragment BGCs
- The correlation-based approach cannot distinguish isomers or regioselectivity of homologs, and new links require proper structure elucidation to confirm planar (2D) and absolute (3D) structures

## Evidence

- [readme] GNPS modified cosine score used for MS/MS fingerprints: "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra"
- [readme] No full molecular networking used: "we did not use the full classical GNPS molecular networking capabilities; we only used the functions required to calculate a modified cosine similarity score"
- [readme] Testing matrix scale in reference release: "for our first release, this testing matrix contained 15 reference MS/MS fingerprints (rows) for MS/MS reference spectra from the GNPS database"
- [discussion] Expected precision outcomes with co-occurrence threshold: "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features"
- [readme] Merging multiple LC-MS/MS files per genome: "multiple LC-MS/MS files for the same genome were merged together into a single set of experimental MS/MS spectra"
