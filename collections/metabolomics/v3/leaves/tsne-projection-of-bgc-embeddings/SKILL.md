---
name: tsne-projection-of-bgc-embeddings
description: Use when you have pfam2vec (or similar domain) embeddings and need to
  visually/quantitatively validate that BGCs of the same product subclass cluster
  together in the learned vector space.
when_to_use_negative:
- You want a functional-activity or product-class prediction of BGCs — the DeepBGC
  random forest classifiers, not t-SNE, are used for that.
- You only need BGC detection in a genome sequence — run `deepbgc pipeline` instead;
  t-SNE projection is an embedding-validation step, not a detector.
- You need statistically rigorous distance comparisons — the intra-clan vs random-pair
  cosine similarity test is the quantitative validation; t-SNE layouts are qualitative
  and distances are not globally meaningful.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_0736
- http://edamontology.org/topic_3473
tools:
- name: pfam2vec
  role: word2vec-like skip-gram network supplying the 100-dimensional Pfam domain
    vectors that are averaged per BGC before t-SNE projection
  repo: https://github.com/Merck/deepbgc
- name: DeepBGC
  role: consumer of the released pfam2vec.csv embeddings via Pfam2VecTransformer;
    the t-SNE validation precedes packaging the embedding table
  repo: https://github.com/Merck/deepbgc
- name: Merck/bgc-pipeline
  role: repository holding the reproduction code and MIBiG sample and Pfam2vec feature
    data (data/features, data/mibig) used for the embedding validation
  repo: https://github.com/Merck/bgc-pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:bf7e339718a59743f8ee992550013dcf038c1af88d638309902dd14b6758e2db
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
  - Merck/bgc-pipeline
  - pfam2vec
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_0736
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# t-SNE projection of BGC embeddings

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Projects averaged pfam2vec vectors, computed per MIBiG biosynthetic gene cluster from its Pfam domain composition, into a 2D t-SNE layout to verify that the embedding space preserves BGC subclass similarity.

## When to use

Use when you have pfam2vec (or similar domain) embeddings and need to visually/quantitatively validate that BGCs of the same product subclass cluster together in the learned vector space.

## When NOT to use

- You want a functional-activity or product-class prediction of BGCs — the DeepBGC random forest classifiers, not t-SNE, are used for that.
- You only need BGC detection in a genome sequence — run `deepbgc pipeline` instead; t-SNE projection is an embedding-validation step, not a detector.
- You need statistically rigorous distance comparisons — the intra-clan vs random-pair cosine similarity test is the quantitative validation; t-SNE layouts are qualitative and distances are not globally meaningful.

## Inputs

- pfam2vec 100-dimensional embedding vectors per Pfam domain (pfam2vec.csv format)
- MIBiG BGC database samples with Pfam domain annotations (data/mibig in bgc-pipeline)
- MIBiG subclass labels for each BGC

## Outputs

- Averaged 100-dimensional pfam2vec vector per MIBiG BGC
- 2D t-SNE projection of BGC vectors colored/inspected by MIBiG subclass
- Validation report that t-SNE layout clusters BGCs by subclass

## How to apply

Train pfam2vec (a word2vec-like skip-gram neural network on Pfam domain 'sentences' from a 3376-genome corpus) to obtain 100-dimensional vectors per Pfam domain. For each MIBiG BGC, average the pfam2vec vectors of all its Pfam domains to produce one vector per BGC. Project these averaged BGC vectors to 2D with t-SNE and check that the layout preserves MIBiG subclass similarity, i.e. BGCs of the same subclass appear as clusters. This serves as the embedding-validation step of the DeepBGC workflow before the vectors are packaged as pfam2vec.csv (release 0.1.0) for use by the DeepBGC detector via Pfam2VecTransformer vector_path config. Complementary validations include intra-clan cosine similarity exceeding random-pair similarity (P < 2.2 × 10−16) and nearest neighbors sharing functional description terms.

## Related tools

- **pfam2vec** (word2vec-like skip-gram network supplying the 100-dimensional Pfam domain vectors that are averaged per BGC before t-SNE projection) — https://github.com/Merck/deepbgc
- **DeepBGC** (consumer of the released pfam2vec.csv embeddings via Pfam2VecTransformer; the t-SNE validation precedes packaging the embedding table) — https://github.com/Merck/deepbgc
- **Merck/bgc-pipeline** (repository holding the reproduction code and MIBiG sample and Pfam2vec feature data (data/features, data/mibig) used for the embedding validation) — https://github.com/Merck/bgc-pipeline

## Examples

```
dvc pull data/features/pfam2vec.dvc && dvc pull data/mibig  # then average pfam2vec vectors per MIBiG BGC and project with t-SNE for subclass clustering check
```

## Evaluation signals

- Intra-clan cosine similarity between domain vectors from the same Pfam clan must be significantly higher than between random domain pairs (P < 2.2 × 10−16).
- Nearest neighbors of each Pfam domain in the embedding space must share functional description terms.
- The t-SNE projection of averaged MIBiG BGC vectors must visibly cluster BGCs by their subclass.
- The resulting embedding table must be writable in the release-0.1.0 pfam2vec.csv format accepted by DeepBGC's Pfam2VecTransformer vector_path parameter.

## Limitations

- Annotation of domains of unknown function via pfam2vec was left incomplete — beyond the study's scope.
- BGC training databases (including MIBiG) are heavily biased towards natural-product 'workhorses' such as Streptomyces, limiting subclass diversity in the projection.
- t-SNE layout is a qualitative check; some MIBiG BGC classes could not be confidently assigned, so not all subclasses will separate cleanly.

## Evidence

- [results] pfam2vec captures biological signal: cosine similarity between domain vectors from the same Pfam superfamily (clan) was significantly higher than between random domain vector pairs (P < 2.2 × 10−16): [evidence span withheld — anchor status: not_found]
- [readme] bgc-pipeline contains Pfam2vec features and MIBiG samples: [evidence span withheld — anchor status: external_doc]
