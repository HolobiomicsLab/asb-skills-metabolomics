---
name: diabetes-cohort-two-group-comparison
description: Use when two subject groups (e.g. DM2 vs NGT) from a microarray expression
  matrix must be compared for pathways with significantly different mean activity
  levels, via t statistics on PLAGE/SVD pathway activities.
when_to_use_negative:
- The goal is gene-level (not pathway-level) differential expression; PLAGE activity
  levels replace individual gene expression as the comparison basis
- The comparison involves more than two groups or a continuous phenotype rather than
  a pairwise diabetic vs non-diabetic contrast
- Pathway membership gene sets are unavailable or not the versions distributed via
  the PLAGE website, since KEGG/Biocarta definitions 'continue to expand and evolve'
tools:
- name: PLAGE
  role: Web program that computes pathway activity levels via SVD from the standardized
    expression matrix and performs the pathway-level t-statistic comparisons between
    cohorts
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:5c151dde38f90f76ab6af48ac2c4be28ab462f54abb792ca5f68a9090c920ee0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# diabetes-cohort-two-group-comparison

## Summary

Statistical comparison of pathway activity levels between a type 2 diabetic (DM2) cohort and a normal-glucose-tolerance (NGT) cohort in the Mootha et al. muscle expression data, using PLAGE SVD-derived activities, per-pathway t statistics, and a 10,000-permutation max-t null distribution.

## When to use

Use when two subject groups (e.g. DM2 vs NGT) from a microarray expression matrix must be compared for pathways with significantly different mean activity levels, via t statistics on PLAGE/SVD pathway activities.

## When NOT to use

- The goal is gene-level (not pathway-level) differential expression; PLAGE activity levels replace individual gene expression as the comparison basis
- The comparison involves more than two groups or a continuous phenotype rather than a pairwise diabetic vs non-diabetic contrast
- Pathway membership gene sets are unavailable or not the versions distributed via the PLAGE website, since KEGG/Biocarta definitions 'continue to expand and evolve'

## Inputs

- Mootha et al. muscle gene expression matrix (17 DM2, 17 NGT, 9 IGT samples; 43 profiles total)
- ~400 predefined KEGG/Biocarta pathway gene sets (versions used available at the PLAGE website)

## Outputs

- dm2_ngt_pathway_pvalues.csv containing pathway names, t statistics, and permutation-derived p-values
- Ranked list of pathways by significance for the DM2 vs NGT comparison

## How to apply

Load the Mootha et al. muscle expression matrix (17 DM2, 17 NGT, 9 IGT; all 43 profiles used) and standardize each gene's expression to zero mean and unit variance over samples. For each of the ~400 KEGG/Biocarta pathways, form the restricted expression matrix and compute the first SVD metagene via PLAGE, taking the first-row coefficient as the pathway activity level in each sample. For the DM2 vs NGT comparison, calculate a t statistic on mean activity levels for every pathway. Run 10,000 label permutations, recording the maximum t statistic across pathways per permutation, and convert each pathway's t statistic to a p-value as the fraction of maximum statistics exceeding it. Rank pathways by significance and write dm2_ngt_pathway_pvalues.csv with pathway names, t statistics, and p-values.

## Related tools

- **PLAGE** (Web program that computes pathway activity levels via SVD from the standardized expression matrix and performs the pathway-level t-statistic comparisons between cohorts) — http://dulci.biostat.edu/pathways

## Evaluation signals

- The top-ranked pathway must be Biocarta 'Activation of cAMP-dependent protein kinase, PKA (protein kinase A)', upregulated in DM2, with p ≈ 0.4
- The result must match the paper's reported finding that no pathway was significant in the DM2 vs NGT comparison
- Output CSV must contain exactly three fields per pathway: name, t statistic, and p-value
- p-values must be computed against a null of 10,000 permutations, each contributing the maximum t statistic across pathways
- Per-gene expression must be standardized to zero mean and unit variance over the 43 samples before SVD metagene extraction

## Limitations

- In the DM2 vs NGT comparison no pathway reached significance; the most significant (PKA activation, upregulated in DM2) had only p = 0.4 under the permutation null
- The KEGG and Biocarta pathway versions used are only available at the PLAGE website; no specific version numbers or accessions are stated
- The t-statistic formula is referenced as an equation placeholder and is not rendered in the section
- The statistical comparison is restricted to pairwise group contrasts of mean activity level, not multi-group designs

## Evidence

- [abstract] A web program PLAGE for pathway-level analysis is provided: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
- [discussion] Pairwise comparison objective: "we mainly perform pairwise comparisons to identify pathways for which the mean activity level in one group (e.g., diabetic) is significantly different from that in the other (non-diabetic)"
- [discussion] Permutation procedure: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] Pathway activity as comparison basis: "the approach quantifies the level of activity of each pathway within each sample and uses the activity levels as the basis for making comparisons"
- [intro] Demonstration dataset: "We demonstrate the approach using the same expression data analyzed by Mootha et al. [9] in their study of type 2 diabetes"
