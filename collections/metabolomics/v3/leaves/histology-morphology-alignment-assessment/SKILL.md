---
name: histology-morphology-alignment-assessment
description: Use when spatial metabolomics cluster maps (pixels labeled by graph-based
  clustering) must be visually or expert-assessed against an H&E-stained tissue image
  to confirm that preprocessing (e.g., SMQVP noise/background removal) recovered true
  neuroanatomical structure.
when_to_use_negative:
- No serial H&E-stained section or morphology reference image is available for the
  sampled tissue
- The goal is quantitative clustering quality (e.g., modularity or silhouette scores)
  rather than anatomical correspondence to histology
- Input data has already been judged as morphology-aligned, and no further preprocessing
  comparison is being evaluated
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0092
tools:
- name: SMQVP
  role: Produces the preprocessed peak table (background pixel removal, noise ion
    filtering) whose cluster map is compared against H&E morphology
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Environment used to run Louvain clustering and render the spatial cluster
    maps for raw vs. preprocessed data
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:fe7e678d656abbbbe8173a61c51b6e58eed035ab1f89114e46e854c138edc2ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# histology-morphology-alignment-assessment

## Summary

Assessment skill for judging whether spatial cluster maps derived from spatial metabolomics peak tables (e.g., Louvain clustering of AFADESI mouse brain data) align with the tissue morphology visible in a serial H&E-stained section, used to validate preprocessing quality (SMQVP) against raw data.

## When to use

Use when spatial metabolomics cluster maps (pixels labeled by graph-based clustering) must be visually or expert-assessed against an H&E-stained tissue image to confirm that preprocessing (e.g., SMQVP noise/background removal) recovered true neuroanatomical structure.

## When NOT to use

- No serial H&E-stained section or morphology reference image is available for the sampled tissue
- The goal is quantitative clustering quality (e.g., modularity or silhouette scores) rather than anatomical correspondence to histology
- Input data has already been judged as morphology-aligned, and no further preprocessing comparison is being evaluated

## Inputs

- Raw AFADESI peak table (53,812 pixels x 2,654 ions) with pixel X/Y coordinates in the first two columns, from OMIX009541
- SMQVP-preprocessed peak table with pixel X/Y coordinates in the first two columns
- H&E-stained mouse brain section image (morphology reference)

## Outputs

- Spatial cluster map for raw data (Figure 3A equivalent)
- Spatial cluster map for SMQVP-preprocessed data (Figure 3B equivalent)
- Morphological-alignment verdict per map (expert_review / llm_judge)

## How to apply

Run Louvain graph-based clustering independently on the raw and preprocessed ion-intensity matrices, treating pixels as observations and ions as features (e.g., 53,812 pixels x 2,654 ions from the AFADESI mouse brain OMIX009541 peak table). Map cluster labels back to the X/Y pixel coordinates in the first two columns of each peak table and render spatial cluster maps in R (raw = Figure 3A equivalent; SMQVP-preprocessed = Figure 3B equivalent). Compare both maps against the H&E-stained mouse brain section image (Figure 2A reference). The pass criterion is that the preprocessed-data map yields distinct spatial regions closely resembling H&E morphology while the raw-data map fails to reflect known neuroanatomical structures; the morphological-alignment judgment is made via expert_review or llm_judge.

## Related tools

- **SMQVP** (Produces the preprocessed peak table (background pixel removal, noise ion filtering) whose cluster map is compared against H&E morphology) — https://github.com/mzlab-research/SMQVP
- **R** (Environment used to run Louvain clustering and render the spatial cluster maps for raw vs. preprocessed data)

## Examples

```
Rscript louvain_spatial_map.R --peak-table smqvp_preprocessed.csv --coords-cols 1:2 --output fig3B_cluster_map.png
```

## Evaluation signals

- Preprocessed-data cluster map shows distinct spatial regions closely resembling the H&E-stained morphology
- Raw-data cluster map fails to reflect known neuroanatomical structures (expected negative control)
- Cluster labels from Louvain clustering map cleanly onto pixel X/Y coordinates with no unmatched pixels
- Morphological-alignment verdict is consistent across expert_review and llm_judge assessments

## Limitations

- The alignment judgment is qualitative (expert_review / llm_judge), relying on visual resemblance to H&E rather than a quantitative anatomical-overlap metric
- H&E is acquired on a serial section, so morphological correspondence is approximate and subject to section-to-section variation

## Evidence

- [abstract] Preprocessing with SMQVP resulted in improved clustering outcomes that more accurately reflected the underlying tissue morphology compared with analyses on unprocessed data: "This rigorous preprocessing resulted in improved clustering outcomes that more accurately reflected the underlying tissue morphology compared with analyses performed on unprocessed data."
- [results] SMQVP improved clustering results by removing noise signals: "SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality."
- [results] The processed data aligns better with tissue morphology: "The processed data aligns better with tissue morphology, highlighting the importance of thorough preprocessing in spatial metabolomics analysis."
- [readme] Example data includes 53,812 pixels and 2,654 ion peaks available from OMIX009541: "The data includes 53,812 pixels and 2,654 ion peaks. This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541."
