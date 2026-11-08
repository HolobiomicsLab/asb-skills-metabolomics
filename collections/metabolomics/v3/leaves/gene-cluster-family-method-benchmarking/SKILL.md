---
name: gene-cluster-family-method-benchmarking
description: Use when choosing or validating the alignment backend for BiG-SCAPE's
  DSS index on a growing BGC dataset — e.g. to confirm that Pfam profile-HMM alignment
  (hmmalign) matches Muscle/MAFFT-based GCF calling while being faster as input BGC
  count increases.
when_to_use_negative:
- The goal is a single definitive GCF clustering on one small dataset where aligner
  runtime is irrelevant — just run one backend rather than benchmarking
- The input is raw genome contigs rather than antiSMASH-processed GenBank BGC predictions;
  BGC and domain prediction must happen first
- The analysis requires multi-locus phylogenies of a single GCF (use CORASON's conserved-core
  approach) rather than method comparison across the whole BGC set
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: BiG-SCAPE
  role: Analysis engine that computes pairwise distances (Jaccard, DSS, Adjacency)
    and GCF calling; re-run with --mix to swap alignment backends for benchmarking
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: hmmalign
  role: HMMER 3.1b2 tool that aligns every predicted domain sequence to its corresponding
    Pfam profile HMM — the fast baseline being benchmarked
- name: antiSMASH v4
  role: Produces the --minimal GenBank BGC predictions and domain sequences that are
    the common benchmark input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:9b3fb4eb8d45d1c30b086eee8502a12345eb7bed5011f6e61353f33a6ba5d106
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - antiSMASH v4
  - hmmalign
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-cluster-family-method-benchmarking

## Summary

Benchmark alternative domain-sequence alignment engines (hmmalign profile-HMM alignment vs Muscle vs MAFFT) used by BiG-SCAPE for computing the Domain Sequence Similarity (DSS) index, by comparing final GCF calls and runtime scaling on the same antiSMASH-predicted BGC input set.

## When to use

Use when choosing or validating the alignment backend for BiG-SCAPE's DSS index on a growing BGC dataset — e.g. to confirm that Pfam profile-HMM alignment (hmmalign) matches Muscle/MAFFT-based GCF calling while being faster as input BGC count increases.

## When NOT to use

- The goal is a single definitive GCF clustering on one small dataset where aligner runtime is irrelevant — just run one backend rather than benchmarking
- The input is raw genome contigs rather than antiSMASH-processed GenBank BGC predictions; BGC and domain prediction must happen first
- The analysis requires multi-locus phylogenies of a single GCF (use CORASON's conserved-core approach) rather than method comparison across the whole BGC set

## Inputs

- antiSMASH v4 (--minimal) GenBank output files with predicted BGCs and domain sequences
- Pfam database version 31 (used with HMMER 3.1b2)
- Optional MIBiG reference BGC GenBank files
- Alternative multiple-sequence aligners: Muscle and MAFFT

## Outputs

- Pairwise BGC distance networks built from Jaccard, Domain Sequence Similarity (DSS), and Adjacency indices
- GCF assignments (final GCF calling) per alignment method for cross-method agreement comparison
- Runtime measurements as a function of the number of input BGCs for hmmalign vs Muscle vs MAFFT

## How to apply

Start from antiSMASH v4 (--minimal) GenBank outputs and analyze the BGC set with BiG-SCAPE using version 31 of the Pfam database together with HMMER version 3.1b2, letting hmmalign align the sequences of every predicted domain type to its corresponding profile HMM (overlapping domains first filtered by hmmer score). For each BGC pair, the DSS component is scored by pairing domains of the same type with the Munkres (Hungarian) optimal one-to-one assignment, then combining copy-number differences and aligned-domain sequence identity with the Jaccard and Adjacency indices. Re-run GCF calling with the alignment engine swapped to Muscle and MAFFT using BiG-SCAPE's --mix parameter, keeping all other parameters fixed. Validate the benchmark by comparing final GCF calls across the three methods for agreement while recording runtimes as the number of input BGCs increases.

## Related tools

- **BiG-SCAPE** (Analysis engine that computes pairwise distances (Jaccard, DSS, Adjacency) and GCF calling; re-run with --mix to swap alignment backends for benchmarking) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **hmmalign** (HMMER 3.1b2 tool that aligns every predicted domain sequence to its corresponding Pfam profile HMM — the fast baseline being benchmarked)
- **antiSMASH v4** (Produces the --minimal GenBank BGC predictions and domain sequences that are the common benchmark input)

## Examples

```
python bigscape.py -i antiSMASH_gbk/ -o results/ --mix
```

## Evaluation signals

- Final GCF calling (using BiG-SCAPE's --mix parameter) shows high agreement between the hmmalign, Muscle, and MAFFT variants
- hmmalign runtimes are shorter than the MSA-based variants and scale favorably as the number of BGCs in the input data increases
- Every pairwise distance decomposes into the three documented indices: percentage of shared domain types (Jaccard), DSS, and Adjacency
- DSS matched pairs are obtained via optimal one-to-one assignment (Munkres/Hungarian) between domains of the same Pfam type, and copy-number differences are reflected in the score
- Domain calls are consistent with the fixed reference stack: Pfam version 31 with HMMER version 3.1b2, with overlapping domains filtered by hmmer score

## Limitations

- The hmmalign runtime advantage is a scaling property — it only becomes apparent as the number of BGCs in the input data increases; for tiny datasets the three backends are effectively equivalent
- Duplicate ORGANISM identifiers across assembly projects collapse genomes in the BiG-SCAPE interactive report (only 96 of 103 Streptomyces genomes were reported), which can skew benchmark bookkeeping
- CORASON-based downstream validation of GCFs can reduce the conserved core to only the query gene for divergent BGCs, limiting phylogenetic cross-checks

## Evidence

- [methods] DSS scoring via Munkres assignment (source card workflow): [evidence span withheld — anchor status: not_found]
- [readme] BiG-SCAPE purpose per README: [evidence span withheld — anchor status: external_doc]
