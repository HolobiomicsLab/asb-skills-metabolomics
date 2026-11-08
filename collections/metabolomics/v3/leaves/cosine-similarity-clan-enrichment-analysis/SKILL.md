---
name: cosine-similarity-clan-enrichment-analysis
description: Use when a word2vec-style embedding of Pfam domain annotations has been
  trained from genomic context and you must demonstrate the vectors capture biological/functional
  meaning before feeding them to a downstream detector.
when_to_use_negative:
- Input is raw nucleotide sequence or protein FASTA rather than already-annotated
  Pfam domains — run Prodigal and Pfam annotation first
- You only need BGC detection/classification and can reuse the release-0.1.0 pfam2vec.csv
  directly instead of re-validating the embedding
- No clan membership or functional description data exists to serve as an independent
  validation signal for the embedding space
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: pfam2vec
  role: word2vec-like skip-gram neural network producing the domain vectors whose
    cosine similarities are validated against Pfam clan relationships
  repo: https://github.com/Merck/bgc-pipeline
- name: DeepBGC
  role: downstream consumer of the validated embedding table via the Pfam2VecTransformer
    vector_path config parameter
  repo: https://github.com/Merck/deepbgc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:f22f65674447668667b1e9684169a76e290233fafb80c28285ab33ef875ccfbf
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cosine-similarity-clan-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that pfam2vec domain embeddings encode functional signal by testing whether cosine similarity between domain vectors from the same Pfam clan exceeds that of random domain-pairs, then sanity-checks neighbors and t-SNE layouts of averaged MIBiG BGC vectors.

## When to use

Use when a word2vec-style embedding of Pfam domain annotations has been trained from genomic context and you must demonstrate the vectors capture biological/functional meaning before feeding them to a downstream detector.

## When NOT to use

- Input is raw nucleotide sequence or protein FASTA rather than already-annotated Pfam domains — run Prodigal and Pfam annotation first
- You only need BGC detection/classification and can reuse the release-0.1.0 pfam2vec.csv directly instead of re-validating the embedding
- No clan membership or functional description data exists to serve as an independent validation signal for the embedding space

## Inputs

- Unlabeled Pfam domain annotations from a bacterial genome corpus (e.g. 3376 genomes), organized as domain 'sentences'
- Pfam clan (superfamily) membership mappings for domains
- MIBiG BGC Pfam domain compositions with subclass labels
- Previously trained pfam2vec.csv embedding table (github.com/Merck/deepbgc releases/tag/v0.1.0)

## Outputs

- 100-dimensional pfam2vec vector per Pfam domain, written as pfam2vec.csv
- Intra-clan vs random-pair cosine similarity distributions with statistical comparison (P < 2.2 × 10−16 expected)
- Nearest-neighbor lists per Pfam domain with shared description-term evidence
- t-SNE projection of averaged MIBiG BGC vectors colored/clustered by subclass

## How to apply

Train a 100-dimensional word2vec-like skip-gram model (pfam2vec) over domain 'sentences' derived from unlabeled Pfam annotations of a large genome corpus (the paper used 3376 genomes). Compute cosine similarity between pairs of domain vectors that belong to the same Pfam clan (superfamily) and between random domain-vector pairs, and statistically compare the two distributions; in the paper intra-clan similarity was significantly higher (P < 2.2 × 10−16). As a second check, retrieve nearest neighbors in embedding space for sampled Pfam domains and confirm neighbors share functional description terms. Finally, average pfam2vec vectors over the Pfam domains of each MIBiG BGC and project the averaged vectors with t-SNE, confirming the 2D layout clusters by BGC subclass. Output the embedding table in the release-0.1.0 pfam2vec.csv format so it can be passed to DeepBGC via the Pfam2VecTransformer vector_path config parameter.

## Related tools

- **pfam2vec** (word2vec-like skip-gram neural network producing the domain vectors whose cosine similarities are validated against Pfam clan relationships) — https://github.com/Merck/bgc-pipeline
- **DeepBGC** (downstream consumer of the validated embedding table via the Pfam2VecTransformer vector_path config parameter) — https://github.com/Merck/deepbgc

## Examples

```
python -c "from scipy.spatial.inp; import numpy as np" 2>/dev/null; python -c "from scipy.spatial.distance import cdist; import numpy as np; S = 1 - cdist(np.vstack([clan_vecs, rand_vecs]), np.vstack([clan_vecs, rand_vecs]), metric='cosine')"
```

## Evaluation signals

- Mean cosine similarity of same-clan domain vector pairs is significantly greater than random-pair similarity (paper: P < 2.2 × 10−16)
- Nearest neighbors of sampled Pfam domains in embedding space share functional description terms
- t-SNE projection of averaged MIBiG BGC vectors clusters by BGC subclass
- Embedding table matches the release-0.1.0 pfam2vec.csv schema (100-dimensional vectors keyed by Pfam domain) and loads via Pfam2VecTransformer vector_path

## Limitations

- Annotation of domains of unknown function via pfam2vec was left incomplete — beyond the scope of the study
- Training data bias: BGC databases were heavily biased towards Streptomyces 'workhorses', limiting generalization to poorly characterized bacteria
- The majority of novel BGC classes identified by the downstream model could not be confidently assigned

## Evidence
