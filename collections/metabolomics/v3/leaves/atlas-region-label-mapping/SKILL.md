---
name: atlas-region-label-mapping
description: Use when registered 2D MSI serial sections must be assigned anatomical
  labels from the Waxholm Space rat brain atlas and per-region Dice-index quality
  scores or regional chemical profiles are the deliverable.
when_to_use_negative:
- Sections have not yet been registered to the MRI atlas — atlas labels will not align
  unless the affine and B-spline transformations have already been applied to all
  ion images
- Working with mouse data mapped to the Allen Brain Atlas (ABA) annotation — that
  reference was used only as a spatial reference in other analyses of this study,
  not for rat 3D regional profiling
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MEISTER
  role: Integrative MS framework whose registration and atlas-mapping code implements
    the Dice-index regional analysis
  repo: https://github.com/richardxie1119/MEISTER
- name: regmri_atlas_coronal.ipynb
  role: Notebook performing brain regional analysis of registered 3D coronal data
    with the MRI atlas
  repo: https://github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:afd485bbb5d5a4e57311a43974d7365a38a79c3f1968b5f957ba7513b94614f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - regmri_atlas_coronal.ipynb
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# atlas-region-label-mapping

## Summary

Assigns brain-region labels from the Waxholm Space atlas of the Sprague-Dawley rat brain to registered 3D MSI sections and scores registration quality with per-region Dice indices, enabling quantitative regional chemical profiling.

## When to use

Use when registered 2D MSI serial sections must be assigned anatomical labels from the Waxholm Space rat brain atlas and per-region Dice-index quality scores or regional chemical profiles are the deliverable.

## When NOT to use

- Sections have not yet been registered to the MRI atlas — atlas labels will not align unless the affine and B-spline transformations have already been applied to all ion images
- Working with mouse data mapped to the Allen Brain Atlas (ABA) annotation — that reference was used only as a spatial reference in other analyses of this study, not for rat 3D regional profiling

## Inputs

- coronal3D_propagated_decoded.h5 — reconstructed and processed 3D coronal rat brain MSI data
- registered_msi_volume.h5 — MSI ion-image volume after affine + B-spline registration transformations were applied
- T2*-weighted anatomical MRI slices from the Waxholm Space atlas of the Sprague-Dawley rat brain
- Human-annotated brain-region masks drawn on registered MSI sections

## Outputs

- dice_scores.csv — per-region Dice index between atlas region labels and human-annotated masks on each registered section
- Atlas-region-labeled MSI sections supporting quantitative regional chemical profiles

## How to apply

Load the reconstructed 3D coronal MSI volume (coronal3D_propagated_decoded.h5 from the MEISTER data repository) and select the T2*-weighted MRI slices from the Waxholm Space atlas that correspond to each tissue section's sectioning distance, verified by manual inspection. Using the registered volume (registered_msi_volume.h5, produced by applying the affine + B-spline transformations from the multimodal registration step to every ion image), compute the Dice index between the ith brain-region label from the atlas and human-annotated masks on the jth registered section, writing per-region scores to dice_scores.csv. In the MEISTER/multiscale_analysis codebase this is implemented in regmri_atlas_coronal.ipynb, run in the conda MEISTER environment. Validation accepts the mapping when the Dice index is computed for all registered sections and atlas labels visually overlap the human-annotated region masks (as in the paper's Supplementary Fig. 10 quality check); no numeric Dice threshold is specified.

## Related tools

- **MEISTER** (Integrative MS framework whose registration and atlas-mapping code implements the Dice-index regional analysis) — https://github.com/richardxie1119/MEISTER
- **regmri_atlas_coronal.ipynb** (Notebook performing brain regional analysis of registered 3D coronal data with the MRI atlas) — https://github.com/richardxie1119/multiscale_analysis

## Examples

```
jupyter notebook regmri_atlas_coronal.ipynb
```

## Evaluation signals

- dice_scores.csv exists and contains a Dice index for every (region, registered-section) pair
- Atlas region labels visually overlap human-annotated masks on each registered section (the paper's Supplementary Fig. 10-style quality check)
- Each MSI section maps to the correct T2*-weighted Waxholm atlas slice as selected by sectioning distance and confirmed by manual inspection
- The input volume is the post-registration product (registered_msi_volume.h5), not the raw reconstructed sections

## Limitations

- No numeric Dice-index threshold is reported; registration quality is accepted by computed DI values and visual mask overlap rather than a quantitative cutoff
- Section-to-MRI-slice correspondence relies on manual inspection of sectioning distance, a subjective decision point
- Validated only against the Waxholm Space atlas of the Sprague-Dawley rat brain with T2*-weighted MRI; transfer to other species/atlas modalities is untested here

## Evidence

- [methods] We registered MSI serial sections to T2*-weighted anatomical MRI: [evidence span withheld — anchor status: not_found]
- [readme] Brain regional analysis notebook: "regmri_atlas_coronal.ipynb: Brain regional analysis of registered 3D coronal data with MRI atlas."
- [abstract] Registration enables regional chemical profiles: [evidence span withheld — anchor status: not_found]
- [readme] Input data availability: "coronal3D_propagated_decoded.h5: reconstructed and processed MSI data of serial sections of rat coronal sections."
