---
name: glocal-bgc-similarity-scoring
description: Use when computing pairwise similarity between Biosynthetic Gene Clusters
  where some clusters are complete and others are fragmented (e.g. from metagenomes
  or pan-genomes), and global-only metrics would penalize length asymmetry.
when_to_use_negative:
- Input clusters lack Pfam domain annotations or antiSMASH 'Core Biosynthetic' gene
  flags — the LCS seed and filter steps depend on ordered domain strings and those
  annotations
- The comparison target is raw nucleotide/protein sequence identity rather than domain
  content, order, copy number and sequence identity, which is what BiG-SCAPE's pairwise
  distance is based on
- A single whole-genome similarity summary is needed, not pairwise BGC-BGC distances
  for network construction or GCF assignment
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: BiG-SCAPE
  role: Hosts the glocal alignment mode; computes pairwise BGC distances from domain
    content and builds the sequence similarity network
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: antiSMASH v4
  role: Produces the GenBank BGC inputs whose ordered Pfam domain strings and 'Core
    Biosynthetic' gene annotations feed the LCS seed and filter
  repo: https://antismash.secondarymetabolites.org
- name: HMMER 3.1b2
  role: Calls conserved domains against the user-supplied Pfam database; overlapping
    domains are filtered by hmmer score before alignment
- name: difflib.SequenceMatcher
  role: Python standard library routine used to locate the longest common subcluster
    between two BGC domain strings
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:56cdba77efc45c97ecf4c711cec457c803f66b0f0b176473026ef437ae8386bb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  - difflib.SequenceMatcher
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# glocal-bgc-similarity-scoring

## Summary

Reconstructs BiG-SCAPE's 'glocal' alignment mode, which seeds each pairwise BGC comparison with the longest common substring (LCS) of their ordered Pfam domain strings and then extends the alignment with match/mismatch/gap scores. This lets pairwise distances correctly compare complete BGCs against fragmented (or merged) ones, which purely global or local similarity metrics mishandle.

## When to use

Use when computing pairwise similarity between Biosynthetic Gene Clusters where some clusters are complete and others are fragmented (e.g. from metagenomes or pan-genomes), and global-only metrics would penalize length asymmetry.

## When NOT to use

- Input clusters lack Pfam domain annotations or antiSMASH 'Core Biosynthetic' gene flags — the LCS seed and filter steps depend on ordered domain strings and those annotations
- The comparison target is raw nucleotide/protein sequence identity rather than domain content, order, copy number and sequence identity, which is what BiG-SCAPE's pairwise distance is based on
- A single whole-genome similarity summary is needed, not pairwise BGC-BGC distances for network construction or GCF assignment

## Inputs

- antiSMASH-processed GenBank files with BGC predictions (per-gene Pfam domain order and 'Core Biosynthetic' annotations)
- user-supplied Pfam database (e.g. version 31) with HMMER 3.1b2 domain calls
- Reference BGC GenBank files from user-defined or MIBiG repositories (optional)

## Outputs

- Glocal distance contributions per BGC pair (Jaccard index, Domain Sequence Similarity, Adjacency index)
- Selected matching region (LCS seed plus extension) for each compared BGC pair
- Sequence similarity network edge list ready for cutoff filtering (e.g. empirical cutoff 0.3)

## How to apply

Parse antiSMASH GenBank inputs to extract each BGC's ordered per-gene Pfam domain string plus antiSMASH 'Core Biosynthetic' gene annotations (domains are called with hmmer against a user-supplied Pfam database, e.g. Pfam version 31, and overlapping domains are filtered by hmmer score). For each BGC pair, run difflib.SequenceMatcher on the two domain strings to locate the longest common subcluster, then apply the filter that the LCS must be either three genes long or contain at least one gene marked by antiSMASH as 'Core Biosynthetic'. Extend the surviving LCS with the scoring scheme match=5, mismatch=-3, gap=-2 to select the matching region of the two clusters, and compute the glocal contribution to the pairwise distance, which BiG-SCAPE divides into Jaccard index, Domain Sequence Similarity and Adjacency index components. Finally, validate on example domain-string pairs where one member is a truncated/fragmented variant, and confirm the module passes the repository's flake8 linting and black formatting checks.

## Related tools

- **BiG-SCAPE** (Hosts the glocal alignment mode; computes pairwise BGC distances from domain content and builds the sequence similarity network) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **antiSMASH v4** (Produces the GenBank BGC inputs whose ordered Pfam domain strings and 'Core Biosynthetic' gene annotations feed the LCS seed and filter) — https://antismash.secondarymetabolites.org
- **HMMER 3.1b2** (Calls conserved domains against the user-supplied Pfam database; overlapping domains are filtered by hmmer score before alignment)
- **difflib.SequenceMatcher** (Python standard library routine used to locate the longest common subcluster between two BGC domain strings)

## Examples

```
from difflib import SequenceMatcher; m = SequenceMatcher(None, pfam_str_a, pfam_str_b); lcs = m.find_longest_match(0, len(pfam_str_a), 0, len(pfam_str_b))
```

## Evaluation signals

- Every retained LCS is either at least three genes long or contains at least one gene annotated by antiSMASH as 'Core Biosynthetic' — count LCS candidates rejected by this filter
- Extension-stage scoring uses match=5, mismatch=-3, gap=-2 around the LCS; spot-check the selected matching region on a hand-computed example pair
- A fragmented BGC aligned against its complete counterpart scores a much higher glocal similarity than a global-alignment baseline would, matching the paper's rationale
- Output distances decompose into the three BiG-SCAPE components (Jaccard index, Domain Sequence Similarity, Adjacency index)
- Module passes the repository's flake8 linting and black formatting checks

## Limitations

- The glocal mode is motivated by fragmented clusters 'which frequently occur in metagenomes and large-scale pan-genome sequencing projects'; for uniformly complete, highly syntenic clusters the extra machinery may add little
- Overlapping domain calls are filtered by hmmer score upstream, so noisy domain predictions (wrong order or spurious domains) directly corrupt the LCS seed
- Distances only become a usable network after cutoff filtering (empirical cutoff distance of 0.3); raw glocal distances are not themselves GCF assignments
- Validation is on example BGC domain-string pairs, not a gold-standard benchmark of fragmented-vs-complete distances

## Evidence

- [readme] README: what the pairwise distance compares: [evidence span withheld — anchor status: external_doc]
