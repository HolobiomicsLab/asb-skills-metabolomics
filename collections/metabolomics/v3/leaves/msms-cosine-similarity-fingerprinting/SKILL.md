---
name: msms-cosine-similarity-fingerprinting
description: Use when a query MS/MS spectrum (reference spectrum from GNPS or a cryptic
  spectrum from a paired genome-MS/MS sample) must be converted into a similarity
  fingerprint so it can be matched to biosynthetic gene cluster families via the NPOmix
  KNN classifier.
when_to_use_negative:
- When the query spectrum is already a reference spectrum with a known GNPS library
  match and no BGC linkage is needed — the fingerprinting step is only useful for
  connecting spectra to gene cluster families
- When the sample has no BGC similar to any BGC in the NPOmix training dataset (e.g.,
  a fungal species with no homologous BGCs); the KNN output would contain no answer
- When full classical GNPS molecular networking annotation is required instead of
  similarity-score-based linkage to gene cluster families
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: GNPS
  role: Used to calculate the pairwise modified cosine similarity score between a
    pair of MS/MS spectra; only the functions required to compute this score are used,
    not the full classical molecular networking capabilities
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: NPOmix (Python)
  role: Consumes the MS/MS fingerprint testing matrix and the BGC fingerprint training
    matrix to run the KNN classifier and assign GCF labels to query spectra
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: Provides the BGC similarity scores (similarity cutoff 0.7) that form the training
    matrix which the MS/MS fingerprints are compared against in the KNN space
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:403ddba6dd6ed2c4a4e2620a53ec36ce7fc77f6c8e6bee5d79b7fcd4214cd5d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - Dereplicator+
  - GNPS
  - MZmine
  - NPOmix
  - NPOmix (Python)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# msms-cosine-similarity-fingerprinting

## Summary

Builds MS/MS fingerprints for NPOmix by computing pairwise modified cosine similarity scores between a query spectrum and every experimental spectrum in each paired-genome sample, yielding a row-vector of maximum per-sample similarities used as the testing matrix for KNN classification.

## When to use

Use when a query MS/MS spectrum (reference spectrum from GNPS or a cryptic spectrum from a paired genome-MS/MS sample) must be converted into a similarity fingerprint so it can be matched to biosynthetic gene cluster families via the NPOmix KNN classifier.

## When NOT to use

- When the query spectrum is already a reference spectrum with a known GNPS library match and no BGC linkage is needed — the fingerprinting step is only useful for connecting spectra to gene cluster families
- When the sample has no BGC similar to any BGC in the NPOmix training dataset (e.g., a fungal species with no homologous BGCs); the KNN output would contain no answer
- When full classical GNPS molecular networking annotation is required instead of similarity-score-based linkage to gene cluster families

## Inputs

- Query MS/MS spectrum (reference spectrum from GNPS or a cryptic MS/MS spectrum from a new paired sample)
- Set of experimental MS/MS spectra per paired-genome sample (e.g., LC-MS/MS files merged into a single set of spectra per genome)
- GNPS molecular networking job task ID used to compute modified cosine scores (e.g., task=b915d1f78c364bf4be9900047760d95c)

## Outputs

- MS/MS fingerprint row: vector of maximum modified cosine similarity scores, one per experimental sample
- Testing matrix whose rows are query MS/MS fingerprints and columns are the paired experimental MS/MS sample sets
- KNN-assigned GCF labels (top-k gene cluster family candidates) for each query spectrum

## How to apply

For each query MS/MS spectrum, use GNPS to calculate the pairwise modified cosine score against every experimental MS/MS spectrum in each sample of the paired-genome-MS/MS dataset (e.g., the 1,040 sets of paired experimental MS/MS spectra in the NPOmix training release). Then identify the maximum similarity of the query spectrum to one of the many MS/MS spectra in each experimental sample, producing a fingerprint row where each column is a different sample. Assemble these rows into a testing matrix that is fed to the k-nearest neighbor (KNN) algorithm alongside the BGC fingerprint training matrix (built from BiG-SCAPE similarity scores with a similarity cutoff of 0.7). The rationale is that the KNN classifier plots both BGC and MS/MS fingerprints in the same space so GCF labels can be assigned from the k-nearest neighbors (k = 3 in the brasilicardin A example), without using structure, class, or library annotations as linking inputs.

## Related tools

- **GNPS** (Used to calculate the pairwise modified cosine similarity score between a pair of MS/MS spectra; only the functions required to compute this score are used, not the full classical molecular networking capabilities) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **NPOmix (Python)** (Consumes the MS/MS fingerprint testing matrix and the BGC fingerprint training matrix to run the KNN classifier and assign GCF labels to query spectra) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (Provides the BGC similarity scores (similarity cutoff 0.7) that form the training matrix which the MS/MS fingerprints are compared against in the KNN space)

## Examples

```
git clone https://github.com/tiagolbiotech/NPOmix_python.git
```

## Evaluation signals

- Each query fingerprint row must contain exactly one maximum similarity value per experimental sample column (same number of columns as the BGC training matrix)
- Similarity scores must range from 0.0 to 1.0, with values below the 0.7 similarity cutoff treated as absent (0.0)
- Reproducing the brasilicardin A validation: the top-ranked GCF for the cryptic spectrum must correspond to MIBiG BGC0000632, with predicted fragment masses matching the observed spectrum at delta m/z < 0.001
- Dataset-level performance check: top-1 precision 81.8% and top-10 precision 92.9% when using similarity and biosynthetic class features with the Jaccard co-occurrence threshold
- GCF labels may repeat across the k-nearest neighbors (k = 3) when multiple neighbors belong to the same gene family — this is expected KNN behavior, not an error

## Limitations

- NPOmix (a correlation-based approach) does not have the resolution to distinguish between isomers and regioselectivity of the various homologs
- If the query sample has no BGC similar to any BGC in the NPOmix training dataset, the output will contain no answer
- The precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set
- Metagenome quality can hinder BGC assembly, leading to fewer BGCs than expected and many fragmented BGCs, which weakens the BGC fingerprint training matrix

## Evidence

- [readme] In the case of MS/MS fingerprints (Fig. 1D), we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample: "In the case of MS/MS fingerprints (Fig. 1D), we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS"
- [readme] we only used the functions required to calculate a modified cosine similarity score between a pair of MS/MS spectra: "Of particular note, we did not use the full classical GNPS molecular networking capabilities; we only used the functions required to calculate a modified cosine similarity score between a pair of"
- [readme] the MS/MS fingerprints also form a testing matrix; in this case, this matrix also contains 1,040 columns due to the 1,040 sets of paired experimental MS/MS spectra: [evidence span withheld — anchor status: not_found]
- [readme] Our approach is suitable for bacterial, fungal, algal, and plant genomes and MS/MS spectra: "Our approach is suitable for bacterial, fungal, algal, and plant genomes and MS/MS spectra. Metagenomes and metagenome-assembled genomes can also be used instead of genomes; however, complete genomes"
- [readme] The KNN algorithm plots the BGC fingerprints in the KNN space: "The KNN algorithm plots the BGC fingerprints in the KNN space (in Fig. 1G, the KNN space is exemplified by only 2 dimensions)."
