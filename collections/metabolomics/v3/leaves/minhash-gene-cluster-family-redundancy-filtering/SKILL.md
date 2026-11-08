---
name: minhash-gene-cluster-family-redundancy-filtering
description: Use when many antiSMASH or gutSMASH-predicted gene clusters (e.g., 1,849
  BGCs from 170 S. mutans genomes) must be reduced to a nonredundant representative
  set before metagenomic read mapping.
when_to_use_negative:
- The anti- or gutSMASH output folders for the genomes have not yet been generated
  — bigmap-family requires these pre-computed predictions as input and cannot mine
  genomes itself
- The gene clusters have already been reduced to a nonredundant GCF set and per-sample
  mapping with bigmap-map is the next step, so a second filtering pass would be redundant
- The analysis does not involve mapping metagenomic/metatranscriptomic reads to gene
  clusters (e.g., the goal is only to enumerate BGCs per genome), in which case the
  redundancy collapse for unambiguous mapping is unnecessary
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: bigmap-family (BiG-MAP)
  role: Module that performs MinHash (MASH) redundancy filtering and GCF grouping
    of antiSMASH/gutSMASH gene cluster predictions
  repo: https://github.com/medema-group/BiG-MAP
- name: BiG-SCAPE
  role: Optional second redundancy-filtering round that groups BGCs into gene cluster
    families (GCFs) more accurately than MASH alone
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: gutSMASH
  role: Predicts the metabolic gene clusters whose output folders are the input to
    bigmap-family
  repo: https://github.com/victoriapascal/gutsmash
- name: antiSMASH
  role: Mines the reference genomes for biosynthetic gene clusters; its output directories
    feed bigmap-family (as in the 1,849 BGCs from 170 S. mutans genomes)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:dc191584544fdcd5acfe6243b456d641ed57d852758755eda27d54a286807861
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/BiG-MAP
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - antiSMASH
  - bigmap-analyse
  - bigmap-family
  - bigmap-family (BiG-MAP)
  - bigmap-map
  - gutSMASH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# MinHash-based gene cluster redundancy filtering and GCF grouping

## Summary

A redundancy-filtering step in the BiG-MAP pipeline that collapses many antiSMASH/gutSMASH-predicted biosynthetic gene clusters (BGCs) into nonredundant representatives and gene cluster families (GCFs) using MinHash (MASH) sequence similarity, optionally refined with BiG-SCAPE. It is essential to avoid ambiguous mapping of metagenomic/metatranscriptomic reads to overlapping cluster sequences downstream.

## When to use

Use when many antiSMASH or gutSMASH-predicted gene clusters (e.g., 1,849 BGCs from 170 S. mutans genomes) must be reduced to a nonredundant representative set before metagenomic read mapping.

## When NOT to use

- The anti- or gutSMASH output folders for the genomes have not yet been generated — bigmap-family requires these pre-computed predictions as input and cannot mine genomes itself
- The gene clusters have already been reduced to a nonredundant GCF set and per-sample mapping with bigmap-map is the next step, so a second filtering pass would be redundant
- The analysis does not involve mapping metagenomic/metatranscriptomic reads to gene clusters (e.g., the goal is only to enumerate BGCs per genome), in which case the redundancy collapse for unambiguous mapping is unnecessary

## Inputs

- Output directories from antiSMASH or gutSMASH runs (predicted gene clusters per genome), required as input to bigmap-family
- List of reference genome assemblies to mine (e.g., 170 S. mutans genomes from Tang et al. and Liu et al. cohorts)
- Processed Pfam-A.hmm database (uncompressed from Interpro and hmmpress-ed) when using the BiG-SCAPE second filtering round
- Locally installed BiG-SCAPE (conda install bigscape) for the optional second redundancy-filtering step

## Outputs

- BiG-MAP.GCF.bed — bedfile to extract core regions in bigmap-map
- BiG-MAP.GCF.fna — reference FASTA of representative gene cluster sequences for mapping WGS reads
- BiG-MAP.GCs.json — dictionary containing the MinHash-filtered gene clusters
- BiG-MAP.GCF.json — dictionary containing the BiG-SCAPE GCF assignments
- A nonredundant representative set of gene clusters (e.g., 41 nonredundant clusters from 1,849 predicted BGCs)

## How to apply

First run antiSMASH or gutSMASH on each input genome so that per-genome output folders of predicted gene clusters exist, then feed those folders to the bigmap-family module of BiG-MAP with -D. MASH performs the first MinHash-based redundancy filter, by default keeping gene clusters with protein-sequence similarity below the 0.8 cut-off (adjustable via -tg); a similarity threshold of 0 would put every cluster in its own family, while 1 would merge all clusters into one family. A second, more accurate redundancy-filtering round is strongly recommended using BiG-SCAPE, which requires a processed Pfam-A.hmm database (uncompressed and hmmpress-ed from the Interpro download). Housekeeping-gene similarity grouping is controlled separately via -th (default 0.1). The module emits BiG-MAP.GCF.bed, BiG-MAP.GCF.fna (the reference sequences used by bigmap-map for read mapping), and dictionaries BiG-MAP.GCs.json / BiG-MAP.GCF.json listing cluster-to-GCF assignments.

## Related tools

- **bigmap-family (BiG-MAP)** (Module that performs MinHash (MASH) redundancy filtering and GCF grouping of antiSMASH/gutSMASH gene cluster predictions) — https://github.com/medema-group/BiG-MAP
- **BiG-SCAPE** (Optional second redundancy-filtering round that groups BGCs into gene cluster families (GCFs) more accurately than MASH alone) — https://github.com/medema-group/BiG-SCAPE/wiki
- **gutSMASH** (Predicts the metabolic gene clusters whose output folders are the input to bigmap-family) — https://github.com/victoriapascal/gutsmash
- **antiSMASH** (Mines the reference genomes for biosynthetic gene clusters; its output directories feed bigmap-family (as in the 1,849 BGCs from 170 S. mutans genomes))

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- All four expected output files are produced: BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, and BiG-MAP.GCF.json
- The nonredundant representative count is substantially smaller than the raw BGC count (in the paper's S. mutans case study, 1,849 antiSMASH-predicted BGCs reduced to 41 representatives)
- The MASH similarity cut-off used is recorded (default 0.8 via -tg) and no two retained representative clusters exceed that similarity threshold
- The GCF.fna representatives are unique enough that bigmap-map mapping of the 96 oral metagenome samples yields unambiguous per-sample core-coverage profiles without cross-mapping artifacts
- When BiG-SCAPE is enabled, the BiG-MAP.GCF.json dictionary reflects the BiG-SCAPE GCF families rather than only MinHash clusters

## Limitations

- MASH's default 0.8 similarity cut-off alone is less accurate; the README strongly recommends adding the BiG-SCAPE second redundancy-filtering round
- Using BiG-SCAPE requires the latest processed Pfam-A.hmm.gz from the Interpro website, uncompressed and processed with hmmpress from HMMER
- Similarity thresholds are tunable and behavior is degenerate at extremes: -tg 0 puts every cluster in its own family, -tg 1 merges all clusters into one family
- BiG-MAP has only been tested on Ubuntu
- MinHash filtering is on protein-sequence similarity; overly permissive thresholds can merge functionally distinct clusters, while overly strict thresholds leave redundant clusters that cause ambiguous read mapping downstream

## Evidence

- [intro] MinHash redundancy filtering avoids ambiguous mapping: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [readme] bigmap-family groups clusters into GCFs by sequence similarity: [evidence span withheld — anchor status: external_doc]
- [readme] MASH first filter uses 0.8 default cut-off: [evidence span withheld — anchor status: external_doc]
- [readme] BiG-SCAPE recommended for more accurate filtering: [evidence span withheld — anchor status: external_doc]
- [readme] GCF.fna is the read-mapping reference: [evidence span withheld — anchor status: external_doc]
