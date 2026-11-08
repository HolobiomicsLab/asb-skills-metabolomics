---
name: longest-common-subcluster-detection
description: Use when comparing complete vs fragmented (or merged) BGCs — e.g. metagenome
  or pan-genome assemblies — where a purely global Pfam-domain alignment would misalign
  non-corresponding genes, and pairwise distances must reflect the shared subcluster.
when_to_use_negative:
- Input BGCs are already known to be complete, mutually collinear, and of similar
  length — a global domain alignment suffices and glocal extension adds unnecessary
  complexity
- The two BGCs share no common subcluster passing the ≥3-gene / ≥1 Core Biosynthetic
  filter — there is no valid anchor and the glocal distance is undefined for the pair
- Domain calls have not yet been deduplicated — overlapping domains must first be
  filtered by hmmer score or the LCS will be spurious
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: BiG-SCAPE
  role: 'Software whose ''glocal'' alignment mode is being reconstructed: computes
    pairwise BGC distances from Pfam domain content, order, copy number and sequence
    identity'
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: antiSMASH v4
  role: Produces the GenBank BGC inputs, the Pfam domain strings, and the 'Core Biosynthetic'
    gene annotations used for the LCS filter
- name: HMMER 3.1b2
  role: Performs domain detection against the user-supplied Pfam v31 database and
    provides the scores used to filter overlapping domains before LCS detection
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:c5d65a5009188f51f5e8f19cff338e17e30d99e0044cd5096bb1d944aef2f340
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# longest-common-subcluster-detection

## Summary

Reconstructs BiG-SCAPE's 'glocal' alignment mode: find the longest common subcluster (LCS) between two ordered Pfam domain strings, filter it, then extend the alignment with a match/mismatch/gap scoring scheme so complete and fragmented BGCs are compared over their truly corresponding regions.

## When to use

Use when comparing complete vs fragmented (or merged) BGCs — e.g. metagenome or pan-genome assemblies — where a purely global Pfam-domain alignment would misalign non-corresponding genes, and pairwise distances must reflect the shared subcluster.

## When NOT to use

- Input BGCs are already known to be complete, mutually collinear, and of similar length — a global domain alignment suffices and glocal extension adds unnecessary complexity
- The two BGCs share no common subcluster passing the ≥3-gene / ≥1 Core Biosynthetic filter — there is no valid anchor and the glocal distance is undefined for the pair
- Domain calls have not yet been deduplicated — overlapping domains must first be filtered by hmmer score or the LCS will be spurious

## Inputs

- antiSMASH-processed GenBank files (BGC predictions, e.g. antiSMASH v4 --minimal output)
- Ordered per-gene Pfam domain strings for each BGC (Pfam v31, HMMER 3.1b2 domain detection)
- antiSMASH 'Core Biosynthetic' gene annotations per BGC

## Outputs

- Selected matching region (glocal alignment) per BGC pair, anchored on the filtered LCS
- Glocal distance contribution per BGC pair usable in BiG-SCAPE pairwise distance matrices (Jaccard / Domain Sequence Similarity / Adjacency indices)

## How to apply

Parse antiSMASH GenBank inputs to extract, per BGC, the ordered per-gene Pfam domain string plus the 'Core Biosynthetic' gene annotations (overlapping domains having first been filtered by hmmer score). For each BGC pair, run difflib.SequenceMatcher on the two domain strings to locate the longest common subcluster. Keep the candidate LCS only if it is at least three genes long OR contains at least one antiSMASH 'Core Biosynthetic' gene — this prevents trivial short matches from driving distance. Extend the alignment around the LCS with the extension-stage scoring scheme (match=5, mismatch=-3, gap=-2) to select the matching region of the two clusters, then compute the glocal distance contribution from that region. The rationale is that comparing the corresponding regions, rather than whole strings, avoids the artifacts of purely global or purely local similarity between complete and fragmented clusters.

## Related tools

- **BiG-SCAPE** (Software whose 'glocal' alignment mode is being reconstructed: computes pairwise BGC distances from Pfam domain content, order, copy number and sequence identity) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **antiSMASH v4** (Produces the GenBank BGC inputs, the Pfam domain strings, and the 'Core Biosynthetic' gene annotations used for the LCS filter)
- **HMMER 3.1b2** (Performs domain detection against the user-supplied Pfam v31 database and provides the scores used to filter overlapping domains before LCS detection)

## Examples

```
from difflib import SequenceMatcher; m = SequenceMatcher(None, domains_a, domains_b); lcs = max(m.get_matching_blocks(), key=lambda b: b.size)
```

## Evaluation signals

- The retained LCS satisfies the article's filter: it is either at least three genes long or contains at least one gene antiSMASH marked 'Core Biosynthetic'
- Extension-stage scoring around the LCS uses exactly the documented weights: match=5, mismatch=-3, gap=-2
- The resulting glocal distance contribution falls in the valid range and matches BiG-SCAPE's output when run on the same BGC pair
- Domain strings were parsed in gene order from the antiSMASH GenBank, with overlapping domains removed by hmmer score before LCS detection
- Module passes the repository's flake8 linting and black formatting checks, and a validated set of example BGC domain-string pairs reproduces expected alignments

## Limitations

- Candidate common regions shorter than three genes and lacking a Core Biosynthetic gene are discarded, so glocal comparisons of very short shared cassettes are not possible
- Spurious LCS matches will result if overlapping Pfam domains are not first filtered by hmmer score
- The current methods for grouping BGCs (the motivation for glocal mode) 'fail to correctly measure the similarity between complete and fragmented gene clusters', so distances from purely global or local modes should not be substituted when fragment edges matter

## Evidence

- [readme] calculating pairwise distances between gene clusters based on a comparison of their protein domain content, order, copy number and sequence identity.: [evidence span withheld — anchor status: external_doc]
