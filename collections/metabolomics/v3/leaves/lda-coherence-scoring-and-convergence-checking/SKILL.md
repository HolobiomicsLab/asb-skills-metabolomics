---
name: lda-coherence-scoring-and-convergence-checking
description: Use when training an LDA topic model over bag-of-words BGC (sub)Pfam
  tokenisations (e.g. iPRESTO PRESTO-TOP) and you must verify topic quality (u_mass
  coherence) and training convergence before using the motifs downstream.
when_to_use_negative:
- Gene order within the BGC is biologically important — LDA assumes a bag-of-words
  model where original gene order is not considered; use the PRESTO-STAT statistical
  method, which represents tokenised genes in their original order
- The corpus is too small to support a 1,000-topic model (e.g. the small Streptomyces/Salinispora
  dataset), where topic granularity and coherence estimates become unreliable
- You need de novo sub-cluster detection rather than quality assessment of an already-trained
  topic model
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iPRESTO
  role: Command-line tool implementing PRESTO-TOP LDA sub-cluster prediction whose
    model quality is assessed by coherence scoring and convergence monitoring
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: Gensim
  role: Multicore LDA implementation (online variational Bayes) used to train the
    1,000-topic model; provides the training log-likelihood used for convergence monitoring
- name: antiSMASH
  role: Source of the BGC predictions that are tokenised into (sub)Pfam bag-of-words
    vectors feeding the LDA model
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:f11a22142bac625eb90e1e912d1515614649ce10db7b156828baf563ce8b6ff6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gensim
  - antiSMASH
  - iPRESTO
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# LDA coherence scoring and convergence checking

## Summary

Quality-assessment procedure for the PRESTO-TOP Latent Dirichlet Allocation topic model used in iPRESTO: it computes the u_mass coherence score over the learned sub-cluster motifs and monitors log-likelihood convergence on held-out BGCs to confirm the 1,000-topic model is stable before motif filtering and validation.

## When to use

Use when training an LDA topic model over bag-of-words BGC (sub)Pfam tokenisations (e.g. iPRESTO PRESTO-TOP) and you must verify topic quality (u_mass coherence) and training convergence before using the motifs downstream.

## When NOT to use

- Gene order within the BGC is biologically important — LDA assumes a bag-of-words model where original gene order is not considered; use the PRESTO-STAT statistical method, which represents tokenised genes in their original order
- The corpus is too small to support a 1,000-topic model (e.g. the small Streptomyces/Salinispora dataset), where topic granularity and coherence estimates become unreliable
- You need de novo sub-cluster detection rather than quality assessment of an already-trained topic model

## Inputs

- Bag-of-words corpus of (sub)Pfam domain-combination frequency vectors for non-redundant antiSMASH-DB BGCs (60,028 BGCs tokenised into 10,539 domain combinations)
- Held-out (test) subset of BGC tokenisations for log-likelihood convergence monitoring
- LDA training configuration: number of topics (1,000), symmetric 1/N alpha and beta, 500 iterations, mini-batch chunk size of 5% (min 2,000)

## Outputs

- u_mass coherence score per LDA topic (sub-cluster motif)
- Log-likelihood trajectory on held-out BGCs demonstrating convergence
- 1,000 filtered sub-cluster motifs, each trimmed to a cumulative domain-combination probability sum of 0.95

## How to apply

Represent each BGC as a bag-of-words frequency vector of its (sub)Pfam domain combinations, ignoring gene order, and train with Gensim's multicore LDA implementation using online variational Bayes (PRESTO-TOP: 1,000 topics, alpha and beta symmetric 1/N, 500 iterations, mini-batch chunk size of 5% of the data with a minimum of 2,000). Hold out a subset of BGCs during training and monitor the log-likelihood on these held-out BGCs across iterations to confirm the model converges rather than overfits. After training, compute the u_mass coherence score on the learned topics as a quantitative quality metric for the sub-cluster motifs. Refine each topic by sorting its domain-combination probability vector and retaining combinations until the cumulative sum reaches 0.95 to filter out noise. Treat the model as acceptable only if it yields the expected 1,000 sub-cluster motifs and passes downstream validation (e.g. recovering 91 of 109 experimentally verified SubClusterBlast sub-clusters at the 0.6 overlap cut-off).

## Related tools

- **iPRESTO** (Command-line tool implementing PRESTO-TOP LDA sub-cluster prediction whose model quality is assessed by coherence scoring and convergence monitoring) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **Gensim** (Multicore LDA implementation (online variational Bayes) used to train the 1,000-topic model; provides the training log-likelihood used for convergence monitoring)
- **antiSMASH** (Source of the BGC predictions that are tokenised into (sub)Pfam bag-of-words vectors feeding the LDA model)

## Examples

```
from gensim.models import LdaMulticore; lda = LdaMulticore(corpus=bgc_bow, num_topics=1000, alpha=1/1000, eta=1/10539, iterations=500, chunksize=2000); print(lda.log_perplexity(heldout_bow))
```

## Evaluation signals

- u_mass coherence score computed over the learned topics indicates non-degenerate motifs
- Log-likelihood on held-out BGCs plateaus (converges) across training iterations rather than diverging
- The trained model yields exactly the configured 1,000 sub-cluster motifs
- After 0.95 cumulative-probability filtering, motif query match rates remain interpretable (gene-to-motif probability > 0.3, summed probabilities > 1.1, overlap score >= 0.15)
- Downstream validation recovers 91 of 109 (83%) experimentally verified SubClusterBlast sub-clusters at the 0.6 overlap cut-off

## Limitations

- u_mass coherence and held-out log-likelihood assess internal model quality but do not guarantee biological validity; external validation against SubClusterBlast sub-clusters is still required
- LDA's bag-of-words assumption discards gene order, so sub-clusters whose diagnostic signal depends on gene adjacency may be missed
- Sub-cluster motifs of length one are almost certainly noise and are filtered out; the 0.95 cumulative-probability trim trades completeness for noise suppression
- No version numbers for iPRESTO or antiSMASH are stated in the supplementary documentation

## Evidence

- [methods] Workflow step 5 of the PRESTO-TOP training procedure: [evidence span withheld — anchor status: not_found]
- [methods] LDA training hyperparameters for PRESTO-TOP: [evidence span withheld — anchor status: not_found]
- [methods] Motif noise filtering after topic training: [evidence span withheld — anchor status: not_found]
- [supplementary] Bag-of-words assumption of the LDA sub-cluster prediction mode: "or by LDA, which assumes a bag of words model where original gene order is not considered"
