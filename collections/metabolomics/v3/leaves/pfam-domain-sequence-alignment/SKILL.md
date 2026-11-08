---
name: pfam-domain-sequence-alignment
description: Use when antiSMASH GenBank outputs must be converted into ordered Pfam
  domain strings for BGC pairwise distance calculation, especially when complete and
  fragmented (or merged) BGCs make purely global or local domain alignment unreliable.
when_to_use_negative:
- Input is raw nucleotide contigs or peptides without prior antiSMASH domain annotation
  — no Pfam domain strings exist to align
- The analysis target is CORASON conserved-core phylogenetics on highly divergent
  BGCs, where the core may be reduced to only the query gene rather than a domain-order
  alignment
- Downstream tasks require full-cluster global identity scores; glocal mode deliberately
  aligns only the LCS-seeded corresponding region
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0157
tools:
- name: antiSMASH
  role: Produces the GenBank BGC predictions and 'Core Biosynthetic' gene annotations
    parsed into Pfam domain strings
  repo: https://antismash.secondarymetabolites.org
- name: BiG-SCAPE
  role: Implements the glocal alignment (LCS seed + extension scoring) and pairwise
    distance calculation on Pfam domain content
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: hmmalign
  role: Aligns sequences of every predicted domain type using each corresponding Pfam
    HMM model for sequence-similarity distance
- name: HMMER 3.1b2
  role: Detects domains against Pfam v31 and provides the scores used to filter overlapping
    domains
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:4671269875d895fdb8ff44a588369049759fc2c3a7879fcc6ba8538eb423664c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH
  - hmmalign
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pfam-domain-sequence-alignment

## Summary

Compares biosynthetic gene clusters by aligning their ordered Pfam domain content — first filtering overlapping domains by hmmer score, then optionally aligning domain sequences per Pfam model with hmmalign, and reconstructing BiG-SCAPE's 'glocal' alignment (LCS seed plus match/mismatch/gap extension) to compute pairwise distances robust to fragmented or merged BGCs.

## When to use

Use when antiSMASH GenBank outputs must be converted into ordered Pfam domain strings for BGC pairwise distance calculation, especially when complete and fragmented (or merged) BGCs make purely global or local domain alignment unreliable.

## When NOT to use

- Input is raw nucleotide contigs or peptides without prior antiSMASH domain annotation — no Pfam domain strings exist to align
- The analysis target is CORASON conserved-core phylogenetics on highly divergent BGCs, where the core may be reduced to only the query gene rather than a domain-order alignment
- Downstream tasks require full-cluster global identity scores; glocal mode deliberately aligns only the LCS-seeded corresponding region

## Inputs

- antiSMASH-processed GenBank files of predicted BGCs (antiSMASH v4, --minimal)
- Pfam database (version 31) with HMMER 3.1b2 for domain detection
- MIBiG reference BGC GenBank files (release 1.3, 1,393 clusters)
- AntiSMASH 'Core Biosynthetic' gene annotations per BGC

## Outputs

- Ordered per-gene Pfam domain strings per BGC
- Pairwise BGC distance contributions from the glocal-selected matching region (Jaccard, Domain Sequence Similarity, Adjacency indices)
- Sequence similarity network edges below the distance cutoff, used for GCF assignment via Affinity Propagation

## How to apply

Parse antiSMASH GenBank inputs to extract each BGC's ordered per-gene Pfam domain strings and antiSMASH 'Core Biosynthetic' gene annotations, filtering overlapping domains based on the score calculated by hmmer (Pfam release 31 with HMMER 3.1b2 in the reference workflow). For sequence-level domain comparison, align the sequences of every predicted domain type using each corresponding Pfam model via hmmalign (high GCF-calling agreement with Muscle and MAFFT, with shorter runtimes as BGC count grows). For the glocal mode, use difflib.SequenceMatcher on each BGC pair's domain strings to find the longest common subcluster; keep candidate subclusters only if they are at least three genes long or contain at least one 'Core Biosynthetic' gene. Extend around the LCS with the extension-stage scoring scheme (match=5, mismatch=-3, gap=-2) to select the matching region, then compute the glocal distance contribution (Jaccard, Domain Sequence Similarity, Adjacency indices) from that region. Feed the resulting pairwise distances into network filtering at the empirical cutoff (0.3 for GCF assignment; 0.75 evaluated for node-count/structural-integrity balance).

## Related tools

- **antiSMASH** (Produces the GenBank BGC predictions and 'Core Biosynthetic' gene annotations parsed into Pfam domain strings) — https://antismash.secondarymetabolites.org
- **BiG-SCAPE** (Implements the glocal alignment (LCS seed + extension scoring) and pairwise distance calculation on Pfam domain content) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **hmmalign** (Aligns sequences of every predicted domain type using each corresponding Pfam HMM model for sequence-similarity distance)
- **HMMER 3.1b2** (Detects domains against Pfam v31 and provides the scores used to filter overlapping domains)

## Examples

```
from difflib import SequenceMatcher; blocks = SequenceMatcher(None, domains_a, domains_b).get_matching_blocks(); lcs = max(blocks, key=lambda b: b.size)
```

## Evaluation signals

- Glocal matching region passes the LCS filter: at least three genes long OR containing at least one antiSMASH 'Core Biosynthetic' gene
- No overlapping domain pairs remain after hmmer-score-based filtering of the domain strings
- Distance matrices agree with BiG-SCAPE's native run on the same antiSMASH GenBank inputs (Jaccard, Domain Sequence Similarity, Adjacency values match)
- Module passes the repository's flake8 linting and black formatting checks
- GCF assignments via network cutoff (0.3) plus Affinity Propagation reproduce the article's family-level grouping on validation BGC pairs

## Limitations

- Current methods for grouping BGCs into gene cluster families fail to correctly measure similarity between complete and fragmented gene clusters — precisely the artifact glocal mode is designed to mitigate, so results should be sanity-checked on known fragmented loci
- Alignment-dependent analyses can fail with an empty alignment; the recommended remedy is to reduce the score cutoff or the scope of taxonomic diversity in the genomic database
- For divergent BGCs, a CORASON conserved core built downstream can collapse to only the query gene
- BiG-SCAPE's interactive report may drop genomes when scaffolds share duplicate ORGANISM identifiers (96 of 103 Streptomyces genomes reported in the study)

## Evidence

- [readme] BiG-SCAPE does this by calculating pairwise distances between gene clusters based on a comparison of their protein domain content, order, copy number and sequence identity.: [evidence span withheld — anchor status: external_doc]
