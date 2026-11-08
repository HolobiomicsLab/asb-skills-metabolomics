---
name: npclassifier-superclass-probability-prediction
description: Use when the only input is a 1H-13C HSQC spectrum (normal or multiplicity-edited)
  and you need predicted NPClassifier superclass labels/probabilities for a compound
  whose authentic NMR reference is absent from spectral libraries.
when_to_use_negative:
- The query molecule already has an authentic HSQC reference spectrum in libraries
  such as NMRShiftDB, BioMagResBank, HMDB, CH-NMR-NP, NP-MRD, or CSEARCH — direct
  library search will be more reliable.
- The input is a mass spectrometry fragmentation spectrum for GNPS molecular networking
  rather than an NMR HSQC spectrum; DeepSAT was designed for NMR input only.
- 'You need a fully explained, interpretable chemical-class assignment: the article
  notes occlusion sensitivity ''does not fully explain the decision-making process
  of DeepSAT'', and rare classes like diazotetronic acids and derivatives (F1 0.64)
  and naphthalenes (F1 0.66) score poorly.'
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0003
- http://edamontology.org/topic_3420
tools:
- name: DeepSAT
  role: Multi-task CNN whose softmax classification head produces the NPClassifier
    superclass probability vector from HSQC input; also available as a web service
    at deepsat.ucsd.edu and via the /api/smart3/search endpoint.
  repo: github.com/mwang87/DeepSAT
- name: NPClassifier
  role: Ontology of chemical structure classes whose superclass labels form the target
    classes of the DeepSAT classification head; model, data, and ontology are CC0-licensed.
- name: PubChem
  role: Downstream structure database searched with the predicted fingerprints/classes
    to retrieve similar small molecules when no authentic NMR reference exists.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:798a05f00c6392297c1902e3f6d42c57156283cfb3be2ad14f3279e84b562c26
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  - NPClassifier
  - PubChem
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0003
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# npclassifier-superclass-probability-prediction

## Summary

Predicts NPClassifier chemical superclasses from a 1H-13C HSQC spectrum via DeepSAT's multi-task CNN softmax classification head, assigning each query spectrum a probability vector over structure classes used to find chemically similar molecules in structure databases.

## When to use

Use when the only input is a 1H-13C HSQC spectrum (normal or multiplicity-edited) and you need predicted NPClassifier superclass labels/probabilities for a compound whose authentic NMR reference is absent from spectral libraries.

## When NOT to use

- The query molecule already has an authentic HSQC reference spectrum in libraries such as NMRShiftDB, BioMagResBank, HMDB, CH-NMR-NP, NP-MRD, or CSEARCH — direct library search will be more reliable.
- The input is a mass spectrometry fragmentation spectrum for GNPS molecular networking rather than an NMR HSQC spectrum; DeepSAT was designed for NMR input only.
- You need a fully explained, interpretable chemical-class assignment: the article notes occlusion sensitivity 'does not fully explain the decision-making process of DeepSAT', and rare classes like diazotetronic acids and derivatives (F1 0.64) and naphthalenes (F1 0.66) score poorly.

## Inputs

- 1H-13C HSQC spectrum as a peak list (1H/13C shift pairs) or rendered spectral image, normal or multiplicity-edited
- Optional: JSON peak list for the DeepSAT web API with 1H and 13C keys

## Outputs

- Per-spectrum NPClassifier superclass probability vector (softmax probabilities in [0,1])
- Per-spectrum Top-1 predicted superclass label
- Optional companion outputs from the same forward pass: 6144-bit Morgan fingerprint bit string and rounded molecular weight

## How to apply

Load the query HSQC spectrum (peak list or rendered spectral image) into the pretrained DeepSAT CNN, which was trained on 143,467 1H-13C HSQC spectra. In the multi-task architecture, the class-prediction head is activated with a softmax function and trained with sparse categorical cross-entropy, in contrast to the sigmoid/binary-cross-entropy fingerprint head and the MAPE regression head for molecular weight. Decode the softmax output into an NPClassifier superclass probability vector per spectrum. Use the predicted classes alongside the predicted 6144-bit Morgan fingerprint (radius 0-2) and rounded molecular weight to search chemical structure databases such as PubChem for similar small molecules, treating two structures as similar when the cosine score of predicted vs. known Morgan fingerprints exceeds 0.8. Benchmark the class head against Top-1 accuracy of 90.4% (normal HSQC) and 90.8% (multiplicity-edited HSQC) reported for DeepSAT.

## Related tools

- **DeepSAT** (Multi-task CNN whose softmax classification head produces the NPClassifier superclass probability vector from HSQC input; also available as a web service at deepsat.ucsd.edu and via the /api/smart3/search endpoint.) — github.com/mwang87/DeepSAT
- **NPClassifier** (Ontology of chemical structure classes whose superclass labels form the target classes of the DeepSAT classification head; model, data, and ontology are CC0-licensed.)
- **PubChem** (Downstream structure database searched with the predicted fingerprints/classes to retrieve similar small molecules when no authentic NMR reference exists.)

## Examples

```
POST peaks=[{'1H': 6.85, '13C': 115.2}, ...] to http://deepsat.ucsd.edu/api/smart3/search; decode NPClassifier superclass probabilities from the returned class head.
```

## Evaluation signals

- Per-spectrum superclass probabilities are all within [0, 1] and the per-head distribution is normalized (softmax property).
- Top-1 class accuracy on a labeled test set should approach the reported 90.4% (normal HSQC) or 90.8% (multiplicity-edited HSQC); large deviations indicate a data or model mismatch.
- Rare classes such as diazotetronic acids and derivatives and naphthalenes should be flagged as expected low-F1 outliers (0.64 and 0.66 respectively) rather than evidence of a broken pipeline.
- Predicted class should be consistent with the independently predicted Morgan fingerprint and molecular weight from the same model pass (e.g. a steroid-class prediction with an implausible MW for steroids suggests an input-formatting error).
- Cross-solvent robustness: over 65% of results remain correctly identified even when spectra were recorded in different solvents; a near-zero correct rate suggests preprocessing misalignment.

## Limitations

- The model's interpretability is incomplete: occlusion-sensitivity results 'do not fully explain the decision-making process of DeepSAT', and were only demonstrated on a single compound (quercetin).
- Low-confidence classes exist: diazotetronic acids and derivatives (F1-score 0.64) and naphthalenes (F1-score 0.66) show poor classification performance.
- Solvent transfer is imperfect — fewer than all spectra are correctly identified across solvents (over 65% correct).
- The paper's README does not document the exact input schema for /api/smart3/search beyond 'peaks as a json list of dicts, with 1H,13C as headers'; if model input names change, code must be updated to match the TensorFlow Serving /model/metadata endpoint.
- Coverage is bounded by the 143,467-spectrum in-house training library; classes absent from NPClassifier or underrepresented in the training set will have poor or absent predictions.

## Evidence

- [methods] Multi-task CNN head design: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [results] Performance of the classification head: "the Top-1 accuracy of normal and Edited HSQC models were 90.4% and 90.8%, respectively"
- [abstract] Use-case framing: "Using only the 1H-13C HSQC spectrum, DeepSAT identifies related known compounds and thus efficiently assists in the identification of molecular structures."
- [results] Class prediction head activation: [evidence span withheld — anchor status: not_found]
- [readme] Web API entry point: "Classify programmatically ```/api/smart3/search```"
- [discussion] Solvent-transfer robustness: "over 65% of the results were still correctly identified even though the data were recorded in different solvents"
- [results] Similarity threshold for downstream search: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [results] Weak classes: "diazotetronic acids and derivatives (F1-score of 0.64) and naphthalenes (F1-score of 0.66), showed low values"
