---
name: knn-gcf-classifier-training
description: Use when paired genome–MS/MS samples are available and the goal is to
  classify query MS/MS spectra (reference or cryptic) into candidate GCFs via a k-nearest
  neighbor model over BGC similarity fingerprints.
when_to_use_negative:
- The query organism has no BGC similar to any BGC in the NPOmix training dataset
  (e.g. a fungal species dissimilar to PoDP-derived bacteria) — output will contain
  no answer
- The goal is to distinguish isomers or regioselectivity of homologs — the correlation-based
  approach lacks this resolution
- The dataset is not similar to the NPOmix training set — precision scores may not
  be reproducible
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0196
tools:
- name: NPOmix
  role: KNN classifier pipeline that trains on BGC fingerprints and classifies MS/MS
    fingerprints into GCF candidates
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity scores (1 minus raw distance) and assigns
    GCFs used to build the training matrix
  repo: https://bigscape-corason.secondarymetabolites.org
- name: GNPS
  role: Computes pairwise modified cosine scores between query and pooled experimental
    MS/MS spectra for the testing matrix
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: scikit-learn
  role: Provides the k-nearest neighbor classifier used to fit the BGC training matrix
    and predict GCF labels
- name: antiSMASH
  role: Detects BGCs from genomes/metagenomes that form the training-set input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:fb35953c930a6e4e087d68ce76078cf34030befc826002a97f43b56eedcfa415
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - NPOmix
  - antiSMASH
  - k-nearest neighbor classifier
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# knn-gcf-classifier-training

## Summary

Train and apply the NPOmix k-nearest neighbor classifier that links MS/MS spectra to biosynthetic gene cluster families (GCFs) using similarity BGC fingerprints built from BiG-SCAPE scores and MS/MS fingerprints built from GNPS modified cosine scores.

## When to use

Use when paired genome–MS/MS samples are available and the goal is to classify query MS/MS spectra (reference or cryptic) into candidate GCFs via a k-nearest neighbor model over BGC similarity fingerprints.

## When NOT to use

- The query organism has no BGC similar to any BGC in the NPOmix training dataset (e.g. a fungal species dissimilar to PoDP-derived bacteria) — output will contain no answer
- The goal is to distinguish isomers or regioselectivity of homologs — the correlation-based approach lacks this resolution
- The dataset is not similar to the NPOmix training set — precision scores may not be reproducible

## Inputs

- antiSMASH BGC GenBank files for the training set (antismash_only_gbk.zip from Zenodo record 6637083)
- Concatenated BiG-SCAPE outputs (--mibig, --include_singletons, concatenated hybrids_glocal tables)
- GNPS job with paired metabolomes (task b915d1f78c364bf4be9900047760d95c) including reference spectra and cryptic spectra
- Pfam-A.hmm (Pfam 35.0) for BiG-SCAPE domain annotation
- GCF label column assigning each BGC fingerprint to a gene cluster family

## Outputs

- BGC similarity fingerprint training matrix (rows = BGCs, columns = genomes/metagenomes, plus GCF label column)
- MS/MS similarity fingerprint testing matrix (rows = query spectra, columns = paired samples)
- Predicted GCF labels for each query MS/MS spectrum at each k value
- Precision and annotation-rate tables (e.g. fig3_precision_evaluation.csv)

## How to apply

Build the BGC training matrix by computing, for each query BGC from the antiSMASH set (e.g. 5,421 BGCs), the maximum BiG-SCAPE-derived similarity score (1 minus raw distance) against every one of the ~1,040 genomes/metagenomes, zeroing scores below the 0.7 similarity cutoff, and append a GCF label column. Build the MS/MS testing matrix by computing pairwise modified cosine scores in GNPS between each query spectrum (from the GNPS job b915d1f78c364bf4be9900047760d95c or cryptic spectra) and the pooled experimental spectra of each paired sample, recording the maximum score per sample column. Optionally augment fingerprints with biosynthetic class and predicted substructure features. Train the scikit-learn KNN classifier on the labeled BGC matrix and classify each MS/MS fingerprint by majority GCF label over the k nearest neighbors (e.g. k=1..10); optionally apply the Jaccard co-occurrence threshold (0.7) that removes GCF candidates absent from the genome of the matched sample. Expected benchmark: ~81.8% precision at k=5 without the threshold and ~92.9% with the threshold using similarity plus biosynthetic class features, with annotation rates ~28% vs ~44%.

## Related tools

- **NPOmix** (KNN classifier pipeline that trains on BGC fingerprints and classifies MS/MS fingerprints into GCF candidates) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (Computes pairwise BGC similarity scores (1 minus raw distance) and assigns GCFs used to build the training matrix) — https://bigscape-corason.secondarymetabolites.org
- **GNPS** (Computes pairwise modified cosine scores between query and pooled experimental MS/MS spectra for the testing matrix) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **scikit-learn** (Provides the k-nearest neighbor classifier used to fit the BGC training matrix and predict GCF labels)
- **antiSMASH** (Detects BGCs from genomes/metagenomes that form the training-set input)

## Examples

```
git clone https://github.com/tiagolbiotech/NPOmix_python.git && wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1 && conda install -c anaconda scikit-learn
```

## Evaluation signals

- Reproduced precision matches Fig. 3: ~81.8% at k=5 without co-occurrence threshold and ~92.9% with the threshold (similarity + biosynthetic class features)
- Annotation rate of linked spectra is ~28% without and ~44% with the Jaccard co-occurrence threshold
- Training matrix dimensions align with the reference scale (5,421 BGCs x 1,040 genomes/metagenomes) with one GCF label column appended
- Similarity scores fall in [0, 1]; scores below the 0.7 cutoff are treated as absent (0.0)
- Automated linking recovers 13 of 14 reference MS/MS spectra correctly labeled (92.9% precision)

## Limitations

- NPOmix does not have the resolution to distinguish isomers or regioselectivity of homologs
- Metagenome quality can hinder BGC assembly, yielding fewer and fragmented BGCs and degrading fingerprint quality
- Precision may not be reproducible for datasets dissimilar to the current training set
- Submitting an organism with no BGC similar to the training set produces no answer
- No RiPP-class validated metabolites exist in the PoDP training data, and paired data are almost exclusively from bacteria

## Evidence

- [intro] we deployed a K-Nearest Neighbor (KNN) algorithm that uses similarity BGC fingerprints and analogously similarity MS/MS fingerprints to classify gene cluster family: "we deployed a K-Nearest Neighbor (KNN) algorithm that uses similarity BGC fingerprints and analogously similarity MS/MS fingerprints to classify gene cluster family"
- [readme] we identify the maximum similarity of the query BGC to one of the many BGCs in each genome: "In order to create a BGC fingerprint (Fig. 1C), we identify the maximum similarity of the query BGC to one of the many BGCs in each genome"
- [readme] for our first release, round 4, we have used 5,421 BGCs in 1,040 genomes/metagenomes: "(e.g., for our first release, round 4, we have used 5,421 BGCs in 1,040 genomes/metagenomes), where each genome is a column"
- [discussion] precision scores of top-1 = 81.8% and top-10 = 92.9%: "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [discussion] by submitting a fungal species that has no BGC similar to any BGC in the NPOmix training dataset: "by submitting a fungal species that has no BGC similar to any BGC in the NPOmix training dataset (obtained from the PoDP database), the output would contain no answer"
