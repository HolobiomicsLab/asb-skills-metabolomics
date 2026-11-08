---
name: paired-omics-fingerprint-vector-analysis
description: Use when you have paired genome/metagenome + LC-MS/MS samples and need
  to link MS/MS spectra (GNPS reference or cryptic) to biosynthetic gene clusters
  with confidence scores.
when_to_use_negative:
- The query genome has no BGC similar to any BGC in the NPOmix training dataset (e.g.,
  a fungal species distant from PoDP bacteria) — the output will contain no answer.
- You need to distinguish isomers or regioselectivity of homologs — the correlation-based
  approach lacks this resolution.
- Your dataset differs substantially from the PoDP training set — the reported precision
  may not be reproducible.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NPOmix
  role: K-nearest neighbor classifier connecting MS/MS fragmentation spectra to BGCs
    via fingerprint vectors
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity scores (1 minus raw distance) used to build
    BGC fingerprint vectors and assign GCFs
  repo: https://bigscape-corason.secondarymetabolites.org
- name: antiSMASH
  role: Predicts BGCs from genomes/metagenomes as input GenBank files for fingerprinting
- name: GNPS
  role: Calculates modified cosine similarity scores between MS/MS spectra for the
    MS/MS fingerprint testing matrix
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:71f8c83b6be51e5de54ae7dd57c8dad0e6d04ee78beadfea720e1b5b6f36c869
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - NPOmix
  - antiSMASH
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# paired-omics-fingerprint-vector-analysis

## Summary

Build paired BGC and MS/MS fingerprint vectors from paired genome–metabolome samples and classify MS/MS spectra to gene cluster families with a k-nearest neighbor classifier (NPOmix), then apply a Jaccard co-occurrence threshold of 0.7 as a confidence filter on candidate BGC–metabolite links.

## When to use

Use when you have paired genome/metagenome + LC-MS/MS samples and need to link MS/MS spectra (GNPS reference or cryptic) to biosynthetic gene clusters with confidence scores.

## When NOT to use

- The query genome has no BGC similar to any BGC in the NPOmix training dataset (e.g., a fungal species distant from PoDP bacteria) — the output will contain no answer.
- You need to distinguish isomers or regioselectivity of homologs — the correlation-based approach lacks this resolution.
- Your dataset differs substantially from the PoDP training set — the reported precision may not be reproducible.

## Inputs

- antiSMASH BGC GenBank files (.gbk) from genomes/metagenomes of paired samples
- GNPS job output with pairwise modified cosine scores between query and experimental MS/MS spectra
- BiG-SCAPE similarity output files (e.g., hybrids_glocal network files) for all BGC classes
- GCF label column assigning each BGC fingerprint row to a gene cluster family
- PoDP validated gene–metabolite links for validation

## Outputs

- BGC fingerprint training matrix (rows = BGCs with max similarity per genome column, e.g., 5,421 BGCs x 1,040 genomes)
- MS/MS fingerprint testing matrix (rows = query spectra, columns = paired experimental samples)
- Per-spectrum GCF label predictions from the KNN classifier at k=3
- Precision metrics (top-1, top-10) and retained-link counts before and after the Jaccard co-occurrence threshold

## How to apply

Run BiG-SCAPE on antiSMASH-predicted BGCs to compute pairwise similarity scores (1 minus BiG-SCAPE raw distance), then build BGC fingerprint vectors where each row is a BGC and each column is a genome holding the maximum similarity score (similarity cutoff 0.7; identical BGCs score 1.0). Analogously, use GNPS modified cosine scores to build MS/MS fingerprint vectors where rows are query spectra and columns are experimental samples. Feed the BGC fingerprint matrix, labeled per-row with GCF assignments, to a k-nearest neighbor classifier (k=3), and plot MS/MS fingerprints into the same KNN space to obtain GCF labels. Optionally add biosynthetic class and substructure features, which improves performance, and apply the Jaccard co-occurrence threshold of 0.7 to filter candidate links. Validate against PoDP validated gene–metabolite links, expecting top-1 = 81.8% and top-10 = 92.9% precision (randomness = 0).

## Related tools

- **NPOmix** (K-nearest neighbor classifier connecting MS/MS fragmentation spectra to BGCs via fingerprint vectors) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (Computes pairwise BGC similarity scores (1 minus raw distance) used to build BGC fingerprint vectors and assign GCFs) — https://bigscape-corason.secondarymetabolites.org
- **antiSMASH** (Predicts BGCs from genomes/metagenomes as input GenBank files for fingerprinting)
- **GNPS** (Calculates modified cosine similarity scores between MS/MS spectra for the MS/MS fingerprint testing matrix) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c

## Examples

```
git clone https://github.com/tiagolbiotech/NPOmix_python.git && wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1
```

## Evaluation signals

- Applying the Jaccard co-occurrence threshold (0.7) with similarity + biosynthetic class features reproduces top-1 = 81.8% and top-10 = 92.9% precision (randomness = 0) against PoDP validated links.
- BGC fingerprint similarity scores fall in [0.0, 1.0], with identical BGCs at 1.0 and scores below the 0.7 similarity cutoff treated as absent.
- The BGC fingerprint matrix shape matches README scale expectations (e.g., ~5,421 BGCs x ~1,040 genomes/metagenomes for release round 4), with one extra column of GCF labels for KNN training.
- Automated linking recovers ~18 previously known mass spectra matched to their experimentally validated biosynthetic genes.
- Dimensional consistency: the MS/MS testing matrix must have the same genome/sample columns (e.g., 1,040) as the BGC training matrix.

## Limitations

- Metagenome quality can hinder BGC assembly, yielding fewer and fragmented BGCs than expected in microbiome samples.
- NPOmix cannot distinguish isomers or regioselectivity of homologs.
- Precision may not be reproducible for datasets dissimilar to the PoDP training set; no RiPP-class validated metabolites existed in PoDP training data, and paired plant data are lacking.
- Links predicted for new (non-PoDP-validated) metabolites still require proper structure elucidation to confirm planar (2D) and absolute (3D) chemistry.

## Evidence

- [intro] we deployed a K-Nearest Neighbor (KNN) algorithm that uses similarity BGC fingerprints and analogously similarity MS/MS fingerprints to classify gene cluster family: "we deployed a K-Nearest Neighbor (KNN) algorithm that uses similarity BGC fingerprints and analogously similarity MS/MS fingerprints to classify gene cluster family"
- [discussion] precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold): "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [readme] we identify the maximum similarity of the query BGC to one of the many BGCs in each genome (which can be considered a pool of BGCs) in the dataset.: "we identify the maximum similarity of the query BGC to one of the many BGCs in each genome (which can be considered a pool of BGCs) in the dataset."
- [readme] we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample.: "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample."
- [readme] for our first release, round 4, we have used 5,421 BGCs in 1,040 genomes/metagenomes: "for our first release, round 4, we have used 5,421 BGCs in 1,040 genomes/metagenomes"
