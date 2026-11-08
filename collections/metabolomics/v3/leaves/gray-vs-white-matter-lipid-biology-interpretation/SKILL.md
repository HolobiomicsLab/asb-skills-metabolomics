---
name: gray-vs-white-matter-lipid-biology-interpretation
description: Use when MSI pixels have been assigned brain-region labels (e.g. Waxholm
  Space atlas) and a lipid annotation table exists, and the goal is to identify which
  lipid species distinguish white-matter structures (corpus callosum, corticofugal
  pathway) from gray-matter structures.
when_to_use_negative:
- The MSI data have not yet been registered to a rat MRI brain atlas or had Waxholm
  Space atlas labels propagated to pixels — interpret regional lipid biology only
  after the multimodal registration step produces the volumetric reconstruction.
- The input is single-cell (SCMS) data rather than tissue MSI; this skill's differential
  analysis is defined over atlas-labeled pixels, not over isolated cells.
- Lipid annotations cannot be made because m/z values cannot be searched against LIPID
  MAPS or similar lipid databases; without annotation, biological interpretation of
  myelin-related species is not possible.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0092
tools:
- name: MEISTER
  role: Provides the deep-learning-reconstructed, registered 3D coronal MSI volume
    and atlas-propagated region labels that are the substrate for regional lipid interpretation;
    its multiscale_analysis notebooks implement brain regional analysis of registered
    3D coronal data with the MRI atlas.
  repo: https://github.com/richardxie1119/MEISTER
- name: MALDI FT-ICR mass spectrometer
  role: Acquired the serial-section coronal MSI data (short-transient, low-resolution
    transients) that MEISTER reconstructs into the high-SNR volume on which regional
    lipid enrichment is computed.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:eee031eadf2ff1bca6094272430c283f82efc3f88b8813a86137d3772aed347e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MALDI FT-ICR mass spectrometer
  - MEISTER
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gray-vs-white-matter-lipid-biology-interpretation

## Summary

Biological interpretation of region-specific lipid profiles from registered 3D rat brain MALDI FT-ICR MSI, focusing on the enrichment of myelin-associated lipids (ceramide, sphingomyelin, HexCer) in white-matter structures versus gray-matter structures.

## When to use

Use when MSI pixels have been assigned brain-region labels (e.g. Waxholm Space atlas) and a lipid annotation table exists, and the goal is to identify which lipid species distinguish white-matter structures (corpus callosum, corticofugal pathway) from gray-matter structures.

## When NOT to use

- The MSI data have not yet been registered to a rat MRI brain atlas or had Waxholm Space atlas labels propagated to pixels — interpret regional lipid biology only after the multimodal registration step produces the volumetric reconstruction.
- The input is single-cell (SCMS) data rather than tissue MSI; this skill's differential analysis is defined over atlas-labeled pixels, not over isolated cells.
- Lipid annotations cannot be made because m/z values cannot be searched against LIPID MAPS or similar lipid databases; without annotation, biological interpretation of myelin-related species is not possible.

## Inputs

- coronal3D_propagated_decoded.h5 — reconstructed, processed, and atlas-propagated 3D coronal MSI volume from the MEISTER pipeline
- Per-pixel Waxholm Space atlas brain region labels (propagated during image registration)
- LIPID MAPS experimental and virtual lipid annotation database (±0.005 m/z search threshold)

## Outputs

- region_lipid_enrichment.tsv — a region-by-lipid table with lipid annotation, log2 fold change, and Benjamini–Hochberg-adjusted Wilcoxon P-value columns per brain structure
- Per-structure regional average lipid profiles extracted from every tissue section
- Optional: ranked list of lipids most contributing to anatomical classification, via mean absolute SHAP values from gradient-boosted trees

## How to apply

Load the reconstructed and registered 3D coronal MSI volume (coronal3D_propagated_decoded.h5) together with the propagated Waxholm Space atlas labels for each pixel. Extract pixels belonging to the 11 atlas-defined brain structures and normalize each pixel by total ion count (TIC). Annotate lipid features by searching m/z values against LIPID MAPS experimental and virtual databases with a ±0.005 m/z threshold, prioritizing experimentally validated lipids sorted by ppm error. Compute per-region average lipid profiles, then perform differential analysis per brain structure using log2 fold change and Wilcoxon rank-sum P-values adjusted by Benjamini–Hochberg. Interpret the regional signatures biologically: expect ceramide — a lipid abundant in the myelin sheath around nerve fibers — elevated in subcortical white matter such as the corpus callosum and corticofugal pathway, with a similar trend for sphingomyelin (SM). Optionally rank the most contributing lipids per region using gradient-boosted tree classifiers and mean absolute SHAP values.

## Related tools

- **MEISTER** (Provides the deep-learning-reconstructed, registered 3D coronal MSI volume and atlas-propagated region labels that are the substrate for regional lipid interpretation; its multiscale_analysis notebooks implement brain regional analysis of registered 3D coronal data with the MRI atlas.) — https://github.com/richardxie1119/MEISTER
- **MALDI FT-ICR mass spectrometer** (Acquired the serial-section coronal MSI data (short-transient, low-resolution transients) that MEISTER reconstructs into the high-SNR volume on which regional lipid enrichment is computed.)

## Examples

```
jupyter nbconvert --execute --to notebook regmri_atlas_coronal.ipynb  # loads coronal3D_propagated_decoded.h5 + Waxholm atlas labels, TIC-normalizes pixels, annotates lipids (LIPID MAPS, ±0.005 m/z), and writes region_lipid_enrichment.tsv
```

## Evaluation signals

- The output table reproduces the reported region-specific lipid contents — in particular, enrichment of myelin-associated HexCer/Cer/SM species in the white-matter-dominant corpus callosum.
- Regional differences are supported by FDR-adjusted Wilcoxon rank-sum P-values (Benjamini–Hochberg correction); flag any structure that shows a significant call without a corresponding fold-change direction consistent with white/gray matter biology.
- Every lipid feature in the table has an annotation traceable to a LIPID MAPS match within the ±0.005 m/z threshold, with experimentally validated lipids prioritized by ppm error.
- Regional average profiles were computed from every tissue section in the 37-section coronal dataset (~2 million pixels total), not from a single section.
- Per-pixel TIC normalization was applied before region averaging; check that no un-normalized intensity leaked into the enrichment table.

## Limitations

- Reproducibility of the biological interpretation depends on the registration step; prior 2D-section-based approaches can produce incoherent image registration across serial sections in large 3D MSI datasets, which motivated MEISTER's 3D approach.
- Mass shift or an inappropriate ppm window in the m/z search can cause false lipid annotations; the article warns that modeling errors may arise from falsely cross-annotated features caused by mass shifts and an inappropriate ppm window.
- Features that fail validation must be flagged for expert review rather than reported as confirmed regional enrichments.

## Evidence

- [results] Ceramide, a lipid abundant in the myelin sheath around nerve fibers, was found elevated in subcortical white matter such as the corpus callosum and corticofugal pathway, with a similar trend observed for sphingomyelin (SM).: [evidence span withheld — anchor status: not_found]
- [methods] we searched the m/z values against LIPID MAPS34 experimental and virtual databases with a ±0.005 m/z threshold: [evidence span withheld — anchor status: not_found]
- [methods] Each pixel was normalized by total ion count (TIC).: "Each pixel was normalized by total ion count (TIC)."
- [readme] Brain regional analysis of registered 3D coronal data with MRI atlas.: "regmri_atlas_coronal.ipynb: Brain regional analysis of registered 3D coronal data with MRI atlas."
- [discussion] which can lead to incoherent image registration across different serial sections in large 3D MSI datasets: [evidence span withheld — anchor status: not_found]
- [results] We also investigated the feature attributions to interpret model decisions and selected the lipids that were contributing most to the anatomical classifications: [evidence span withheld — anchor status: not_found]
