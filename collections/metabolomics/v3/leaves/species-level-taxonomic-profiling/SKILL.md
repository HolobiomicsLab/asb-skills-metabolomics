---
name: species-level-taxonomic-profiling
description: Use when you have paired-end shallow shotgun metagenomic reads (e.g.,
  fecal samples from the SPF Sacrifice cohort, n=666) plus genotype metadata, and
  you need species-level taxonomic profiles to run differential abundance testing
  (e.g., ANCOM-BC2) between 3xTg and WT mice.
when_to_use_negative:
- If the input is already a feature/abundance table (e.g., from a prior profiling
  step) and only differential testing is needed — skip re-profiling and proceed directly
  to ANCOM-BC2.
- If the goal is metabolomics feature-level analysis rather than microbial taxonomic
  profiling — this skill is not applicable.
tools:
- name: tissueMASST
  role: mass spectrometry search tool used in the broader pipeline; not required for
    taxonomic profiling itself, but available for downstream molecular feature tracking
- name: ANCOM-BC2
  role: performs species-level differential abundance testing with genotype (3xTg
    vs WT) as the primary contrast, producing log-fold changes and adjusted p-values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:8f924ab300821bc5cf03d8a883bbc7c29ad7891b3af60119367080788583ff5e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANCOM-BC2
  - tissueMASST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# species-level-taxonomic-profiling

## Summary

Profile shotgun metagenomic reads to species-level taxonomic abundance tables for the SPF Sacrifice cohort fecal samples, then run ANCOM-BC2 differential abundance testing with genotype as the primary contrast. This skill produces the species-level outputs that underlie the reported 3xTg gut microbiome dysbiosis findings (e.g., M. schaedleri enrichment and A. muciniphila depletion).

## When to use

Use when you have paired-end shallow shotgun metagenomic reads (e.g., fecal samples from the SPF Sacrifice cohort, n=666) plus genotype metadata, and you need species-level taxonomic profiles to run differential abundance testing (e.g., ANCOM-BC2) between 3xTg and WT mice.

## When NOT to use

- If the input is already a feature/abundance table (e.g., from a prior profiling step) and only differential testing is needed — skip re-profiling and proceed directly to ANCOM-BC2.
- If the goal is metabolomics feature-level analysis rather than microbial taxonomic profiling — this skill is not applicable.

## Inputs

- paired-end shallow shotgun metagenomic reads (FASTQ) for SPF Sacrifice-cohort fecal samples (3xTg and WT C57BL/6J animals)
- sample metadata table mapping each sample to its genotype (3xTg vs WT)

## Outputs

- species-level taxonomic abundance table (samples × species counts/relative abundances)
- ANCOM-BC2 species-level differential abundance table with log-fold changes and adjusted p-values, written to ancom_bc2_results.csv

## How to apply

Download the SPF Sacrifice-cohort fecal shallow shotgun metagenomics data (n=666 paired samples from 3xTg and WT C57BL/6J animals) from the public Qiita/MassIVE deposit associated with the paper. Attach the 3xTg vs WT genotype metadata to each sample. Profile the reads to a species-level taxonomic abundance table using a shotgun-based taxonomic profiler, attaching the genotype column so differential abundance models can be fit per species. Run ANCOM-BC2 at the species level with genotype as the primary contrast to estimate log-fold changes and adjusted p-values per species. Extract significant species-level results, verifying that the direction of the log-fold changes reproduces the reported Figure 5D findings (increased M. schaedleri, Helicobacter, Desulfovibrio, Escherichia, Shigella; decreased A. muciniphila strains and Bifidobacterium species in 3xTg mice), then write the full ANCOM-BC2 output table to ancom_bc2_results.csv.

## Related tools

- **tissueMASST** (mass spectrometry search tool used in the broader pipeline; not required for taxonomic profiling itself, but available for downstream molecular feature tracking)
- **ANCOM-BC2** (performs species-level differential abundance testing with genotype (3xTg vs WT) as the primary contrast, producing log-fold changes and adjusted p-values)

## Evaluation signals

- The species-level abundance table should contain ~666 samples (paired samples from the SPF Sacrifice cohort) with species-level columns.
- ANCOM-BC2 output should include log-fold change and adjusted p-value columns for every species tested.
- Direction of log-fold changes should match the reported Figure 5D findings: increased M. schaedleri and Helicobacter/Desulfovibrio/Escherichia/Shigella species, decreased A. muciniphila strains and Bifidobacterium species in 3xTg mice.
- The output file ancom_bc2_results.csv should exist and contain the full species-level results table.
- Genotype metadata should be present for every sample included in the abundance table.

## Limitations

- Longitudinal fecal samples lack animal IDs (collected randomly per cage), preventing single-animal developmental analysis.
- 3xTg and WT mice were bred and housed separately before the experiment, a potential confound.
- Feature detection across reference datasets can be influenced by study-specific extraction methods, workflows, and instrumentation; lack of detection does not necessarily indicate absence in tissue.

## Evidence

- [results] 3xTg mice harbored increased abundance of M. schaedleri and species of the Helicobacter, Desulfovibrio, Escherichia, and Shigella genera, and decreased abundance of A. muciniphila strains.: [evidence span withheld — anchor status: not_found]
- [intro] 3xTg animals were characterized by depletion of Akkermansia muciniphila and an enrichment of Mucispirillum schaedleri, Helicobacter, Desulfovibrio, Escherichia, and Shigella species: "3xTg animals were characterized by depletion of Akkermansia muciniphila and an enrichment of Mucispirillum schaedleri, Helicobacter, Desulfovibrio, Escherichia, and Shigella species"
- [full_text] Profile the reads to a species-level taxonomic abundance table with a shotgun-based taxonomic profiler and attach the 3xTg vs WT genotype metadata.: [evidence span withheld — anchor status: not_found]
