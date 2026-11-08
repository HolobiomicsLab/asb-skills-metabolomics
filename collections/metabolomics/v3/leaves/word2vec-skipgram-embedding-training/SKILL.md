---
name: word2vec-skipgram-embedding-training
description: Use when you have unlabeled Pfam domain annotations from a large genome
  corpus and need dense per-domain feature vectors (e.g., pfam2vec.csv) to feed a
  downstream detector such as the DeepBGC Pfam2VecTransformer.
when_to_use_negative:
- You only need to run DeepBGC detection on annotated genomes — reuse the released
  pfam2vec.csv instead of retraining the embedding
- Your input is raw nucleotide sequence rather than Pfam domain annotations; first
  run gene prediction (Prodigal) and domain annotation to obtain the domain 'sentences'
- You lack a sufficiently large unlabeled domain corpus; the embedding signal derives
  from a 3376-genome corpus and small corpora will not reproduce the clan-similarity
  validation
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_0736
- http://edamontology.org/topic_3474
tools:
- name: pfam2vec
  role: word2vec-like skip-gram neural network trained on Pfam domain sentences to
    produce the 100-dimensional domain embedding
- name: DeepBGC
  role: downstream consumer of the pfam2vec.csv embedding via the Pfam2VecTransformer
    vector_path config parameter for BGC detection
  repo: https://github.com/Merck/deepbgc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:423eb9dafc891b765268beace2c4b9a04fbf0e33c97be477e38093b99f4369da
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
  - DeepBGC
  - pfam2vec
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_0736
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# word2vec-skipgram-embedding-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a word2vec-like skip-gram neural network (pfam2vec) that converts Pfam domain annotation 'sentences' from thousands of genomes into 100-dimensional numeric vectors encoding genomic-context function, then validate the embedding via intra-clan cosine similarity and MIBiG t-SNE clustering.

## When to use

Use when you have unlabeled Pfam domain annotations from a large genome corpus and need dense per-domain feature vectors (e.g., pfam2vec.csv) to feed a downstream detector such as the DeepBGC Pfam2VecTransformer.

## When NOT to use

- You only need to run DeepBGC detection on annotated genomes — reuse the released pfam2vec.csv instead of retraining the embedding
- Your input is raw nucleotide sequence rather than Pfam domain annotations; first run gene prediction (Prodigal) and domain annotation to obtain the domain 'sentences'
- You lack a sufficiently large unlabeled domain corpus; the embedding signal derives from a 3376-genome corpus and small corpora will not reproduce the clan-similarity validation

## Inputs

- Unlabeled Pfam domain annotations from a ~3376-genome bacterial corpus, formatted as domain 'sentences' (ordered domain lists per genomic region)
- Pfam clan membership mapping (for intra-clan vs random-pair validation)
- MIBiG BGC database samples with subclass labels (for t-SNE validation)
- Optionally, pre-trained pfam2vec.csv from deepbgc release v0.1.0 as a substitute for retraining

## Outputs

- pfam2vec.csv embedding table (one 100-dimensional vector per Pfam domain) compatible with DeepBGC Pfam2VecTransformer vector_path
- Intra-clan vs random-pair cosine similarity distributions with significance test (P < 2.2 × 10−16 expected)
- Nearest-neighbor lists per Pfam domain with shared functional description terms
- t-SNE projection of averaged MIBiG BGC vectors colored/checked by subclass

## How to apply

Convert the Pfam domain annotations from the 3376-genome unlabeled corpus into domain 'sentences' and train a word2vec-like skip-gram neural network to yield 100-dimensional vectors per Pfam domain (pfam2vec). Validate biologically: compute cosine similarity between domain pairs from the same Pfam clan and between random domain pairs — intra-clan similarity must be significantly elevated (P < 2.2 × 10−16) — and check that nearest neighbors in the embedding space share functional description terms. Then average pfam2vec vectors over the Pfam domains of each MIBiG BGC and project with t-SNE, confirming the layout clusters BGCs by subclass. Finally, write the embedding table in the release-0.1.0 pfam2vec.csv format so it can be supplied to the DeepBGC detector via the Pfam2VecTransformer vector_path config parameter; if retraining is unnecessary, reuse the released pfam2vec.csv from deepbgc v0.1.0.

## Related tools

- **pfam2vec** (word2vec-like skip-gram neural network trained on Pfam domain sentences to produce the 100-dimensional domain embedding)
- **DeepBGC** (downstream consumer of the pfam2vec.csv embedding via the Pfam2VecTransformer vector_path config parameter for BGC detection) — https://github.com/Merck/deepbgc

## Examples

```
# train/validation carried out with Merck/bgc-pipeline (DVC-managed); to supply the embedding to DeepBGC:
deepbgc pipeline mySequence.fa --detector pfam2vec --pfam2vec path/to/pfam2vec.csv
```

## Evaluation signals

- Cosine similarity between Pfam domain vectors from the same Pfam clan is significantly higher than between random domain pairs (P < 2.2 × 10−16)
- Nearest neighbors retrieved in the embedding space share functional description terms for each Pfam domain
- t-SNE projection of averaged MIBiG BGC vectors clusters BGCs by their MIBiG subclass
- Output embedding table matches the release-0.1.0 pfam2vec.csv schema so Pfam2VecTransformer can load it via vector_path

## Limitations

- Annotation of domains of unknown function via pfam2vec was left incomplete and explicitly beyond the scope of the study
- Downstream DeepBGC models trained on existing BGC databases are heavily biased towards Streptomyces BGCs, which limits extrapolation regardless of embedding quality
- The embedding is trained from unlabeled genomic context, so vector geometry reflects genomic co-occurrence rather than direct biochemical function

## Evidence
