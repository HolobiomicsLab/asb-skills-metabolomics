---
name: gene-cluster-abundance-benchmarking
description: Use when validating a metagenomic gene-cluster abundance tool against
  simulated libraries with known abundances, or when choosing a redundancy-filtering
  cutoff and read-mapping mode for BiG-MAP.
when_to_use_negative:
- Real metagenomic/metatranscriptomic data without simulated ground-truth abundances
  — F1 cannot be computed and this benchmark design does not apply
- You already have validated tool defaults and only need to run the standard BiG-MAP
  pipeline (bigmap-download → bigmap-family → bigmap-map → bigmap-analyse) on real
  samples
- Your goal is differential abundance/expression analysis of gene clusters between
  conditions rather than accuracy assessment of the profiler
edam_operation: http://edamontology.org/operation_3226
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: BiG-MAP
  role: 'Pipeline under test: bigmap-family performs MASH-based redundancy filtering,
    bigmap-map maps reads with Bowtie2, bigmap-analyse quantifies abundances'
  repo: github.com/medema-group/BiG-MAP
- name: Grinder
  role: Simulates metagenomic libraries with known ground-truth gene cluster abundances
- name: Bowtie2
  role: Aligner used by bigmap-map; benchmarked in end-to-end vs local modes to select
    the best-performing preset
- name: BiG-SCAPE
  role: Optional second redundancy-filtering step within bigmap-family, recommended
    for more accurate filtering
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: seaborn
  role: Visualization library for assembling the heatmap-style F1 results grid
  repo: https://raw.githubusercontent.com/mwaskom/seaborn/master/doc/_static/logo-wide-lightbg.svg
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:e62228222202a491d911389ba0aa26d9b4a06e76e40f89967498812b2033d030
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - Bowtie2
  - Grinder
  - seaborn
  edam_operation: http://edamontology.org/operation_3226
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# gene-cluster-abundance-benchmarking

## Summary

Benchmark a gene-cluster abundance profiler (BiG-MAP) by mapping simulated metagenomic reads with known Grinder ground-truth abundances and scoring precision/recall/F1 across a MASH-similarity cutoff × Bowtie2-alignment-mode grid, to select tool defaults.

## When to use

Use when validating a metagenomic gene-cluster abundance tool against simulated libraries with known abundances, or when choosing a redundancy-filtering cutoff and read-mapping mode for BiG-MAP.

## When NOT to use

- Real metagenomic/metatranscriptomic data without simulated ground-truth abundances — F1 cannot be computed and this benchmark design does not apply
- You already have validated tool defaults and only need to run the standard BiG-MAP pipeline (bigmap-download → bigmap-family → bigmap-map → bigmap-analyse) on real samples
- Your goal is differential abundance/expression analysis of gene clusters between conditions rather than accuracy assessment of the profiler

## Inputs

- Set of reference genomes (e.g. 101 CGR genomes) with predicted gene clusters from antiSMASH/gutSMASH output folders
- Grinder-simulated metagenomic FASTQ libraries with known ground-truth gene cluster abundances
- Grid of MASH similarity cutoffs and Bowtie2 alignment modes (end-to-end vs local, fast preset)

## Outputs

- Precision, recall, and F1 values per MASH-cutoff × Bowtie2-mode combination
- Heatmap-style results table over the parameter grid
- Selected best-performing parameter combination (e.g. MASH similarity 0.8, Bowtie2 fast end-to-end) as tool defaults

## How to apply

Simulate metagenomic libraries (e.g. 8 libraries at 0.5× and 0.05× coverage with uniform, linear, power-law, and exponential abundance structures) from reference genomes using Grinder, so that per-cluster ground-truth abundances are known. Run bigmap-family to perform MASH-based MinHash redundancy filtering of the gene clusters at each MASH-similarity threshold in the grid (BiG-MAP's default is 0.8), then map each simulated library to the non-redundant representative clusters with bigmap-map, running Bowtie2 in both end-to-end and local modes (including the fast preset). Quantify per-cluster abundances with bigmap-analyse and compute precision, recall, and F1 against the Grinder ground truth for every MASH-threshold × Bowtie2-mode cell. Assemble the metrics into a heatmap-style grid and adopt the best-scoring cell as the tool defaults — in this paper, MASH similarity 0.8 with Bowtie2 'fast' end-to-end alignment.

## Related tools

- **BiG-MAP** (Pipeline under test: bigmap-family performs MASH-based redundancy filtering, bigmap-map maps reads with Bowtie2, bigmap-analyse quantifies abundances) — github.com/medema-group/BiG-MAP
- **Grinder** (Simulates metagenomic libraries with known ground-truth gene cluster abundances)
- **Bowtie2** (Aligner used by bigmap-map; benchmarked in end-to-end vs local modes to select the best-performing preset)
- **BiG-SCAPE** (Optional second redundancy-filtering step within bigmap-family, recommended for more accurate filtering) — https://github.com/medema-group/BiG-SCAPE/wiki
- **seaborn** (Visualization library for assembling the heatmap-style F1 results grid) — https://raw.githubusercontent.com/mwaskom/seaborn/master/doc/_static/logo-wide-lightbg.svg

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- All computed precision, recall, and F1 values fall within [0, 1]
- The optimal grid cell matches the reported best defaults: MASH similarity cutoff 0.8 and Bowtie2 'fast' end-to-end mode
- Every MASH-threshold × Bowtie2-mode combination yields a complete entry in the heatmap-style results table (no missing cells)
- F1 scores are computed by comparing bigmap-analyse abundance calls against the known Grinder ground-truth abundances for each simulated library
- The non-redundant representative gene cluster reference set (BiG-MAP.GCF.fna) is regenerated for each MASH cutoff in the grid

## Limitations

- BiG-MAP 'has been only tested on Ubuntu' (README), so porting the benchmark to other OSes is unverified
- MASH redundancy filtering alone may be insufficient; the README recommends adding the BiG-SCAPE second filtering step for more accurate redundancy filtering
- The MASH cutoff is configurable ('can be changed as desired'), so results and defaults are sensitive to the grid range explored

## Evidence

- [intro] We validated the tool using simulated metagenomic data and show how MGC abundance and expression levels are accurately recapitulated.: "We validated the tool using simulated metagenomic data and show how MGC abundance and expression levels are accurately recapitulated."
- [intro] MinHash filtering: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [readme] MASH cutoff: [evidence span withheld — anchor status: external_doc]
- [readme] bigmap-family input: [evidence span withheld — anchor status: external_doc]
