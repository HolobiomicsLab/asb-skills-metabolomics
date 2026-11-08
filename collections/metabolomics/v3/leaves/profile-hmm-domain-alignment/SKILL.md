---
name: profile-hmm-domain-alignment
description: Use when computing BiG-SCAPE's Domain Sequence Similarity (DSS) index,
  or when many antiSMASH-predicted domain sequences must be aligned per Pfam domain
  type against profile HMMs instead of running all-versus-all sequence alignment between
  BGC pairs.
when_to_use_negative:
- Goal is evolutionary relationships between clusters (multi-locus phylogenies, genomic
  context) — that is CORASON's phylogenomic approach, not the profile-HMM DSS alignment
- Input is raw genome contigs without antiSMASH domain predictions — there are no
  domain sequences to align; run antiSMASH v4 first
- Pairwise BGC distances are already computed and only downstream GCF/network clustering
  is needed
edam_operation: http://edamontology.org/operation_0292
tools:
- name: hmmalign
  role: Aligns the sequences of every predicted domain type to its corresponding Pfam
    profile HMM, producing the alignments underlying the DSS index
- name: HMMER 3.1b2
  role: HMMER suite version used with Pfam version 31 for domain search and hmmalign
    profile alignment
- name: BiG-SCAPE
  role: Consumes the profile-HMM alignments to compute the Domain Sequence Similarity
    index within pairwise BGC distances (with Jaccard and Adjacency indices) using
    Munkres assignment
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: antiSMASH v4
  role: Generates the --minimal GenBank output from which predicted domain sequences
    are extracted prior to hmmalign alignment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:8cd526279a99efad83913a375a10ad0817191d317446a513914b14eae2e54af2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  - hmmalign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# profile-hmm-domain-alignment

## Summary

Align antiSMASH-predicted BGC domain sequences to their Pfam profile HMMs using hmmalign (HMMER 3.1b2) so that BiG-SCAPE can compute its Domain Sequence Similarity (DSS) index — copy-number differences plus sequence identity — without quadratic all-versus-all sequence comparison.

## When to use

Use when computing BiG-SCAPE's Domain Sequence Similarity (DSS) index, or when many antiSMASH-predicted domain sequences must be aligned per Pfam domain type against profile HMMs instead of running all-versus-all sequence alignment between BGC pairs.

## When NOT to use

- Goal is evolutionary relationships between clusters (multi-locus phylogenies, genomic context) — that is CORASON's phylogenomic approach, not the profile-HMM DSS alignment
- Input is raw genome contigs without antiSMASH domain predictions — there are no domain sequences to align; run antiSMASH v4 first
- Pairwise BGC distances are already computed and only downstream GCF/network clustering is needed

## Inputs

- antiSMASH v4 (--minimal) GenBank output files containing BGC predictions
- Predicted domain amino-acid sequences per BGC, grouped by domain type
- Pfam database version 31 profile HMM models (used with HMMER 3.1b2)

## Outputs

- Per-domain-type multiple alignments of domain sequences against their Pfam profile HMMs
- Per-BGC-pair DSS index scores (copy-number differences + aligned-domain sequence identity)
- Combined pairwise distance matrix feeding BiG-SCAPE GCF network construction and Affinity Propagation clustering

## How to apply

Load antiSMASH v4 (--minimal) GenBank output and extract the predicted domain sequence for every domain of every BGC. Align the sequences of every predicted domain type to its corresponding Pfam (version 31) profile HMM with hmmalign from HMMER 3.1b2; filter overlapping domains based on the score calculated by hmmer before or during this step. In BiG-SCAPE, per-BGC-pair the alignments are consumed via the Munkres (Hungarian) assignment algorithm to find the optimal one-to-one matching between domains of the same type; matched pairs are scored on copy-number differences and aligned-sequence identity and combined with the Jaccard and Adjacency indices into the pairwise distance (GCF network filtered at cutoff 0.3). Rationale: profile-based alignment linearizes the comparison cost relative to BGC count, which the article validated by showing hmmalign yields GCF calling in high agreement with Muscle and MAFFT variants (--mix) while having shorter runtimes as input BGC count increases.

## Related tools

- **hmmalign** (Aligns the sequences of every predicted domain type to its corresponding Pfam profile HMM, producing the alignments underlying the DSS index)
- **HMMER 3.1b2** (HMMER suite version used with Pfam version 31 for domain search and hmmalign profile alignment)
- **BiG-SCAPE** (Consumes the profile-HMM alignments to compute the Domain Sequence Similarity index within pairwise BGC distances (with Jaccard and Adjacency indices) using Munkres assignment) — https://github.com/medema-group/BiG-SCAPE/wiki
- **antiSMASH v4** (Generates the --minimal GenBank output from which predicted domain sequences are extracted prior to hmmalign alignment)

## Evaluation signals

- Every predicted domain type has an alignment against its own Pfam v31 model; overlapping domains are removed based on the hmmer score before scoring
- Repeated GCF calling with the --mix Muscle and MAFFT alignment variants shows high agreement with the hmmalign-based GCF calls
- Runtime scaling check: hmmalign runtimes stay shorter than Muscle/MAFFT variants as the number of input BGCs increases
- Each BiG-SCAPE pairwise distance decomposes into the expected three components: Jaccard index, Domain Sequence Similarity index, and Adjacency index
- GCF network is filtered at the empirical cutoff distance of 0.3, and cutoff 0.75 maximizes node count while minimizing damage to network structural integrity in cutoff scans

## Limitations

- Validated specifically with Pfam version 31 and HMMER 3.1b2 on antiSMASH v4 (--minimal) output; other domain-model or predictor versions were not benchmarked for agreement
- The benchmark covers final GCF calling and runtime scaling; per-pair distance differences between hmmalign and Muscle/MAFFT variants were not reported as identical, only highly concordant
- Requires well-formed antiSMASH domain annotations; BiG-SCAPE's interactive report can silently drop scaffolds whose ORGANISM identifiers duplicate across assembly projects (96 of 103 Streptomyces genomes reported)

## Evidence

- [readme] BiG-SCAPE does this by calculating pairwise distances between gene clusters based on a comparison of their protein domain content, order, copy number and sequence identity.: [evidence span withheld — anchor status: external_doc]
