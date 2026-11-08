---
name: shap-feature-attribution-interpretation
description: Use when a supervised classifier (e.g., gradient-boosted trees) has been
  trained on normalized MSI pixel features (m/z bins) to predict brain regions and
  you need to identify the most contributing lipids per anatomical structure.
when_to_use_negative:
- Input is already a differential expression table or ranked lipid list; SHAP interpretation
  requires the trained classifier and raw pixel-level features.
- Unsupervised analysis (e.g., UMAP embedding only) with no supervised classifier;
  SHAP requires a supervised model output.
- Single-cell SCMS data lacking spatial/atlas labels for region classification; SHAP
  attribution on anatomical regions requires spatially annotated pixel data.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: MEISTER
  role: Provides the reconstructed and registered 3D MSI data and computational framework
    (including multiscale_analysis notebooks) on which the gradient-boosted tree classifier
    and SHAP interpretation are performed.
  repo: https://github.com/richardxie1119/MEISTER
- name: MALDI FT-ICR mass spectrometer
  role: Source instrument for the pixel-level lipid mass spectra whose features are
    attributed via SHAP values after classification.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:e528afc39a49da63dfcf57a9b2d552fa78dea2e407bd1ccf34edf3fad9808e6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MALDI FT-ICR mass spectrometer
  - MEISTER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# shap-feature-attribution-interpretation

## Summary

Interprets gradient-boosted tree classifiers trained on MALDI FT-ICR mass spectrometry imaging pixel data by ranking lipid features with mean absolute SHAP values, revealing which lipids drive anatomical region classification.

## When to use

Use when a supervised classifier (e.g., gradient-boosted trees) has been trained on normalized MSI pixel features (m/z bins) to predict brain regions and you need to identify the most contributing lipids per anatomical structure.

## When NOT to use

- Input is already a differential expression table or ranked lipid list; SHAP interpretation requires the trained classifier and raw pixel-level features.
- Unsupervised analysis (e.g., UMAP embedding only) with no supervised classifier; SHAP requires a supervised model output.
- Single-cell SCMS data lacking spatial/atlas labels for region classification; SHAP attribution on anatomical regions requires spatially annotated pixel data.

## Inputs

- Trained gradient-boosted tree classifier on MSI pixel data
- TIC-normalized MSI pixel spectra (m/z bins, 3-ppm)
- Atlas-derived brain region labels per pixel (e.g., Waxholm Space)
- LIPID MAPS annotation table with m/z and ppm error

## Outputs

- Ranked lipid feature list by mean absolute SHAP value
- Interpretable region-specific lipid signature (e.g., myelin-associated HexCer/Cer/SM species in corpus callosum)
- Feature contribution summary table

## How to apply

After training a gradient-boosted tree classifier on TIC-normalized MSI pixel spectra (m/z bins in 3-ppm increments) with atlas-derived region labels, compute SHAP values for each feature prediction. Rank lipid features by their mean absolute SHAP value to select those contributing most to the anatomical classifications. Validate that top-ranked features (e.g., ceramide, sphingomyelin in white matter like corpus callosum) align with reported regional lipid enrichment results and differential analysis (log2 fold change, Wilcoxon rank-sum test with Benjamini–Hochberg FDR). The rationale is to move beyond black-box classification toward interpretable region-specific lipid signatures.

## Related tools

- **MEISTER** (Provides the reconstructed and registered 3D MSI data and computational framework (including multiscale_analysis notebooks) on which the gradient-boosted tree classifier and SHAP interpretation are performed.) — https://github.com/richardxie1119/MEISTER
- **MALDI FT-ICR mass spectrometer** (Source instrument for the pixel-level lipid mass spectra whose features are attributed via SHAP values after classification.)

## Examples

```
python regmri_atlas_coronal.ipynb
```

## Evaluation signals

- Top-ranked features by mean absolute SHAP value should reproduce the article's reported region-specific lipid enrichments (e.g., ceramide/SM elevated in corpus callosum).
- SHAP-derived feature importance should be consistent with differential analysis results (log2 fold change direction and Wilcoxon BH-adjusted P-value significance).
- Feature importance ranking should remain stable across independent tissue sections or folds within the same 3D MSI volume.
- Number of features contributing to classification should be reasonable relative to total annotated features (e.g., 728 of 1,156 features matched LIPID MAPS with <3 ppm error).

## Limitations

- SHAP interpretation depends on classifier quality; if the gradient-boosted tree underfits or the atlas registration is misaligned, top features may be artifacts rather than true biological drivers.
- Correlation-based interpretation of SHAP values may be misleading when lipid features are highly collinear (common in lipidomics); consider additional feature-selection steps.
- The article notes that FDR-controlled metabolite annotation methods cannot easily be applied to image-guided SCMS data due to lack of spatial information; SHAP interpretation of single-cell data therefore carries similar annotation uncertainty.
- Only features retained after m/z binning and prevalence filtering (e.g., features in ≥5% of cells/pixels) are eligible for SHAP-based interpretation.

## Evidence

- [results] We performed both single-pixel analysis of individual tissue sections through UMAP visualization (Fig. 3c and Supplementary Fig. 5) and supervised classification with gradient-boosted trees: [evidence span withheld — anchor status: not_found]
- [results] We also investigated the feature attributions to interpret model decisions and selected the lipids that were contributing most to the anatomical classifications: [evidence span withheld — anchor status: not_found]
- [results] Ceramide, a lipid abundant in the myelin sheath around nerve fibers, was found elevated in subcortical white matter such as the corpus callosum and corticofugal pathway: [evidence span withheld — anchor status: not_found]
- [methods] Each pixel was normalized by total ion count (TIC).: "Each pixel was normalized by total ion count (TIC)."
- [methods] The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift: "The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift"
- [readme] coronal3D_propagated_decoded.h5: reconstructed and processed MSI data of serial sections of rat coronal sections.: "coronal3D_propagated_decoded.h5: reconstructed and processed MSI data of serial sections of rat coronal sections."
