---
name: pearson-correlation-of-pathway-activity-with-clinical-phenotype
description: Use when you have a gene expression matrix paired with a continuous clinical
  variable (e.g. 1-hour OGTT blood glucose) and need to test which of ~400 predefined
  KEGG/Biocarta pathways' activity levels correlate with the phenotype.
when_to_use_negative:
- The analysis goal is comparing mean pathway activity between two sample groups (e.g.
  diabetic vs non-diabetic) rather than correlating with a continuous phenotype —
  PLAGE's pairwise t-statistic comparison is the appropriate variant.
- No predefined pathway gene sets (KEGG/Biocarta) are available and de novo pathway
  discovery is required.
- The phenotype is a binary label rather than a continuous variable — Pearson correlation
  of activity levels is not the right test.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: PLAGE
  role: Web program implementing the SVD-based pathway activity computation used to
    derive the activity levels that are correlated with the phenotype
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:4e92064510048ba23ab9efe99bff022120ac80ea7047285907871ba986dc36e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# pearson-correlation-of-pathway-activity-with-clinical-phenotype

## Summary

Compute PLAGE pathway activity levels via SVD of standardized pathway gene-expression matrices, then Pearson-correlate each pathway's activity levels with a continuous clinical phenotype (e.g. 1-hour OGTT blood glucose in the Mootha et al. type 2 diabetes dataset), with permutation-based p-value estimation.

## When to use

Use when you have a gene expression matrix paired with a continuous clinical variable (e.g. 1-hour OGTT blood glucose) and need to test which of ~400 predefined KEGG/Biocarta pathways' activity levels correlate with the phenotype.

## When NOT to use

- The analysis goal is comparing mean pathway activity between two sample groups (e.g. diabetic vs non-diabetic) rather than correlating with a continuous phenotype — PLAGE's pairwise t-statistic comparison is the appropriate variant.
- No predefined pathway gene sets (KEGG/Biocarta) are available and de novo pathway discovery is required.
- The phenotype is a binary label rather than a continuous variable — Pearson correlation of activity levels is not the right test.

## Inputs

- Genes-by-samples gene expression matrix (e.g. the Mootha et al. type 2 diabetes expression data, 43 samples)
- Continuous phenotype vector aligned to samples (e.g. 1-hour OGTT blood glucose concentrations)
- Pathway gene sets (~400 pathways from KEGG and Biocarta)

## Outputs

- Per-pathway Pearson correlation coefficient r between pathway activity level and phenotype
- Per-pathway permutation p-value (10,000 permutations of phenotype values)
- Ranked table of pathways by correlation strength and significance (e.g. KEGG Oxidative Phosphorylation, Biocarta PKA activation, KEGG ATP synthesis)

## How to apply

Standardize each gene's expression to zero mean and unit variance across the samples (43 samples in the type 2 diabetes study), and for each of the ~400 KEGG/Biocarta pathways form the genes-by-samples matrix Y, compute its singular value decomposition Y = WDC, and take the first-row coefficients cj of C as the pathway activity levels (PLAGE). Compute the Pearson correlation r between each pathway's activity levels and the clinical phenotype (e.g. 1-hour OGTT glucose concentrations). Estimate p-values by permuting the phenotype values 10,000 times, recording the extremal (minimum and maximum) correlations per permutation, and taking the fraction of extremal values stronger than the observed correlation in the corresponding direction. The rationale, per Tomfohr et al., is that features defined by groups of genes are more robust to variation at the level of individual genes than single-gene analyses. Optionally run the analysis through the PLAGE web program rather than reimplementing the SVD pipeline.

## Related tools

- **PLAGE** (Web program implementing the SVD-based pathway activity computation used to derive the activity levels that are correlated with the phenotype) — http://dulci.biostat.edu/pathways

## Evaluation signals

- Computed Pearson r for KEGG Oxidative Phosphorylation vs 1-hour OGTT glucose matches the reported Table 1 value within ±0.05 (reported r = -0.52, p = 0.03)
- Computed r for Biocarta PKA activation (reported +0.47, p = 0.06) and KEGG ATP synthesis (reported -0.43, p = 0.17) match reported values within ±0.05 with the same sign
- Significance ordering preserved: OxPhos most significant, PKA activation second, ATP synthesis least significant
- Permutation p-values (10,000 permutations) fall within the sampling uncertainty of the reported p-values
- Gene expression standardization (zero mean, unit variance per gene across samples) was applied before forming each pathway matrix Y and computing its SVD

## Limitations

- KEGG and Biocarta pathways continue to expand and evolve; results depend on the specific pathway versions used, and no version numbers are stated — the versions used for the article are only available at the PLAGE website.
- Permutation p-values carry sampling uncertainty, so p-values should be compared against reported values only within that uncertainty, not exactly.
- The t-statistic formula and cited reference details (e.g. Mootha et al. [9]) are placeholders in the article section and must be resolved from the full text.

## Evidence

- [abstract] A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
- [intro] Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD).: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [discussion] we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11].: "The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11]."
