---
name: subpfam-hmm-construction-from-msa-clades
description: Use when standard Pfam pHMMs are too coarse for downstream BGC tokenisation
  and per-clade subfamily-level pHMMs are needed from existing Pfam MSAs.
when_to_use_negative:
- The target Pfams are rare or already functionally specific, since subPfams were
  only constructed for the 112 most frequent Pfam domains in antiSMASH-DB
- Only raw unaligned protein sequences are available and no curated Pfam MSA exists
  to phylogenetically split
- The downstream analysis does not require subfamily-level domain tokenisation (ordinary
  Pfam pHMMs suffice)
edam_topics:
- http://edamontology.org/topic_0623
tools:
- name: build_subpfam
  role: Primary pipeline that cuts each Pfam MSA into clades via tree-cutting and
    builds per-clade subPfam pHMMs
  repo: satriaphd/build_subpfam
- name: hmmscan
  role: HMMER3 search tool against which the assembled subPfam pHMM database is used
    to tokenise BGC genes
- name: Pfam
  role: Source of the domain MSAs (Pfam 32.0) from which subPfams are derived
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:077d53fd9929d2888dfdef732c8d2d6ce28f741228168036b21c5987dc472421
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satriaphd/build_subpfam
  tools:
  - ECDomainMiner
  - Pfam
  - antiSMASH
  - build_subpfam
  - hmmscan
  - iPRESTO
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# subPfam HMM construction from MSA clades

## Summary

Builds finer-grained 'subPfam' pHMM models by splitting each Pfam multiple sequence alignment into phylogenetic clades and converting every clade into its own pHMM, so that overly broad Pfam domains (e.g. the 112 most abundant biosynthetic Pfams in antiSMASH-DB) can be tokenised at higher resolution with hmmscan.

## When to use

Use when standard Pfam pHMMs are too coarse for downstream BGC tokenisation and per-clade subfamily-level pHMMs are needed from existing Pfam MSAs.

## When NOT to use

- The target Pfams are rare or already functionally specific, since subPfams were only constructed for the 112 most frequent Pfam domains in antiSMASH-DB
- Only raw unaligned protein sequences are available and no curated Pfam MSA exists to phylogenetically split
- The downstream analysis does not require subfamily-level domain tokenisation (ordinary Pfam pHMMs suffice)

## Inputs

- Pfam multiple sequence alignments in multifasta format (per domain)
- Set of target Pfam domains (here: the 112 most frequent biosynthetic Pfams in antiSMASH-DB, taken from Pfam 32.0)
- Tree-cut branch-length parameter defining clade boundaries

## Outputs

- Per-clade subPfam pHMM models
- Custom HMMER3 pHMM database in which abundant biosynthetic Pfams are replaced by clade-derived subPfams (usable as input to hmmscan)

## How to apply

For each target Pfam, start from the aligned protein sequences in a multifasta MSA. Programmatically define clades by cutting a mid-rooted, ladderized, ultrametricized phylogenetic tree of the aligned sequences at a specified branch-length point, then convert each clade into a new pHMM. In the iPRESTO workflow this was done with the build_subpfam pipeline for the 112 most frequent biosynthetic Pfam domains in the antiSMASH-DB, based on Pfam 32.0, and the resulting subPfam database replaced those 112 abundant Pfams inside the custom pHMM database used by HMMER3 hmmscan. The decision point is the tree-cut length point, which determines clade granularity and hence subPfam resolution.

## Related tools

- **build_subpfam** (Primary pipeline that cuts each Pfam MSA into clades via tree-cutting and builds per-clade subPfam pHMMs) — satriaphd/build_subpfam
- **hmmscan** (HMMER3 search tool against which the assembled subPfam pHMM database is used to tokenise BGC genes)
- **Pfam** (Source of the domain MSAs (Pfam 32.0) from which subPfams are derived)

## Evaluation signals

- SubPfams are constructed exactly for the 112 most frequent Pfam domains in the antiSMASH-DB, not for every Pfam
- Each resulting clade-derived pHMM must be usable with HMMER3 hmmscan as part of a custom database built from Pfam 32.0
- SubPfam-scanned tokenisation of antiSMASH-DB v2 downstream should reproduce the paper's preprocessed dataset: 60,028 non-redundant BGCs and 10,539 distinct domain combinations (Table A of S1 Text)
- Every clade in a subPfam traces back to a cut of the mid-rooted, ladderized, ultrametricized tree of that Pfam's MSA, so clade counts reflect the chosen cut-length point

## Limitations

- The build_subpfam README marks its usage documentation as 'under construction', so parameters (e.g. the tree cut point) must be inferred from the paper rather than the tool docs
- SubPfam granularity depends entirely on where the phylogenetic tree is cut; a poorly chosen cut point can under- or over-split domains
- Only the 112 most abundant Pfam domains were sub-divided; rare Pfams keep their original broad pHMM granularity

## Evidence

- [readme] Build per-clade pHMM models from a collection of aligned protein sequences (in a multifasta format).: [evidence span withheld — anchor status: external_doc]
- [readme] Clades defined by cutting an ultrametricized tree: [evidence span withheld — anchor status: external_doc]
- [supplementary] Dividing Pfam MSAs into clades as new pHMMs: "dividing the multiple sequence alignment of a Pfam into clades and converting each clade into a new pHMM"
- [supplementary] SubPfams built for the 112 most frequent Pfams: "subPfams are constructed for the 112 most frequent Pfam domains in the antiSMASH-DB"
- [other] hmmscan tokenisation against the custom subPfam database: [evidence span withheld — anchor status: not_found]
