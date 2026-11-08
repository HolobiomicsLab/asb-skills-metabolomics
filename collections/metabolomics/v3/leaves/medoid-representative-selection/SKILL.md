---
name: medoid-representative-selection
description: Use when a redundant set of antiSMASH/gutSMASH gene-cluster GenBank outputs
  must be deduplicated before metagenomic read mapping, to avoid ambiguous mapping
  of reads to near-identical clusters.
when_to_use_negative:
- The gene-cluster collection is already nonredundant (e.g. one cluster per genome),
  so medoid selection would discard genuine diversity
- The input is raw sequencing reads or abundance tables rather than antiSMASH/gutSMASH
  GenBank outputs — this skill operates upstream of bigmap-map
- The goal is metagenome assembly or taxonomy profiling rather than profiling metabolic
  gene-cluster (MGC) abundance and expression
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: BiG-MAP
  role: Pipeline whose bigmap-family module implements the redundancy filtering and
    medoid representative selection
  repo: https://github.com/medema-group/BiG-MAP
- name: MASH
  role: MinHash sketching of extracted protein sequences to compute pairwise distances
    and cluster gene clusters at the 0.8 similarity cutoff
- name: BiG-SCAPE
  role: 'Second redundancy-filtering round: groups representative gene clusters into
    GCFs; recommended by the README for more accurate filtering'
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: gutSMASH
  role: Upstream predictor of metabolic gene clusters whose GenBank output folders
    serve as the input to bigmap-family
  repo: https://github.com/victoriapascal/gutsmash
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:665440ec01ea6a4733bba807cdab4aa22e4dd8d0cb48c082b9b7bcbf1abdcce0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/BiG-MAP
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - MASH
  - gutSMASH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# medoid-representative-selection

## Summary

Collapses redundant antiSMASH/gutSMASH-predicted gene clusters into a nonredundant representative collection by MASH MinHash sketching of protein sequences, selecting the medoid (minimal cumulative distance) per similarity cluster, with optional BiG-SCAPE-based GCF grouping.

## When to use

Use when a redundant set of antiSMASH/gutSMASH gene-cluster GenBank outputs must be deduplicated before metagenomic read mapping, to avoid ambiguous mapping of reads to near-identical clusters.

## When NOT to use

- The gene-cluster collection is already nonredundant (e.g. one cluster per genome), so medoid selection would discard genuine diversity
- The input is raw sequencing reads or abundance tables rather than antiSMASH/gutSMASH GenBank outputs — this skill operates upstream of bigmap-map
- The goal is metagenome assembly or taxonomy profiling rather than profiling metabolic gene-cluster (MGC) abundance and expression

## Inputs

- antiSMASH or gutSMASH output directories containing predicted gene-cluster GenBank files (run beforehand)
- Optional: BiG-SCAPE installation path and processed Pfam-A.hmm database (from InterPro, uncompressed and hmmpress-processed)
- Optional: similarity thresholds -tg (gene clusters, default 0.8) and -th (housekeeping genes, default 0.1)

## Outputs

- BiG-MAP.GCF.fna — reference FASTA of nonredundant representative gene clusters for read mapping
- BiG-MAP.GCF.bed — bedfile to extract core regions in bigmap-map
- BiG-MAP.GCs.json — dictionary of MASH-based gene-cluster families
- BiG-MAP.GCF.json — dictionary of BiG-SCAPE GCF assignments

## How to apply

Load the antiSMASH or gutSMASH output directories and extract the protein sequences of all predicted gene clusters from their GenBank files. Sketch these protein sequences with MASH MinHash sketches and cluster them pairwise at the default 0.8 similarity threshold (adjustable via the -tg flag, a fraction between 0 and 1) so that clusters above the cutoff are considered redundant. Within each similarity group, select the medoid — the member with minimal cumulative distance to the others — as the nonredundant representative; this avoids ambiguous mapping when metagenomic/metatranscriptomic reads are later mapped to representatives. Optionally run a second, more accurate redundancy-filtering round by clustering representatives into GCFs with BiG-SCAPE (requires a processed Pfam-A.hmm database, hmmpress-processed), as recommended by the README. Write the representative collection as BiG-MAP.GCF.fna (mapping reference), BiG-MAP.GCF.bed (core-region extraction), and the BiG-MAP.GCs.json / BiG-MAP.GCF.json GCF dictionaries.

## Related tools

- **BiG-MAP** (Pipeline whose bigmap-family module implements the redundancy filtering and medoid representative selection) — https://github.com/medema-group/BiG-MAP
- **MASH** (MinHash sketching of extracted protein sequences to compute pairwise distances and cluster gene clusters at the 0.8 similarity cutoff)
- **BiG-SCAPE** (Second redundancy-filtering round: groups representative gene clusters into GCFs; recommended by the README for more accurate filtering) — https://github.com/medema-group/BiG-SCAPE/wiki
- **gutSMASH** (Upstream predictor of metabolic gene clusters whose GenBank output folders serve as the input to bigmap-family) — https://github.com/victoriapascal/gutsmash

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- The output collection file exists and contains exactly one nonredundant representative per MASH similarity cluster
- Each representative corresponds to the medoid of its cluster (minimal cumulative MASH distance to other members)
- BiG-MAP.GCF.fna contains only representative sequences and can be used as the mapping reference in bigmap-map
- BiG-MAP.GCF.json GCF assignments are consistent with the -tg threshold used (0 → every cluster its own family, 1 → one family containing all clusters)
- When BiG-SCAPE is enabled, Pfam-A.hmm is present, uncompressed, and hmmpress-processed before the run

## Limitations

- The 0.8 MASH similarity cutoff is a default that may over- or under-cluster; it must be tuned per dataset via -tg
- BiG-SCAPE-based second-round filtering is optional and requires extra installation plus a maintained Pfam database; the README notes the tool has only been tested on Ubuntu
- Redundant clusters collapsed to a single medoid lose member-level resolution in downstream abundance profiling

## Evidence

- [readme] The first redundancy filtering step is performed by MASH, that by default uses 0.8 sequence similarity cut-off but can be changed as desired.: [evidence span withheld — anchor status: external_doc]
- [readme] This step is optional but to make use of the second redundancy filtering step in the bigmap-family module, install BiG-SCAPE: [evidence span withheld — anchor status: external_doc]
- [intro] BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19): "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
