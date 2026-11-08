---
name: plink2-linear-regression-gwas
description: Use when testing SNPs for association with UK Biobank NMR biomarker concentrations,
  or when quantifying whether ukbnmr technical-variation QC increases the number of
  genome-wide significant loci (P < 5e-8) versus the original data.
when_to_use_negative:
- The NMR data has not yet been processed with ukbnmr::remove_technical_variation()
  — no prefiltering or partial QC should be applied before running that function,
  so pre-QC GWAS must use the raw extracted concentrations as-is.
- The biomarkers of interest are derived ratios or composite biomarkers — adjustment
  can introduce new unwanted variation into these, and the article recommends filtering
  to the 107 non-derived biomarkers and re-deriving them post-adjustment.
- The analysis goal is epidemiological (non-genetic) association testing rather than
  comparing genetic association signal before and after technical variation removal.
tools:
- name: PLINK2
  role: Runs the generalized linear model GWAS on each biomarker per data state, adjusting
    for age, sex, genotyping chip, and 10 genetic PCs
- name: ukbnmr
  role: Provides remove_technical_variation() to produce the post-QC NMR biomarker
    concentrations and extract_biomarkers() for the original data
  repo: sritchie73/ukbnmr
- name: R
  role: Environment for running ukbnmr, extracting biomarkers, and tabulating locus
    counts
  repo: sritchie73/ukbnmr
- name: data.table
  role: Fast reading/writing of biomarker and summary statistics CSV files via fread()
    and fwrite()
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:aac5dbc655fa875c9972eb2a7561542951a23de7441a89ffc03bb72220753b74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - PLINK2
  - R
  - data.table
  - ukbnmr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# plink2-linear-regression-gwas

## Summary

Runs PLINK2 generalized linear model GWAS on UK Biobank NMR metabolic biomarker concentrations (e.g. alanine, albumin), comparing genome-wide significant locus counts before versus after ukbnmr removal of technical variation to quantify gains in genetic association power.

## When to use

Use when testing SNPs for association with UK Biobank NMR biomarker concentrations, or when quantifying whether ukbnmr technical-variation QC increases the number of genome-wide significant loci (P < 5e-8) versus the original data.

## When NOT to use

- The NMR data has not yet been processed with ukbnmr::remove_technical_variation() — no prefiltering or partial QC should be applied before running that function, so pre-QC GWAS must use the raw extracted concentrations as-is.
- The biomarkers of interest are derived ratios or composite biomarkers — adjustment can introduce new unwanted variation into these, and the article recommends filtering to the 107 non-derived biomarkers and re-deriving them post-adjustment.
- The analysis goal is epidemiological (non-genetic) association testing rather than comparing genetic association signal before and after technical variation removal.

## Inputs

- Exported UK Biobank NMR field data (Table Exporter CSV/TSV) extractable via ukbnmr::extract_biomarkers()
- Post-QC biomarker data from ukbnmr::remove_technical_variation()
- PLINK2 genotype files for ~8.5 million common SNPs (MAF > 1%, INFO > 0.4)
- Covariate file: age, sex, genotyping chip, first 10 genetic principal components
- Sample subset: 111,450 unrelated 'White British' ancestry participants

## Outputs

- Four PLINK2 GWAS summary statistics files (alanine and albumin, original and post-QC data)
- Genome-wide significant SNP lists at P < 5e-8 partitioned into independent loci by LD blocks
- gwas_loci_counts.csv comparing pre- vs post-QC locus counts per biomarker

## How to apply

Load the original and post-QC (remove_technical_variation()) NMR biomarker concentrations with data.table, and restrict to the unrelated 'White British' ancestry subset of 111,450 participants. Assemble a covariate file with age, sex, genotyping chip, and the first 10 genetic principal components. Run four PLINK2 generalized linear model GWAS (alanine-original, alanine-post-QC, albumin-original, albumin-post-QC) over the ~8.5 million common SNPs filtered to MAF > 1% and INFO > 0.4. Select SNPs at P < 5e-8 from each summary-statistics file and partition them into independent loci using LD blocks, then tabulate pre- versus post-QC locus counts per biomarker. The rationale is that removal of technical variation (plate row/column effects, within-spectrometer drift, outlier plates) increases signal for genetic studies without changing the overall correlation structure.

## Related tools

- **PLINK2** (Runs the generalized linear model GWAS on each biomarker per data state, adjusting for age, sex, genotyping chip, and 10 genetic PCs)
- **ukbnmr** (Provides remove_technical_variation() to produce the post-QC NMR biomarker concentrations and extract_biomarkers() for the original data) — sritchie73/ukbnmr
- **R** (Environment for running ukbnmr, extracting biomarkers, and tabulating locus counts) — sritchie73/ukbnmr
- **data.table** (Fast reading/writing of biomarker and summary statistics CSV files via fread() and fwrite())

## Examples

```
processed <- remove_technical_variation(exported); fwrite(processed$biomarkers, file="nmr_biomarker_data.csv")
```

## Evaluation signals

- gwas_loci_counts.csv must show post-QC locus counts exceeding pre-QC counts: 47 vs 30 for alanine and 43 vs 32 for albumin
- Only SNPs passing MAF > 1% and INFO > 0.4 filters (~8.5 million common SNPs) appear in the summary statistics
- Genome-wide significant SNP selection uses exactly P < 5e-8 and loci are partitioned using LD blocks into independent loci
- Covariate model in all four GWAS runs includes age, sex, genotyping chip, and the first 10 genetic principal components
- Cohort is restricted to the 111,450 unrelated 'White British' participants, identical across all four GWAS runs

## Limitations

- No filtering or treatment of outlier samples is performed by the ukbnmr R package when removing technical variation; sensitivity analysis to inclusion/exclusion of QC-flagged samples is recommended.
- Regressing on technical covariates may remove biological variation of interest, and other sources of technical variation may not be captured by the technical covariates examined.
- Only a small fraction of samples (2.6%) could be assessed for reproducibility via blind duplicates, limiting direct validation of QC improvements.

## Evidence

- [abstract] abstract: "and demonstrate that it increases signal for genetic and epidemiological studies of the NMR metabolic biomarker data in UK Biobank"
- [methods] methods: "Each biomarker is adjusted for drift over time within each of the six spectrometers."
- [discussion] discussion: "regressing on these covariates may remove biological variation of interest."
- [readme] readme: "The `remove_technical_variation()` function removes additional technical variation present in the UK Biobank NMR data"
