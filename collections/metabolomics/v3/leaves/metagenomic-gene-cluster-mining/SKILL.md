---
name: metagenomic-gene-cluster-mining
description: Use when starting from microbial genome assemblies and needing predicted
  MGCs/BGCs reduced to representative GCFs as mapping targets for metagenomic or metatranscriptomic
  abundance analysis.
when_to_use_negative:
- The gene clusters are already predicted and dereplicated and you only need read
  mapping/abundance — use bigmap-map/bigmap-download directly instead.
- The dataset is already an abundance/feature table (e.g., RPKM per sample per cluster);
  mining genomes again would not apply.
- You need pathway-level functional profiling from reads alone without reference genomes
  (tools like HUMAnN2, MetaPath, FMAP, Metatrans are the alternatives the paper contrasts
  against).
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3697
tools:
- name: gutSMASH
  role: Predicts specialized primary metabolic gene clusters (MGCs) from anaerobic/microbiome
    genomes; run on the 1,440 HMOD oral reference genomes before family grouping
  repo: https://github.com/victoriapascal/gutsmash
- name: BiG-MAP
  role: Provides the bigmap-family module that performs MinHash redundancy filtering
    and GCF grouping from gutSMASH/antiSMASH output folders
  repo: https://github.com/medema-group/BiG-MAP
- name: BiG-SCAPE
  role: Optional second redundancy-filtering round inside bigmap-family for more accurate
    GCF assignment; requires processed Pfam-A.hmm
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: ncbi/sra-tools
  role: Supports SRA retrieval of shotgun metagenomic reads (via bigmap-download/fastq-dump)
    that will be mapped to the mined clusters
  repo: https://github.com/ncbi/sra-tools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:5da81eae718e56f7e2cc641ba2ca6dfc0af5eb4596656314ba5263eca0737533
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/victoriapascal/gutsmash
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - bigmap-analyse
  - bigmap-map
  - gutSMASH
  - ncbi/sra-tools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metagenomic-gene-cluster-mining

## Summary

Predict metabolic gene clusters (MGCs) from microbial reference genomes with gutSMASH/antiSMASH and group them into non-redundant gene cluster families (GCFs) with BiG-MAP's MinHash and BiG-SCAPE filtering, so metagenomic reads can be mapped to distinct cluster representatives for abundance profiling.

## When to use

Use when starting from microbial genome assemblies and needing predicted MGCs/BGCs reduced to representative GCFs as mapping targets for metagenomic or metatranscriptomic abundance analysis.

## When NOT to use

- The gene clusters are already predicted and dereplicated and you only need read mapping/abundance — use bigmap-map/bigmap-download directly instead.
- The dataset is already an abundance/feature table (e.g., RPKM per sample per cluster); mining genomes again would not apply.
- You need pathway-level functional profiling from reads alone without reference genomes (tools like HUMAnN2, MetaPath, FMAP, Metatrans are the alternatives the paper contrasts against).

## Inputs

- Microbial genome assemblies or existing antiSMASH/gutSMASH output folders (one per genome)
- Pfam-A.hmm.gz downloaded from Interpro and processed with hmmpress (for BiG-SCAPE filtering)
- SRA accession list file for downstream shotgun metagenome download (e.g., BioProject Acc_list.txt)
- BiG-SCAPE installation location (optional, for the second redundancy filtering step)

## Outputs

- BiG-MAP.GCF.bed (bedfile to extract core regions in bigmap-map)
- BiG-MAP.GCF.fna (reference FASTA to map WGS reads against)
- BiG-MAP.GCs.json (dictionary of grouped gene clusters)
- BiG-MAP.GCF.json (dictionary of BiG-SCAPE gene cluster families)
- Representative GCF sequences used later to compute RPKM abundance profiles across samples

## How to apply

First run gutSMASH (for specialized primary metabolism) or antiSMASH on the genomes; in the paper's case, gutSMASH on 1,440 HMOD oral reference genomes yielded 3,352 predicted MGCs. Collect the per-genome prediction output folders and pass them to bigmap-family, which performs MinHash (MASH)-based redundancy filtering at a default sequence similarity cut-off of 0.8 (adjustable via -tg, with a separate housekeeping-gene threshold -th defaulting to 0.1). Strongly recommended: enable the second filtering round with BiG-SCAPE by supplying its install location (-b) and a processed Pfam-A.hmm database (hmmpress-ed Pfam-A.hmm.gz from Interpro). The module outputs BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, and BiG-MAP.GCF.json, which downstream bigmap-map uses to extract core regions and map WGS reads. Rationale: redundant near-identical clusters cause ambiguous read mappings, so redundancy filtering before mapping is essential to accurate RPKM profiles.

## Related tools

- **gutSMASH** (Predicts specialized primary metabolic gene clusters (MGCs) from anaerobic/microbiome genomes; run on the 1,440 HMOD oral reference genomes before family grouping) — https://github.com/victoriapascal/gutsmash
- **BiG-MAP** (Provides the bigmap-family module that performs MinHash redundancy filtering and GCF grouping from gutSMASH/antiSMASH output folders) — https://github.com/medema-group/BiG-MAP
- **BiG-SCAPE** (Optional second redundancy-filtering round inside bigmap-family for more accurate GCF assignment; requires processed Pfam-A.hmm) — https://github.com/medema-group/BiG-SCAPE/wiki
- **ncbi/sra-tools** (Supports SRA retrieval of shotgun metagenomic reads (via bigmap-download/fastq-dump) that will be mapped to the mined clusters) — https://github.com/ncbi/sra-tools

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- bigmap-family emits all four expected artifacts: BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, and BiG-MAP.GCF.json
- Redundancy-filtering settings match the defaults (MASH similarity cut-off 0.8; -th 0.1 for housekeeping genes) unless deliberately overridden
- Cluster yield is plausible against the paper's scale: gutSMASH on 1,440 HMOD oral genomes produced 3,352 predicted MGCs before/while grouping
- Each representative GCF in BiG-MAP.GCF.fna traces back to input genome clusters in the GCs/GCF JSON dictionaries (no orphan representatives)
- Downstream mapping of reads to these representatives yields measurable RPKM values and the expected differential statistics (e.g., 246 differentially abundant clusters, healthy 5.62 vs diseased 3.38 RPKM, Kruskal-Wallis P = 0.00049 for the pdu operon)

## Limitations

- BiG-MAP has been 'only tested on Ubuntu' per the README, so installation/runtime issues may occur on other platforms
- gutSMASH/antiSMASH must be run beforehand on every genome; bigmap-family cannot predict clusters itself
- BiG-SCAPE-based second filtering is optional; skipping it leaves only MASH filtering, which the README says is less accurate
- Validation of abundance profiling was performed with simulated metagenomic data; real-sample accuracy depends on mapping to representative sequences from reference genomes or metagenomic assemblies

## Evidence

- [intro] MinHash-based redundancy filtering of gene clusters to avoid ambiguous mapping: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [readme] bigmap-family MASH threshold: [evidence span withheld — anchor status: external_doc]
- [readme] bigmap-family input type: [evidence span withheld — anchor status: external_doc]
- [abstract] BiG-MAP purpose: "This report presents a new software tool called BiG-MAP, which allows assessing gene cluster abundance and expression in microbiome samples using metagenomic and metatranscriptomic data."
