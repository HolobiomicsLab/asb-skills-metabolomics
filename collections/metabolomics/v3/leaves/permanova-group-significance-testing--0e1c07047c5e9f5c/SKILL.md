---
name: permanova-group-significance-testing
description: Use when testing whether compositional (beta-diversity) distances separate
  predefined sample groups — e.g. genotype or sex in SPF mouse metagenomics — and
  you need a per-factor pseudo-F and p-value rather than abundance-level tests.
when_to_use_negative:
- The question concerns alpha diversity or differential abundance of individual taxa/features
  rather than community-composition (beta-diversity) separation between groups.
- You intend to pool 3xTg and 5xFAD samples into a single PERMANOVA — the genotype
  effect was observed only in the 3xTg Sacrifice cohort, so per-study stratification
  is required.
- The data are from the Longitudinal cohort, whose fecal samples lack animal IDs and
  differ in collection method (fresh fecal vs. colon after sacrifice) from the Sacrifice
  cohort.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0092
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:6b4809ce6f7ff98403d9af8caac1b1e820222d384816c48280272c92a297d7e5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# permanova-group-significance-testing

## Summary

Runs PERMANOVA on beta-diversity distances (here phylo-RPCA distances from fecal shotgun metagenomics) to test whether predefined groups such as genotype or sex differ significantly in community composition, reporting pseudo-F and p-value per factor and study.

## When to use

Use when testing whether compositional (beta-diversity) distances separate predefined sample groups — e.g. genotype or sex in SPF mouse metagenomics — and you need a per-factor pseudo-F and p-value rather than abundance-level tests.

## When NOT to use

- The question concerns alpha diversity or differential abundance of individual taxa/features rather than community-composition (beta-diversity) separation between groups.
- You intend to pool 3xTg and 5xFAD samples into a single PERMANOVA — the genotype effect was observed only in the 3xTg Sacrifice cohort, so per-study stratification is required.
- The data are from the Longitudinal cohort, whose fecal samples lack animal IDs and differ in collection method (fresh fecal vs. colon after sacrifice) from the Sacrifice cohort.

## Inputs

- Compositional fecal shotgun metagenomics feature table from the Qiita deposit (n=666 paired SPF fecal samples)
- Sample metadata with genotype, study, sex, and cohort (Sacrifice vs Longitudinal) columns
- Phylo-RPCA (phylogenetic Robust Aitchison PCA) distance matrix computed from the compositional feature table

## Outputs

- permanova_results.csv with pseudo-F and p-value per study and factor (3xTg genotype, 5xFAD genotype, 5xFAD sex)

## How to apply

Retrieve the fecal shotgun metagenomics feature table (n=666 paired SPF fecal samples) and sample metadata (genotype, study, sex, cohort) from the Qiita deposit, subset to the Sacrifice-cohort SPF samples, and stratify by study (3xTg vs 5xFAD) because the genotype effect is study-specific. Compute beta diversity via phylogenetic Robust Aitchison PCA (phylo-RPCA) on the compositional feature table, then run PERMANOVA on genotype within each study separately using the phylo-RPCA distance matrix, adding a sex test in the 5xFAD study as a positive control. Record pseudo-F and p-value per study and factor to permanova_results.csv. The expected outcome pattern is a strong 3xTg genotype effect but a null 5xFAD genotype effect, so never pool the two studies into a single PERMANOVA test.

## Examples

```
from skbio.stats.distance import permanova; permanova(phylo_rpcra_dm, metadata, column='genotype')
```

## Evaluation signals

- 3xTg Sacrifice-cohort genotype test yields pseudo-F ≈ 28 with p < 0.001.
- 5xFAD genotype test yields pseudo-F = 0.14 with p = 0.885 (non-significant).
- 5xFAD sex test yields pseudo-F ≈ 3.9 with p ≈ 0.026, confirming the test detects a known smaller effect.
- Results file permanova_results.csv contains one row per (study, factor) with both pseudo-F and p-value columns populated.
- Distance matrix used for PERMANOVA is the phylo-RPCA distance computed on the compositional (not raw-count) feature table.

## Limitations

- The genotype effect is study-dependent: significant in the 3xTg Sacrifice cohort but absent in the 5xFAD study, so pooling or generalizing across studies is invalid.
- 3xTg and WT mice were bred and housed separately before the experiment, a potential confound that PERMANOVA cannot adjust for.
- Cohort differences (Longitudinal vs Sacrifice sample collection: fresh fecal vs. colon after sacrifice) affect both microbiome and metabolomics data, so cohort must be controlled when subsetting.

## Evidence

- [results] A significant genotype effect (PERMANOVA, pseudo-F = 28, p < 0.001) was observed in the 3xTg Sacrifice cohort, but no significant genotype effect was detected in the 5xFAD study (PERMANOVA, pseudo-F = 0.14, p = 0.885).: [evidence span withheld — anchor status: not_found]
- [methods] Compute beta diversity via phylogenetic Robust Aitchison PCA (phylo-RPCA) on the compositional feature table.: [evidence span withheld — anchor status: not_found]
- [methods] Retrieve the fecal shotgun metagenomics feature table (n=666 paired SPF fecal samples) and associated sample metadata (genotype, study, sex, cohort) from the Qiita deposit.: [evidence span withheld — anchor status: not_found]
