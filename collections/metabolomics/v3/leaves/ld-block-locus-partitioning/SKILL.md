---
name: ld-block-locus-partitioning
description: Use when counting independent genetic association signals from PLINK2
  GWAS summary statistics, e.g. comparing genome-wide significant locus counts between
  pre- and post-QC versions of NMR biomarker data.
when_to_use_negative:
- The input is raw UK Biobank NMR field data that has not yet passed through ukbnmr
  remove_technical_variation() and the goal is QC correction rather than locus counting
- GWAS has not been run or summary statistics lack P-values, so there are no significant
  SNPs to partition
- You need SNP-level rather than locus-level association summaries (SNPs within a
  block are deliberately collapsed)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: PLINK2
  role: Runs the generalized linear model GWAS whose significant SNPs (P < 5e-8) are
    then partitioned into LD-block loci
- name: ukbnmr
  role: Supplies the post-QC biomarker concentrations (remove_technical_variation())
    whose GWAS results are compared against the original data at the locus level
  repo: https://github.com/sritchie73/ukbnmr
- name: R
  role: Environment for loading biomarker data with data.table and tabulating locus
    counts into gwas_loci_counts.csv
- name: data.table
  role: Fast reading/writing (fread/fwrite) of biomarker matrices and summary statistics
    tables
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:f0a5a179e68839721ddfd02ff2f1df5f2a8ff96e6e137ebfbfcb1659f73ebef3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - PLINK2
  - R
  - data.table
  - ukbnmr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# ld-block-locus-partitioning

## Summary

Partitions genome-wide significant SNPs (P < 5e-8) from GWAS summary statistics into independent loci using LD blocks, enabling accurate locus-level counting of genetic associations before and after technical-variation removal.

## When to use

Use when counting independent genetic association signals from PLINK2 GWAS summary statistics, e.g. comparing genome-wide significant locus counts between pre- and post-QC versions of NMR biomarker data.

## When NOT to use

- The input is raw UK Biobank NMR field data that has not yet passed through ukbnmr remove_technical_variation() and the goal is QC correction rather than locus counting
- GWAS has not been run or summary statistics lack P-values, so there are no significant SNPs to partition
- You need SNP-level rather than locus-level association summaries (SNPs within a block are deliberately collapsed)

## Inputs

- PLINK2 GWAS summary statistics files (.glm.linear or similar) for each biomarker/data-state combination
- LD block definitions (genomic block boundaries) for partitioning significant SNPs
- Post-QC biomarker matrix produced by ukbnmr remove_technical_variation() (nmr_biomarker_data.csv)
- Covariate file with age, sex, genotyping chip, and 10 genetic principal components for the unrelated White British subset

## Outputs

- Per-biomarker counts of genome-wide significant independent loci (e.g. gwas_loci_counts.csv) comparing pre- vs. post-QC states
- List of genome-wide significant SNPs (P < 5e-8) grouped into LD-block-defined loci

## How to apply

Run PLINK2 generalized linear model GWAS on the biomarker of interest — restricted to the unrelated 'White British' subset of 111,450 participants — over ~8.5 million common SNPs filtered to MAF > 1% and INFO > 0.4, adjusting for age, sex, genotyping chip, and the first 10 genetic principal components. Extract SNPs passing the genome-wide significance threshold of P < 5e-8 from the summary statistics file. Partition these significant SNPs into independent loci by assigning each to its LD block, so correlated SNPs within a block collapse to a single locus rather than inflating the count. Tabulate locus counts per biomarker per data state (original vs. post ukbnmr remove_technical_variation) to quantify the gain in detection power; for the source study this yielded alanine 30→47 loci and albumin 32→43 loci.

## Related tools

- **PLINK2** (Runs the generalized linear model GWAS whose significant SNPs (P < 5e-8) are then partitioned into LD-block loci)
- **ukbnmr** (Supplies the post-QC biomarker concentrations (remove_technical_variation()) whose GWAS results are compared against the original data at the locus level) — https://github.com/sritchie73/ukbnmr
- **R** (Environment for loading biomarker data with data.table and tabulating locus counts into gwas_loci_counts.csv)
- **data.table** (Fast reading/writing (fread/fwrite) of biomarker matrices and summary statistics tables)

## Examples

```
plink2 --bfile ukb_white_british_unrelated --glm --pheno alanine_postqc.tsv --pheno-name Alanine --covar covariates.tsv --covar-name age,sex,chip,PC1-PC10 --out alanine_postqc
```

## Evaluation signals

- gwas_loci_counts.csv shows post-QC locus counts exceeding pre-QC counts: 47 vs 30 for alanine and 43 vs 32 for albumin
- All counted loci derive from SNPs meeting the genome-wide significance threshold P < 5e-8
- SNPs falling within the same LD block are counted as a single locus, not aggregated at the chromosome level
- The covariate model used for the GWAS matches the specification: age, sex, genotyping chip, and 10 genetic PCs in the 111,450-participant unrelated White British subset
- Re-running the partitioning on identical summary statistics reproduces the same locus counts

## Limitations

- LD-block partitioning collapses correlated SNPs, so it reports locus counts rather than independent causal variant counts
- Interpretation assumes the gain in loci (e.g. 30→47 for alanine) reflects increased power after removal of technical variation, not changes in the sample subset
- The comparison depends on using the same filtered SNP set (MAF > 1%, INFO > 0.4) and covariate model for both pre- and post-QC GWAS

## Evidence

- [other] Workflow source card: locus partitioning step: [evidence span withheld — anchor status: not_found]
- [other] Workflow source card: expected validation counts: [evidence span withheld — anchor status: not_found]
- [readme] README: source of the post-QC data being compared: "The `remove_technical_variation()` function removes additional technical variation present in the UK Biobank NMR data"
- [abstract] Article abstract: rationale that QC increases genetic signal: "and demonstrate that it increases signal for genetic and epidemiological studies of the NMR metabolic biomarker data in UK Biobank"
