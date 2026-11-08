---
name: untargeted-metabolomics-ordination-analysis
description: Use when you have per-tissue untargeted metabolomics peak-area feature
  tables with sample metadata (study, cohort, genotype) and need to verify whether
  samples cluster across conditions before running differential or ML analyses.
when_to_use_negative:
- Longitudinal fecal samples lack associated animal IDs (collected randomly per cage),
  so ordination cannot be interpreted as single-animal developmental trajectories
- Cohort clustering may reflect collection method (fresh fecal vs. colon after sacrifice)
  rather than biology, so do not interpret PC separation as a genotype effect without
  checking cohort balance
- Input is already ordinated or the goal is MS/MS spectral annotation — use GNPS library
  spectral matching instead of PCA on peak areas
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0092
tools:
- name: GNPS
  role: Source repository from which the deposited untargeted metabolomics peak-area
    tables and sample metadata for the 3xTg and 5xFAD studies (Sacrifice and Longitudinal
    cohorts) are downloaded before RCLR transformation and PCA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:ba24ec05ae3ebcf0e4c98244fcdd5872bcc75fc59d1e3c15dc6cb97fd0b2ee90
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# untargeted-metabolomics-ordination-analysis

## Summary

Ordination (PCA on RCLR-transformed untargeted metabolomics peak-area matrices) to test whether samples cluster by experimental variables such as study, cohort, or tissue, and to decide if downstream analyses must be stratified accordingly.

## When to use

Use when you have per-tissue untargeted metabolomics peak-area feature tables with sample metadata (study, cohort, genotype) and need to verify whether samples cluster across conditions before running differential or ML analyses.

## When NOT to use

- Longitudinal fecal samples lack associated animal IDs (collected randomly per cage), so ordination cannot be interpreted as single-animal developmental trajectories
- Cohort clustering may reflect collection method (fresh fecal vs. colon after sacrifice) rather than biology, so do not interpret PC separation as a genotype effect without checking cohort balance
- Input is already ordinated or the goal is MS/MS spectral annotation — use GNPS library spectral matching instead of PCA on peak areas

## Inputs

- Untargeted metabolomics peak-area feature table per tissue (from GNPS/MassIVE deposits)
- Sample metadata mapping samples to study (3xTg vs 5xFAD), cohort (Sacrifice vs Longitudinal), genotype, sex, age, and colonization status
- Tissue subset definition (brain, serum, liver, cecal content, feces)

## Outputs

- PC1 vs PC2 score plots per tissue, colored by study and cohort (reproducing Figure S2)
- Per-tissue RCLR-transformed peak-area matrices feeding downstream analyses
- Decision on whether downstream per-tissue analyses must be stratified by study and cohort

## How to apply

Subset the deposited peak-area table to a single tissue (brain, serum, liver, cecal content, or feces) and merge sample metadata (study: 3xTg vs 5xFAD; cohort: Sacrifice vs Longitudinal; genotype, sex, age, colonization status). Apply robust center log-ratio (RCLR) transformation to each per-tissue peak-area matrix because compositional metabolomics data contain many zeros that a plain CLR cannot handle. Run PCA on the RCLR-transformed matrix with scikit-learn, retaining PC1 and PC2 for score plotting. Plot PC1 vs PC2 per tissue, coloring samples by study and then by cohort, to visually reproduce the reported clustering and justify stratifying all downstream per-tissue analyses. Validate by confirming that the score plots show separation consistent with the paper's statement that clustering was observed across studies and cohorts for each tissue type.

## Related tools

- **GNPS** (Source repository from which the deposited untargeted metabolomics peak-area tables and sample metadata for the 3xTg and 5xFAD studies (Sacrifice and Longitudinal cohorts) are downloaded before RCLR transformation and PCA)

## Examples

```
from sklearn.decomposition import PCA; pcs = PCA(n_components=2).fit_transform(rclr_transform(peak_area_tissue))
```

## Evaluation signals

- Score plots per tissue visually separate samples by study (3xTg vs 5xFAD) and by cohort (Sacrifice vs Longitudinal), matching the reported Figure S2 pattern
- RCLR transformation is applied per tissue to the raw peak-area matrix, correctly handling zero-count features (no division-by-zero or NaNs in the transformed matrix)
- The same PC1/PC2 retention and coloring scheme is applied consistently across all five tissues (brain, serum, liver, cecal content, feces)
- The stratification decision carried forward (downstream analyses stratified by study and cohort) is explicitly justified by the ordination result

## Limitations

- Differences between Longitudinal and Sacrifice cohorts can be attributed to differences in sample collection (fresh fecal vs. colon after sacrifice), so cohort clustering is a technical confound, not necessarily biological
- 3xTg and WT mice were bred and housed separately before the start of the experiment, a potential confound that ordination cannot disentangle
- Feature detection can be influenced by study-specific extraction methods, workflows, and instrumentation, which may drive apparent clustering

## Evidence

- [supplementary] Clustering of the untargeted metabolomics data was observed across studies (3xTg vs 5xFAD) and cohorts (Sacrifice vs Longitudinal) for each tissue type: "Clustering of the untargeted metabolomics data was observed across studies (3xTg vs 5xFAD) and cohorts (Sacrifice vs Longitudinal) for each tissue type"
- [discussion] Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample: "Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data and can be attributed to differences in sample"
- [abstract] a multi-organ untargeted metabolomics dataset (2,271 samples) generated from five tissue types in two genetic AD mouse models under colonized or germ-free conditions: "a multi-organ untargeted metabolomics dataset (2,271 samples) generated from five tissue types in two genetic AD mouse models under colonized or germ-free conditions"
