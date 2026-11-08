---
name: biosynthetic-domain-curation-via-ec-annotation
description: Use when tokenised BGCs (antiSMASH-detected, (sub)Pfam-annotated) contain
  non-biosynthetic domains that must be removed prior to sub-cluster prediction or
  redundancy filtering.
when_to_use_negative:
- The BGCs are already tokenised and contain only biosynthetic Pfams/subPfams (e.g.
  already processed with this skill) — re-filtering would double-remove rare domains.
- The dataset is too small for frequency-based thresholds to be meaningful (e.g. a
  single genome or a small MIBiG-only analysis), where dropping Pfams occurring <3
  times could strip real biosynthetic signal.
- The analysis requires non-biosynthetic domains (e.g. tailoring or regulatory-gene
  annotation) rather than biosynthetic sub-cluster discovery.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: ECDomainMiner
  role: Source of the curated list of 1,839 biosynthetic Pfams used as the retention
    whitelist during domain curation
- name: hmmscan
  role: HMMER3 tool that tokenises every gene by scanning against the custom Pfam
    32.0/subPfam pHMM database before biosynthetic-domain filtering
- name: antiSMASH
  role: Produces the BGC predictions whose genes are tokenised and then curated to
    biosynthetic domains only
- name: build_subpfam
  role: Builds the per-clade subPfam pHMMs (from the 112 most frequent biosynthetic
    Pfams) that replace full-length Pfams in the scanning database
  repo: https://github.com/satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:e9624b883400c400dc17a96c70801369701b08b1c10b20df0db221e3229b505f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satriaphd/build_subpfam
  tools:
  - ECDomainMiner
  - antiSMASH
  - build_subpfam
  - hmmscan
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# biosynthetic-domain-curation-via-ec-annotation

## Summary

Curation step that restricts tokenised BGC genes to the 1,839 biosynthetic Pfams curated via ECDomainMiner, removes non-biosynthetic domains and rare Pfams (<3 occurrences), and discards BGCs with fewer than two non-empty genes before redundancy filtering.

## When to use

Use when tokenised BGCs (antiSMASH-detected, (sub)Pfam-annotated) contain non-biosynthetic domains that must be removed prior to sub-cluster prediction or redundancy filtering.

## When NOT to use

- The BGCs are already tokenised and contain only biosynthetic Pfams/subPfams (e.g. already processed with this skill) — re-filtering would double-remove rare domains.
- The dataset is too small for frequency-based thresholds to be meaningful (e.g. a single genome or a small MIBiG-only analysis), where dropping Pfams occurring <3 times could strip real biosynthetic signal.
- The analysis requires non-biosynthetic domains (e.g. tailoring or regulatory-gene annotation) rather than biosynthetic sub-cluster discovery.

## Inputs

- antiSMASH-detected BGCs (e.g. antiSMASH-DB v2 download asdb_20180828_all_results.tar.xz, 152,122 BGCs; MIBiG 1.4, 1,819 BGCs; Streptomyces/Salinispora dataset, 5,927 BGCs)
- Custom pHMM database built from Pfam 32.0 with 112 clade-derived subPfams replacing the most abundant biosynthetic Pfams
- List of 1,839 biosynthetic Pfams curated via ECDomainMiner

## Outputs

- Tokenised BGC table restricted to biosynthetic (sub)Pfam domains only
- Filtered BGC set with rare Pfams (<3 occurrences) and low-content BGCs (<2 non-empty genes) removed
- Curated BGCs ready for graph-based redundancy filtering (output downstream: preprocessed_nonredundant_bgcs.tsv with 60,028 BGCs and 10,539 domain combinations)

## How to apply

Start from antiSMASH-predicted BGCs tokenised by running HMMER3 hmmscan against a custom pHMM database built from Pfam 32.0, in which the 112 most abundant biosynthetic Pfams in antiSMASH-DB are replaced by clade-derived subPfams (built with the build_subpfam pipeline). Retain only the 1,839 biosynthetic Pfams curated via ECDomainMiner, removing all non-biosynthetic domains from every gene. Then drop individual Pfams that occur fewer than three times across the dataset, and discard BGCs left with fewer than two non-empty genes, since single-domain gene clusters carry no sub-cluster signal. The curated, tokenised BGCs then feed the Adjacency Index (>0.95) graph-based redundancy filter and downstream PRESTO-STAT/PRESTO-TOP sub-cluster prediction.

## Related tools

- **ECDomainMiner** (Source of the curated list of 1,839 biosynthetic Pfams used as the retention whitelist during domain curation)
- **hmmscan** (HMMER3 tool that tokenises every gene by scanning against the custom Pfam 32.0/subPfam pHMM database before biosynthetic-domain filtering)
- **antiSMASH** (Produces the BGC predictions whose genes are tokenised and then curated to biosynthetic domains only)
- **build_subpfam** (Builds the per-clade subPfam pHMMs (from the 112 most frequent biosynthetic Pfams) that replace full-length Pfams in the scanning database) — https://github.com/satriaphd/build_subpfam

## Examples

```
hmmscan --domtblout domains.domtbl subpfam_db.hmm bgc_genes.faa
```

## Evaluation signals

- Every domain label surviving in the output must map to one of the 1,839 ECDomainMiner-curated biosynthetic Pfams (or their clade-derived subPfams).
- No Pfam present in the output occurs fewer than 3 times across the dataset.
- Every surviving BGC contains at least 2 non-empty (domain-bearing) genes.
- Downstream invariant: after the subsequent Adjacency Index >0.95 redundancy filter, the antiSMASH-DB v2 pipeline should yield exactly 60,028 BGCs and 10,539 distinct domain combinations (Table A of S1 Text).
- BGCs flagged by antiSMASH as lying on a contig edge must have been discarded before curation.

## Limitations

- Frequency-based domain removal (Pfams occurring <3 times) is calibrated to the large antiSMASH-DB dataset and may over-prune small datasets.
- Curation depends on the Pfam 32.0-based custom database with 112 subPfams; other Pfam versions or subPfam definitions will change the retained domain set.
- BGCs on contig edges are discarded upstream, so partially sequenced clusters never reach this curation step; no antiSMASH version number is stated in the supplementary methods.

## Evidence

- [supplementary] Tokenisation removes non-biosynthetic domains: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [supplementary] SubPfams built for the 112 most frequent Pfam domains: "subPfams are constructed for the 112 most frequent Pfam domains in the antiSMASH-DB"
- [readme] build_subpfam constructs clade-derived pHMMs: [evidence span withheld — anchor status: external_doc]
- [supplementary] Final curated dataset size (validation invariant): "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
