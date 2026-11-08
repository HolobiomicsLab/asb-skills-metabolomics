---
name: train-test-split-interpolation-modeling
description: Use when pooled QC feature tables from large-scale LC-MS (e.g., RPLC)
  runs exhibit retention time drift across chromatograms and an nontargeted pipeline
  needs nonlinear RT correction via validated anchor peaks.
when_to_use_negative:
- Input is already an aligned feature table with negligible RT drift, so anchor-based
  interpolation would add no value
- Single-sample or small-scale LC-MS batches lacking pooled QC injections from which
  unique model anchors can be selected
- Tasks requiring metabolite identification or statistical/phenotype modeling — MassCube
  does not directly perform metabolite identification and downstream biological experiments
  remain necessary to confirm function
edam_topics:
- http://edamontology.org/topic_3370
tools:
- name: MassCube
  role: Selects unique model peaks (by quality and intensity) in QC pools as RT anchors,
    fits the linear interpolation model on train anchors, and applies it to correct
    nonlinear RT shifts across chromatograms
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Implementation language in which the anchor filtering (spacing = 0.01, noise
    < 0.3), top-50 height ranking, and interpolation fitting are executed
  repo: https://github.com/huaxuyu/masscube
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:2d9970d931766f7022e2ca8bb55dbfdfdd44986fa5770d1cc339dc6947c8dcf1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3370
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# train-test-split-interpolation-modeling

## Summary

A retention-time drift correction procedure from MassCube in which unique, high-quality model peaks in pooled QC samples serve as RT anchors; a linear interpolation model fitted on a training subset of anchors corrects nonlinear RT shifts across chromatograms in large-scale LC-MS studies.

## When to use

Use when pooled QC feature tables from large-scale LC-MS (e.g., RPLC) runs exhibit retention time drift across chromatograms and an nontargeted pipeline needs nonlinear RT correction via validated anchor peaks.

## When NOT to use

- Input is already an aligned feature table with negligible RT drift, so anchor-based interpolation would add no value
- Single-sample or small-scale LC-MS batches lacking pooled QC injections from which unique model anchors can be selected
- Tasks requiring metabolite identification or statistical/phenotype modeling — MassCube does not directly perform metabolite identification and downstream biological experiments remain necessary to confirm function

## Inputs

- MassCube-processed QC pool feature table with RT, m/z, height, spacing, and noise columns
- Anchor validity thresholds: m/z spacing = 0.01, noise < 0.3
- Ranking rule: top 50 anchors by height, split into train and test subsets

## Outputs

- RT-corrected test-set feature table (residual RT shift ~1.0 s vs 40.5 s pre-correction in RPLC negative mode)
- Train/test anchor subsets of unique model peaks ranked by height
- Reported reduction in RT shift after interpolation-based correction

## How to apply

Load the MassCube-processed QC feature table containing RT, m/z, height, spacing, and noise metadata per feature. Filter candidate anchors using m/z spacing = 0.01 and noise < 0.3 as validity criteria, keeping only unique model peaks selected by quality and intensity. Rank valid anchors by height and retain the top 50, then split them into train and test subsets. Fit a linear interpolation model on the train-set anchor RTs and apply it to correct nonlinear RT shifts in the test subset. Validate by reporting the residual RT shift after correction against the uncorrected shift; the published benchmark reduces test-set shifts from 40.5 s to ~1.0 s in RPLC negative mode.

## Related tools

- **MassCube** (Selects unique model peaks (by quality and intensity) in QC pools as RT anchors, fits the linear interpolation model on train anchors, and applies it to correct nonlinear RT shifts across chromatograms) — https://github.com/huaxuyu/masscube
- **Python** (Implementation language in which the anchor filtering (spacing = 0.01, noise < 0.3), top-50 height ranking, and interpolation fitting are executed) — https://github.com/huaxuyu/masscube

## Examples

```
pip install masscube
```

## Evaluation signals

- Residual RT shift on held-out test anchors must be substantially reduced versus the uncorrected shift — benchmark ~1.0 s residual against 40.5 s pre-correction in RPLC negative mode
- Every retained anchor satisfies the validity criteria (m/z spacing = 0.01, noise < 0.3) and is a unique model peak selected by quality and intensity
- Anchor set is limited to the top 50 features ranked by height before the train/test split
- Corrected RT values for test anchors fall within ~1 s of reference, confirming the interpolation removes nonlinear rather than only linear shifts

## Limitations

- External validated benchmarking datasets with large-scale ground-truth data are still missing, so the 40.5 s → 1.0 s result cannot be independently cross-checked
- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios, which can compromise anchor quality
- The reported benchmark is specific to RPLC negative mode; generalization to other chromatography/ionization modes is not established in the provided context

## Evidence

- [results] MassCube selects unique model peaks (by quality and intensity) in QC pools as anchors, uses linear interpolation to correct nonlinear RT shifts: [evidence span withheld — anchor status: not_found]
- [intro] adduct grouping including in-source fragments (ISF), chromatographic alignments, exhaustive structural annotations to statistical analysis and visualization: "adduct grouping including in-source fragments (ISF), chromatographic alignments, exhaustive structural annotations to statistical analysis and visualization"
- [methods] Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source.: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
- [readme] masscube is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing.: "masscube is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing."
