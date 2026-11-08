---
name: expression-matrix-standardization-and-subsetting
description: Use when a raw gene-by-sample expression matrix (e.g., Mootha DM2 or
  smoker airway epithelium data) must be standardized and subset to ~400 KEGG/Biocarta
  pathway gene-sets before SVD-based pathway activity computation.
when_to_use_negative:
- When the goal is gene-level ranking/enrichment itself (e.g., GSEA signal-to-noise
  ranking with 1000 permutations) rather than producing standardized pathway-level
  inputs for SVD.
- When the input is already a pathway activity score table or the gene-sets are not
  defined gene lists (e.g., de novo clustered features), since subsetting requires
  explicit KEGG/Biocarta gene memberships.
- When sample sizes are too small relative to pathway gene count for a stable restricted-matrix
  decomposition.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: PLAGE
  role: Web program implementing the SVD-based pathway analysis that consumes the
    standardized, pathway-restricted matrices; used to compute pathway activity levels
    and comparison statistics.
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:772fe4b69ce2ccf2e35a5f3d0f3b6e0e8000cdc1c8104fdd5d63c8fb70736e9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# expression-matrix-standardization-and-subsetting

## Summary

A preprocessing skill from the PLAGE (Pathway Level Analysis of Gene Expression) method: restrict a gene-by-sample microarray matrix to a pathway's genes and standardize each gene to zero mean and unit variance over samples, so that singular value decomposition of the subset matrix yields pathway 'activity' levels.

## When to use

Use when a raw gene-by-sample expression matrix (e.g., Mootha DM2 or smoker airway epithelium data) must be standardized and subset to ~400 KEGG/Biocarta pathway gene-sets before SVD-based pathway activity computation.

## When NOT to use

- When the goal is gene-level ranking/enrichment itself (e.g., GSEA signal-to-noise ranking with 1000 permutations) rather than producing standardized pathway-level inputs for SVD.
- When the input is already a pathway activity score table or the gene-sets are not defined gene lists (e.g., de novo clustered features), since subsetting requires explicit KEGG/Biocarta gene memberships.
- When sample sizes are too small relative to pathway gene count for a stable restricted-matrix decomposition.

## Inputs

- Gene-by-sample expression matrix from a two-group microarray study (e.g., Mootha et al. type 2 diabetes profiles: 43 samples = 17 DM2, 17 NGT, 9 IGT; or airway epithelium profiles: 75 subjects = 34 current, 18 former, 23 never smokers)
- Pathway gene-set definitions (~400 pathways and complexes from the KEGG and Biocarta websites)

## Outputs

- Standardized, pathway-restricted gene-by-sample matrix Y for each gene-set
- Per-pathway SVD quantities: first eigenvalue lambda, pathway gene count ng, sample count ns
- Per-pathway first-eigenvalue fraction lambda^2/(ng*ns) of total variation (e.g., ribosome ~0.53 in the smoking data, Oxidative Phosphorylation ~0.23, ~0.05 for size-matched Gaussian noise)
- Joined table of pathway first-eigenvalue fractions versus downstream rankings (e.g., eigenvalue_fraction_vs_gsea_rank.csv)

## How to apply

For each pathway gene-set from the ~400 KEGG and Biocarta collections, extract the rows (pathway genes, count ng) of the gene-by-sample matrix (columns = samples, count ns) to form the restricted matrix Y. Standardize every gene's expression levels to zero mean and unit variance across samples so that no single highly variable gene dominates the decomposition. Compute the singular value decomposition Y = WDC and record the first eigenvalue lambda together with ng and ns. Quantify pathway activity by the first metagene/sample scores, then compute the first-eigenvalue fraction lambda^2/(ng*ns) of total variation as a measure of how coordinately the pathway's genes are expressed. The rationale is that features defined by groups of genes are more robust to individual-gene variation than single-gene statistics, and the activity levels can then be compared across sample groups (e.g., diabetic vs non-diabetic) with permutation t-tests.

## Related tools

- **PLAGE** (Web program implementing the SVD-based pathway analysis that consumes the standardized, pathway-restricted matrices; used to compute pathway activity levels and comparison statistics.) — http://dulci.biostat.edu/pathways

## Examples

```
Ys = (Y - Y.mean(axis=1, keepdims=True)) / Y.std(axis=1, ddof=0, keepdims=True); W, d, C = numpy.linalg.svd(Ys); frac = d[0]**2 / (Ys.shape[0] * Ys.shape[1])
```

## Evaluation signals

- Each row of the restricted matrix Y has mean exactly 0 and variance 1 across samples after standardization.
- The first-eigenvalue fraction lambda^2/(ng*ns) for the ribosome gene-set in the smoking dataset is ~0.53 (over half of total variation), versus ~0.23 for Oxidative Phosphorylation.
- A size-matched simulated uncorrelated Gaussian gene-set (ribosome size, averaged over 1000 simulated sets) yields a fraction of ~0.05, providing a noise baseline.
- The joined output table (e.g., eigenvalue_fraction_vs_gsea_rank.csv) contains one row per pathway with ng, ns, lambda, the fraction, and the GSEA ES/rank in each of the four two-group comparisons.
- Downstream permutation testing uses a large number of label permutations (10,000 in the article), recording the t statistic for the most significant pathway per permutation.

## Limitations

- KEGG and Biocarta pathways continue to expand and evolve; the article's pathway versions are only available at the PLAGE website and no specific version numbers or accessions are stated, so reproducibility depends on obtaining those versions.
- The method quantifies coordinate expression (variation captured by the first metagene), which does not by itself distinguish pathways with consistent group differences from pathways whose genes are merely coordinately expressed — the very confound observed when comparing against GSEA top-ranked sets.
- Validation results are specific to the two demonstration datasets (Mootha type 2 diabetes profiles and airway epithelium smoker/non-smoker profiles); expected eigenvalue fractions (0.53, 0.23, 0.05) need not transfer to other datasets.

## Evidence

- [intro] SVD is the core transform applied after standardization/subsetting: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [discussion] Activity levels from the processed matrices are the basis of comparisons: "the approach quantifies the level of activity of each pathway within each sample and uses the activity levels as the basis for making comparisons"
- [discussion] Pathway gene-sets come from KEGG and Biocarta: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [abstract] Demonstration dataset 1: type 2 diabetes expression data: "expression data from a study of type 2 diabetes"
- [discussion] Pathway version reproducibility caveat: "The versions used for this article are available at the PLAGE website [11]."
