---
name: pfam-domain-corpus-preparation
description: Use when you have unlabeled Pfam domain annotations from many genomes
  and need genomic-context domain embeddings (pfam2vec.csv) to feed DeepBGC or evaluate
  embedding biological signal.
when_to_use_negative:
- A trained pfam2vec.csv already exists — supply it directly via the Pfam2VecTransformer
  vector_path config parameter instead of retraining the corpus
- The corpus is too small (few genomes) to provide meaningful unlabeled genomic context
  for skip-gram training
- The goal is per-domain sequence/function annotation rather than genomic-context
  embeddings — pfam2vec does not resolve domains of unknown function (left beyond
  the study's scope)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pfam2vec
  role: word2vec-like skip-gram neural network trained on the domain 'sentences' to
    produce 100-dimensional Pfam domain vectors
  repo: https://github.com/Merck/bgc-pipeline
- name: DeepBGC
  role: consumer of the pfam2vec.csv embedding via the Pfam2VecTransformer vector_path
    config parameter; provides `deepbgc download` for Pfam database dependencies
  repo: https://github.com/Merck/deepbgc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:0e518ea2cb2294fa307831b7d94b8f89a0f39a683e67446f225a2e0dd65da5db
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Merck/deepbgc
  repo_url: https://github.com/Merck/bgc-pipeline
  tools:
  - DeepBGC
  - pfam2vec
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# pfam-domain-corpus-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts unlabeled Pfam domain annotations from a large multi-genome corpus (3376 bacterial genomes) into domain 'sentences' and trains a word2vec-like skip-gram network (pfam2vec) yielding 100-dimensional vectors per Pfam domain, so that domain numeric representations reflect genomic context rather than sequence homology.

## When to use

Use when you have unlabeled Pfam domain annotations from many genomes and need genomic-context domain embeddings (pfam2vec.csv) to feed DeepBGC or evaluate embedding biological signal.

## When NOT to use

- A trained pfam2vec.csv already exists — supply it directly via the Pfam2VecTransformer vector_path config parameter instead of retraining the corpus
- The corpus is too small (few genomes) to provide meaningful unlabeled genomic context for skip-gram training
- The goal is per-domain sequence/function annotation rather than genomic-context embeddings — pfam2vec does not resolve domains of unknown function (left beyond the study's scope)

## Inputs

- Unlabeled Pfam domain annotation TSVs (Pfam TSV format) from a 3376-genome bacterial corpus
- Pfam database and Pfam clan definitions (downloadable via `deepbgc download`)
- MIBiG BGC database samples (for t-SNE subclass validation)
- Previously trained pfam2vec.csv from deepbgc release v0.1.0 (optional reuse instead of retraining)

## Outputs

- Domain 'sentences' corpus (ordered per-genome Pfam domain sequences)
- pfam2vec.csv: 100-dimensional embedding table, one vector per Pfam domain (release-0.1.0 format)
- Validation artifacts: intra-clan vs random-pair cosine similarity statistics, nearest-neighbor description-term check, t-SNE projection of averaged MIBiG BGC vectors colored by subclass

## How to apply

Treat each genome's ordered Pfam domain annotation list as a 'sentence' and train a word2vec-like skip-gram neural network (pfam2vec) over the 3376-genome unlabeled corpus to obtain 100-dimensional vectors per Pfam domain. Download required Pfam database dependencies first (e.g. via `deepbgc download`). Validate the embedding before reuse: cosine similarity between domain vectors from the same Pfam clan must be significantly higher than between random domain pairs (P < 2.2 × 10−16), nearest neighbors must share functional description terms, and t-SNE projection of Pfam-domain-averaged MIBiG BGC vectors must cluster by BGC subclass. Write the embedding table in the release-0.1.0 pfam2vec.csv format so it can be supplied to the DeepBGC detector via the Pfam2VecTransformer vector_path config parameter. Alternatively reuse the pre-trained pfam2vec.csv from the deepbgc v0.1.0 release rather than retraining.

## Related tools

- **pfam2vec** (word2vec-like skip-gram neural network trained on the domain 'sentences' to produce 100-dimensional Pfam domain vectors) — https://github.com/Merck/bgc-pipeline
- **DeepBGC** (consumer of the pfam2vec.csv embedding via the Pfam2VecTransformer vector_path config parameter; provides `deepbgc download` for Pfam database dependencies) — https://github.com/Merck/deepbgc

## Examples

```
deepbgc download
```

## Evaluation signals

- Cosine similarity between domain vectors from the same Pfam clan is significantly higher than between random domain vector pairs (P < 2.2 × 10−16)
- Nearest neighbors of each Pfam domain in the embedding space share functional description terms
- t-SNE projection of Pfam-domain-averaged MIBiG BGC vectors clusters by MIBiG subclass
- Output table parses as release-0.1.0 pfam2vec.csv format and loads through Pfam2VecTransformer vector_path
- Each Pfam domain receives a 100-dimensional vector

## Limitations

- Domains of unknown function were not annotated via pfam2vec — this was beyond the study's scope
- Downstream BGC models trained on pfam2vec features inherit bias from BGC databases heavily skewed towards Streptomyces
- Reproducing the full corpus and embeddings requires DVC-managed data files from Merck/bgc-pipeline (AWS credentials, `dvc pull data/path/to/file.dvc`)

## Evidence
