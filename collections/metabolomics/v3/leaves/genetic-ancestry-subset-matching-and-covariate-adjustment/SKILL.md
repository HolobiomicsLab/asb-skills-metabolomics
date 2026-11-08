---
name: genetic-ancestry-subset-matching-and-covariate-adjustment
description: Use when running genome-wide association studies on UK Biobank NMR biomarker
  concentrations (e.
when_to_use_negative:
- The biomarker data has not yet been processed with ukbnmr remove_technical_variation()
  — no prefiltering of samples or columns should be done before running that function
- The goal is QC of the NMR data itself rather than assessing genetic association
  signal before/after QC
- The cohort is not UK Biobank NMR metabolomics data with available genotypes and
  ancestry/PC information
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: PLINK2
  role: Runs the generalized linear model GWAS for each biomarker in each data state
    (original vs post-QC)
- name: ukbnmr
  role: R package providing remove_technical_variation() to produce the post-QC biomarker
    data and extract_biomarkers() to pull alanine/albumin from raw UK Biobank field
    data
  repo: https://github.com/sritchie73/ukbnmr
- name: R
  role: Environment for running ukbnmr and assembling phenotype/covariate files
- name: data.table
  role: Fast reading/writing (fread/fwrite) of biomarker, covariate, and summary statistics
    CSV files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:4a4a1b7bd4631ab4f05901f7e3fbe4062164f0b31eb1297004f59a4e2f1f0311
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
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# genetic-ancestry-subset-matching-and-covariate-adjustment

## Summary

A GWAS design skill for UK Biobank NMR biomarkers: restrict analysis to the unrelated 'White British' genetic ancestry subset (111,450 participants) and run PLINK2 generalized linear models adjusting for age, sex, genotyping chip, and 10 genetic principal components, on both original and ukbnmr post-QC biomarker data to quantify power gained from removing technical variation.

## When to use

Use when running genome-wide association studies on UK Biobank NMR biomarker concentrations (e.g. alanine, albumin) where ancestry restriction to unrelated White British participants and covariate adjustment (age, sex, chip, 10 PCs) are needed, and when comparing association signal before vs after ukbnmr technical variation removal.

## When NOT to use

- The biomarker data has not yet been processed with ukbnmr remove_technical_variation() — no prefiltering of samples or columns should be done before running that function
- The goal is QC of the NMR data itself rather than assessing genetic association signal before/after QC
- The cohort is not UK Biobank NMR metabolomics data with available genotypes and ancestry/PC information

## Inputs

- Original UK Biobank NMR biomarker concentration data (alanine, albumin) extracted via ukbnmr extract_biomarkers()
- Post-QC biomarker data produced by ukbnmr remove_technical_variation()
- List of unrelated 'White British' genetic ancestry participants (111,450 individuals)
- Covariate file with age, sex, genotyping chip, and first 10 genetic principal components
- Genotype data: 8.5 million common SNPs (MAF > 1%, INFO > 0.4)

## Outputs

- Four GWAS summary statistics files (PLINK2 .glm output): alanine-original, alanine-post-QC, albumin-original, albumin-post-QC
- Lists of genome-wide significant SNPs (P < 5e-8) partitioned into independent LD-block loci per GWAS
- gwas_loci_counts.csv tabulating locus counts before vs after QC (alanine: 30 vs 47; albumin: 32 vs 43)

## How to apply

Extract alanine and albumin concentrations from both the original NMR biomarker data and the ukbnmr remove_technical_variation() output using data.table in R. Restrict to the unrelated 'White British' genetic ancestry subset of 111,450 participants and build a covariate file with age, sex, genotyping chip, and the first 10 genetic principal components. Run four PLINK2 generalized linear model GWAS (alanine-original, alanine-post-QC, albumin-original, albumin-post-QC) over 8.5 million common SNPs filtered to MAF > 1% and INFO > 0.4. Select genome-wide significant SNPs at P < 5e-8 from each summary statistics file and partition them into independent loci using LD blocks, then tabulate locus counts before versus after QC. The rationale is that removing plate-, spectrometer-, and drift-related technical variation should increase genetic signal, as demonstrated by the article's finding that QC 'increases signal for genetic and epidemiological studies'.

## Related tools

- **PLINK2** (Runs the generalized linear model GWAS for each biomarker in each data state (original vs post-QC))
- **ukbnmr** (R package providing remove_technical_variation() to produce the post-QC biomarker data and extract_biomarkers() to pull alanine/albumin from raw UK Biobank field data) — https://github.com/sritchie73/ukbnmr
- **R** (Environment for running ukbnmr and assembling phenotype/covariate files)
- **data.table** (Fast reading/writing (fread/fwrite) of biomarker, covariate, and summary statistics CSV files)

## Examples

```
plink2 --bfile ukb_common --glm --pheno nmr_pheno.txt --pheno-name Alanine --covar covariates.txt --covar-name age,sex,chip,PC1-PC10 --maf 0.01 --out alanine_postqc
```

## Evaluation signals

- gwas_loci_counts.csv must show post-QC locus counts exceeding pre-QC counts: 47 vs 30 for alanine and 43 vs 32 for albumin
- Genome-wide significance threshold applied is P < 5e-8 in all four GWAS
- SNP set restricted to common variants with MAF > 1% and INFO > 0.4 (approximately 8.5 million SNPs)
- All four analyses are restricted to the same 111,450 unrelated White British participants and identical covariate set (age, sex, genotyping chip, 10 PCs)
- Independent loci are counted after partitioning significant SNPs by LD blocks, not by raw SNP counts

## Limitations

- Regressing on technical covariates may remove biological variation of interest, so post-QC GWAS signal gains should be interpreted with care
- There may be other sources of technical variation not captured by the technical covariates examined
- remove_technical_variation() requires all data present — no prefiltering of samples or columns beforehand — and needs at least 32 GB of RAM and about an hour to run

## Evidence

- [other] Restrict to the unrelated 'White British' genetic ancestry subset of 111,450 participants and assemble the covariate file containing age, sex, genotyping chip, and the first 10 genetic principal components.: [evidence span withheld — anchor status: not_found]
- [other] Run four GWAS with PLINK2 generalized linear models over common SNPs filtered to MAF > 1% and INFO > 0.4.: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract: the QC procedure increases signal for genetic studies.: "and demonstrate that it increases signal for genetic and epidemiological studies of the NMR metabolic biomarker data in UK Biobank"
- [readme] README: no prefiltering before remove_technical_variation().: "Note that the no prefiltering of samples or columns should be performed prior to running this function: the algorithms used for removing technical variation expect all the data to be present."
- [methods] Article methods: the remove_technical_variation() function.: "The `remove_technical_variation()` function removes additional technical variation present in the UK Biobank NMR data"
