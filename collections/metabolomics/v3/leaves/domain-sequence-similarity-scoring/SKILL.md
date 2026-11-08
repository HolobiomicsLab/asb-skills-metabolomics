---
name: domain-sequence-similarity-scoring
description: Use when computing pairwise distances between biosynthetic gene clusters
  at scale and the distance must capture both Pfam domain copy-number differences
  and domain sequence identity, e.g. for GCF network construction from antiSMASH output.
when_to_use_negative:
- Input gene clusters are not antiSMASH-processed GenBank files with Pfam domain annotations,
  so no domain sequences exist to align to profile HMMs
- The analysis goal is a high-resolution multi-locus phylogeny of a gene cluster family
  (CORASON-style conserved-core analysis), where full alignment methods like Muscle
  or MAFFT are used instead
- Only a handful of BGCs are compared and full all-vers-all sequence alignment cost
  is negligible
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: hmmalign
  role: Aligns the sequences of every predicted domain type to its corresponding Pfam
    profile HMM, producing the profile-based alignments on which the DSS index is
    scored
- name: HMMER 3.1b2
  role: HMMER version used with Pfam v31 for domain detection against the user-supplied
    Pfam database and for the hmmalign profile alignments
- name: antiSMASH v4
  role: Produces the --minimal GenBank BGC predictions from which predicted domain
    sequences are extracted
- name: BiG-SCAPE
  role: Implements the DSS index, runs the Munkres (Hungarian) assignment of same-type
    domains, and combines DSS with the Jaccard and Adjacency indices into the pairwise
    distance
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:314ac88ec9982970d26fe6915a5cb78f47f17c185540d6b57149fc49e1231700
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  - hmmalign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# domain-sequence-similarity-scoring

## Summary

Computes the Domain Sequence Similarity (DSS) index of BiG-SCAPE's pairwise BGC distance by aligning each predicted domain sequence to its Pfam profile HMM with hmmalign, then scoring matched domain pairs for copy-number differences and sequence identity. This profile-based approach avoids the quadratic all-vers-all sequence comparison bottleneck in large-scale BGC comparisons.

## When to use

Use when computing pairwise distances between biosynthetic gene clusters at scale and the distance must capture both Pfam domain copy-number differences and domain sequence identity, e.g. for GCF network construction from antiSMASH output.

## When NOT to use

- Input gene clusters are not antiSMASH-processed GenBank files with Pfam domain annotations, so no domain sequences exist to align to profile HMMs
- The analysis goal is a high-resolution multi-locus phylogeny of a gene cluster family (CORASON-style conserved-core analysis), where full alignment methods like Muscle or MAFFT are used instead
- Only a handful of BGCs are compared and full all-vers-all sequence alignment cost is negligible

## Inputs

- antiSMASH v4 (--minimal) GenBank output files with predicted BGCs
- Predicted domain sequences for every BGC (filtered for overlapping domains by hmmer score)
- Pfam database version 31 profile HMMs (used with HMMER 3.1b2)

## Outputs

- Domain Sequence Similarity (DSS) index per BGC pair, reflecting Pfam domain copy-number differences and domain sequence identity
- Pairwise BGC distance values combining DSS with the Jaccard and Adjacency indices
- BGC sequence similarity network input for GCF assignment

## How to apply

Load antiSMASH v4 (--minimal) GenBank output files and extract the predicted domain sequences for every BGC, first filtering overlapping domains based on the hmmer score. Align the sequences of every predicted domain type to its corresponding profile HMM using hmmalign from HMMER 3.1b2 with Pfam database version 31 — each domain is aligned only to its own model, not against all other domains. For each BGC pair, apply the Munkres (Hungarian) assignment algorithm to find the optimal one-to-one matching between domains of the same type. Score the matched pairs on copy-number differences and sequence identity of the aligned domains, and combine the resulting DSS index with the Jaccard and Adjacency indices into BiG-SCAPE's pairwise distance calculation. The rationale is that aligning to the profile HMM replaces expensive all-vs-all domain sequence alignment while retaining sufficient resolution for GCF calling.

## Related tools

- **hmmalign** (Aligns the sequences of every predicted domain type to its corresponding Pfam profile HMM, producing the profile-based alignments on which the DSS index is scored)
- **HMMER 3.1b2** (HMMER version used with Pfam v31 for domain detection against the user-supplied Pfam database and for the hmmalign profile alignments)
- **antiSMASH v4** (Produces the --minimal GenBank BGC predictions from which predicted domain sequences are extracted)
- **BiG-SCAPE** (Implements the DSS index, runs the Munkres (Hungarian) assignment of same-type domains, and combines DSS with the Jaccard and Adjacency indices into the pairwise distance) — https://git.wageningenur.nl/medema-group/BiG-SCAPE

## Evaluation signals

- Repeat GCF calling with Muscle- and MAFFT-based alignment variants using BiG-SCAPE's --mix parameter and confirm high agreement in final GCF calling between the three methods
- Verify that hmmalign runtimes remain shorter than Muscle/MAFFT alternatives as the number of input BGCs increases
- Confirm every domain sequence was aligned only to its own corresponding Pfam model (one model per domain type), not cross-aligned
- Check that overlapping domains were filtered by hmmer score before alignment
- Verify the DSS score reflects both copy-number differences and sequence identity of optimally Munkres-matched domain pairs within each BGC pair

## Limitations

- Validated against Muscle- and MAFFT-based variants within BiG-SCAPE's --mix parameter; agreement is high but the profile-based DSS trades alignment resolution for runtime
- Depends on the quality of antiSMASH v4 domain predictions and the chosen Pfam version (v31); switching Pfam versions changes the DSS index
- Duplicate ORGANISM identifiers across assembly projects can cause BiG-SCAPE's interactive report to omit genomes (e.g. only 96 of 103 Streptomyces genomes reported), which affects downstream interpretation of the distance network

## Evidence

- [readme] BiG-SCAPE distance definition: [evidence span withheld — anchor status: external_doc]
