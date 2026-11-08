---
name: atlas-label-based-region-extraction-msi
description: Use when a registered 3D MSI volume (e.g., coronal3D_propagated_decoded.h5)
  carries pixel-level brain-region labels from an atlas, and per-structure lipid enrichment
  or differential abundance statistics are needed.
when_to_use_negative:
- Input is a raw unregistered MSI stack with no atlas label per pixel — registration
  and label propagation must happen first.
- Goal is single-cell lipid mapping or cross-annotation of SCMS data, not region-level
  anatomical profiling.
- Data are from a tissue lacking an available atlas registration (e.g., rat pancreas),
  where no anatomical label structure exists.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: MEISTER
  role: Framework that reconstructs high-resolution MSI data and provides the registered
    3D volume and propagated atlas labels that this skill consumes
  repo: https://github.com/richardxie1119/MEISTER
- name: MALDI FT-ICR mass spectrometer
  role: Instrument that acquired the serial-section MSI data from which regional lipid
    profiles are extracted
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:3d51b831fbd70492692b4836bb5c026bd846d4a9cfca1b9962f9fabe201e4a3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MALDI FT-ICR mass spectrometer
  - MEISTER
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# atlas-label-based-region-extraction-msi

## Summary

Extracts and normalizes mass-spec imaging pixels assigned to anatomical brain structures via propagated atlas labels (e.g., Waxholm Space), enabling region-level lipid profiling and differential testing across structures such as corpus callosum and neocortex.

## When to use

Use when a registered 3D MSI volume (e.g., coronal3D_propagated_decoded.h5) carries pixel-level brain-region labels from an atlas, and per-structure lipid enrichment or differential abundance statistics are needed.

## When NOT to use

- Input is a raw unregistered MSI stack with no atlas label per pixel — registration and label propagation must happen first.
- Goal is single-cell lipid mapping or cross-annotation of SCMS data, not region-level anatomical profiling.
- Data are from a tissue lacking an available atlas registration (e.g., rat pancreas), where no anatomical label structure exists.

## Inputs

- Reconstructed and registered 3D MSI volume (coronal3D_propagated_decoded.h5) with per-pixel atlas region labels
- Peak-binned m/z feature table aligned across serial tissue sections (3-ppm bins, TIC-normalized)
- Annotated lipid list (e.g., LIPID MAPS matches with ppm error)

## Outputs

- Region-by-lipid enrichment table (region_lipid_enrichment.tsv) with annotation, log2 fold change, and BH-adjusted P-value per structure
- Per-structure mean lipid profile across sections

## How to apply

Load the registered 3D MSI volume together with the propagated atlas brain-region labels for every pixel. Subset pixels by the 11 target structures (neocortex, hippocampus, thalamus, hypothalamus, corpus callosum, corticofugal pathway, superior colliculus, basal forebrain, brainstem, striatum, septal regions), then normalize each pixel by total ion count (TIC) before aggregating. Annotate candidate lipid features by searching m/z values against LIPID MAPS experimental and virtual databases with a ±0.005 m/z threshold, prioritizing experimentally validated lipids sorted by ppm error. Compute per-structure average lipid profiles from all serial sections, then run differential analysis (log2 fold change, Wilcoxon rank-sum with Benjamini–Hochberg FDR correction) between regions; optionally rank the most discriminative lipids by mean absolute SHAP values from a gradient-boosted tree classifier.

## Related tools

- **MEISTER** (Framework that reconstructs high-resolution MSI data and provides the registered 3D volume and propagated atlas labels that this skill consumes) — https://github.com/richardxie1119/MEISTER
- **MALDI FT-ICR mass spectrometer** (Instrument that acquired the serial-section MSI data from which regional lipid profiles are extracted)

## Examples

```
jupyter nbconvert --to notebook --execute regmri_atlas_coronal.ipynb --inplace
```

## Evaluation signals

- The output table reproduces the reported enrichment of myelin-associated HexCer/Cer/SM species (e.g., ceramide and sphingomyelin) in the white-matter-dominant corpus callosum and corticofugal pathway.
- Every pixel in the 3D volume is assigned to exactly one of the 11 listed atlas structures; no unassigned or duplicate labels remain.
- Wilcoxon rank-sum P-values are Benjamini–Hochberg adjusted; only features passing FDR threshold are reported as significant.
- Lipid annotations are limited to LIPID MAPS matches within ±0.005 m/z (approximately <3 ppm for the reported 728 annotations of 1,156 total features).
- Per-structure profiles are computed from every serial section in the 3D reconstruction, not just one slice.

## Limitations

- FDR-controlled metabolite annotation methods cannot easily be applied to image-guided SCMS data due to lack of spatial information, so annotation confidence relies on the m/z threshold approach here.
- UoSS fitting modeling errors can arise from falsely cross-annotated features caused by mass shifts, inappropriate ppm window, or nonlinearities between MSI and SCMS data.
- Pixels outside the 11 defined brain structures are excluded from regional statistics and have no label-based profile.

## Evidence

- [methods] m/z searched against LIPID MAPS with ±0.005 threshold: [evidence span withheld — anchor status: not_found]
- [methods] registration to Waxholm Space atlas T2*-weighted MRI: [evidence span withheld — anchor status: not_found]
- [methods] TIC normalization per pixel: "Each pixel was normalized by total ion count (TIC)."
- [results] Ceramide and SM elevated in corpus callosum: [evidence span withheld — anchor status: not_found]
- [readme] README notebook for brain regional analysis of registered 3D coronal data with MRI atlas: "regmri_atlas_coronal.ipynb: Brain regional analysis of registered 3D coronal data with MRI atlas."
