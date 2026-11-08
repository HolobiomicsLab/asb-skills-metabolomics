---
name: hsqc-spectral-input-processing
description: Use when you have measured 1H-13C HSQC spectra (normal or multiplicity
  edited) and want to run DeepSAT structure annotation against large chemical databases
  like PubChem.
when_to_use_negative:
- Input is a full 1D 1H or 13C NMR spectrum rather than a 2D 1H-13C HSQC spectrum
  — DeepSAT uses only the HSQC spectrum.
- No reference chemical structure database is available to generate fingerprints for
  cosine comparison — DeepSAT searches known molecules, not novel structures.
- Baseline retrieval against a curated reference spectral library (e.g. NMRshiftDB)
  is desired — simple shift matching achieved under 3% correct identifications, so
  DeepSAT is the intended alternative.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: DeepSAT
  role: CNN multi-task model that consumes HSQC peak lists and outputs fingerprints,
    molecular weights, and structure classes for structure annotation
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:ec3c053f55b1c2b71e67f2ccb3283f010e7d83647e9a1a89089ec506a67bf5d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# hsqc-spectral-input-processing

## Summary

Prepare 1H-13C HSQC spectra as DeepSAT-compatible peak lists so a trained CNN multi-task model can predict fingerprint bit strings, molecular weights, and structure classes for compound identification.

## When to use

Use when you have measured 1H-13C HSQC spectra (normal or multiplicity edited) and want to run DeepSAT structure annotation against large chemical databases like PubChem.

## When NOT to use

- Input is a full 1D 1H or 13C NMR spectrum rather than a 2D 1H-13C HSQC spectrum — DeepSAT uses only the HSQC spectrum.
- No reference chemical structure database is available to generate fingerprints for cosine comparison — DeepSAT searches known molecules, not novel structures.
- Baseline retrieval against a curated reference spectral library (e.g. NMRshiftDB) is desired — simple shift matching achieved under 3% correct identifications, so DeepSAT is the intended alternative.

## Inputs

- 1H-13C HSQC spectrum as a JSON list of dicts with '1H' and '13C' peak keys
- Multiplicity edited HSQC spectra (alternatively supported)
- Pretrained DeepSAT model served via TensorFlow Serving

## Outputs

- Predicted fingerprint bit strings
- Predicted molecular weights
- Predicted compound classes (Top-1 accuracy ~90.4-90.8%)
- Ranked list of candidate small-molecule structures with cosine similarity scores (annotation rate at cosine threshold 0.8)

## How to apply

Convert the HSQC spectrum into a JSON list of peak dictionaries with '1H' and '13C' headers, then submit to the DeepSAT TensorFlow Serving endpoint at /model/metadata to confirm the model input names still match the code before querying /api/smart3/search. The pretrained CNN multi-task architecture, trained on 143,467 HSQC spectra, predicts Morgan-fingerprint-like bit strings, molecular weights, and structure classes from the spectrum alone. Compare each predicted fingerprint to fingerprints generated from known small molecules using cosine similarity, treating structures with cosine score over 0.8 as 'similar'. Multiplicity edited HSQC spectra are supported as alternative inputs; the edited model achieved an average fingerprint cosine score of 0.8574, molecular weight R-squared of 0.9336, and Top-1 class accuracy of 90.8%, slightly better than the normal HSQC model (0.8450, 0.9183, 90.4%).

## Related tools

- **DeepSAT** (CNN multi-task model that consumes HSQC peak lists and outputs fingerprints, molecular weights, and structure classes for structure annotation) — github.com/mwang87/DeepSAT

## Examples

```
curl -X POST /api/smart3/search -d '[{"1H": 3.5, "13C": 55.2}, {"1H": 6.1, "13C": 120.4}]'
```

## Evaluation signals

- Average cosine score between predicted and ground-truth fingerprints should be ~0.8450 (normal HSQC) or ~0.8574 (edited HSQC).
- Molecular weight prediction R-squared should be ~0.9183 (normal) or ~0.9336 (edited).
- Top-1 compound class accuracy should be ~90.4% (normal) or ~90.8% (edited).
- Annotation rate should be computed at a cosine score threshold of 0.8 for structure similarity.
- Check /model/metadata endpoint output: if model input names changed, the calling code must be updated before /api/smart3/search will accept peaks.

## Limitations

- Interpretability analysis via occlusion sensitivity was limited to a single compound (quercetin) and does not fully explain model decisions.
- Solvent transfer is incomplete: over 65% (not all) results were still correctly identified when spectra were recorded in methanol-d4 vs chloroform-d (74.4% vs 66.7% top-1).
- Privacy note: server logs record which structures were classified, not which user queried them.
- All data, models, and ontology are CC0-licensed; the software itself is MIT-licensed.

## Evidence

- [intro] CNN multi-task model on HSQC spectra: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [results] Similarity threshold for structures: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [readme] API endpoint for programmatic search: "Classify programmatically ```/api/smart3/search``` You can put in your peaks as a json list of dicts, with 1H,13C as headers."
- [readme] Model metadata check: "We pass through tensorflow serving at this url: ```/model/metadata``` If the model input names change, then we need to change it in the code"
- [discussion] Multiplicity edited HSQC support: "DeepSAT supports the use of Multiplicity Edited HSQC spectra as inputs"
- [discussion] Solvent sensitivity retention rate: "over 65% of the results were still correctly identified even though the data were recorded in different solvents"
- [results] Model performance on fingerprints: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
