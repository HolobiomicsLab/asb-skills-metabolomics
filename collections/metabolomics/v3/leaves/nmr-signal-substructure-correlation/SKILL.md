---
name: nmr-signal-substructure-correlation
description: 'Use when interpreting a trained DeepSAT HSQC model: you need to verify
  that specific HSQC peaks drive predicted chemical fingerprints or structure classes
  by masking spectral regions and measuring attribution changes.'
when_to_use_negative:
- The goal is de novo structure elucidation or spectral-library search from an HSQC
  query — use DeepSAT's fingerprint-similarity retrieval path instead of occlusion
  interpretability.
- No trained DeepSAT model weights are available; occlusion sensitivity requires a
  trained network and cannot be applied to raw spectra alone.
- The compound lacks any known diagnostic cross-peaks or substructure assignment,
  leaving no reference against which to validate the attribution map.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3422
- http://edamontology.org/topic_0154
tools:
- name: DeepSAT
  role: Trained CNN whose occlusion sensitivity is analyzed to attribute HSQC peaks
    to molecular substructures; also provides the quercetin HSQC input via its repository.
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:a27a5eba42cf15c3328c8e1d142159cfa7797ac0b14916f5d576b3f8ad201200
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3422
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# nmr-signal-substructure-correlation

## Summary

Correlates specific 1H-13C HSQC cross-peaks with molecular substructures using occlusion sensitivity analysis on a trained DeepSAT CNN, providing interpretability evidence that the network associates NMR signals with functional groups.

## When to use

Use when interpreting a trained DeepSAT HSQC model: you need to verify that specific HSQC peaks drive predicted chemical fingerprints or structure classes by masking spectral regions and measuring attribution changes.

## When NOT to use

- The goal is de novo structure elucidation or spectral-library search from an HSQC query — use DeepSAT's fingerprint-similarity retrieval path instead of occlusion interpretability.
- No trained DeepSAT model weights are available; occlusion sensitivity requires a trained network and cannot be applied to raw spectra alone.
- The compound lacks any known diagnostic cross-peaks or substructure assignment, leaving no reference against which to validate the attribution map.

## Inputs

- Trained DeepSAT CNN model (multi-task supervised architecture for HSQC spectra)
- 1H-13C HSQC spectrum input (e.g. quercetin), including 1H/13C chemical-shift coordinates
- Known diagnostic cross-peaks / substructure assignments for the target compound

## Outputs

- Per-peak attribution map aligned to HSQC 1H/13C chemical-shift coordinates
- Attribution-map overlay rendered on the original HSQC spectrum (e.g. quercetin_occlusion_map.png)
- Qualitative (expert-review) confirmation that high-attribution peaks match diagnostic substructure signals

## How to apply

Load the trained DeepSAT CNN (a multi-task supervised model trained on 143,467 1H-13C HSQC spectra) and the quercetin HSQC spectrum from github.com/mwang87/DeepSAT. Slide an occlusion window across the HSQC image, masking each region in turn and recording how the network's predicted chemical fingerprint, molecular weight, and structure-class outputs change — the rationale being that the most influential regions are those most important for the prediction. Aggregate the occlusion scores into a per-peak attribution map aligned to the 1H/13C chemical-shift coordinates of the spectrum. Compare the highest-attribution peaks against the compound's known diagnostic cross-peaks (for quercetin, the A-ring signals at δC/δH 92.9/6.30 and 97.8/6.19) to confirm signal–substructure correlation. Render the attribution map as an overlay on the original HSQC spectrum; validate qualitatively by expert review that high-attribution regions coincide with known diagnostic signals.

## Related tools

- **DeepSAT** (Trained CNN whose occlusion sensitivity is analyzed to attribute HSQC peaks to molecular substructures; also provides the quercetin HSQC input via its repository.) — github.com/mwang87/DeepSAT

## Examples

```
from deepsat import occlusion; occlusion.run(model_path='deepsat_model.h5', spectrum='quercetin_hsqc.csv', out='quercetin_occlusion_map.png')
```

## Evaluation signals

- Occlusion scores for the reported diagnostic peaks (quercetin: δC/δH 92.9/6.30 and 97.8/6.19) are substantially higher than for non-diagnostic spectral regions.
- Attribution map overlay renders on the original HSQC spectrum with 1H/13C coordinates correctly aligned.
- Output file (e.g. quercetin_occlusion_map.png) is produced and saved.
- Expert qualitative review confirms high-attribution regions coincide with the compound's known functional-group NMR signals, consistent with the paper's occlusion sensitivity analysis on quercetin.
- Masking a spectral region produces a measurable change in the network's predicted chemical fingerprint and class outputs, confirming the occlusion pipeline is functioning.

## Limitations

- Interpretability analysis was performed on a single compound (quercetin) and does not fully explain the model's decision-making process.
- Qualitative expert review is the validation method, so results are not quantitatively thresholded.
- Occlusion sensitivity is specific to image-style HSQC inputs of the trained DeepSAT CNN and does not transfer to other spectral representations or untrained networks.

## Evidence

- [results] occlusion sensitivity to correlate NMR signals with substructures: [evidence span withheld — anchor status: not_found]
- [discussion] using the 'occlusion sensitivity' method on the compound quercetin: [evidence span withheld — anchor status: not_found]
- [discussion] these results do not fully explain the decision-making process of DeepSAT: "Even though these results do not fully explain the decision-making process of DeepSAT"
- [readme] github.com/mwang87/DeepSAT: [evidence span withheld — anchor status: not_found]
- [intro] DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
