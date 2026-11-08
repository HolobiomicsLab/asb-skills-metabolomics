---
name: bgc-similarity-fingerprint-construction
description: Use when building the labeled BGC training matrix for NPOmix from antiSMASH
  .gbk files and BiG-SCAPE pairwise similarity outputs, prior to KNN classification
  of MS/MS fingerprints.
when_to_use_negative:
- The input query genome/fungal species has no BGC similar to any BGC in the NPOmix
  training dataset — the output would contain no answer
- The metagenome quality is too low for BGC assembly, yielding fragmented BGCs and
  an uninformative fingerprint
- You already possess an MS/MS fingerprint testing matrix and only need classification,
  not BGC feature construction
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3172
tools:
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity (raw distances) used to derive maximum similarity
    scores per genome and to assign GCFs
  repo: https://bigscape-corason.secondarymetabolites.org
- name: antiSMASH
  role: Produces the .gbk BGC annotation files that form the query and reference BGC
    sets
- name: NPOmix
  role: Python pipeline in which the BGC fingerprint training matrix is assembled
    and fed to the KNN classifier
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: scikit-learn
  role: Provides the k-nearest-neighbor classifier trained on the BGC fingerprint
    matrix with GCF labels
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:5d00b5e27e6effc33370d7a69a1eb861f4de23dce8273f6757a594d48d5a0f05
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - MIBiG database
  - NPOmix
  - antiSMASH
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# bgc-similarity-fingerprint-construction

## Summary

Construction of BGC similarity fingerprints for the NPOmix KNN classifier: each query BGC is represented as a vector of maximum BiG-SCAPE-derived similarity scores (1 − raw distance) against the pooled BGCs of every genome/metagenome, binarized at the 0.7 similarity cutoff, and labeled with a GCF for training.

## When to use

Use when building the labeled BGC training matrix for NPOmix from antiSMASH .gbk files and BiG-SCAPE pairwise similarity outputs, prior to KNN classification of MS/MS fingerprints.

## When NOT to use

- The input query genome/fungal species has no BGC similar to any BGC in the NPOmix training dataset — the output would contain no answer
- The metagenome quality is too low for BGC assembly, yielding fragmented BGCs and an uninformative fingerprint
- You already possess an MS/MS fingerprint testing matrix and only need classification, not BGC feature construction

## Inputs

- antiSMASH BGC GenBank files (antismash_only_gbk.zip / antismash_bgcs.zip from Zenodo record 6637083)
- BiG-SCAPE pairwise similarity output network (concatenated per-class outputs, e.g. hybrids_glocal), run with --include_singletons --mibig
- hmmpressed Pfam-A.hmm database (Pfam 35.0)
- GCF label assignments from the BiG-SCAPE network

## Outputs

- BGC fingerprint training matrix (rows = 5,421 query BGC max-similarity vectors, columns = 1,040 genomes/metagenomes, binarized at 0.7)
- GCF label column appended to the training matrix for KNN training

## How to apply

Run BiG-SCAPE (with --pfam_dir pointing at hmmpressed Pfam-A.hmm from Pfam 35.0, --include_singletons --mibig, e.g. -c 8) over the antiSMASH BGC set from Zenodo record 6637083, then concatenate per-class outputs (e.g. with concat_bigscape.sh into a hybrids_glocal network). For each query BGC, compute the maximum BiG-SCAPE-derived similarity score (1 minus raw distance) against all BGCs in each of the 1,040 genomes/metagenomes; the result is one row of the training matrix (5,421 BGCs x 1,040 genome columns). Binarize scores below the 0.7 similarity cutoff to absent (0.0), since identical BGCs score 1.0 and a 0.8 score indicates a homolog. Append a final column of GCF labels so the scikit-learn KNN classifier knows which fingerprint patterns belong together. Concatenating the hybrids_glocal class output is essential so hybrid BGCs are not lost in the per-class BiG-SCAPE split.

## Related tools

- **BiG-SCAPE** (Computes pairwise BGC similarity (raw distances) used to derive maximum similarity scores per genome and to assign GCFs) — https://bigscape-corason.secondarymetabolites.org
- **antiSMASH** (Produces the .gbk BGC annotation files that form the query and reference BGC sets)
- **NPOmix** (Python pipeline in which the BGC fingerprint training matrix is assembled and fed to the KNN classifier) — https://github.com/tiagolbiotech/NPOmix_python
- **scikit-learn** (Provides the k-nearest-neighbor classifier trained on the BGC fingerprint matrix with GCF labels)

## Examples

```
bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig; sh concat_bigscape.sh /path/to/bigscape_output/ 2023-11-11_11-11-11_hybrids_glocal/
```

## Evaluation signals

- Training matrix dimensions match the reference scale: 5,421 BGCs x 1,040 genomes/metagenomes
- All similarity scores below the 0.7 cutoff are binarized to absent (0.0); identical BGCs score exactly 1.0
- Every row carries a valid GCF label in the appended column; hybrid BGCs present in the concatenated hybrids_glocal output are included
- Downstream KNN run with similarity + biosynthetic class features reproduces Fig. 3 behavior: ~81.8% top-1 precision without and ~92.9% top-10 precision with the Jaccard co-occurrence threshold
- BiG-SCAPE run used --include_singletons --mibig and per-class outputs were concatenated with concat_bigscape.sh (no class-specific tables missing)

## Limitations

- Metagenome quality can hinder BGC assembly, producing fragmented BGCs and fewer fingerprints than expected
- The 0.7 cutoff and training set (PoDP-derived, almost exclusively bacterial) may not transfer; precision may not be reproducible for every dataset
- The correlation-based fingerprint cannot resolve isomers or regioselectivity of homologs

## Evidence

- [readme] In order to create a BGC fingerprint (Fig. 1C), we identify the maximum similarity of the query BGC to one of the many BGCs in each genome: "In order to create a BGC fingerprint (Fig. 1C), we identify the maximum similarity of the query BGC to one of the many BGCs in each genome"
- [readme] for our first release, round 4, we have used 5,421 BGCs in 1,040 genomes/metagenomes: "for our first release, round 4, we have used 5,421 BGCs in 1,040 genomes/metagenomes"
- [readme] a score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome: "a score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
- [readme] This matrix can be fed to the k-nearest neighbor (KNN) algorithm in order to train it with the genomic data: "This matrix can be fed to the k-nearest neighbor (KNN) algorithm in order to train it with the genomic data"
- [readme] one extra column is required in this genomic data matrix, a column that labels each BGC fingerprint with a GCF: "one extra column is required in this genomic data matrix, a column that labels each BGC fingerprint with a GCF"
- [readme] bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig: "bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ \ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig"
