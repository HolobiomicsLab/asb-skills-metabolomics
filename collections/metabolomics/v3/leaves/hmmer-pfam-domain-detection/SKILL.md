---
name: hmmer-pfam-domain-detection
description: Use when antiSMASH GenBank BGC outputs must be assigned conserved protein
  domains before BGC similarity networking (BiG-SCAPE), or when per-domain-type alignments
  are needed for GCF/clan clustering or CORASON-style analysis.
when_to_use_negative:
- Input BGCs are not in antiSMASH GenBank format (BiG-SCAPE expects antiSMASH-processed
  GenBank files, not raw FASTA genomes)
- Domain annotations already exist at the same resolution (e.g. pre-computed Pfam
  domain tables); re-running HMMER would only add runtime
- The goal is full-sequence similarity rather than domain-content-based BGC comparison,
  since this skill's distances are defined on shared domain types
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: HMMER 3.1b2
  role: Searches every sequence in the antiSMASH GenBank files against the user-supplied
    Pfam v31 HMM database; its scores also resolve overlapping-domain filtering
- name: hmmalign
  role: Aligns the sequences of every predicted domain type using each corresponding
    Pfam model, providing the domain sequence comparisons behind the Domain Sequence
    Similarity index
- name: BiG-SCAPE
  role: Consumes the HMMER/Pfam domain annotations, filtering overlapping domains
    by hmmer score and computing Jaccard/Domain Sequence Similarity/Adjacency distances
    for GCF assignment
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: antiSMASH v4
  role: Produces the BGC GenBank inputs (--minimal parameter setting) whose sequences
    undergo Pfam/HMMER domain detection
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:27090e2013b7cdd351880e2219063c26f2dfa8e5866746ae7d1a25dafaf2c0ed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  - hmmalign
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# hmmer-pfam-domain-detection

## Summary

Annotates antiSMASH-predicted biosynthetic gene clusters with conserved protein domains by searching sequences against a user-supplied Pfam database (version 31 with HMMER 3.1b2 in this study), then aligns every predicted domain type with hmmalign — the domain content that BiG-SCAPE converts into Jaccard, Domain Sequence Similarity and Adjacency pairwise distances.

## When to use

Use when antiSMASH GenBank BGC outputs must be assigned conserved protein domains before BGC similarity networking (BiG-SCAPE), or when per-domain-type alignments are needed for GCF/clan clustering or CORASON-style analysis.

## When NOT to use

- Input BGCs are not in antiSMASH GenBank format (BiG-SCAPE expects antiSMASH-processed GenBank files, not raw FASTA genomes)
- Domain annotations already exist at the same resolution (e.g. pre-computed Pfam domain tables); re-running HMMER would only add runtime
- The goal is full-sequence similarity rather than domain-content-based BGC comparison, since this skill's distances are defined on shared domain types

## Inputs

- antiSMASH v4 GenBank output files of predicted BGCs (e.g. --minimal runs)
- User-supplied Pfam database (version 31) HMM files
- Reference BGC GenBank files such as MIBiG release 1.3 (1,393 BGCs) for the combined analysis

## Outputs

- Per-BGC domain annotation (domain types, orders, copy numbers) with overlapping domains resolved by hmmer score
- Per-domain-type multiple sequence alignments produced by hmmalign against each corresponding HMM model
- Domain-content tables consumed by BiG-SCAPE pairwise distance computation (Jaccard, Domain Sequence Similarity, Adjacency)

## How to apply

Take antiSMASH GenBank outputs as input and search all sequences from each file for conserved domains using a user-supplied external Pfam database — here Pfam version 31 searched with HMMER 3.1b2. Overlapping domains are filtered based on the score calculated by hmmer, keeping the highest-scoring domain so copy-number counting downstream is consistent. The sequences of every predicted domain type are then aligned using each corresponding model by hmmalign; these domain alignments feed the Domain Sequence Similarity index (and the BiG-SCAPE --mix GCF calling). Rationale for choosing hmmalign over Muscle/MAFFT: comparison of final GCF calling showed high agreement between the three methods, with hmmalign showing shorter runtimes as the number of BGCs in the input data increases — an important property for large datasets such as 74,652 actinobacterial BGCs.

## Related tools

- **HMMER 3.1b2** (Searches every sequence in the antiSMASH GenBank files against the user-supplied Pfam v31 HMM database; its scores also resolve overlapping-domain filtering)
- **hmmalign** (Aligns the sequences of every predicted domain type using each corresponding Pfam model, providing the domain sequence comparisons behind the Domain Sequence Similarity index)
- **BiG-SCAPE** (Consumes the HMMER/Pfam domain annotations, filtering overlapping domains by hmmer score and computing Jaccard/Domain Sequence Similarity/Adjacency distances for GCF assignment) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **antiSMASH v4** (Produces the BGC GenBank inputs (--minimal parameter setting) whose sequences undergo Pfam/HMMER domain detection)

## Evaluation signals

- All overlapping domain calls are resolved by hmmer score (highest-scoring domain retained) with no unfiltered overlaps carried into the distance computation
- A domain alignment is generated for every predicted domain type using its corresponding Pfam model (hmmalign output present per domain type)
- Recorded pipeline versions match the study configuration: Pfam version 31 with HMMER version 3.1b2
- Downstream GCF calling using hmmalign-derived domains shows high agreement with Muscle- and MAFFT-derived GCF calls (BiG-SCAPE --mix comparison)
- Reproduced clustering (e.g. 17,718 GCFs / 801 GCCs on 74,652 actinobacterial BGCs at cutoff 0.3) is consistent with published counts when the same domain annotations are used

## Limitations

- If the alignment itself is empty, it is recommended to reduce the score cutoff or the scope of the taxonomic diversity on the genomic database — manual parameter intervention may be required for divergent inputs
- hmmalign's runtime advantage accrues only as the number of BGCs in the input data increases; for small datasets the choice among hmmalign, Muscle and MAFFT matters mainly for consistency, not speed
- Results depend on the exact Pfam release and HMMER version; using a different Pfam version than 31 changes domain content and can alter GCF membership

## Evidence

- [readme] BiG-SCAPE calculates pairwise distances between gene clusters based on a comparison of their protein domain content, order, copy number and sequence identity.: [evidence span withheld — anchor status: external_doc]
