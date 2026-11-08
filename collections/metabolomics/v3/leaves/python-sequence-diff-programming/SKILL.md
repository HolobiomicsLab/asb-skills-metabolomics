---
name: python-sequence-diff-programming
description: Use when comparing complete vs fragmented (or merged) BGCs via their
  ordered Pfam domain strings and you need pairwise distances that avoid artifacts
  of purely global or purely local similarity.
when_to_use_negative:
- Inputs are raw nucleotide/amino-acid sequences rather than ordered Pfam domain strings
  — BiG-SCAPE distances are built from protein domain content, order and copy number,
  not raw sequence
- A candidate LCS fails the filter (shorter than three genes and no 'Core Biosynthetic'
  gene) — no glocal matching region exists for that pair
- You only need the already-computed Jaccard/Domain Sequence Similarity/Adjacency
  distance values rather than the underlying alignment reconstruction
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: BiG-SCAPE
  role: Reference implementation whose 'glocal' alignment mode (LCS seed + extension
    scoring) this Python module reconstructs for pairwise BGC distance calculation
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: antiSMASH v4
  role: Source of the GenBank BGC predictions, Pfam domain strings and 'Core Biosynthetic'
    gene annotations consumed by the module
- name: difflib.SequenceMatcher
  role: Python standard-library sequence-diff engine used to locate the longest common
    subcluster between two BGC domain strings
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:95fc8c295de6cdc204c296cb6ed583d8489dea93c5d148c62d640af5d2428e08
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - antiSMASH v4
  - difflib.SequenceMatcher
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# python-sequence-diff-programming

## Summary

Reconstructs BiG-SCAPE's 'glocal' BGC alignment mode in Python: the longest common subcluster between two Pfam domain strings is located with difflib.SequenceMatcher and then extended with match/mismatch/gap penalties, so complete and fragmented gene clusters are compared over corresponding regions only.

## When to use

Use when comparing complete vs fragmented (or merged) BGCs via their ordered Pfam domain strings and you need pairwise distances that avoid artifacts of purely global or purely local similarity.

## When NOT to use

- Inputs are raw nucleotide/amino-acid sequences rather than ordered Pfam domain strings — BiG-SCAPE distances are built from protein domain content, order and copy number, not raw sequence
- A candidate LCS fails the filter (shorter than three genes and no 'Core Biosynthetic' gene) — no glocal matching region exists for that pair
- You only need the already-computed Jaccard/Domain Sequence Similarity/Adjacency distance values rather than the underlying alignment reconstruction

## Inputs

- antiSMASH-processed GenBank files (BGC predictions) with per-gene Pfam domain annotations
- antiSMASH 'Core Biosynthetic' gene annotations per BGC
- Ordered Pfam domain strings for each BGC pair (from a user-supplied external Pfam database, e.g. version 31 with HMMER 3.1b2)

## Outputs

- Selected matching region (subcluster span) per BGC pair after LCS filtering and extension scoring
- Glocal distance contribution per BGC pair, usable alongside Jaccard, Domain Sequence Similarity and Adjacency indices
- Validated Python module (flake8- and black-clean) with example domain-string pair tests

## How to apply

Parse antiSMASH GenBank inputs to extract each BGC's ordered per-gene Pfam domain string plus the antiSMASH 'Core Biosynthetic' gene annotations (overlapping domains should already be filtered by hmmer score). For each BGC pair, run difflib.SequenceMatcher on the two domain strings to locate the longest common subcluster (LCS). Filter candidate subclusters, keeping only those that are at least three genes long OR contain at least one gene marked 'Core Biosynthetic'. Around the selected LCS, run the extension-stage scoring scheme (match=5, mismatch=-3, gap=-2) to choose the matching region of the two clusters, then compute the glocal distance contribution from that region. Validate the module on example BGC domain-string pairs and confirm it passes the repository's flake8 linting and black formatting checks.

## Related tools

- **BiG-SCAPE** (Reference implementation whose 'glocal' alignment mode (LCS seed + extension scoring) this Python module reconstructs for pairwise BGC distance calculation) — https://github.com/medema-group/BiG-SCAPE/wiki
- **antiSMASH v4** (Source of the GenBank BGC predictions, Pfam domain strings and 'Core Biosynthetic' gene annotations consumed by the module)
- **difflib.SequenceMatcher** (Python standard-library sequence-diff engine used to locate the longest common subcluster between two BGC domain strings)

## Examples

```
from difflib import SequenceMatcher; m = SequenceMatcher(None, domains_a, domains_b); lcs = max(m.get_matching_blocks(), key=lambda b: b.size)
```

## Evaluation signals

- Every retained matching region is either at least three genes long or contains at least one antiSMASH 'Core Biosynthetic' gene
- Extension stage applies exactly match=5, mismatch=-3, gap=-2 around the LCS-selected region
- Module passes the repository's flake8 linting and black formatting checks
- Fragmented-vs-complete BGC pairs yield different (non-artifactual) glocal distance contributions than a purely global Jaccard comparison would
- SequenceMatcher matching blocks agree with the expected longest common subcluster on curated example domain-string pairs

## Limitations

- LCS candidates that are neither three genes long nor contain a Core Biosynthetic gene are discarded, so very short shared subclusters contribute no glocal match
- Distances depend on the Pfam version used for domain detection (this study used version 31 with HMMER 3.1b2) and on hmmer-score-based filtering of overlapping domains
- The glocal mode is motivated by fragmented/merged clusters from metagenomes and large-scale pan-genome sequencing; for complete, near-identical clusters global indices may suffice

## Evidence

- [readme] calculating pairwise distances between gene clusters based on a comparison of their protein domain content, order, copy number and sequence identity: [evidence span withheld — anchor status: external_doc]
