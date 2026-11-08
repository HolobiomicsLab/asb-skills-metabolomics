---
name: natural-product-identification-benchmarking
description: Reference for evaluating NMR-based structure annotation tools on measured
  natural product HSQC spectra, especially when testing performance under changed
  solvent conditions (methanol-d4 vs chloroform-d) or comparing against NMRshiftDB
  shift-search baselines.
when_to_use_negative:
- When the query is not an HSQC spectrum but a full 1D proton/carbon NMR spectrum
  processed by reference-search tools like MetaboMiner or COLMAR — the article reports
  those achieved under 3% correct identification via NMRshiftDB shift search, so a
  shift-matching baseline is the wrong frame for DeepSAT-style fingerprint prediction.
- When the goal is de novo structure elucidation rather than identification of related
  known compounds or similar-structure retrieval.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_0154
tools:
- name: DeepSAT
  role: CNN multi-task model that predicts Morgan fingerprints, molecular weights,
    and structure classes from 1H-13C HSQC spectra for similarity-based identification.
  repo: https://github.com/mwang87/DeepSAT
- name: NMRshiftDB
  role: Reference spectral library used for baseline chemical-shift matching (thresholds
    0.5 ppm carbon / 0.05 ppm proton) in benchmark comparisons.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:56c3436e873633a847a80b132fbe13e6888e2e31bfcedcd8eceb8eb998599be2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mwang87/DeepSAT
  tools:
  - DeepSAT
  - NMRshiftDB
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# natural-product-identification-benchmarking

## Summary

Benchmark DeepSAT, a CNN multi-task model trained on 143,467 1H-13C HSQC spectra, for natural product structure identification by scoring predicted Morgan fingerprints, molecular weights, and structure classes against known small molecules using a cosine-score threshold of 0.8, including cross-solvent robustness testing in methanol-d4 vs chloroform-d.

## When to use

Reference for evaluating NMR-based structure annotation tools on measured natural product HSQC spectra, especially when testing performance under changed solvent conditions (methanol-d4 vs chloroform-d) or comparing against NMRshiftDB shift-search baselines.

## When NOT to use

- When the query is not an HSQC spectrum but a full 1D proton/carbon NMR spectrum processed by reference-search tools like MetaboMiner or COLMAR — the article reports those achieved under 3% correct identification via NMRshiftDB shift search, so a shift-matching baseline is the wrong frame for DeepSAT-style fingerprint prediction.
- When the goal is de novo structure elucidation rather than identification of related known compounds or similar-structure retrieval.

## Inputs

- Measured 1H-13C HSQC spectra of natural products (also Multiplicity Edited HSQC) as lists of {1H shift, 13C shift} peaks
- Known small-molecule structures for generating reference Morgan fingerprints
- Pretrained DeepSAT CNN multi-task model

## Outputs

- Per-compound top-1 identification/annotation rate under each solvent condition
- Cross-solvent retention rate (fraction correctly identified when solvent changes)
- Average cosine score between predicted and ground-truth fingerprints
- R-squared for molecular weight prediction
- Top-1 compound class prediction accuracy (with F1-score per class)
- Occlusion-sensitivity maps linking HSQC signals to substructures (e.g. quercetin)

## How to apply

Load measured 1H-13C HSQC spectra of natural products (e.g. 18 compounds in methanol-d4 and the same 18 in chloroform-d) and input each into the pretrained DeepSAT CNN multi-task model to predict fingerprint bit strings, molecular weights, and structure classes. Compare predicted fingerprints against Morgan fingerprints generated from known small molecules using cosine similarity, applying the threshold of cosine score 0.8 used for annotation rate measurement. For baseline comparison, match query chemical shifts against a reference library using thresholds of 0.5 ppm for carbon signals and 0.05 ppm for proton signals. Record average cosine score (reported as 0.8450 normal / 0.8574 edited HSQC), R-squared for molecular weight prediction (0.9183 / 0.9336), and Top-1 compound class accuracy (90.4% / 90.8%). For solvent robustness, match results across the two solvent conditions and compute the fraction of compounds still correctly identified when the solvent changes — the article reports over 65% retention, with DeepSAT's top-1 rate at 74.4% (chloroform-d) and 66.7% (methanol-d4). Optionally run occlusion sensitivity on a representative compound (quercetin) to probe which HSQC signals drive predictions.

## Related tools

- **DeepSAT** (CNN multi-task model that predicts Morgan fingerprints, molecular weights, and structure classes from 1H-13C HSQC spectra for similarity-based identification.) — https://github.com/mwang87/DeepSAT
- **NMRshiftDB** (Reference spectral library used for baseline chemical-shift matching (thresholds 0.5 ppm carbon / 0.05 ppm proton) in benchmark comparisons.)

## Examples

```
curl -X POST https://deepsat.ucsd.edu/api/smart3/search -H 'Content-Type: application/json' -d '[{"1H": 6.88, "13C": 116.5}, {"1H": 7.42, "13C": 121.3}]'
```

## Evaluation signals

- Average cosine score between predicted and ground-truth fingerprints ≥ 0.8450 (normal HSQC) or ≥ 0.8574 (Edited HSQC) as reported.
- R-squared for molecular weight prediction in the range 0.9183–0.9336.
- Top-1 compound class accuracy ≥ 90.4%.
- Over 65% of results still correctly identified when the solvent changes from chloroform-d to methanol-d4 (reported top-1 rates 74.4% and 66.7%).
- Annotation rate computed with the fixed cosine-score threshold of 0.8, reproducible from solvent_comparison.csv style per-compound results.

## Limitations

- Over 65% of the results were still correctly identified even though the data were recorded in different solvents — i.e. cross-solvent robustness is incomplete, with a per-compound drop between chloroform-d (74.4%) and methanol-d4 (66.7%).
- Classification F1-scores are low for some classes (diazotetronic acids and derivatives 0.64, naphthalenes 0.66).
- Interpretability via occlusion sensitivity was only demonstrated on a single compound (quercetin) and does not fully explain the model's decision-making process.
- README does not state model weight download details or full web-service parameters; only /api/smart3/search and /model/metadata endpoints are described.

## Evidence

- [intro] DeepSAT is a CNN-based multi-task supervised learning model trained on 143,467 1H-13C HSQC spectra that predicts chemical fingerprints, molecular weights, and structure classes: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [results] cosine threshold 0.8: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [results] shift matching thresholds: "The thresholds for the peak shift differences were set as 0.5 ppm for carbon signals and 0.05 ppm for proton signals"
- [results] reported cosine scores: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [results] cross-solvent retention: "over 65% of the results were still correctly identified even though the data were recorded in different solvents"
- [results] per-solvent top-1 rates: [evidence span withheld — anchor status: not_found]
- [discussion] occlusion sensitivity on quercetin: "using the ‘occlusion sensitivity’ method on the compound quercetin"
- [results] NMRshiftDB baseline: "The results from searching NMRshiftDB by proton and carbon NMR shifts, however, achieved under 3% of correct identifications."
- [readme] README API endpoint: "Classify programmatically ```/api/smart3/search```"
- [readme] input format: "You can put in your peaks as a json list of dicts, with 1H,13C as headers."
