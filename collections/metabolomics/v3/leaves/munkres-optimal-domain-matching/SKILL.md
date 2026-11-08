---
name: munkres-optimal-domain-matching
description: Use when computing pairwise distances between biosynthetic gene clusters
  with differing numbers of Pfam domains per type, and you need an optimal, non-quadratic
  one-to-one domain assignment to score sequence similarity and copy-number differences.
when_to_use_negative:
- When the goal is full evolutionary reconstruction of a gene cluster family — CORASON's
  multi-locus core phylogeny, not Munkres matching, is the appropriate mechanism
- When comparing clusters with largely non-overlapping domain-type inventories — with
  few or no shared domain types the assignment is uninformative and the Jaccard index
  alone will dominate the distance
- When BGCs are not closely related enough for hmmalign-based profile alignments to
  be informative; alternative alignment backends (Muscle, MAFFT via --mix) should
  be benchmarked instead
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_0797
tools:
- name: BiG-SCAPE
  role: Implements the Munkres-based one-to-one domain matching and combines the resulting
    DSS index with the Jaccard and Adjacency indices into the pairwise BGC distance;
    provides --mix to benchmark Muscle/MAFFT variants
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: hmmalign
  role: Aligns every predicted domain sequence to its corresponding Pfam profile HMM
    so domain identity scores are computed from profile-based alignments rather than
    all-versus-all sequence comparison
- name: HMMER 3.1b2
  role: Provides hmmalign and the hmmer scores used to filter overlapping domains
    before matching; used with Pfam version 31
- name: antiSMASH v4
  role: Produces the --minimal GenBank files from which predicted BGC domain sequences
    are extracted as matching input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:75d0dd9446723c1bf0ac30fee64f2ed1a12e71dc46fbb8b4817c02c09c432cd2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  - hmmalign
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# munkres-optimal-domain-matching

## Summary

A pairwise BGC comparison strategy that uses the Munkres (Hungarian) assignment algorithm to find the optimal one-to-one matching between domains of the same Pfam type in two gene clusters, so that copy-number differences and domain sequence identity (Domain Sequence Similarity, DSS) can be scored efficiently instead of relying on slow all-versus-all sequence alignment.

## When to use

Use when computing pairwise distances between biosynthetic gene clusters with differing numbers of Pfam domains per type, and you need an optimal, non-quadratic one-to-one domain assignment to score sequence similarity and copy-number differences.

## When NOT to use

- When the goal is full evolutionary reconstruction of a gene cluster family — CORASON's multi-locus core phylogeny, not Munkres matching, is the appropriate mechanism
- When comparing clusters with largely non-overlapping domain-type inventories — with few or no shared domain types the assignment is uninformative and the Jaccard index alone will dominate the distance
- When BGCs are not closely related enough for hmmalign-based profile alignments to be informative; alternative alignment backends (Muscle, MAFFT via --mix) should be benchmarked instead

## Inputs

- antiSMASH v4 (--minimal) GenBank output files containing predicted BGC domain sequences
- Pfam database version 31 profile HMMs
- hmmalign (HMMER 3.1b2) per-domain-type alignments of predicted domain sequences

## Outputs

- Optimal one-to-one assignment of same-type Pfam domains for each BGC pair
- Domain Sequence Similarity (DSS) index per BGC pair capturing copy-number differences and aligned domain sequence identity
- Pairwise BGC distance values combining Jaccard, DSS, and Adjacency indices, feeding the sequence similarity network and GCF calling

## How to apply

First extract the predicted domain sequences from antiSMASH v4 (--minimal) GenBank files, aligning every domain of each Pfam type to its corresponding profile HMM with hmmalign (HMMER 3.1b2, Pfam 31) — this profile-based alignment is what makes the approach fast at scale. For each pair of BGCs, restrict to domains shared by type and run the Munkres (Hungarian) assignment algorithm to obtain the optimal one-to-one matching between domains of the same type. Score the matched pairs on copy-number differences and sequence identity of the aligned domains, then combine the DSS index with the Jaccard and Adjacency indices into BiG-SCAPE's pairwise distance. Overlapping domains should already have been filtered by hmmer score before matching, and gene clusters with empty aligned domains should be excluded from the pair. As validation, repeat GCF calling with Muscle- and MAFFT-based alignment variants (BiG-SCAPE's --mix parameter) and confirm high agreement in GCF calling, with hmmalign showing shorter runtimes as input BGC count increases.

## Related tools

- **BiG-SCAPE** (Implements the Munkres-based one-to-one domain matching and combines the resulting DSS index with the Jaccard and Adjacency indices into the pairwise BGC distance; provides --mix to benchmark Muscle/MAFFT variants) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **hmmalign** (Aligns every predicted domain sequence to its corresponding Pfam profile HMM so domain identity scores are computed from profile-based alignments rather than all-versus-all sequence comparison)
- **HMMER 3.1b2** (Provides hmmalign and the hmmer scores used to filter overlapping domains before matching; used with Pfam version 31)
- **antiSMASH v4** (Produces the --minimal GenBank files from which predicted BGC domain sequences are extracted as matching input)

## Examples

```
python bigscape.py -i antismash_gbk/ -o bigscape_out/ --pfam-dir pfam31 --mix --cutoffs 0.3,0.75
```

## Evaluation signals

- GCF calling from the hmmalign-based DSS shows high agreement with GCF calling using Muscle- and MAFFT-based alignment variants under BiG-SCAPE's --mix parameter
- hmmalign runtimes remain shorter than Muscle/MAFFT variants as the number of input BGCs increases (no quadratic blow-up in runtime)
- Every matched domain pair in a BGC pair is one-to-one and restricted to domains of the same Pfam type; no domain is matched twice
- Copy-number differences are reflected in the DSS index (clusters with duplicated domain types score less similar), and combined distances stay within the network-filtering cutoff space (e.g. the empirical 0.3 GCF cutoff)
- Overlapping domains were filtered by hmmer score before matching, so no pair scores on overlapping duplicate domain calls

## Limitations

- The efficiency gain depends on profile-based (hmmalign) pre-alignment; benchmarking must confirm agreement with Muscle/MAFFT since profile alignments may lose information relative to full multiple sequence alignment
- GCF network filtering is sensitive to the distance cutoff (the article found 0.75 maximized nodes while preserving network structural integrity, and an empirical 0.3 cutoff is used for GCF assignment), so cutoff choice affects downstream families
- Interactive BiG-SCAPE reports can silently drop genomes when ORGANISM identifiers are duplicated across assembly projects (only 96 of 103 Streptomyces genomes were reported), which distorts pairwise comparisons if unnoticed

## Evidence

- [methods] Munkres assignment rationale and DSS scoring: [evidence span withheld — anchor status: not_found]
