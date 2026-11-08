---
name: msms-spectral-fingerprint-construction
description: Use when query MS/MS spectra (validated reference spectra or cryptic
  spectra without GNPS library matches) must be converted into per-sample similarity
  vectors over paired genome-MS/MS samples so a KNN classifier can assign gene cluster
  family (GCF) labels.
when_to_use_negative:
- The goal is metabolite identification against a spectral library rather than linking
  spectra to biosynthetic gene clusters via paired data.
- The dataset lacks paired genome-MS/MS samples (e.g., a fungal species with no paired
  data or no BGCs similar to the PoDP training set), so fingerprints have no meaningful
  training counterpart.
- The analysis requires distinguishing isomers or regioselectivity of homologs; the
  correlation-based fingerprinting cannot resolve these.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GNPS
  role: Computes the pairwise modified cosine similarity scores between query spectra
    and pooled experimental sample spectra used to build the fingerprint vectors
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: NPOmix
  role: Consumes the MS/MS fingerprint testing matrix and classifies each fingerprint
    against the BGC training matrix via KNN
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: scikit-learn
  role: Provides the k-nearest neighbor classifier applied to the constructed fingerprints
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:366488f6c55316b5bd86577ac41c07962937b42e7fca0d9717e759092177e268
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - GNPS
  - NPOmix
  - antiSMASH
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# msms-spectral-fingerprint-construction

## Summary

Construction of MS/MS similarity fingerprints for the NPOmix KNN classifier: each query spectrum (GNPS reference or cryptic) is represented as a vector of maximum modified cosine scores against the pooled experimental spectra of every paired genome-MS/MS sample. This testing matrix is the metabolomic analogue of the BGC fingerprint training matrix and is required for linking spectra to gene cluster families.

## When to use

Use when query MS/MS spectra (validated reference spectra or cryptic spectra without GNPS library matches) must be converted into per-sample similarity vectors over paired genome-MS/MS samples so a KNN classifier can assign gene cluster family (GCF) labels.

## When NOT to use

- The goal is metabolite identification against a spectral library rather than linking spectra to biosynthetic gene clusters via paired data.
- The dataset lacks paired genome-MS/MS samples (e.g., a fungal species with no paired data or no BGCs similar to the PoDP training set), so fingerprints have no meaningful training counterpart.
- The analysis requires distinguishing isomers or regioselectivity of homologs; the correlation-based fingerprinting cannot resolve these.

## Inputs

- Query MS/MS reference spectra from GNPS database (from cloned GNPS job b915d1f78c364bf4be9900047760d95c)
- Cryptic MS/MS spectra from new paired genome-MS/MS samples
- Pooled experimental LC-MS/MS spectra for each of the 1,040 paired samples (multiple files per genome merged)
- Paired genome-MS/MS sample metadata from the Paired Omics Data Platform (PoDP)

## Outputs

- MS/MS fingerprint testing matrix (rows = query spectra, columns = one per paired sample, entries = maximum modified cosine score, e.g., 15 fingerprints x 1,040 columns)
- Input testing matrix for the NPOmix KNN classifier GCF-label assignment

## How to apply

Use GNPS to compute pairwise modified cosine scores between each query MS/MS spectrum and every experimental spectrum in each paired sample; notably only the modified cosine pairwise function is used, not full GNPS molecular networking. For each sample, record the maximum score obtained against its pooled experimental spectra (multiple LC-MS/MS files for the same genome are merged into a single experimental spectra set). Each query spectrum becomes one row in the testing matrix; columns match the genomes/metagenomes in the BGC training matrix (1,040 in the first release), so the two matrices share dimensionality for KNN classification. Reference spectra are taken from the cloned GNPS job b915d1f78c364bf4be9900047760d95c; cryptic spectra come from new samples containing both a genome and LC-MS/MS data. The resulting matrix (e.g., 15 reference MS/MS fingerprints in the first release) is fed to the scikit-learn KNN classifier alongside the labeled BGC training matrix.

## Related tools

- **GNPS** (Computes the pairwise modified cosine similarity scores between query spectra and pooled experimental sample spectra used to build the fingerprint vectors) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **NPOmix** (Consumes the MS/MS fingerprint testing matrix and classifies each fingerprint against the BGC training matrix via KNN) — https://github.com/tiagolbiotech/NPOmix_python
- **scikit-learn** (Provides the k-nearest neighbor classifier applied to the constructed fingerprints)

## Evaluation signals

- Fingerprint matrix columns must equal the number of paired genomes/metagenomes in the BGC training matrix (1,040 columns in the first release) so rows are plotted in the same KNN space.
- First-release reference matrix should contain 15 reference MS/MS fingerprint rows from the GNPS database.
- All fingerprint entries are modified cosine scores bounded in the 0.0-1.0 range, with per-sample entries being the maximum over the sample's pooled spectra.
- Downstream KNN precision on the benchmark should reach ~81.8% top-1 without the Jaccard co-occurrence threshold and ~92.9% top-10 with the threshold (similarity + biosynthetic class features), with annotation rates of ~28% and ~44% respectively.
- The reproduced metrics should be written to fig3_precision_evaluation.csv when combined with the BGC-side fingerprint construction.

## Limitations

- Full classical GNPS molecular networking capabilities were not used; only pairwise modified cosine scoring, so no molecular network topology is produced.
- NPOmix (a correlation-based approach) does not have the resolution to distinguish between isomers and regioselectivity of the various homologs.
- The precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set.
- Metagenome quality can hinder BGC assembly, leading to fragmented BGCs that weaken the counterpart training matrix and degrade classification.

## Evidence

- [readme] GNPS pairwise modified cosine scoring builds fingerprints: "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample."
- [readme] Only pairwise modified cosine, not molecular networking: "we did not use the full classical GNPS molecular networking capabilities; we only used the functions required to calculate a modified cosine similarity score"
- [readme] First-release testing matrix dimensions: "for our first release, this testing matrix contained 15 reference MS/MS fingerprints (rows) for MS/MS reference spectra from the GNPS database"
- [discussion] Benchmark precision with and without co-occurrence threshold: "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
