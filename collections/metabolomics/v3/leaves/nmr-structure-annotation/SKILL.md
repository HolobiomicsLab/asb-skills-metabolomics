---
name: nmr-structure-annotation
description: Use when you have 1H-13C HSQC (optionally multiplicity edited) NMR peak
  lists from a small natural product or metabolite and need to retrieve candidate
  structures or nearest known compounds from large chemical databases rather than
  limited NMR spectral libraries.
when_to_use_negative:
- Input is a raw FID or 2D matrix rather than an HSQC-derived peak list with 1H and
  13C chemical shifts — DeepSAT consumes peak lists, not processed images of full
  spectra
- Query spectra are 1D proton/carbon chemical-shift tables from reference libraries
  such as NMRShiftDB — the article reports searching NMRshiftDB by proton and carbon
  shifts achieved under 3% correct identifications, so classical library matching
  is the wrong tool here
- The target is a protein/oligomer or other macromolecule outside the small-molecule
  metabolite scope the model was trained on
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3332
tools:
- name: DeepSAT
  role: Core CNN multi-task model that ingests the HSQC peak list and predicts fingerprint
    bit string, molecular weight, and structure class for database similarity search
  repo: github.com/mwang87/DeepSAT
- name: SMART 3 / smart3 search API
  role: Programmatic classification endpoint (/api/smart3/search) that accepts peaks
    as a JSON list of dicts with 1H,13C headers
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:157a6a3c5ca3ed22d3f6f3368e9c27de79ee533dcb13d95157b2f2ebee2a5468
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  - SMART 3 / smart3 search API
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# nmr-structure-annotation

## Summary

Annotate and identify small-molecule structures from a single 1H-13C HSQC spectrum by running it through the DeepSAT CNN multi-task model, which predicts chemical fingerprint bit strings, molecular weights, and structure classes, then matches predictions against known-molecule fingerprints via cosine similarity to search chemical databases.

## When to use

Use when you have 1H-13C HSQC (optionally multiplicity edited) NMR peak lists from a small natural product or metabolite and need to retrieve candidate structures or nearest known compounds from large chemical databases rather than limited NMR spectral libraries.

## When NOT to use

- Input is a raw FID or 2D matrix rather than an HSQC-derived peak list with 1H and 13C chemical shifts — DeepSAT consumes peak lists, not processed images of full spectra
- Query spectra are 1D proton/carbon chemical-shift tables from reference libraries such as NMRShiftDB — the article reports searching NMRshiftDB by proton and carbon shifts achieved under 3% correct identifications, so classical library matching is the wrong tool here
- The target is a protein/oligomer or other macromolecule outside the small-molecule metabolite scope the model was trained on

## Inputs

- 1H-13C HSQC NMR spectrum peak list (JSON list of dicts with '1H' and '13C' shift headers)
- Multiplicity edited HSQC peak list (optional, uses the Edited-HSQC model variant)
- In-house reference HSQC spectral library of candidate molecules (e.g. the n=143,467 Moliverse library) or any molecular structure database such as PubChem
- Known small-molecule Morgan fingerprints for cosine-similarity comparison

## Outputs

- Predicted fingerprint bit string
- Predicted molecular weight
- Predicted structure class (Top-1 class prediction)
- Ranked list of similar structures with cosine similarity scores (similar structures flagged when score > 0.8)
- Annotation rate / top-1 correct identification rate over a compound set
- Optional occlusion-sensitivity map highlighting HSQC regions driving the prediction

## How to apply

Load the measured 1H-13C HSQC spectrum and submit its peak list as a JSON list of dicts with 1H and 13C headers to the DeepSAT API at /api/smart3/search, or use the web service; the CNN multi-task model, trained on 143,467 HSQC spectra from the in-house Moliverse reference library, predicts a fingerprint bit string, molecular weight, and structure class. Compare the predicted fingerprint against Morgan fingerprints generated from known small molecules in the candidate database and compute cosine similarity; treat two structures as 'similar' when the cosine score exceeds 0.8, the same threshold used to define the annotation rate. Rank candidate structures by cosine score and report the top-1 prediction and structure class; for multiplicity edited HSQC inputs, the corresponding edited-HSQC model variant should be used, which yielded a higher average fingerprint cosine score (0.8574) and molecular weight R-squared (0.9336) than the normal HSQC model. For cross-solvent work (e.g. methanol-d4 vs chloroform-d), apply the pipeline per solvent and retain compounds still correctly identified across solvents, validating that over 65% of results remain correct. To sanity-check that signals drive the prediction, run the occlusion sensitivity technique (e.g. on quercetin) to see which HSQC regions most influence the network's output.

## Related tools

- **DeepSAT** (Core CNN multi-task model that ingests the HSQC peak list and predicts fingerprint bit string, molecular weight, and structure class for database similarity search) — github.com/mwang87/DeepSAT
- **SMART 3 / smart3 search API** (Programmatic classification endpoint (/api/smart3/search) that accepts peaks as a JSON list of dicts with 1H,13C headers) — github.com/mwang87/DeepSAT

## Examples

```
POST /api/smart3/search with body [{"1H": 6.12, "13C": 128.4}, {"1H": 7.05, "13C": 116.9}]
```

## Evaluation signals

- Cosine similarity between predicted and ground-truth Morgan fingerprint should average around 0.8450 (normal HSQC) or 0.8574 (multiplicity edited HSQC) on the evaluation set
- Molecular weight prediction R-squared should be approximately 0.9183 (normal HSQC) or 0.9336 (Edited HSQC)
- Top-1 compound class accuracy should be about 90.4% (normal HSQC) or 90.8% (Edited HSQC)
- Structures are declared 'similar' only when the cosine score exceeds 0.8 — verify the same threshold is applied when computing annotation rate
- In the cross-solvent benchmark (18 compounds in methanol-d4 vs chloroform-d), top-1 correct identification rates were 74.4% (chloroform-d) and 66.7% (methanol-d4), with over 65% of results still correctly identified across solvents

## Limitations

- Interpretability analysis via occlusion sensitivity was only demonstrated on a single compound (quercetin) and does not fully explain the decision-making process of DeepSAT
- Performance is bounded by the reference structure library — the in-house HSQC library contained n=143,467 spectra and chemical diversity coverage remains limited relative to nature
- Some compound classes classify poorly, e.g. diazotetronic acids and derivatives (F1-score 0.64) and naphthalenes (F1-score 0.66)
- Solvent transfer is not perfect — fewer than all results are correctly identified across methanol-d4 and chloroform-d measurements (over 65% retention, not 100%)
- The MIT-licensed code, models, and ontology are the reusable artifacts; the README notes the TensorFlow Serving model metadata endpoint must match expected input names, so API input-name changes would break integration

## Evidence

- [intro] DeepSAT trained on HSQC spectra: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [results] Similarity threshold 0.8: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [results] Similarity threshold 0.8 for annotation: "For the measurement of annotation rate, cosine score of 0.8 was set as the threshold"
- [results] Cosine score 0.8450 normal HSQC: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [results] R-squared for molecular weight prediction: "for the normal and multiplicity edited HSQC models the results were 0.9183 and 0.9336, respectively"
- [results] Top-1 accuracy 90.4% and 90.8%: "the Top-1 accuracy of normal and Edited HSQC models were 90.4% and 90.8%, respectively"
- [discussion] Cross-solvent retention rate: "over 65% of the results were still correctly identified even though the data were recorded in different solvents"
- [discussion] Occlusion sensitivity on quercetin: "using the ‘occlusion sensitivity’ method on the compound quercetin"
- [readme] SMART 3 API peak format: "You can put in your peaks as a json list of dicts, with 1H,13C as headers."
