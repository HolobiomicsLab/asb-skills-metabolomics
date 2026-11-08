---
name: metagenomic-sample-metadata-stratification
description: Use when a Qiita-deposited fecal shotgun metagenomics feature table (n=666
  paired SPF fecal samples) with metadata fields genotype/study/sex/cohort must be
  subset (e.g., Sacrifice cohort, per-study) before PERMANOVA or beta-diversity testing.
when_to_use_negative:
- 'The analysis targets Longitudinal-cohort fecal samples: they were collected randomly
  per cage without animal IDs, so single-animal metadata linkage (and genotype-stratified
  testing) is not possible.'
- The question concerns untargeted metabolomics feature abundance (e.g., serum carnitines)
  rather than metagenomic community composition tested via beta diversity/PERMANOVA.
- The input is already a single-study, single-cohort table with no confounding metadata
  dimensions, so no stratification is needed before testing.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: Qiita
  role: Repository hosting the fecal shotgun metagenomics feature table (n=666 paired
    SPF fecal samples) and sample metadata (genotype, study, sex, cohort) that are
    retrieved and then stratified.
- name: manual expert review
  role: Validation of the stratified PERMANOVA results against the reported pseudo-F
    and p-value targets (3xTg genotype pseudo-F ≈ 28, p < 0.001; 5xFAD genotype pseudo-F
    = 0.14, p = 0.885; 5xFAD sex pseudo-F ≈ 3.9, p = 0.026).
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:c308f7898d2ccdfc3ae12832f1d9fcfd81c8b6e56165ca335e80abd8a1aa4720
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Qiita
  - manual expert review
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# metagenomic-sample-metadata-stratification

## Summary

Stratify a shotgun metagenomics feature table and its sample metadata by experimental factors (genotype, study, sex, cohort) before community-level hypothesis testing, so that PERMANOVA on phylo-RPCA beta diversity distances is computed within homogeneous strata (e.g., the 3xTg Sacrifice cohort) rather than across confounded groupings.

## When to use

Use when a Qiita-deposited fecal shotgun metagenomics feature table (n=666 paired SPF fecal samples) with metadata fields genotype/study/sex/cohort must be subset (e.g., Sacrifice cohort, per-study) before PERMANOVA or beta-diversity testing.

## When NOT to use

- The analysis targets Longitudinal-cohort fecal samples: they were collected randomly per cage without animal IDs, so single-animal metadata linkage (and genotype-stratified testing) is not possible.
- The question concerns untargeted metabolomics feature abundance (e.g., serum carnitines) rather than metagenomic community composition tested via beta diversity/PERMANOVA.
- The input is already a single-study, single-cohort table with no confounding metadata dimensions, so no stratification is needed before testing.

## Inputs

- Fecal shotgun metagenomics feature table (n=666 paired SPF fecal samples) from the Qiita deposit
- Sample metadata table with genotype, study, sex, and cohort columns

## Outputs

- Study-stratified metadata subsets (e.g., 3xTg Sacrifice-cohort SPF samples, 5xFAD Sacrifice-cohort SPF samples)
- permanova_results.csv with pseudo-F and p-values per study and factor
- phylo-RPCA beta-diversity distance matrix per stratified cohort

## How to apply

Retrieve the fecal shotgun metagenomics feature table (n=666 paired SPF fecal samples) and associated sample metadata (genotype, study, sex, cohort) from the Qiita deposit, then subset to the Sacrifice-cohort SPF samples and stratify them by study (3xTg vs 5xFAD) so each PERMANOVA stratum shares one study and one cohort. On each stratum, compute beta diversity via phylogenetic Robust Aitchison PCA (phylo-RPCA) on the compositional feature table, then run PERMANOVA on the target factor (genotype; use sex in the 5xFAD study as a control factor) and record pseudo-F and p-values per study and factor to permanova_results.csv. Stratification by study is essential because the article reports that the 3xTg genotype effect (pseudo-F = 28, p < 0.001) is absent in 5xFAD (pseudo-F = 0.14, p = 0.885), so pooling studies would mask the signal. Keep Longitudinal and Sacrifice samples separate: cohort differences are attributed to differing sample collection (fresh fecal vs. colon after sacrifice).

## Related tools

- **Qiita** (Repository hosting the fecal shotgun metagenomics feature table (n=666 paired SPF fecal samples) and sample metadata (genotype, study, sex, cohort) that are retrieved and then stratified.)
- **manual expert review** (Validation of the stratified PERMANOVA results against the reported pseudo-F and p-value targets (3xTg genotype pseudo-F ≈ 28, p < 0.001; 5xFAD genotype pseudo-F = 0.14, p = 0.885; 5xFAD sex pseudo-F ≈ 3.9, p = 0.026).)

## Examples

```
import pandas as pd; md = pd.read_csv('sample_metadata.tsv', sep='\t'); md_3xTg = md[(md.study == '3xTg') & (md.cohort == 'Sacrifice')]
```

## Evaluation signals

- The 3xTg Sacrifice-cohort genotype PERMANOVA reproduces pseudo-F ≈ 28, p < 0.001.
- The 5xFAD genotype PERMANOVA reproduces the reported non-significant result, pseudo-F = 0.14, p = 0.885.
- The 5xFAD sex control test yields pseudo-F ≈ 3.9, p = 0.026.
- permanova_results.csv exists and contains one pseudo-F and p-value row per study and factor combination.
- Stratified sample subsets derive only from the Sacrifice-cohort SPF samples of the n=666 paired SPF fecal feature table, with no Longitudinal-cohort leakage.

## Limitations

- Differences between the Longitudinal and Sacrifice cohorts were observed for both microbiome and metabolomics data and are attributed to differing sample collection (fresh fecal vs. colon after sacrifice), so cross-cohort pooling is invalid.
- 3xTg and WT mice were bred and housed separately before the start of the experiment, a potential confound for genotype effects even after study-level stratification.
- Longitudinal fecal samples lack animal IDs (collected randomly per cage), preventing single-animal metadata-driven analysis.

## Evidence

- [other] Retrieve the fecal shotgun metagenomics feature table (n=666 paired SPF fecal samples) and associated sample metadata (genotype, study, sex, cohort) from the Qiita deposit.: [evidence span withheld — anchor status: not_found]
- [other] Subset to the Sacrifice-cohort SPF samples and stratify them by study (3xTg study vs 5xFAD study).: [evidence span withheld — anchor status: not_found]
- [discussion] Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample: "Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample"
- [discussion] Fecal samples in the Longitudinal study were collected randomly from animals in the same cage and they did not have an associated animal ID so investigation of single animal development was not: "Fecal samples in the Longitudinal study were collected randomly from animals in the same cage and they did not have an associated animal ID so investigation of single animal development was not"
