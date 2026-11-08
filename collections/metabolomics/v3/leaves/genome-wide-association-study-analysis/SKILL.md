---
name: genome-wide-association-study-analysis
description: Use when testing whether ukbnmr's removal of technical variation from
  NMR biomarker data increases genetic association power, or when counting genome-wide
  significant loci (P < 5e-8) for a metabolomics trait.
when_to_use_negative:
- The NMR data have not yet passed through remove_technical_variation() — the README
  states no prefiltering of samples or columns should be performed before that function,
  so GWAS input preparation must follow QC, not precede it
- The trait is a derived biomarker ratio or composite — the article recommends filtering
  to the 107 non-derived biomarkers and re-deriving the rest post-adjustment, since
  adjustment can introduce new unwanted variation into ratios
- The analysis is not about UK Biobank NMR metabolomics traits or lacks per-sample
  biomarker concentrations keyed to genotypes (eid)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: PLINK2
  role: Runs the generalized linear model GWAS per biomarker per data state over common
    SNPs with MAF > 1% and INFO > 0.4, adjusted for age, sex, genotyping chip, and
    10 PCs
- name: ukbnmr
  role: Provides remove_technical_variation() to generate the post-QC biomarker concentrations
    compared against the original data, and extract_biomarkers() to build GWAS phenotype
    files
  repo: https://github.com/sritchie73/ukbnmr
- name: R
  role: Environment for running the ukbnmr package and preparing biomarker/covariate
    files with data.table's fread()/fwrite()
- name: data.table
  role: Fast reading and writing of CSV biomarker and QC-flag files used as GWAS phenotypes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:a616dab66d1079eb46321538b66b8001d3667c837b664546b52f86bd66da5ec8
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
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# genome-wide-association-study-analysis

## Summary

Runs GWAS of UK Biobank NMR metabolic biomarkers (e.g., alanine, albumin) with PLINK2 generalized linear models on original versus ukbnmr QC-corrected concentrations to quantify the gain in genetic association signal after removal of technical variation.

## When to use

Use when testing whether ukbnmr's removal of technical variation from NMR biomarker data increases genetic association power, or when counting genome-wide significant loci (P < 5e-8) for a metabolomics trait.

## When NOT to use

- The NMR data have not yet passed through remove_technical_variation() — the README states no prefiltering of samples or columns should be performed before that function, so GWAS input preparation must follow QC, not precede it
- The trait is a derived biomarker ratio or composite — the article recommends filtering to the 107 non-derived biomarkers and re-deriving the rest post-adjustment, since adjustment can introduce new unwanted variation into ratios
- The analysis is not about UK Biobank NMR metabolomics traits or lacks per-sample biomarker concentrations keyed to genotypes (eid)

## Inputs

- UK Biobank NMR biomarker concentration table (original field data extracted via ukbnmr::extract_biomarkers, eid + visit_index keyed)
- Post-QC NMR biomarker data produced by ukbnmr::remove_technical_variation()
- PLINK binary genotype files for ~8.5 million common SNPs (MAF > 1%, INFO > 0.4)
- Sample ancestry/unrelatedness subset definition ('White British' unrelated, n=111,450)
- Covariate file: age, sex, genotyping chip, first 10 genetic principal components

## Outputs

- PLINK2 GLM summary statistics per biomarker per data state (alanine-original, alanine-post-QC, albumin-original, albumin-post-QC)
- Genome-wide significant SNP list (P < 5e-8) per GWAS
- Independent locus counts partitioned by LD blocks (gwas_loci_counts.csv)

## How to apply

Load the original NMR biomarker data and the post-QC output of ukbnmr's remove_technical_variation() using data.table in R, extracting the biomarker concentrations of interest (e.g., alanine and albumin). Restrict to the unrelated 'White British' genetic ancestry subset of 111,450 participants and build a covariate file with age, sex, genotyping chip, and the first 10 genetic principal components. Run one PLINK2 GLM per biomarker per data state over the ~8.5 million common SNPs filtered to MAF > 1% and INFO > 0.4, adjusting for the covariates. Select SNPs with P < 5e-8 from each summary statistics file and partition them into independent loci using LD blocks, then tabulate pre- versus post-QC locus counts. The rationale is that the QC procedure regresses out sample preparation time, shipment plate row/column effects, spectrometer batch effects, and within-spectrometer drift, so any increase in significant loci reflects restored genetic signal rather than technical artifacts.

## Related tools

- **PLINK2** (Runs the generalized linear model GWAS per biomarker per data state over common SNPs with MAF > 1% and INFO > 0.4, adjusted for age, sex, genotyping chip, and 10 PCs)
- **ukbnmr** (Provides remove_technical_variation() to generate the post-QC biomarker concentrations compared against the original data, and extract_biomarkers() to build GWAS phenotype files) — https://github.com/sritchie73/ukbnmr
- **R** (Environment for running the ukbnmr package and preparing biomarker/covariate files with data.table's fread()/fwrite())
- **data.table** (Fast reading and writing of CSV biomarker and QC-flag files used as GWAS phenotypes)

## Examples

```
plink2 --bfile ukb_white_british --pheno nmr_pheno.txt --pheno-name alanine --covar covariates.txt --covar-name age,sex,chip,PC1-PC10 --maf 0.01 --glm --out alanine_original
```

## Evaluation signals

- gwas_loci_counts.csv shows post-QC locus counts exceeding pre-QC counts: 47 vs 30 loci for alanine and 43 vs 32 for albumin
- Each GWAS is restricted to the 111,450-participant unrelated White British subset with identical covariates (age, sex, chip, 10 PCs) across original and post-QC runs, so locus-count differences are attributable to QC
- Summary statistics are filtered to common variants (MAF > 1%, INFO > 0.4) before significance testing at P < 5e-8
- Significant SNPs are partitioned into independent loci via LD blocks rather than counted as raw SNPs
- Post-QC CV% of blind duplicate measurements decreases and R2 increases, consistent with the article's QC validation

## Limitations

- No filtering or treatment of outlier samples is performed by the ukbnmr R package when removing technical variation, so outlier plates are set to missing and residual outliers may remain in GWAS phenotypes
- Regressing on technical covariates may remove biological variation of interest, and other sources of technical variation may not be captured by the covariates examined
- Only a small fraction of samples (2.6%) could be assessed for reproducibility via blind duplicates, limiting QC validation coverage
- Strong inter-correlation among biomarkers challenges multiple testing strategy choice and multivariable modelling
- Sensitivity analysis to inclusion/exclusion of QC-flagged samples and measurements is recommended

## Evidence

- [abstract] Increased genetic association power after QC: "and demonstrate that it increases signal for genetic and epidemiological studies of the NMR metabolic biomarker data in UK Biobank"
- [methods] Source of post-QC phenotype data: "The `remove_technical_variation()` function removes additional technical variation present in the UK Biobank NMR data"
- [readme] No prefiltering before QC step: "Note that the no prefiltering of samples or columns should be performed prior to running this function: the algorithms used for removing technical variation expect all the data to be present."
- [discussion] Outlier samples untouched by package: "no filtering or treatment of outlier samples is performed by the ukbnmr R package when removing technical variation."
