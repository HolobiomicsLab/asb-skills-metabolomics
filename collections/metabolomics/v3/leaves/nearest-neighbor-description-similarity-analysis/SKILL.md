---
name: nearest-neighbor-description-similarity-analysis
description: Use when a trained protein/domain embedding must be validated for functional
  coherence, e.g. checking that pfam2vec vectors place domains with related Pfam descriptions
  close together in cosine space.
when_to_use_negative:
- The embedding has not yet been trained — validate clustering (intra-clan cosine
  similarity, MIBiG t-SNE) only after pfam2vec vectors exist
- The goal is BGC detection in a genome rather than embedding validation — use the
  deepbgc pipeline instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3474
tools:
- name: pfam2vec
  role: Supplies the 100-dimensional word2vec-like skip-gram domain vectors whose
    nearest neighbors are compared for shared functional description terms
  repo: https://github.com/Merck/deepbgc
- name: DeepBGC
  role: Consumer of the validated pfam2vec.csv embedding via the Pfam2VecTransformer
    vector_path config parameter
  repo: https://github.com/Merck/deepbgc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:2ee20ad77f062f2430217d94599cb6e28dce0d6ed6fa75786ea1e953915cafe2
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# nearest-neighbor-description-similarity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a learned domain-embedding (pfam2vec) by retrieving each Pfam domain's nearest neighbors in the 100-dimensional embedding space and checking that neighbors share functional description terms, providing evidence the embedding captures biological meaning.

## When to use

Use when a trained protein/domain embedding must be validated for functional coherence, e.g. checking that pfam2vec vectors place domains with related Pfam descriptions close together in cosine space.

## When NOT to use

- The embedding has not yet been trained — validate clustering (intra-clan cosine similarity, MIBiG t-SNE) only after pfam2vec vectors exist
- The goal is BGC detection in a genome rather than embedding validation — use the deepbgc pipeline instead

## Inputs

- Trained pfam2vec embedding table (100-dimensional vectors per Pfam domain, pfam2vec.csv release-0.1.0 format)
- Pfam domain annotations (domain accession to functional description mappings) from the 3376-genome corpus
- Pfam clan membership assignments for domain pairs

## Outputs

- Per-domain nearest-neighbor lists in the embedding space with cosine similarity scores
- Shared-description-term comparison between each query domain and its neighbors
- Validation verdict on whether the embedding preserves functional relationships

## How to apply

After training pfam2vec (a word2vec-like skip-gram network producing 100-dimensional vectors per Pfam domain from domain 'sentences' built from the 3376-genome corpus), compute cosine similarity between every domain vector and all others and retrieve the top nearest neighbors for each Pfam domain. For each domain, compare the retrieved neighbors' Pfam functional description terms against the query domain's description and verify that neighbors share meaningful description terms (e.g. conserved functional keywords). Complement this with a clan-level check: cosine similarity between domain pairs from the same Pfam clan should be significantly higher than between random domain pairs (reported as P < 2.2 × 10−16). A passing validation supports using the embedding table (pfam2vec.csv, release-0.1.0 format) downstream in DeepBGC via the Pfam2VecTransformer vector_path config parameter.

## Related tools

- **pfam2vec** (Supplies the 100-dimensional word2vec-like skip-gram domain vectors whose nearest neighbors are compared for shared functional description terms) — https://github.com/Merck/deepbgc
- **DeepBGC** (Consumer of the validated pfam2vec.csv embedding via the Pfam2VecTransformer vector_path config parameter) — https://github.com/Merck/deepbgc

## Examples

```
python -c "import pandas as pd; from sklearn.neighbors import NearestNeighbors; X = pd.read_csv('pfam2vec.csv', index_col=0); nn = NearestNeighbors(metric='cosine').fit(X); print(nn.kneighbors(X, n_neighbors=6))"
```

## Evaluation signals

- Nearest neighbors of each Pfam domain share functional description terms with the query domain
- Intra-clan cosine similarity is significantly higher than random-pair similarity (paper reports P < 2.2 × 10−16)
- Embedding table is written in the release-0.1.0 pfam2vec.csv format loadable by Pfam2VecTransformer via vector_path
- Averaged MIBiG BGC vectors project with t-SNE into clusters that preserve MIBiG subclass similarity

## Limitations

- Annotation of domains of unknown function via pfam2vec was left incomplete and declared beyond the scope of the study
- Neighbors sharing description terms is a qualitative check; some domains may retrieve neighbors whose functional annotations are sparse or uninformative

## Evidence

- [results] pfam2vec captures biological signal via nearest-neighbor description sharing: [evidence span withheld — anchor status: not_found]
- [results] intra-clan similarity elevates over random pairs: [evidence span withheld — anchor status: not_found]
