---
name: bgc-gene-cluster-family-fingerprint-construction
description: Use when paired genome-MS/MS samples have been antiSMASH-annotated and
  BiG-SCAPE-scored, and you need per-BGC genome-wide similarity vectors plus GCF labels
  as a KNN training matrix.
when_to_use_negative:
- You only need BGC clustering/GCF grouping itself rather than feature vectors for
  linking BGCs to MS/MS spectra
- The query BGCs have no BiG-SCAPE similarity to any training-set BGC (e.g. a fungal
  species with distant BGCs), so fingerprints carry no signal
- Input is already a feature/instance matrix ready for KNN training
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3474
tools:
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity scores (raw distance converted to 1 minus
    distance) used as fingerprint feature values
  repo: https://bigscape-corason.secondarymetabolites.org
- name: antiSMASH
  role: Produces the BGC GenBank annotations that constitute the fingerprint inputs
- name: NPOmix_python
  role: Repository scripts (concat_bigscape.sh, notebooks) that concatenate BiG-SCAPE
    outputs and assemble the fingerprint matrix
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: scikit-learn
  role: Consumes the BGC fingerprint matrix with GCF labels to train the k=3 nearest-neighbor
    classifier
- name: MIBiG database
  role: Reference BGCs included via BiG-SCAPE --mibig to anchor GCF labels of validated
    clusters
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:ff9650bfe8058fa8a962a103fe932ece0d4ccb7bd58fc92628e15e906077a032
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - MIBiG database
  - NPOmix
  - NPOmix_python
  - antiSMASH
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# bgc-gene-cluster-family-fingerprint-construction

## Summary

Builds NPOmix BGC fingerprint training matrices by computing, for each query BGC, the maximum BiG-SCAPE similarity score (1 minus raw distance) against every genome/metagenome in a paired dataset, then appending a GCF label column for KNN training.

## When to use

Use when paired genome-MS/MS samples have been antiSMASH-annotated and BiG-SCAPE-scored, and you need per-BGC genome-wide similarity vectors plus GCF labels as a KNN training matrix.

## When NOT to use

- You only need BGC clustering/GCF grouping itself rather than feature vectors for linking BGCs to MS/MS spectra
- The query BGCs have no BiG-SCAPE similarity to any training-set BGC (e.g. a fungal species with distant BGCs), so fingerprints carry no signal
- Input is already a feature/instance matrix ready for KNN training

## Inputs

- antiSMASH BGC GenBank files (antismash_only_gbk.zip, training BGCs plus query BGCs from samples)
- BiG-SCAPE pairwise similarity output tables per biosynthetic class (e.g. *_hybrids_glocal)
- Pfam-A.hmm (Pfam 35.0) for BiG-SCAPE domain annotation
- GCF label assignments for each training BGC

## Outputs

- BGC fingerprint training matrix: rows = 5,421 BGC max-similarity vectors, columns = 1,040 genomes/metagenomes, scores 0.0-1.0 with <0.7 set to absent
- GCF label column appended to the training matrix for KNN supervision

## How to apply

Run BiG-SCAPE on the antiSMASH BGCs (including training-set and query BGCs) with --pfam_dir pointing to the prepared Pfam-A.hmm, using --include_singletons and --mibig with multiple cores, then concatenate per-class outputs (e.g. the *_hybrids_glocal directory) with concat_bigscape.sh. For each of the 5,421 query BGCs, compute the maximum BiG-SCAPE similarity score (1 minus raw distance) against each of the 1,040 genomes/metagenomes, treating scores below the 0.7 similarity cutoff as absent (0.0). Assemble these max-similarity vectors as rows of the training matrix, with one column per genome, and append a GCF label column so the scikit-learn KNN classifier (k=3) knows which fingerprints belong together. Optionally add biosynthetic class features, which the paper reports improves NPOmix performance.

## Related tools

- **BiG-SCAPE** (Computes pairwise BGC similarity scores (raw distance converted to 1 minus distance) used as fingerprint feature values) — https://bigscape-corason.secondarymetabolites.org
- **antiSMASH** (Produces the BGC GenBank annotations that constitute the fingerprint inputs)
- **NPOmix_python** (Repository scripts (concat_bigscape.sh, notebooks) that concatenate BiG-SCAPE outputs and assemble the fingerprint matrix) — https://github.com/tiagolbiotech/NPOmix_python
- **scikit-learn** (Consumes the BGC fingerprint matrix with GCF labels to train the k=3 nearest-neighbor classifier)
- **MIBiG database** (Reference BGCs included via BiG-SCAPE --mibig to anchor GCF labels of validated clusters)

## Examples

```
conda activate bigscape && bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs/ -c 8 --include_singletons --mibig && sh concat_bigscape.sh /path/to/bigscape_output/ 2023-11-11_11-11-11_hybrids_glocal/
```

## Evaluation signals

- Training matrix shape matches the reference run: 5,421 BGC rows x 1,040 genome/metagenome columns plus a GCF label column
- All fingerprint values fall in [0.0, 1.0], with entries below the 0.7 similarity cutoff set to 0.0 (absent)
- BiG-SCAPE run completed with --include_singletons and --mibig and per-class outputs were concatenated into a single table per class (e.g. *_hybrids_glocal)
- Downstream KNN (k=3) on the matrix with similarity + biosynthetic class features and the Jaccard co-occurrence threshold of 0.7 reproduces top-1 = 81.8% and top-10 = 92.9% precision against MIBiG/PoDP validated links
- Every row carries a GCF label; repeated GCF labels among the k=3 neighbors are expected KNN behavior, not an error

## Limitations

- The 0.7 similarity cutoff is applied for fingerprint binarization/absence; scores below it are treated as the BGC being absent from that genome
- Metagenome quality can hinder BGC assembly, yielding fragmented BGCs and smaller-than-expected BGC counts, weakening fingerprints
- NPOmix is a correlation-based approach and cannot distinguish isomers or regioselectivity of homologs
- Precision may not be reproducible for datasets dissimilar to the current PoDP-derived training set

## Evidence

- [readme] BGC fingerprint construction with max similarity and 0.7 cutoff: "we identify the maximum similarity of the query BGC to one of the many BGCs in each genome (which can be considered a pool of BGCs) in the dataset"
- [readme] Similarity score semantics and cutoff: "the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
- [readme] Training matrix dimensions and label column: "we have used 5,421 BGCs in 1,040 genomes/metagenomes), where each genome is a column. This matrix can be fed to the k-nearest neighbor (KNN) algorithm"
- [readme] BiG-SCAPE run parameters from README: "bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ \ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig"
- [intro] Biosynthetic class features improve performance: "We showed that the addition of biosynthetic class and substructure features improves the performance of our Natural Products Mixed Omics tool"
