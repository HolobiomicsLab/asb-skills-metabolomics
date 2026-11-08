---
name: differential-pathway-activity-t-testing
description: Use when comparing two sample groups (e.g., DM2 vs NGT microarray profiles)
  to find pathways whose PLAGE-derived activity levels differ significantly, using
  t statistics calibrated by max-t label permutations.
when_to_use_negative:
- The goal is differential expression of individual genes rather than pathway-level
  activity — the article positions pathway-based results as complementary to individual-gene-focused
  analyses
- No predefined pathway sets (KEGG/Biocarta memberships) are available for the genes
  in the expression matrix
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PLAGE
  role: 'Web program that implements the SVD-based pathway analysis: computes pathway
    activity levels and the permutation-calibrated pairwise comparisons used by this
    skill'
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:bda7d1e4910495e8aa647aedb7933a94e18c8b8df518b5cf9fae054d52879235
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# differential-pathway-activity-t-testing

## Summary

Tests whether a pathway's PLAGE activity level differs between two sample groups by computing a t statistic on SVD-derived pathway activities and calibrating significance with a max-t label-permutation null across ~400 KEGG/Biocarta pathways.

## When to use

Use when comparing two sample groups (e.g., DM2 vs NGT microarray profiles) to find pathways whose PLAGE-derived activity levels differ significantly, using t statistics calibrated by max-t label permutations.

## When NOT to use

- The goal is differential expression of individual genes rather than pathway-level activity — the article positions pathway-based results as complementary to individual-gene-focused analyses
- No predefined pathway sets (KEGG/Biocarta memberships) are available for the genes in the expression matrix

## Inputs

- Gene expression matrix from the Mootha et al. type 2 diabetes study (17 DM2, 17 NGT, 9 IGT profiles; all 43 used for the comparison)
- Pathway membership definitions (~400 pathways and complexes from Biocarta and KEGG)

## Outputs

- Ranked table of pathways with t statistics and permutation-calibrated p-values (dm2_ngt_pathway_pvalues.csv)
- Pathway activity levels per sample (first SVD metagene coefficient per pathway)

## How to apply

Standardize each gene's expression to zero mean and unit variance over samples, then for each of the ~400 KEGG/Biocarta pathways form the restricted expression matrix and compute the first SVD metagene via PLAGE, taking the first-row coefficient as the pathway activity in each sample. For the two-group comparison (e.g., DM2 vs NGT), calculate a t statistic on mean activity levels for every pathway. Build the null by running 10,000 label permutations, recording the maximum t statistic across pathways per permutation, and convert each pathway's t statistic to a p-value as the fraction of maximum statistics exceeding it. Rank pathways by significance and write the results table (pathway names, t statistics, p-values, e.g., dm2_ngt_pathway_pvalues.csv). The max-t permutation scheme accounts for testing many pathways at once, which is why the top pathway in the DM2 vs NGT comparison only reached p = 0.4 and no pathway was called significant.

## Related tools

- **PLAGE** (Web program that implements the SVD-based pathway analysis: computes pathway activity levels and the permutation-calibrated pairwise comparisons used by this skill) — http://dulci.biostat.edu/pathways

## Evaluation signals

- Each pathway's p-value equals the fraction of the 10,000 per-permutation maximum t statistics that exceed the pathway's observed t statistic
- Genes are standardized to zero mean and unit variance over samples before SVD
- Reproducing the DM2 vs NGT comparison must rank the Biocarta 'Activation of cAMP-dependent protein kinase, PKA (protein kinase A)' pathway (upregulated in DM2) as most significant with p ≈ 0.4
- Output table contains one row per tested pathway with pathway name, t statistic, and p-value columns
- Exactly 10,000 label permutations were performed, with the maximum t statistic across pathways recorded per permutation

## Limitations

- No pathway was significant in the DM2 vs NGT comparison (best p = 0.4); the max-t correction across ~400 pathways makes individual significance hard to attain in small cohorts
- KEGG and Biocarta pathway definitions evolve over time; the versions used in the article are only available via the PLAGE website and no version numbers or accessions are stated
- The t-statistic formula is referenced but not rendered in the extracted section, so implementation details must be sourced elsewhere

## Evidence

- [abstract] A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
- [discussion] we mainly perform pairwise comparisons to identify pathways for which the mean activity level in one group (e.g., diabetic) is significantly different from that in the other (non-diabetic): "we mainly perform pairwise comparisons to identify pathways for which the mean activity level in one group (e.g., diabetic) is significantly different from that in the other (non-diabetic)"
- [discussion] we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] The SVD-based pathways analysis method has been implemented in a web program [11] called PLAGE (pathway level analysis of gene expression).: "The SVD-based pathways analysis method has been implemented in a web program [11] called PLAGE (pathway level analysis of gene expression)."
