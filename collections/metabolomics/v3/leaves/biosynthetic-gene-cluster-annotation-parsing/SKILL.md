---
name: biosynthetic-gene-cluster-annotation-parsing
description: Use when antiSMASH GenBank outputs must be converted into ordered per-gene
  Pfam domain strings with Core Biosynthetic gene annotations, e.g. to seed LCS-based
  BGC similarity comparisons.
when_to_use_negative:
- Input is raw nucleotide FASTA or unassembled contigs without antiSMASH domain annotations
  — run BGC prediction and domain detection first
- Downstream analysis already consumes BiG-SCAPE's processed domain/domain-sequence
  files rather than the original GenBank records
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: antiSMASH
  role: Source of the GenBank files whose per-gene Pfam domain order and 'Core Biosynthetic'
    annotations are parsed; antiSMASH v4 (--minimal) was used in the reference study
  repo: https://antismash.secondarymetabolites.org
- name: BiG-SCAPE
  role: 'Consumer of the parsed annotations: it uses antiSMASH GenBank inputs to compute
    pairwise BGC distances via Jaccard, Domain Sequence Similarity and Adjacency indices'
  repo: https://github.com/medema-group/BiG-SCAPE
- name: HMMER 3.1b2
  role: Detects conserved domains against the user-supplied Pfam database; hmmer scores
    resolve overlapping-domain conflicts during parsing
- name: hmmalign
  role: Aligns the sequences of every predicted domain type with its corresponding
    model after domain parsing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:b94824b72897028ff6f51c233ba3d7d1ca40bc82f3662d0df60fc8237c8a460d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH
  - hmmalign
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# biosynthetic-gene-cluster-annotation-parsing

## Summary

Extracts ordered per-gene Pfam domain strings and antiSMASH 'Core Biosynthetic' gene flags from antiSMASH-processed GenBank files, producing the structured domain-sequence representations that BiG-SCAPE's glocal pairwise distance calculations require.

## When to use

Use when antiSMASH GenBank outputs must be converted into ordered per-gene Pfam domain strings with Core Biosynthetic gene annotations, e.g. to seed LCS-based BGC similarity comparisons.

## When NOT to use

- Input is raw nucleotide FASTA or unassembled contigs without antiSMASH domain annotations — run BGC prediction and domain detection first
- Downstream analysis already consumes BiG-SCAPE's processed domain/domain-sequence files rather than the original GenBank records

## Inputs

- antiSMASH-processed GenBank files (BGC predictions), e.g. antiSMASH v4 run with --minimal
- User-supplied Pfam database (version 31 in the reference study) with HMMER 3.1b2 domain hits
- Reference BGC GenBank files such as MIBiG (release 1.3) entries

## Outputs

- Ordered per-gene Pfam domain string for each BGC
- Per-BGC list of genes flagged 'Core Biosynthetic' by antiSMASH
- Deduplicated domain annotations after hmmer-score-based filtering of overlapping domains

## How to apply

Parse the antiSMASH GenBank files with a GenBank-aware reader (e.g. Biopython SeqIO) to obtain, for each BGC, the ordered per-gene Pfam domain string and the set of genes annotated by antiSMASH as 'Core Biosynthetic'. Domains are detected against a user-supplied external Pfam database (Pfam v31 with HMMER 3.1b2 in the reference study); when domains overlap, keep the hit with the higher hmmer score so each gene carries a single, non-redundant domain order. Preserve gene order in the domain string because downstream glocal comparison (difflib.SequenceMatcher to find the LCS, then extension with match=5, mismatch=-3, gap=-2) is order-sensitive. Retain the Core Biosynthetic flags alongside the domain string, since the LCS between two clusters is only accepted if it is at least three genes long or contains at least one Core Biosynthetic gene.

## Related tools

- **antiSMASH** (Source of the GenBank files whose per-gene Pfam domain order and 'Core Biosynthetic' annotations are parsed; antiSMASH v4 (--minimal) was used in the reference study) — https://antismash.secondarymetabolites.org
- **BiG-SCAPE** (Consumer of the parsed annotations: it uses antiSMASH GenBank inputs to compute pairwise BGC distances via Jaccard, Domain Sequence Similarity and Adjacency indices) — https://github.com/medema-group/BiG-SCAPE
- **HMMER 3.1b2** (Detects conserved domains against the user-supplied Pfam database; hmmer scores resolve overlapping-domain conflicts during parsing)
- **hmmalign** (Aligns the sequences of every predicted domain type with its corresponding model after domain parsing)

## Examples

```
from difflib import SequenceMatcher; m = SequenceMatcher(None, domains_a, domains_b); lcs = m.find_longest_match(0, len(domains_a), 0, len(domains_b))
```

## Evaluation signals

- Every gene in each parsed BGC yields exactly one domain in the ordered string once overlapping domains are filtered by hmmer score
- The set of Core Biosynthetic genes extracted matches the antiSMASH annotations in the source GenBank records
- LCS candidates derived from the parsed domain strings satisfy the acceptance rule: length >= 3 genes or containing at least one Core Biosynthetic gene
- Parsed domain strings reproduce the same LCS matches and glocal distances on known BGC pairs (extension scoring match=5, mismatch=-3, gap=-2)
- Module code passes the repository's flake8 linting and black formatting checks

## Limitations

- Overlapping domain predictions are ambiguous and must be resolved by hmmer score, which can alter the domain order fed to downstream alignment
- Genomic scaffolds from the same organism grouped by ORGANISM identifier can collapse genomes in downstream reports (BiG-SCAPE reported only 96 of 103 Streptomyces genomes for this reason)
- Results depend on the antiSMASH version and Pfam release used (antiSMASH v4 and Pfam 31 in the reference study); re-parsing is needed when versions change

## Evidence

- [discussion] All the sequences from each file are searched for conserved domains using a user-supplied external Pfam database.: "All the sequences from each file are searched for conserved domains using a user-supplied external Pfam database."
- [readme] BiG-SCAPE uses antiSMASH processed GenBank files, i.e. BGC predictions, as well as reference BGC GenBank files: [evidence span withheld — anchor status: external_doc]
