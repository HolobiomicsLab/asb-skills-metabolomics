---
name: anchor-peak-selection-by-spacing-and-noise
description: Use when aligning LC-MS feature tables across many chromatograms where
  retention-time drift is nonlinear and QC-pool peak metadata (RT, m/z, height, spacing,
  noise) are available to define anchors.
when_to_use_negative:
- Input is raw vendor data (mzML or Bruker .d) rather than a processed QC feature
  table — anchors require MassCube peak detection and feature tables first
- The goal is metabolite identification or annotation rather than chromatographic
  alignment — MassCube does not directly perform metabolite identification
- No QC pools with unique, high-quality model peaks exist in the study, so spacing/noise-based
  anchor filtering cannot be populated
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassCube
  role: Produces the QC feature table with RT, m/z, height, spacing and noise metadata
    from which anchors are selected, and provides the chromatographic alignment module
    the interpolation correction feeds into
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime in which the anchor filtering (spacing = 0.01, noise < 0.3), top-50
    height ranking, train/test split, and linear-interpolation RT correction are implemented
  repo: https://github.com/huaxuyu/masscube/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:e70c24d08342311a8c492c7d03e35d164a56b5cb414a1f86dfe9f9ecf8136109
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# anchor-peak-selection-by-spacing-and-noise

## Summary

Reconstructs MassCube's retention-time anchor selection: unique model peaks from QC pools are filtered by m/z spacing (= 0.01) and noise (< 0.3), ranked by height, and used with linear interpolation to correct nonlinear RT drift across chromatograms (40.5 s → 1.0 s residual shift in RPLC negative mode).

## When to use

Use when aligning LC-MS feature tables across many chromatograms where retention-time drift is nonlinear and QC-pool peak metadata (RT, m/z, height, spacing, noise) are available to define anchors.

## When NOT to use

- Input is raw vendor data (mzML or Bruker .d) rather than a processed QC feature table — anchors require MassCube peak detection and feature tables first
- The goal is metabolite identification or annotation rather than chromatographic alignment — MassCube does not directly perform metabolite identification
- No QC pools with unique, high-quality model peaks exist in the study, so spacing/noise-based anchor filtering cannot be populated

## Inputs

- MassCube-processed QC feature table containing RT, m/z, height, spacing, and noise metadata per peak
- QC-pool chromatograms from a large-scale LC-MS study (e.g. RPLC negative mode)

## Outputs

- Filtered set of up to 50 anchor peaks (m/z spacing = 0.01, noise < 0.3, top-ranked by height)
- Linear interpolation model mapping RT drift between chromatograms
- Corrected RT values for test-set anchors
- Residual RT-shift metric after correction (e.g. 1.0 s vs 40.5 s pre-correction)

## How to apply

Load the MassCube-processed QC feature table carrying RT, m/z, height, spacing, and noise metadata for each peak. Filter candidate anchors using validity criteria of m/z spacing = 0.01 and noise < 0.3, so only well-resolved, low-noise model peaks are retained. Rank the valid anchors by peak height and keep the top 50, then split them into train and test subsets. Fit a linear interpolation model on train-set anchor RTs and apply it to correct RT shifts of test-set anchors, which handles nonlinear retention-time drift between chromatograms. Report the residual test-set RT shift after correction; in RPLC negative mode the uncorrected shift of 40.5 s should drop to about 1.0 s.

## Related tools

- **MassCube** (Produces the QC feature table with RT, m/z, height, spacing and noise metadata from which anchors are selected, and provides the chromatographic alignment module the interpolation correction feeds into) — https://github.com/huaxuyu/masscube
- **Python** (Runtime in which the anchor filtering (spacing = 0.01, noise < 0.3), top-50 height ranking, train/test split, and linear-interpolation RT correction are implemented) — https://github.com/huaxuyu/masscube/

## Examples

```
pip install masscube
```

## Evaluation signals

- All retained anchors satisfy the validity criteria: m/z spacing = 0.01 and noise < 0.3
- Anchor set is limited to the top 50 candidates ranked by peak height, split into train and test subsets
- Test-set RT shift after linear-interpolation correction is substantially reduced versus uncorrected, reaching approximately 1.0 s against a 40.5 s pre-correction shift in RPLC negative mode
- Corrected RT shifts are consistent between train and test anchor subsets, indicating the interpolation generalizes beyond fitting anchors

## Limitations

- Linear interpolation between a limited anchor set approximates nonlinear RT drift and depends on anchors being unique, high-quality model peaks in QC pools
- The reported benchmark (40.5 s → 1.0 s) is specific to RPLC negative mode; behavior in other chromatography/ionization modes is not cited in the surfaced cards
- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all such scenarios, which can affect anchor quality

## Evidence

- [methods] Filter anchors using m/z spacing = 0.01 and noise < 0.3 as validity criteria.: [evidence span withheld — anchor status: not_found]
- [methods] Rank valid anchors by height and retain the top 50.: [evidence span withheld — anchor status: not_found]
- [results] uses linear interpolation to correct nonlinear RT shifts: [evidence span withheld — anchor status: not_found]
- [readme] Highly accurate nontargeted peak detection and segmentation.: "Highly accurate nontargeted peak detection and segmentation."
