---
name: precision-evaluation-across-neighborhood-sizes
description: Use when you have a trained NPOmix BGC fingerprint matrix and MS/MS test
  fingerprints and need to quantify how classifier precision and annotation rate vary
  with k, feature set, and co-occurrence filtering.
when_to_use_negative:
- Your query organism has no BGC similar to any BGC in the NPOmix training dataset
  (e.g. a fungal species with no homologs) — the output would contain no answer, so
  precision evaluation is meaningless
- Your dataset differs substantially from the PoDP-derived training set — the article
  warns the precision score may not be reproducible for every dataset
- You want validated metabolite identification rather than precision benchmarking
  of GCF classification (NPOmix cannot distinguish isomers or regioselectivity of
  homologs)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: NPOmix
  role: KNN classifier pipeline that links MS/MS spectra to GCFs using similarity
    BGC and MS/MS fingerprints; the Python version is the implementation under evaluation
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: scikit-learn
  role: Provides the k-nearest neighbor classifier (KNeighborsClassifier) trained
    on the BGC fingerprint training matrix with GCF labels
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity (raw distances converted to 1-minus-distance
    scores) and GCF assignments used to build the training fingerprints
  repo: https://bigscape-corason.secondarymetabolites.org
- name: GNPS
  role: Computes pairwise modified cosine scores between query MS/MS spectra and pooled
    experimental spectra of each paired sample for the testing fingerprints
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:0b2bf5cdf0a0660c44eb505b92d1095a22e18ddea1972f16b23a36967d3d0562
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - NPOmix
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# precision-evaluation-across-neighborhood-sizes

## Summary

Benchmarks NPOmix's k-nearest neighbor BGC-to-MS/MS linker by sweeping k across three feature sets (similarity only; +biosynthetic class; +class+substructure), with and without the Jaccard co-occurrence threshold, reporting precision and annotation rates (Fig. 3 reproduction).

## When to use

Use when you have a trained NPOmix BGC fingerprint matrix and MS/MS test fingerprints and need to quantify how classifier precision and annotation rate vary with k, feature set, and co-occurrence filtering.

## When NOT to use

- Your query organism has no BGC similar to any BGC in the NPOmix training dataset (e.g. a fungal species with no homologs) — the output would contain no answer, so precision evaluation is meaningless
- Your dataset differs substantially from the PoDP-derived training set — the article warns the precision score may not be reproducible for every dataset
- You want validated metabolite identification rather than precision benchmarking of GCF classification (NPOmix cannot distinguish isomers or regioselectivity of homologs)

## Inputs

- antiSMASH BGC training set (antismash_bgcs.zip / antismash_only_gbk.zip from Zenodo record 6637083)
- Concatenated BiG-SCAPE similarity outputs (hybrids_glocal tables, run with --include_singletons --mibig)
- BGC fingerprint training matrix with GCF label column (e.g. 5,421 BGCs x 1,040 genomes)
- MS/MS fingerprint testing matrix (max modified cosine scores vs pooled spectra of each paired sample, 1,040 columns)
- Reference MS/MS spectra from the cloned GNPS job b915d1f78c364bf4be9900047760d95c plus cryptic spectra

## Outputs

- fig3_precision_evaluation.csv with precision and annotation-rate values for the three feature sets, across k values, with and without the Jaccard co-occurrence threshold

## How to apply

Build the BGC training matrix (5,421 antiSMASH BGCs x 1,040 genomes) from maximum BiG-SCAPE similarity scores (1 minus raw distance, 0.7 cutoff) plus a GCF label column, and the MS/MS testing matrix from maximum modified cosine scores computed in GNPS per paired sample. Construct three feature sets: similarity fingerprints alone, similarity plus biosynthetic class features, and similarity plus biosynthetic class and predicted substructure features. Train the scikit-learn KNeighborsClassifier on each training matrix and classify each query MS/MS fingerprint by majority GCF label, sweeping k (e.g. k=1..10). At each k, evaluate precision both without filtering and after applying the Jaccard co-occurrence threshold that removes GCF candidates absent from the genome of the matched sample. Record precision and annotation rates per k, per feature set, and per filtering condition to fig3_precision_evaluation.csv; the article benchmark is ~81.8% precision at k=5 without the threshold and 92.9% top-10 with the threshold (similarity + biosynthetic class features), with annotation rates of ~28% and ~44% respectively.

## Related tools

- **NPOmix** (KNN classifier pipeline that links MS/MS spectra to GCFs using similarity BGC and MS/MS fingerprints; the Python version is the implementation under evaluation) — https://github.com/tiagolbiotech/NPOmix_python
- **scikit-learn** (Provides the k-nearest neighbor classifier (KNeighborsClassifier) trained on the BGC fingerprint training matrix with GCF labels)
- **BiG-SCAPE** (Computes pairwise BGC similarity (raw distances converted to 1-minus-distance scores) and GCF assignments used to build the training fingerprints) — https://bigscape-corason.secondarymetabolites.org
- **GNPS** (Computes pairwise modified cosine scores between query MS/MS spectra and pooled experimental spectra of each paired sample for the testing fingerprints) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c

## Examples

```
from sklearn.neighbors import KNeighborsClassifier; knn = KNeighborsClassifier(n_neighbors=5); knn.fit(bgc_fingerprint_matrix, gcf_labels); preds = knn.predict(msms_test_matrix)
```

## Evaluation signals

- Reproduced precision matches the reported Fig. 3 values: ~81.8% at k=5 without the co-occurrence threshold and ~92.9% (13 of 14 reference MS/MS spectra correctly labeled) with the Jaccard threshold of 0.7, for similarity + biosynthetic class features
- Annotation rates for linked spectra fall near ~28% (without threshold) and ~44% (with threshold)
- Training matrix shape matches the reference scale: 5,421 BGC rows x 1,040 genome/metagenome columns plus a GCF label column
- Similarity scores are bounded in [0, 1]; entries at or below the 0.7 BiG-SCAPE cutoff are treated as BGC absence (score 0.0)
- Repeated GCF labels in the returned k-neighbor list are expected KNN behavior, not an error; majority voting over k neighbors must be applied

## Limitations

- The precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set
- NPOmix (a correlation-based approach) does not have the resolution to distinguish between isomers and regioselectivity of the various homologs
- Metagenome quality can hinder BGC assembly, leading to fewer and fragmented BGCs and thus degraded fingerprints
- Current validated PoDP training metabolites are almost exclusively bacterial (no RiPP class; lack of paired data for plants), so precision estimates may not transfer to other taxa

## Evidence

- [discussion] Reported top-k precision: "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [intro] Feature-set effect: "We showed that the addition of biosynthetic class and substructure features improves the performance of our Natural Products Mixed Omics tool"
- [readme] Fingerprint construction and cutoff: "the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
- [readme] Training matrix scale: "for our first release, round 4, we have used 5,421 BGCs in 1,040 genomes/metagenomes"
- [methods] Training data acquisition: "wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1"
