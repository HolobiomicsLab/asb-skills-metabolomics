---
name: bigscape-bgc-similarity-fingerprinting
description: Use when you have antiSMASH .gbk BGCs from paired genome–metabolome samples
  and need BGC fingerprint rows (with GCF labels) to train the NPOmix KNN classifier
  or to query against its space.
when_to_use_negative:
- The query organism (e.g., a fungal species) has no BGCs similar to any BGC in the
  NPOmix PoDP training set — similarity-only features will produce no answer
- BGCs come from low-quality metagenomes where assembly hinders BGC recovery, leaving
  fragmented or missing BGCs that distort similarity fingerprints
- The goal is MS/MS spectral fingerprinting (GNPS modified cosine scores against experimental
  samples), which is the analogous test-side procedure, not the BGC-side fingerprinting
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity (1 − raw distance) and groups BGCs into GCFs;
    outputs are converted into the fingerprint training matrix
  repo: https://bigscape-corason.secondarymetabolites.org
- name: antiSMASH
  role: Produces the .gbk BGC annotation files that serve as BiG-SCAPE input for training
    genomes and query samples
- name: hmmer (hmmpress + Pfam-A.hmm)
  role: Downloads, parses, and presses the Pfam 35.0 domain database required as --pfam_dir
    for BiG-SCAPE runs
- name: NPOmix
  role: Consumes the BGC fingerprint matrix (with GCF labels) as the KNN (k = 3) training
    set against which MS/MS fingerprints are classified
  repo: https://github.com/tiagolbiotech/NPOmix_python
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:5a391ace6a0fcf5c14ed6b6e0d4f2b858708d9bf59bda1a5cca58c16314dd6d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - NPOmix
  - antiSMASH
  - hmmer (hmmpress + Pfam-A.hmm)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# bigscape-bgc-similarity-fingerprinting

## Summary

Builds BGC similarity fingerprint matrices for NPOmix by running BiG-SCAPE pairwise comparisons over antiSMASH-derived .gbk BGCs and converting raw distances to per-genome maximum similarity vectors (similarity = 1 − raw distance, cutoff 0.7).

## When to use

Use when you have antiSMASH .gbk BGCs from paired genome–metabolome samples and need BGC fingerprint rows (with GCF labels) to train the NPOmix KNN classifier or to query against its space.

## When NOT to use

- The query organism (e.g., a fungal species) has no BGCs similar to any BGC in the NPOmix PoDP training set — similarity-only features will produce no answer
- BGCs come from low-quality metagenomes where assembly hinders BGC recovery, leaving fragmented or missing BGCs that distort similarity fingerprints
- The goal is MS/MS spectral fingerprinting (GNPS modified cosine scores against experimental samples), which is the analogous test-side procedure, not the BGC-side fingerprinting

## Inputs

- antiSMASH-annotated BGC GenBank files (.gbk) from training genomes (Zenodo antismash_only_gbk.zip) and from query samples
- Pfam-A.hmm domain database (Pfam 35.0, hmmpress-formatted)
- Paired genome–metabolome sample metadata from the Paired Omics Data Platform (PoDP)

## Outputs

- BiG-SCAPE pairwise distance outputs (per BGC class: hybrids, NRPS, RiPP, etc.), concatenated across classes
- Gene cluster family (GCF) assignments for each BGC
- BGC similarity fingerprint training matrix (rows = BGC fingerprints of maximum similarity scores, columns = 1,040 genomes/metagenomes, plus a GCF label column)

## How to apply

Download and hmmpress the Pfam-A.hmm database (Pfam 35.0), then run BiG-SCAPE on all antiSMASH BGCs — the training set from the PoDP/MIBiG-derived Zenodo archive plus the query sample BGCs — with the flags --include_singletons and --mibig, and concatenate the outputs for all BGC classes. For each BGC, compute similarity to every other BGC as 1 minus the BiG-SCAPE raw distance, treating scores below the 0.7 similarity cutoff as absent (0.0). Build the fingerprint by taking, for each genome (or metagenome) column, the maximum similarity of the query BGC to any BGC in that genome, yielding a training matrix (e.g., 5,421 BGCs × 1,040 genomes/metagenomes in the round-4 release) plus an extra column labeling each fingerprint with its GCF. This BGC fingerprint matrix is the training input for the NPOmix k-nearest neighbor classifier (k = 3), which is queried later with MS/MS fingerprints built from GNPS modified cosine scores.

## Related tools

- **BiG-SCAPE** (Computes pairwise BGC similarity (1 − raw distance) and groups BGCs into GCFs; outputs are converted into the fingerprint training matrix) — https://bigscape-corason.secondarymetabolites.org
- **antiSMASH** (Produces the .gbk BGC annotation files that serve as BiG-SCAPE input for training genomes and query samples)
- **hmmer (hmmpress + Pfam-A.hmm)** (Downloads, parses, and presses the Pfam 35.0 domain database required as --pfam_dir for BiG-SCAPE runs)
- **NPOmix** (Consumes the BGC fingerprint matrix (with GCF labels) as the KNN (k = 3) training set against which MS/MS fingerprints are classified) — https://github.com/tiagolbiotech/NPOmix_python

## Examples

```
conda activate bigscape; bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig
```

## Evaluation signals

- All fingerprint similarity scores fall in [0.0, 1.0], with identical BGCs scoring 1.0 and scores below the 0.7 cutoff recorded as absent (0.0)
- The training matrix has one column per genome/metagenome in the paired dataset (1,040 columns in the round-4 release) plus one GCF label column
- BiG-SCAPE was run with --include_singletons and --mibig, and outputs from all BGC classes were concatenated before fingerprinting (concat_bigscape.sh)
- The expected release-scale matrix size is reproduced (e.g., 5,421 BGCs in 1,040 genomes/metagenomes)
- Each BGC fingerprint row is assigned to a GCF so the KNN algorithm can group fingerprint patterns belonging to the same gene cluster family

## Limitations

- Complete genomes are preferred; metagenome-assembled genomes can be used but metagenome quality can hinder BGC assembly, yielding fragmented BGCs and fewer BGCs than expected
- NPOmix is correlation-based and cannot distinguish isomers or regioselectivity of homologous BGC/metabolite pairs
- Precision (top-1 = 81.8%, top-10 = 92.9%) may not be reproducible for datasets dissimilar to the current PoDP-derived training set

## Evidence

- [readme] each biosynthetic gene cluster (BGC) in the genomes will go through a pairwise similarity comparison (Fig. 1A) to every other BGC in the same set of genomes: "By applying [BiG-SCAPE](https://bigscape-corason.secondarymetabolites.org), each biosynthetic gene cluster (BGC) in the genomes will go through a pairwise similarity comparison (Fig. 1A) to every"
- [readme] maximum similarity of the query BGC to one of the many BGCs in each genome: "we identify the maximum similarity of the query BGC to one of the many BGCs in each genome (which can be considered a pool of BGCs) in the dataset."
- [readme] similarity cutoff of 0.7 indicates BGC absence: "a score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome."
- [intro] BiG-SCAPE reliably calculates BGC pairwise similarity and GCFs: "since 2018, BiG-SCAPE (7) can reliably calculate the similarity between pairs of BGCs, grouping them into gene cluster families (GCFs)"
- [readme] BiG-SCAPE CLI invocation with pfam_dir, singletons, and mibig flags: "bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ \ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig"
