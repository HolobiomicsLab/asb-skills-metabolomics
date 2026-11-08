---
name: hsqc-spectral-peak-annotation
description: Use when a trained DeepSAT model has already predicted fingerprints/classes
  from a 1H-13C HSQC spectrum and you need to attribute the prediction to specific
  spectral peaks or substructures, e.g. quercetin's A-ring HSQC peaks.
when_to_use_negative:
- When you need quantitative, statistically validated peak assignments — occlusion
  sensitivity is a qualitative interpretability aid that 'does not fully explain the
  decision-making process of DeepSAT'.
- When you have no trained DeepSAT model or HSQC image input — this workflow operates
  on the model's internal outputs, not on raw FIDs or other NMR spectra types (1D
  1H/13C alone won't work).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: DeepSAT
  role: Trained CNN used to generate the predictions being attributed via occlusion
    sensitivity; also provides the scaffold ontology and molecular fingerprint output
    space.
  repo: https://github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:ab6f7507dad14759368de5590156decee9f95288851055168114b02ed50b75d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mwang87/DeepSAT
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hsqc-spectral-peak-annotation

## Summary

An interpretability workflow from the DeepSAT paper that uses occlusion sensitivity to map which 1H-13C HSQC peaks drive a CNN's prediction of molecular fingerprints and scaffold classes, then links those peaks to substructures via the molecular ontology.

## When to use

Use when a trained DeepSAT model has already predicted fingerprints/classes from a 1H-13C HSQC spectrum and you need to attribute the prediction to specific spectral peaks or substructures, e.g. quercetin's A-ring HSQC peaks.

## When NOT to use

- When you need quantitative, statistically validated peak assignments — occlusion sensitivity is a qualitative interpretability aid that 'does not fully explain the decision-making process of DeepSAT'.
- When you have no trained DeepSAT model or HSQC image input — this workflow operates on the model's internal outputs, not on raw FIDs or other NMR spectra types (1D 1H/13C alone won't work).

## Inputs

- Trained DeepSAT CNN model (normal or multiplicity-edited HSQC)
- 1H-13C HSQC spectrum image/peak list for the query compound (e.g. quercetin)
- Known diagnostic cross-peaks of the query compound for validation (e.g. quercetin δC/δH 92.9/6.30 and 97.8/6.19)
- DeepSAT scaffold ontology / ClassyFire class list

## Outputs

- Occlusion sensitivity attribution map over the HSQC chemical-shift space
- Ranked list of HSQC peaks by attribution score
- Qualitative link between high-attribution peaks and molecular substructures

## How to apply

Take the trained DeepSAT CNN (trained on ~143,467 1H-13C HSQC spectra from the Moliverse database) and run the 'occlusion sensitivity' method: systematically occlude regions of the input HSQC image and measure the change in the network's output (predicted fingerprint bits, molecular weight, or scaffold class), yielding an attribution map over chemical-shift space. Compare the highest-attribution peaks against known diagnostic cross-peaks of the target compound — for quercetin, the peaks at δC/δH 92.9/6.30 and 97.8/6.19 correlate with its A-ring substructure. Use the model's scaffold ontology (ClassyFire-style classes, e.g. naphthalenes with F1-score 0.66) to interpret which substructures the peaks encode, and render the attribution as an overlay on the original HSQC plot. Note this is qualitative/expert-validation, not a quantitative cutoff-based method.

## Related tools

- **DeepSAT** (Trained CNN used to generate the predictions being attributed via occlusion sensitivity; also provides the scaffold ontology and molecular fingerprint output space.) — https://github.com/mwang87/DeepSAT

## Examples

```
POST /api/smart3/search with peaks JSON; then run occlusion sensitivity: occlude HSQC regions and record delta in predicted fingerprint bits / class probabilities (implementation in DeepSAT repo).
```

## Evaluation signals

- High-attribution peaks should coincide with the query compound's known diagnostic NMR signals (e.g. quercetin's A-ring peaks at δC/δH 92.9/6.30 and 97.8/6.19).
- Attribution map should be a 2D image aligned to the 1H/13C axes of the input HSQC, not a 1D projection.
- Occlusion scores should show spatial localization (peaks) rather than uniform noise across the spectrum.
- For a well-trained model (cosine score 0.8450-0.8574 for fingerprint prediction), attributions should be stable across repeated runs with small occlusion perturbations.

## Limitations

- Results do not fully explain the decision-making process of DeepSAT — this is a qualitative, single-compound case study, not a general interpretability guarantee.
- Attribution depends on the quality of the trained model; low-confidence predictions (e.g. scaffold classes with F1-score ~0.64-0.66) will have unreliable attributions.
- Applicable to HSQC spectra (normal or multiplicity-edited) as supported by DeepSAT's input; other NMR experiments or non-NMR data are out of scope.

## Evidence

- [results] the occlusion sensitivity technique was applied to analyze which parts of an image are most important for a deep network's prediction: [evidence span withheld — anchor status: not_found]
- [discussion] using the 'occlusion sensitivity' method on the compound quercetin: [evidence span withheld — anchor status: not_found]
- [discussion] Even though these results do not fully explain the decision-making process of DeepSAT: "Even though these results do not fully explain the decision-making process of DeepSAT"
- [results] the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [readme] The license as included for the software is MIT. Additionally, all data, models, and ontology are licensed as [CC0](https://creativecommons.org/share-your-work/public-domain/cc0/).: "The license as included for the software is MIT. Additionally, all data, models, and ontology are licensed as [CC0](https://creativecommons.org/share-your-work/public-domain/cc0/)."
