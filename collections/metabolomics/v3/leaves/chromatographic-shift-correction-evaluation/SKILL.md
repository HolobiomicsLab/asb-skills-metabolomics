---
name: chromatographic-shift-correction-evaluation
description: Use when aligning feature retention times across many LC-MS chromatograms
  (e.g., large QC-pooled metabolomics studies) and you need quantifiable residual
  RT-shift benchmarks after correction.
when_to_use_negative:
- Input is already an aligned feature table with no residual RT drift to correct
- The study lacks QC pool samples from which unique, high-quality model peaks can
  be selected as anchors
- The goal is metabolite identification or MS/MS annotation — MassCube does not directly
  perform metabolite identification
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MassCube
  role: Produces the QC feature table with RT, m/z, height, spacing, and noise metadata
    used for anchor selection and RT-shift correction
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Language in which MassCube implements the anchor filtering, ranking, and linear
    interpolation RT correction
  repo: https://github.com/huaxuyu/masscube
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:de28431f9f9d911ac3bd9f4d176aaea22c3e08e99f6572c83883d2a63f99d7c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# chromatographic-shift-correction-evaluation

## Summary

A retention-time drift correction and benchmarking method from MassCube: unique high-quality model peaks in QC pools are selected as anchors, and linear interpolation over those anchors corrects nonlinear RT shifts across LC-MS chromatograms, evaluated by residual RT shift.

## When to use

Use when aligning feature retention times across many LC-MS chromatograms (e.g., large QC-pooled metabolomics studies) and you need quantifiable residual RT-shift benchmarks after correction.

## When NOT to use

- Input is already an aligned feature table with no residual RT drift to correct
- The study lacks QC pool samples from which unique, high-quality model peaks can be selected as anchors
- The goal is metabolite identification or MS/MS annotation — MassCube does not directly perform metabolite identification

## Inputs

- MassCube-processed QC feature table with columns for RT, m/z, height, spacing, and noise metadata
- Raw LC-MS data (mzML or supported Bruker .d files) processed through the MassCube peak-detection and alignment workflow

## Outputs

- Corrected retention times for test-set features
- List of selected RT anchor peaks (top 50 by height passing spacing/noise filters)
- Residual RT shift metric comparing pre- and post-correction drift (e.g., 40.5 s to 1.0 s in RPLC negative mode)

## How to apply

Load the MassCube-processed QC feature table containing RT, m/z, height, spacing, and noise metadata. Filter candidate anchors by validity criteria of m/z spacing = 0.01 and noise < 0.3, rank valid anchors by height, and retain the top 50 as model peaks. Split the top 50 anchors into train and test subsets, fit a linear interpolation model on the train-set anchor RTs, and apply it to correct test-set RT shifts. Rationale: selecting unique, high-intensity, low-noise model peaks in QC pools gives stable landmarks so that interpolation approximates nonlinear RT drift. Judge success by the reduction in residual RT shift — the MassCube benchmark reduced RPLC negative-mode test-set shifts from 40.5 s to ~1.0 s.

## Related tools

- **MassCube** (Produces the QC feature table with RT, m/z, height, spacing, and noise metadata used for anchor selection and RT-shift correction) — https://github.com/huaxuyu/masscube
- **Python** (Language in which MassCube implements the anchor filtering, ranking, and linear interpolation RT correction) — https://github.com/huaxuyu/masscube

## Examples

```
pip install masscube; from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj("sample.d")
```

## Evaluation signals

- Residual RT shift after correction is substantially reduced versus the uncorrected shift, targeting ~1.0 s residual against a 40.5 s pre-correction shift in RPLC negative mode
- All selected anchors satisfy the validity criteria of m/z spacing = 0.01 and noise < 0.3
- Exactly the top 50 anchors by height are retained and split into train and test subsets
- Corrected test-set RT shifts are computed only from train-set anchor RTs (no leakage of test anchors into the interpolation model)
- Anchor peaks are unique model peaks in QC pools, ranked by quality and intensity

## Limitations

- Linear interpolation is a linear approximation to nonlinear RT shifts, so correction accuracy depends on the density and quality of selected anchors
- External validated benchmarking datasets with large-scale ground-truth data are still missing, making independent evaluation of alignment accuracy difficult
- Correction depends on QC pool peak quality; noisy or non-unique peaks failing the spacing (0.01) and noise (< 0.3) filters cannot serve as anchors

## Evidence

- [other] uses linear interpolation to correct nonlinear RT shifts, reducing test-set shifts from 40.5 s to 1.0 s in RPLC negative mode.: [evidence span withheld — anchor status: not_found]
- [other] Filter anchors using m/z spacing = 0.01 and noise < 0.3 as validity criteria. 3. Rank valid anchors by height and retain the top 50.: [evidence span withheld — anchor status: not_found]
- [readme] masscube is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing.: "masscube is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing."
- [abstract] Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction: "Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction"
