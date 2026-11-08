---
name: phylogenetic-compositional-pca-analysis
description: Use when you have a paired metagenomics feature table (e.g. n=666 fecal
  SPF samples) and sample metadata, and need to test whether community composition
  differs by a factor such as genotype.
when_to_use_negative:
- Input is already a rarefied or non-compositional distance matrix — phylo-RPCA's
  compositional and phylogenetic assumptions would be redundant
- Analysis targets longitudinal fecal samples lacking animal IDs (collected randomly
  per cage), where single-animal tracking is not possible
- The question concerns taxonomic differential abundance rather than community-wide
  beta diversity, where model-based approaches are more appropriate
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: manual expert review
  role: source card lists the workflow as executed by expert manual review rather
    than a named software tool
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:73d51f60a0a98d9fa183606a227fc05bedf4a9ac9615cda0b5bc1d70c10513b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - manual expert review
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# phylogenetic-compositional-pca-analysis

## Summary

Compute beta diversity on compositional shotgun metagenomics feature tables using phylogenetic Robust Aitchison PCA (phylo-RPCA), then test group effects (e.g. genotype) via PERMANOVA on the resulting distance matrix.

## When to use

Use when you have a paired metagenomics feature table (e.g. n=666 fecal SPF samples) and sample metadata, and need to test whether community composition differs by a factor such as genotype.

## When NOT to use

- Input is already a rarefied or non-compositional distance matrix — phylo-RPCA's compositional and phylogenetic assumptions would be redundant
- Analysis targets longitudinal fecal samples lacking animal IDs (collected randomly per cage), where single-animal tracking is not possible
- The question concerns taxonomic differential abundance rather than community-wide beta diversity, where model-based approaches are more appropriate

## Inputs

- fecal shotgun metagenomics feature table (n=666 paired SPF fecal samples) from the Qiita deposit
- sample metadata (genotype, study, sex, cohort)

## Outputs

- permanova_results.csv with pseudo-F and p-values per study and factor (genotype, sex)
- phylo-RPCA distance matrix for the subsetted samples

## How to apply

Retrieve the fecal shotgun metagenomics feature table and sample metadata (genotype, study, sex, cohort) from the Qiita deposit. Subset to the Sacrifice-cohort SPF samples and stratify them by study (3xTg vs 5xFAD). Compute beta diversity via phylogenetic Robust Aitchison PCA (phylo-RPCA) on the compositional feature table, then run PERMANOVA on genotype within each study using the phylo-RPCA distances, recording pseudo-F and p-values. Validate against reported values: 3xTg genotype pseudo-F ≈ 28, p < 0.001; 5xFAD genotype pseudo-F = 0.14, p = 0.885.

## Related tools

- **manual expert review** (source card lists the workflow as executed by expert manual review rather than a named software tool)

## Evaluation signals

- 3xTg genotype PERMANOVA reproduces pseudo-F ≈ 28, p < 0.001
- 5xFAD genotype PERMANOVA reproduces pseudo-F = 0.14, p = 0.885
- 5xFAD sex effect near pseudo-F = 3.9, p = 0.026
- Distance matrix computed on the Sacrifice-cohort SPF subset, not the full 666-sample table
- permanova_results.csv contains one row per study × factor combination with pseudo-F and p-value columns

## Limitations

- Differences between Longitudinal and Sacrifice cohorts in microbiome data are attributed to differing sample collection (fresh fecal vs. colon after sacrifice), so cohort must be stratified before testing
- 3xTg and WT mice were bred and housed separately before the experiment, a potential confound in genotype effects
- Feature detection can be influenced by study-specific extraction methods, workflows, and instrumentation

## Evidence

- [results] 3xTg genotype significant, 5xFAD genotype not: [evidence span withheld — anchor status: not_found]
- [discussion] cohort differences caveat: "Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample"
- [discussion] breeding confound caveat: "Finally, 3xTg and WT mice were bred and housed separately before the start of the experiment"
