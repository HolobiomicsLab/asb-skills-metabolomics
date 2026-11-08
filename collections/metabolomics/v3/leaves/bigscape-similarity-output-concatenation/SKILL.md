---
name: bigscape-similarity-output-concatenation
description: Use when you have antiSMASH .gbk BGCs from multiple genomes and need
  a unified pairwise BGC similarity table per biosynthetic class to build NPOmix training
  fingerprints.
when_to_use_negative:
- You only need GCF clustering memberships and not per-pair similarity scores for
  fingerprint construction — plain BiG-SCAPE network output suffices without concatenation.
- Your BGCs come from a fungal species with no BGC similar to any NPOmix training-set
  BGC, since the downstream classifier will return no answer.
- You need MS/MS-to-MS/MS similarity rather than BGC-to-BGC similarity — use GNPS
  modified cosine scoring instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: BiG-SCAPE
  role: Computes pairwise similarity between all antiSMASH BGCs (with --include_singletons
    and --mibig) to produce the per-class distance outputs that are concatenated
- name: antiSMASH
  role: Produces the BGC GenBank files for both training genomes and user samples
    that serve as BiG-SCAPE input
- name: NPOmix
  role: Consumes the concatenated similarity tables to build the BGC fingerprint training
    matrix for the KNN classifier
  repo: https://github.com/tiagolbiotech/NPOmix_python
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:201c1dd2ba8cc75e14d9fef0133d30d66401a5cd99f38595976fd873879c2ef8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - MIBiG database
  - NPOmix
  - antiSMASH
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# BiG-SCAPE similarity output concatenation

## Summary

Run BiG-SCAPE on antiSMASH-predicted BGCs (including MIBiG and singletons) and concatenate the per-class pairwise similarity outputs (e.g. *_hybrids_glocal) into a single similarity table that feeds NPOmix BGC fingerprint construction.

## When to use

Use when you have antiSMASH .gbk BGCs from multiple genomes and need a unified pairwise BGC similarity table per biosynthetic class to build NPOmix training fingerprints.

## When NOT to use

- You only need GCF clustering memberships and not per-pair similarity scores for fingerprint construction — plain BiG-SCAPE network output suffices without concatenation.
- Your BGCs come from a fungal species with no BGC similar to any NPOmix training-set BGC, since the downstream classifier will return no answer.
- You need MS/MS-to-MS/MS similarity rather than BGC-to-BGC similarity — use GNPS modified cosine scoring instead.

## Inputs

- antiSMASH BGC GenBank files (antismash_only_gbk.zip from Zenodo record 6637083, plus sample antiSMASH output placed in the antismash_gbk_only folder)
- Pfam-A.hmm (Pfam 35.0), gunzipped and hmmpress-ed
- concat_bigscape.sh script from the NPOmix_python repository

## Outputs

- BiG-SCAPE output directories per biosynthetic class and mode (e.g. *_hybrids_glocal) containing pairwise distance files
- A single concatenated similarity table per class, usable to compute maximum BGC-to-genome similarity scores (1 - raw distance) with a 0.7 similarity cutoff

## How to apply

After antiSMASH annotation, run BiG-SCAPE with --pfam_dir pointing to the processed Pfam-A.hmm (hmmpress-ed Pfam 35.0), --include_singletons and --mibig, and 8 cores, so every query BGC is compared against the MIBiG-backed training set. BiG-SCAPE emits separate output directories per biosynthetic class and distance mode (e.g. 2023-11-11_11-11-11_hybrids_glocal); run concat_bigscape.sh on the output root plus the class directory name to merge them into one similarity table per class. Convert raw BiG-SCAPE distances to similarity as (1 - raw distance), and treat scores below the 0.7 similarity cutoff as absent when later building the BGC fingerprint matrix (5,421 BGCs x 1,040 genomes in the NPOmix release). The concatenated table is the direct input for computing, per query BGC, the maximum similarity against each genome's pool of BGCs.

## Related tools

- **BiG-SCAPE** (Computes pairwise similarity between all antiSMASH BGCs (with --include_singletons and --mibig) to produce the per-class distance outputs that are concatenated)
- **antiSMASH** (Produces the BGC GenBank files for both training genomes and user samples that serve as BiG-SCAPE input)
- **NPOmix** (Consumes the concatenated similarity tables to build the BGC fingerprint training matrix for the KNN classifier) — https://github.com/tiagolbiotech/NPOmix_python

## Examples

```
bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig && sh concat_bigscape.sh /path/to/bigscape_output/ 2023-11-11_11-11-11_hybrids_glocal/
```

## Evaluation signals

- BiG-SCAPE run completes with --mibig so MIBiG reference BGCs are present in every pairwise comparison, enabling similarity scores against training genomes.
- concat_bigscape.sh succeeds given the BiG-SCAPE output root and the class-mode directory name (e.g. 2023-11-11_11-11-11_hybrids_glocal), producing one merged table per class.
- Similarity values derived from the concatenated table fall in [0.0, 1.0], where identical BGCs score 1.0 and scores below the 0.7 cutoff are treated as absent.
- Downstream BGC fingerprint matrix dimensions match expectations (e.g. 5,421 query BGCs as rows x 1,040 genomes/metagenomes as columns in the NPOmix first release).
- Reproducing the full NPOmix pipeline yields top-1 = 81.8% and top-10 = 92.9% precision against validated MIBiG/PoDP links, indicating the concatenated similarity table was built correctly.

## Limitations

- Metagenome quality can hinder BGC assembly, yielding fewer, fragmented BGCs that weaken similarity comparisons.
- Similarity-based linking cannot distinguish isomers or regioselectivity of homologous BGCs.
- Precision scores may not be reproducible for datasets dissimilar to the PoDP-derived training set.

## Evidence

- [readme] BiG-SCAPE run with antiSMASH BGCs and concatenation: "bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ \ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig"
- [readme] Concatenation command: "sh concat_bigscape.sh /path/to/bigscape_output/ 2023-11-11_11-11-11_hybrids_glocal/"
- [readme] Similarity definition and cutoff: "each biosynthetic gene cluster (BGC) in the genomes will go through a pairwise similarity comparison (Fig. 1A) to every other BGC in the same set of genomes to compute similarity scores (1 minus"
- [readme] Absent BGC cutoff: "the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
- [readme] Training matrix dimensions: "for our first release, round 4, we have used 5,421 BGCs in 1,040 genomes/metagenomes"
