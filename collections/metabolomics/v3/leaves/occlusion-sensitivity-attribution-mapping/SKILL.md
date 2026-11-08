---
name: occlusion-sensitivity-attribution-mapping
description: Use when a trained DeepSAT CNN model has made predictions on a 1H-13C
  HSQC spectrum and you need to interpret which spectral peaks (e.g. δC/δH 92.9/6.30
  in quercetin) drive the fingerprint and class outputs, producing an attribution
  map aligned to chemical-shift coordinates.
when_to_use_negative:
- The goal is structure annotation rather than model interpretability — use the standard
  DeepSAT search/classification workflow instead
- The network input is not a 1H-13C HSQC spectrum (e.g. multiplicity-edited HSQC is
  supported, but other spectral types are out of scope)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: DeepSAT
  role: Trained CNN model whose fingerprint, molecular-weight, and class predictions
    are perturbed by occlusion to produce attribution scores
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:559838aeb24b17492a86ada3f9586b2929a600283e5e7e707f3eddd7d4db16e1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# occlusion-sensitivity-attribution-mapping

## Summary

Occlusion sensitivity analysis that slides a masking window over an input 1H-13C HSQC spectrum to measure how occlusion of each region shifts a trained DeepSAT CNN's predicted chemical fingerprints, molecular weight, and structure classes, yielding a per-peak attribution map that links specific NMR signals to molecular substructures.

## When to use

Use when a trained DeepSAT CNN model has made predictions on a 1H-13C HSQC spectrum and you need to interpret which spectral peaks (e.g. δC/δH 92.9/6.30 in quercetin) drive the fingerprint and class outputs, producing an attribution map aligned to chemical-shift coordinates.

## When NOT to use

- The goal is structure annotation rather than model interpretability — use the standard DeepSAT search/classification workflow instead
- The network input is not a 1H-13C HSQC spectrum (e.g. multiplicity-edited HSQC is supported, but other spectral types are out of scope)

## Inputs

- trained DeepSAT CNN model checkpoint or served model endpoint
- 1H-13C HSQC spectrum of a target compound (e.g. quercetin) with 1H/13C chemical-shift coordinates

## Outputs

- per-peak occlusion attribution map aligned to HSQC chemical-shift coordinates
- rendered attribution overlay on the original HSQC spectrum (e.g. quercetin_occlusion_map.png)
- comparison of high-attribution peaks against known diagnostic cross-peaks

## How to apply

Load the trained DeepSAT CNN (from mwang87/DeepSAT, served via the /model/metadata API) together with the quercetin 1H-13C HSQC spectrum. Slide an occlusion window across the spectrum, masking each region in turn and recording the change in the network's predicted fingerprint bit string, molecular weight, and structure class outputs. Aggregate occlusion scores into a per-peak attribution map aligned to the 1H/13C chemical-shift coordinates of the spectrum. Compare high-attribution peaks against known diagnostic cross-peaks (e.g. quercetin's A-ring substructure) to confirm the correlation between NMR signals and functional-group fingerprints. Render the attribution map as an overlay on the original HSQC spectrum and save it (e.g. quercetin_occlusion_map.png).

## Related tools

- **DeepSAT** (Trained CNN model whose fingerprint, molecular-weight, and class predictions are perturbed by occlusion to produce attribution scores) — github.com/mwang87/DeepSAT

## Examples

```
python occlusion_map.py --model deepsat_cnn --input quercetin_hsqc.csv --out quercetin_occlusion_map.png
```

## Evaluation signals

- High-attribution regions of the map coincide with quercetin's diagnostic NMR signals (δC/δH 92.9/6.30 and 97.8/6.19), consistent with the paper's occlusion sensitivity analysis
- Attribution map peaks align to valid HSQC chemical-shift coordinates rather than edge artefacts
- Occlusion scores show a clear separation between diagnostic peaks and background spectral noise
- Rendered overlay visually matches the original HSQC spectrum without distortion of peak positions

## Limitations

- Occlusion sensitivity does not fully explain the decision-making process of DeepSAT — the paper notes these results do not fully explain the model's decisions
- Interpretability analysis in the article is limited to a single compound (quercetin) and is not a general validation across diverse scaffolds

## Evidence

- [results] occlusion sensitivity technique applied to correlate NMR signals with substructures: "the occlusion sensitivity technique was applied to analyze which parts of an image are most important for a deep network’s prediction"
- [results] Occlusion sensitivity analysis on quercetin correlates HSQC peaks with A-ring substructure: [evidence span withheld — anchor status: not_found]
- [discussion] Occlusion sensitivity does not fully explain model decisions: "Even though these results do not fully explain the decision-making process of DeepSAT"
