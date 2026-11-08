---
name: gnps-modified-cosine-scoring
description: 'Use when building MS/MS testing-matrix fingerprints for NPOmix: each
  query spectrum needs a maximum modified cosine score against experimental spectra
  of each genome-matched sample.'
when_to_use_negative:
- You need full molecular networking or spectral library dereplication outputs — the
  article explicitly used only the modified cosine pairwise scoring functions, not
  classical GNPS molecular networking
- The query spectra have no comparable experimental spectra in the paired samples
  (e.g. a fungal species with no similar BGCs/spectra in the training set yields no
  answer)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: GNPS
  role: Computes the pairwise modified cosine similarity scores between query MS/MS
    spectra and pooled experimental sample spectra
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: NPOmix
  role: Consumes the resulting MS/MS fingerprint testing matrix as input to its k-nearest
    neighbor classifier
  repo: https://github.com/tiagolbiotech/NPOmix_python
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:580f30bf8056908302fe258ad63e65ff1b222cbd4a88e49cde083e51a99df686
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - GNPS
  - NPOmix
  - antiSMASH
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# gnps-modified-cosine-scoring

## Summary

Computes pairwise modified cosine similarity scores between a query MS/MS spectrum (reference or cryptic) and the pooled experimental spectra of every paired sample via GNPS, producing the per-sample maximum scores that form NPOmix MS/MS fingerprint columns.

## When to use

Use when building MS/MS testing-matrix fingerprints for NPOmix: each query spectrum needs a maximum modified cosine score against experimental spectra of each genome-matched sample.

## When NOT to use

- You need full molecular networking or spectral library dereplication outputs — the article explicitly used only the modified cosine pairwise scoring functions, not classical GNPS molecular networking
- The query spectra have no comparable experimental spectra in the paired samples (e.g. a fungal species with no similar BGCs/spectra in the training set yields no answer)

## Inputs

- Query MS/MS spectra (GNPS reference spectra or cryptic spectra from new samples)
- Pooled experimental MS/MS spectra per paired genome-MS/MS sample (multiple LC-MS/MS files per genome merged)
- GNPS job clone with sample metabolomes (e.g. task b915d1f78c364bf4be9900047760d95c, files added to G3 group)

## Outputs

- MS/MS fingerprint testing matrix (rows = query spectra, columns = samples, values = maximum modified cosine score per sample)
- Per-query vector of maximum modified cosine scores fed to the KNN classifier

## How to apply

For each query MS/MS spectrum — either a GNPS reference spectrum (e.g. from GNPS job task b915d1f78c364bf4be9900047760d95c) or a cryptic spectrum from a new sample — compute pairwise modified cosine scores in GNPS against the pooled experimental spectra of every paired sample. Multiple LC-MS/MS files for the same genome are merged into a single set of experimental spectra first. Record only the maximum modified cosine score per sample; these maxima become the columns of the MS/MS fingerprint vector. Note that full GNPS molecular networking is NOT used — only the pairwise spectrum-pair scoring functions — because the MS/MS fingerprint must be dimensionally aligned with the BGC training matrix (e.g. 1,040 columns).

## Related tools

- **GNPS** (Computes the pairwise modified cosine similarity scores between query MS/MS spectra and pooled experimental sample spectra) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **NPOmix** (Consumes the resulting MS/MS fingerprint testing matrix as input to its k-nearest neighbor classifier) — https://github.com/tiagolbiotech/NPOmix_python

## Examples

```
git clone https://github.com/tiagolbiotech/NPOmix_python.git  # then clone GNPS job task=b915d1f78c364bf4be9900047760d95c and add query spectra to the G3 group
```

## Evaluation signals

- Each MS/MS fingerprint vector has exactly one column per paired sample (matching the BGC training matrix dimensionality, e.g. 1,040 columns)
- Each entry is the maximum (not mean or sum) modified cosine score across the sample's pooled experimental spectra
- Scores range 0.0–1.0, consistent with the BGC fingerprint scale where identical spectra score 1.0
- Reference-spectrum queries (e.g. the 15 reference MS/MS fingerprints in the first release) yield nonzero scores in the expected matched samples
- Downstream NPOmix precision on the testing matrix reproduces reported values (top-1 = 81.8%, top-10 = 92.9% with Jaccard co-occurrence threshold)

## Limitations

- Modified cosine scoring cannot distinguish isomers or regioselectivity of homologs — correlation-based fingerprints lack that resolution
- Precision is dataset-dependent: scores may not be reproducible for datasets dissimilar to the PoDP-derived training set
- Cryptic spectra (no GNPS library match) can be scored, but resulting links require proper structure elucidation to confirm

## Evidence

- [readme] modified cosine scoring in GNPS, not molecular networking: "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample"
- [readme] not full molecular networking: "we did not use the full classical GNPS molecular networking capabilities; we only used the functions required to calculate a modified cosine similarity score between a pair of MS/MS spectra"
- [readme] merging multiple LC-MS/MS files per genome: "multiple LC-MS/MS files for the same genome were merged together into a single set of experimental MS/MS spectra"
- [discussion] cryptic spectra as queries: "spectra from cryptic metabolites (links without GNPS library matches and therefore absent from the MS/MS database"
