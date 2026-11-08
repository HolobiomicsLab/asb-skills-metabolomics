---
name: cnn-model-interpretation-nmr
description: 'Use when a trained NMR spectrum-to-structure CNN (e.g., DeepSAT) needs
  interpretability: you want to know which regions of an HSQC spectrum drive fingerprint
  or class predictions, to validate that the model associates chemical-shift peaks
  with real functional groups.'
when_to_use_negative:
- You need to identify or annotate an unknown compound's structure — this skill only
  explains model decisions, it does not perform the retrieval or classification itself
  (use DeepSAT's /api/smart3/search endpoint for that).
- The model's input format or metadata endpoint has changed — the README notes that
  model input names must be updated in the code if they change, so interpretability
  output would be invalid until re-integrated.
- You require a formal or quantitative attribution metric with error bars — the article
  notes the occlusion results 'do not fully explain the decision-making process of
  DeepSAT' and were assessed qualitatively for a single compound.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0593
tools:
- name: DeepSAT
  role: The trained CNN model being interpreted; provides the prediction function
    whose outputs (Morgan fingerprint bits and structure classes) are occluded to
    compute attribution.
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:699b1ae46e5fedfb247f454e2420d3cc688567fcfe95e77f3fd322457bd17fc7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cnn-model-interpretation-nmr

## Summary

Applies occlusion sensitivity to a trained DeepSAT convolutional neural network that predicts chemical fingerprints, molecular weights, and structure classes from 1H-13C HSQC spectra, generating per-peak attribution maps that link specific HSQC cross-peaks to molecular substructures.

## When to use

Use when a trained NMR spectrum-to-structure CNN (e.g., DeepSAT) needs interpretability: you want to know which regions of an HSQC spectrum drive fingerprint or class predictions, to validate that the model associates chemical-shift peaks with real functional groups.

## When NOT to use

- You need to identify or annotate an unknown compound's structure — this skill only explains model decisions, it does not perform the retrieval or classification itself (use DeepSAT's /api/smart3/search endpoint for that).
- The model's input format or metadata endpoint has changed — the README notes that model input names must be updated in the code if they change, so interpretability output would be invalid until re-integrated.
- You require a formal or quantitative attribution metric with error bars — the article notes the occlusion results 'do not fully explain the decision-making process of DeepSAT' and were assessed qualitatively for a single compound.

## Inputs

- Trained DeepSAT CNN model weights (normal or multiplicity-edited HSQC variant)
- Query 1H-13C HSQC spectrum (or multiplicity-edited HSQC spectrum) as peak list or processed image, e.g., quercetin HSQC with peaks at δC/δH 92.9/6.30 and 97.8/6.19
- Reference compound structure (SMILES/InChI) and its known Morgan fingerprint for validation

## Outputs

- Per-peak occlusion attribution map aligned to 1H/13C chemical-shift coordinates
- Overlaid visualization (e.g., PNG) of attribution map on the original HSQC spectrum
- List of high-attribution cross-peaks ranked by importance, optionally annotated with substructure assignments

## How to apply

Load the trained DeepSAT CNN (trained on 143,467 1H-13C HSQC spectra) and the query HSQC spectrum. Slide an occlusion window across the spectrum, masking each region in turn and recording the change in the network's predicted Morgan fingerprint bits and class outputs. Aggregate occlusion scores into a per-peak attribution map aligned to the 1H/13C chemical-shift coordinates. Compare high-attribution peaks against the compound's known diagnostic cross-peaks — for quercetin, the A-ring peaks at δC/δH 92.9/6.30 and 97.8/6.19 should rank among the most attributive regions. Overlay the attribution map on the original HSQC spectrum and validate by expert review that high-attribution regions coincide with diagnostic NMR signals.

## Related tools

- **DeepSAT** (The trained CNN model being interpreted; provides the prediction function whose outputs (Morgan fingerprint bits and structure classes) are occluded to compute attribution.) — github.com/mwang87/DeepSAT

## Examples

```
curl -X POST https://deepsat.ucsd.edu/api/smart3/search -H 'Content-Type: application/json' -d '[{"1H":6.30,"13C":92.9},{"1H":6.19,"13C":97.8}]'
```

## Evaluation signals

- High-attribution regions of the occlusion map must overlap the compound's known diagnostic HSQC cross-peaks (e.g., quercetin's A-ring peaks at δC/δH 92.9/6.30 and 97.8/6.19).
- The occlusion score distribution must be sparse — only a small subset of spectral regions should show large prediction deltas, consistent with localized chemical-shift features.
- The overlay image must show the attribution map aligned to the same chemical-shift axes as the original HSQC spectrum.
- For the normal HSQC model, the un-occluded baseline prediction should reproduce the reported metrics (fingerprint cosine ~0.845, MW R² ~0.918, Top-1 class accuracy ~90.4%) before occlusion.
- The attribution map must be reproducible when re-run with the same model weights and input spectrum (deterministic occlusion sweep).

## Limitations

- Interpretability analysis was performed on only one compound (quercetin); results 'do not fully explain the decision-making process of DeepSAT'.
- Occlusion sensitivity provides qualitative, expert-reviewed attribution rather than a calibrated per-peak importance score with statistical uncertainty.
- The analysis assumes the occlusion window covers exactly one spectral feature; overlapping peaks or broad regions may confound attribution.
- DeepSAT was trained on a specific in-house HSQC library (n=143,467) — attribution reflects features learned from that distribution and may not transfer to spectra from very different chemical space or solvent conditions.
- Model input names must match the serving metadata; if the input schema changes, the occlusion pipeline must be updated accordingly (per the README note on /model/metadata).

## Evidence

- [results] Occlusion sensitivity analysis showed that the two HSQC peaks of quercetin at δC/δH 92.9/6.30 and 97.8/6.19 were significantly correlated with its A-ring substructure: "the occlusion sensitivity technique was applied to analyze which parts of an image are most important for a deep network’s prediction"
- [discussion] Interpretability limited to one compound; does not fully explain model decisions: "Even though these results do not fully explain the decision-making process of DeepSAT"
- [intro] DeepSAT is a CNN-based multi-task supervised learning model trained on 143,467 1H-13C HSQC spectra that predicts chemical fingerprints, molecular weights, and structure classes: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [results] Average fingerprint cosine between predicted and ground truth was 0.8450 for the normal HSQC model: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [results] Top-1 class accuracy of the normal HSQC model was 90.4%: "the Top-1 accuracy of normal and Edited HSQC models were 90.4% and 90.8%, respectively"
- [readme] Model input names must match the serving metadata; if changed, update the code accordingly: "We pass through tensorflow serving at this url: ```/model/metadata``` If the model input names change, then we need to change it in the code"
