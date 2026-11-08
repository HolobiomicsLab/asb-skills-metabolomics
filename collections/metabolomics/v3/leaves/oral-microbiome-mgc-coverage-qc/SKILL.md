---
name: oral-microbiome-mgc-coverage-qc
description: Use when metagenomic reads have been mapped to gutSMASH/antiSMASH-predicted
  MGC or GCF representatives with bigmap-map and you must gate differential abundance
  (RPKM, Kruskal-Wallis) calls by per-sample core-region coverage, as in the 96-sample
  healthy vs. caries oral analysis.
when_to_use_negative:
- The goal is de novo prediction of metabolic gene clusters from genomes rather than
  profiling their abundance in metagenomes — run gutSMASH/antiSMASH alone instead.
- No shotgun read data or SRA accessions are available (only genome annotations or
  a precomputed abundance table without mapped reads), since coverage cannot be computed
  without mapping.
- Differential abundance is already established and you only need statistical testing,
  not per-sample coverage validation of cluster presence.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: BiG-MAP
  role: Pipeline whose bigmap-map module maps metagenomic reads to representative
    cluster sequences and computes RPKM and core-region coverage used for the 0.5
    coverage gate
  repo: https://github.com/medema-group/BiG-MAP
- name: gutSMASH
  role: Predicts specialized primary metabolic gene clusters (e.g. 3,352 MGCs from
    1,440 HMOD oral genomes) that serve as mapping targets whose coverage is QC'd
  repo: https://github.com/victoriapascal/gutsmash
- name: ncbi/sra-tools
  role: Supports retrieval of the SRA shotgun metagenomic reads that bigmap-download
    fetches for mapping
  repo: https://github.com/ncbi/sra-tools
- name: BiG-SCAPE
  role: Second redundancy-filtering round in bigmap-family so GCF representatives
    (and their core regions) are unambiguous mapping targets
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:26647820168cb0ecd0e0cdfc782770a9c4fa5dbbce0a11cb9aa0f7b857440844
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/BiG-MAP
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - bigmap-analyse
  - bigmap-map
  - gutSMASH
  - ncbi/sra-tools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# oral-microbiome-mgc-coverage-qc

## Summary

Quality-controls the presence of metabolic gene clusters (MGCs/GCFs) in oral shotgun metagenomes by inspecting core-region coverage after mapping reads with BiG-MAP, so that RPKM abundance and differential-abundance calls (e.g. Kruskal-Wallis on healthy vs. caries samples) are only trusted for samples with adequate cluster coverage (threshold 0.5).

## When to use

Use when metagenomic reads have been mapped to gutSMASH/antiSMASH-predicted MGC or GCF representatives with bigmap-map and you must gate differential abundance (RPKM, Kruskal-Wallis) calls by per-sample core-region coverage, as in the 96-sample healthy vs. caries oral analysis.

## When NOT to use

- The goal is de novo prediction of metabolic gene clusters from genomes rather than profiling their abundance in metagenomes — run gutSMASH/antiSMASH alone instead.
- No shotgun read data or SRA accessions are available (only genome annotations or a precomputed abundance table without mapped reads), since coverage cannot be computed without mapping.
- Differential abundance is already established and you only need statistical testing, not per-sample coverage validation of cluster presence.

## Inputs

- biogmap-family output directory containing BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, BiG-MAP.GCF.json
- Shotgun metagenomic FASTQ files from oral (healthy vs. caries-associated) samples downloaded from SRA via bigmap-download
- antiSMASH or gutSMASH output folders for the reference genomes (e.g. the 1,440 HMOD oral genomes)

## Outputs

- Per-sample RPKM abundance profiles of MGCs/GCFs across the 96 metagenome samples
- Per-sample core-region coverage values for each gene cluster (used against the 0.5 threshold)
- Kruskal-Wallis differential-abundance statistics (e.g. P = 0.00049, healthy 5.62 vs. diseased 3.38 RPKM for the pdu-like operon)
- Verification record such as pdu_operon_verification.json with cluster counts (246 whole set / 173 core) and coverage summaries (mean 0.77, 13/34 healthy samples below 0.5)

## How to apply

First run bigmap-family on gutSMASH output directories to produce BiG-MAP.GCF.bed (the bedfile used to extract core regions) and BiG-MAP.GCF.fna (the reference to map WGS reads against); MinHash redundancy filtering at the default 0.8 sequence-similarity cut-off prevents ambiguous mapping, and a second BiG-SCAPE-based filtering round is recommended. Then run bigmap-map to map the metagenomic reads to the representative cluster sequences and compute RPKM abundance plus core-region coverage per sample. Apply the coverage threshold of 0.5: samples whose mean core coverage falls below 0.5 are flagged as low-confidence for cluster presence (in the F0442 pdu-like operon case, 13 of 34 healthy oral samples fell below this threshold, while samples passing it averaged 0.77 core coverage). Only then interpret RPKM differences (healthy 5.62 vs. diseased 3.38, Kruskal-Wallis P = 0.00049) and record verification numbers such as counts of differentially abundant clusters (246 whole set, 173 core) in a results file like pdu_operon_verification.json.

## Related tools

- **BiG-MAP** (Pipeline whose bigmap-map module maps metagenomic reads to representative cluster sequences and computes RPKM and core-region coverage used for the 0.5 coverage gate) — https://github.com/medema-group/BiG-MAP
- **gutSMASH** (Predicts specialized primary metabolic gene clusters (e.g. 3,352 MGCs from 1,440 HMOD oral genomes) that serve as mapping targets whose coverage is QC'd) — https://github.com/victoriapascal/gutsmash
- **ncbi/sra-tools** (Supports retrieval of the SRA shotgun metagenomic reads that bigmap-download fetches for mapping) — https://github.com/ncbi/sra-tools
- **BiG-SCAPE** (Second redundancy-filtering round in bigmap-family so GCF representatives (and their core regions) are unambiguous mapping targets) — https://github.com/medema-group/BiG-SCAPE/wiki

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, and BiG-MAP.GCF.json are all emitted by bigmap-family before mapping
- All coverage values fall in [0, 1]; mean core coverage among healthy samples passing the 0.5 threshold is 0.77
- Coverage gate reproduces the reported 13 of 34 healthy oral samples below the 0.5 coverage threshold for the Streptococcus sp. F0442 pdu-like operon
- RPKM and statistics match the reported values: healthy 5.62 vs. diseased 3.38 RPKM, Kruskal-Wallis P = 0.00049, 246 (whole set) and 173 (core) differentially abundant clusters
- Verification numbers are recorded in pdu_operon_verification.json

## Limitations

- Coverage filtering is lossy: 13 of 34 healthy samples fell below the 0.5 coverage threshold for the pdu-like operon, so enrichment in healthy samples is 'not exclusively found in healthy samples'
- BiG-MAP 'has been only tested on Ubuntu', so other platforms may need extra setup
- The MinHash redundancy cut-off defaults to 0.8 and can be changed; an overly permissive threshold risks ambiguous mapping that corrupts coverage estimates
- For unpaired (e.g. ancient) read data, the optional filterBAM tool must be installed separately

## Evidence

- [readme] BiG-MAP.GCF.bed = Bedfile to extract core regions in bigmap-map: [evidence span withheld — anchor status: external_doc]
- [intro] BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [abstract] 96 metagenomic samples from healthy and caries-associated human oral microbiome samples: "we analyzed 96 metagenomic samples from healthy and caries-associated human oral microbiome samples and identified 252 gene clusters"
- [intro] Differential abundance or expression is calculated with parametric or nonparametric tests after coverage profiling: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [readme] filterBAM is needed for unpaired read data before coverage profiling: [evidence span withheld — anchor status: not_found]
