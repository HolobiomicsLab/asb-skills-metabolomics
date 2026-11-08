---
name: pfam-domain-based-feature-extraction
description: Use when a genome sequence (FASTA, e.g. RefSeq bacterial genomes) needs
  to be converted into Pfam-domain feature vectors as the input representation for
  BGC detection, pfam2vec embedding, or random forest product-class classification.
when_to_use_negative:
- Input is already a Pfam TSV or feature table (e.g. GeneSwap_Negatives.pfam.tsv)
  — reuse it directly rather than re-running gene calling and hmmscan
- Target molecules are non-ribosomal structures requiring direct metabolite detection
  rather than sequence-based BGC prediction
- The genome lacks an installed 'prodigal' binary on PATH and re-running domain extraction
  on raw sequence cannot proceed
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: DeepBGC
  role: Runs the pipeline ('deepbgc pipeline') that performs Pfam-domain feature extraction
    and applies the BiLSTM detector and random forest classifiers to the domain representation
  repo: https://github.com/Merck/deepbgc
- name: Prodigal
  role: Predicts protein-coding genes from the input genome so that Pfam domains can
    be detected per gene
  repo: https://github.com/hyattpd/Prodigal
- name: HMMER
  role: Runs hmmscan to assign Pfam domains to predicted proteins, producing the Pfam
    TSV feature table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:6b984398e5455bafa887c68e5b172316f575ea6ff74a76c66f997abccd5d46e6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Merck/deepbgc
  repo_url: https://github.com/Merck/deepbgc
  tools:
  - ClusterFinder
  - DeepBGC
  - HMMER
  - Prodigal
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pfam-domain-based-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract Pfam protein-domain features from bacterial or fungal genome sequences (Prodigal gene calling + HMMER hmmscan against the Pfam database) to produce a per-genome Pfam TSV that serves as the numerical input representation for DeepBGC's BiLSTM detector and pfam2vec/random-forest classifiers.

## When to use

Use when a genome sequence (FASTA, e.g. RefSeq bacterial genomes) needs to be converted into Pfam-domain feature vectors as the input representation for BGC detection, pfam2vec embedding, or random forest product-class classification.

## When NOT to use

- Input is already a Pfam TSV or feature table (e.g. GeneSwap_Negatives.pfam.tsv) — reuse it directly rather than re-running gene calling and hmmscan
- Target molecules are non-ribosomal structures requiring direct metabolite detection rather than sequence-based BGC prediction
- The genome lacks an installed 'prodigal' binary on PATH and re-running domain extraction on raw sequence cannot proceed

## Inputs

- Bacterial or fungal genome sequences in FASTA format (e.g. RefSeq bacterial genomes, 'mySequence.fa')
- Optionally, pre-annotated genomes with existing gene calls or Pfam domain TSVs
- Trained DeepBGC models and Pfam database fetched via 'deepbgc download'

## Outputs

- Per-genome Pfam TSV of detected protein domains (hmmscan results)
- Prodigal gene predictions (genes and proteins)
- Domain-scored BGC predictions with detector scores and predicted product class/activity from random forest classifiers

## How to apply

First run 'deepbgc download' to fetch the trained DeepBGC detector models and the Pfam database, and verify the setup with 'deepbgc info'. For each input genome, run 'deepbgc pipeline', which automatically calls Prodigal for protein-coding gene prediction (install Prodigal and place the 'prodigal' binary on the PATH) and HMMER hmmscan for Pfam domain detection when genes/domains are not already annotated. The result is a per-genome Pfam TSV of domains, which is the vocabulary used by pfam2vec word-embedding vectors and the downstream BiLSTM detector; genes/domains are labeled in or out of predicted clusters (an 'in_cluster' column of 0/1 is used in training-style TSVs). Apply the strict detector domain-score cutoff corresponding to the chosen false positive rate (e.g. 2% FPR) when aggregating domains into predicted BGCs. Precomputed Pfam TSVs can be reused (e.g. GeneSwap_Negatives.pfam.tsv or a previously trained pfam2vec.csv from the deepbgc v0.1.0 release) instead of re-running domain detection.

## Related tools

- **DeepBGC** (Runs the pipeline ('deepbgc pipeline') that performs Pfam-domain feature extraction and applies the BiLSTM detector and random forest classifiers to the domain representation) — https://github.com/Merck/deepbgc
- **Prodigal** (Predicts protein-coding genes from the input genome so that Pfam domains can be detected per gene) — https://github.com/hyattpd/Prodigal
- **HMMER** (Runs hmmscan to assign Pfam domains to predicted proteins, producing the Pfam TSV feature table)

## Examples

```
deepbgc pipeline mySequence.fa
```

## Evaluation signals

- The Pfam TSV per genome contains hmmscan-derived domain assignments for all Prodigal-predicted genes; 'deepbgc info' confirms the Pfam database and trained models are downloaded
- Reproduced per-genome BGC counts from the domain features across the 3376 RefSeq genomes match the reported values: DeepBGC 10926 (strict 2% FPR cutoff), antiSMASH 13865, ClusterFinder 62491
- Pairwise overlap fractions of predicted BGC sequence match Figure 5A: ~90% of DeepBGC predictions overlap ClusterFinder, ~39% overlap antiSMASH (35% both), and ~5% (566) are unique to DeepBGC
- Training-style Pfam TSVs contain a binary 'in_cluster' column (0 = pfam outside a BGC, 1 = inside) with no missing labels
- Detector scores fall at or above the strict domain-score cutoff corresponding to the 2% false positive rate before a gene set is called a BGC

## Limitations

- Training data are heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, limiting detection of novel BGCs in poorly characterized bacteria
- BGC class recovery is threshold-dependent: some well-studied classes were recovered regardless of threshold while others were not
- Some BGC classes were mis-identified in the S. coelicolor reference genome example, and the majority of novel-class BGCs could not be confidently assigned

## Evidence
