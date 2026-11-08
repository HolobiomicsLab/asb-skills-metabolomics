---
name: microbiome-beta-diversity-ordination
description: Use when a paired microbiome/metabolomics study needs a genotype- or
  treatment-level test of community composition from a compositional feature table
  (e.
when_to_use_negative:
- When the question is differential abundance of individual taxa rather than whole-community
  composition (beta diversity) — use a differential-abundance method instead.
- 'When longitudinal fecal samples are analyzed at the single-animal level: the Longitudinal
  cohort lacks animal IDs (collected randomly per cage), so single-animal developmental
  beta-diversity analysis is not possible.'
- 'When pooling 3xTg and 5xFAD samples into a single PERMANOVA test: the article demonstrates
  the two models must be tested separately, as only 3xTg shows a significant genotype
  effect.'
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: phylo-RPCA
  role: 'Beta-diversity ordination method: computes phylogenetic Robust Aitchison
    PCA distances on the compositional feature table before PERMANOVA testing'
- name: PERMANOVA
  role: Statistical test applied to the phylo-RPCA distances to obtain pseudo-F and
    p-value for genotype (and sex as control) within each mouse-model study
- name: Qiita
  role: Public deposit from which the fecal shotgun metagenomics feature table and
    sample metadata are retrieved
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:ba2b5fa2585487fcd05dd4b38b52cd16678aad5da3de5d67caf60a8a567af4af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PERMANOVA
  - Qiita
  - phylo-RPCA
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# microbiome-beta-diversity-ordination

## Summary

Assess whether gut microbial community composition (beta diversity) differs across conditions by ordinating shotgun-metagenomic feature tables with phylogenetic Robust Aitchison PCA (phylo-RPCA) and testing group separation with PERMANOVA, as applied to 3xTg and 5xFAD Alzheimer's disease mouse fecal samples.

## When to use

Use when a paired microbiome/metabolomics study needs a genotype- or treatment-level test of community composition from a compositional feature table (e.g. Qiita deposit of 666 paired SPF fecal samples), specifically to answer whether beta diversity differs by genotype within one mouse model at a time.

## When NOT to use

- When the question is differential abundance of individual taxa rather than whole-community composition (beta diversity) — use a differential-abundance method instead.
- When longitudinal fecal samples are analyzed at the single-animal level: the Longitudinal cohort lacks animal IDs (collected randomly per cage), so single-animal developmental beta-diversity analysis is not possible.
- When pooling 3xTg and 5xFAD samples into a single PERMANOVA test: the article demonstrates the two models must be tested separately, as only 3xTg shows a significant genotype effect.

## Inputs

- Fecal shotgun metagenomics feature table (n=666 paired SPF fecal samples) from the Qiita deposit
- Sample metadata (genotype, study, sex, cohort) matching the feature table sample IDs

## Outputs

- Phylo-RPCA ordination distances for the subsetted Sacrifice-cohort SPF samples
- permanova_results.csv with pseudo-F and p-value per study and factor (e.g. genotype in 3xTg, genotype and sex in 5xFAD)

## How to apply

Retrieve the fecal shotgun metagenomics feature table (n=666 paired SPF fecal samples) and sample metadata (genotype, study, sex, cohort) from the Qiita deposit, then subset to the Sacrifice-cohort SPF samples and stratify them by study (3xTg vs 5xFAD). Compute beta diversity via phylogenetic Robust Aitchison PCA (phylo-RPCA) on the compositional feature table, which is the appropriate compositional approach for these data. Run PERMANOVA on genotype within the 3xTg study using the phylo-RPCA distances and record pseudo-F and p-value; run the same test within the 5xFAD study, additionally testing sex in 5xFAD as a control factor. Write pseudo-F and p-values per study and factor to a results CSV. The rationale for stratifying by study is that the two models must be tested separately — 3xTg shows a strong genotype effect (pseudo-F = 28, p < 0.001) whereas 5xFAD shows none (pseudo-F = 0.14, p = 0.885) — so pooling would mask the model-specific signal.

## Related tools

- **phylo-RPCA** (Beta-diversity ordination method: computes phylogenetic Robust Aitchison PCA distances on the compositional feature table before PERMANOVA testing)
- **PERMANOVA** (Statistical test applied to the phylo-RPCA distances to obtain pseudo-F and p-value for genotype (and sex as control) within each mouse-model study)
- **Qiita** (Public deposit from which the fecal shotgun metagenomics feature table and sample metadata are retrieved)

## Evaluation signals

- 3xTg Sacrifice-cohort genotype test should reproduce the reported significant effect: pseudo-F ≈ 28, p < 0.001
- 5xFAD genotype test should reproduce the reported non-significant result: pseudo-F = 0.14, p = 0.885
- 5xFAD sex-effect control should land near pseudo-F = 3.9, p = 0.026
- permanova_results.csv must contain a row per study–factor combination (3xTg/genotype, 5xFAD/genotype, 5xFAD/sex) with pseudo-F and p-value columns populated
- Sample count in the ordination must match the Sacrifice-cohort SPF subset of the 666 paired fecal samples, stratified by study before testing

## Limitations

- Differences between Longitudinal and Sacrifice cohorts in microbiome data are attributed to differing sample collection (fresh fecal vs. colon after sacrifice), so the beta-diversity test should be restricted to the Sacrifice cohort when reproducing these values.
- 3xTg and WT mice were bred and housed separately before the start of the experiment, a potential confound that could influence community composition independent of genotype.
- The workflow relies on a manual expert-review tooling path rather than an automated pipeline, so parameter choices (e.g. exact phylo-RPCA settings) are not fully specified in the article.

## Evidence

- [methods] phylo-RPCA genotype PERMANOVA workflow: [evidence span withheld — anchor status: not_found]
- [results] reported genotype effect: [evidence span withheld — anchor status: not_found]
- [abstract] dataset scope: "complemented by shotgun metagenomics sequencing data (666 samples)"
- [discussion] cohort collection caveat: "Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample"
