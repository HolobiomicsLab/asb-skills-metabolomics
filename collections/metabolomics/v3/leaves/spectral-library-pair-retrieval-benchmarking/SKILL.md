---
name: spectral-library-pair-retrieval-benchmarking
description: Use when you hold MS/MS spectra with SMILES/InChI structural annotations
  (e.g. a held-out GNPS-derived test set) and must compare similarity measures on
  retrieving structurally related pairs.
when_to_use_negative:
- Spectra lack structural annotations (SMILES/InChI), so Tanimoto ground-truth pairs
  (e.g. threshold 0.6) cannot be computed
- You need absolute compound identification or discrimination between Tanimoto 0.8–0.9
  and a near-complete chemical match, which the paper reports MS2DeepScore cannot
  reliably resolve
- Data are embeddings/clustering or uncertainty-filtering tasks rather than retrieval-benchmark
  comparisons across similarity measures
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2DeepScore
  role: Primary similarity measure under test; pretrained Siamese model predicting
    Tanimoto scores from MS/MS spectrum pairs
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Python library used to import/handle spectra and compute modified Cosine scores;
    spectra are cleaned with its filtering pipeline
  repo: https://github.com/matchms/matchms
- name: Spec2Vec
  role: Unsupervised spectral similarity baseline compared against MS2DeepScore in
    the retrieval benchmark
- name: RDKit
  role: Computes molecular fingerprints (Morgan-2/Morgan-3 / ECFP pendants) for ground-truth
    Tanimoto structural similarity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:afe32d63822efb7e3ca65c00bc718af357d21b4ff3a04d3faec70578c6bd481d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - RDKit
  - Spec2Vec
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# spectral-library-pair-retrieval-benchmarking

## Summary

Benchmarks spectral similarity measures (e.g. MS2DeepScore vs. modified Cosine and Spec2Vec) by threshold-sweep precision/recall retrieval of chemically related MS/MS spectrum pairs, using structural Tanimoto similarity from fingerprints as ground truth.

## When to use

Use when you hold MS/MS spectra with SMILES/InChI structural annotations (e.g. a held-out GNPS-derived test set) and must compare similarity measures on retrieving structurally related pairs.

## When NOT to use

- Spectra lack structural annotations (SMILES/InChI), so Tanimoto ground-truth pairs (e.g. threshold 0.6) cannot be computed
- You need absolute compound identification or discrimination between Tanimoto 0.8–0.9 and a near-complete chemical match, which the paper reports MS2DeepScore cannot reliably resolve
- Data are embeddings/clustering or uncertainty-filtering tasks rather than retrieval-benchmark comparisons across similarity measures

## Inputs

- Annotated MS/MS test-set spectra (GNPS-derived; e.g. 3601 spectra of 500 unique compounds) with SMILES/InChI annotations
- Trained or pretrained MS2DeepScore model (e.g. ms2deepscore 0.2.0 .pt weights)
- Molecular structures (SMILES/InChI) for RDKit fingerprint computation
- Candidate similarity measures: modified Cosine and Spec2Vec alongside MS2DeepScore

## Outputs

- Pairwise spectral similarity score matrices for each measure over the test spectra
- Ground-truth Tanimoto similarity matrix over all spectrum pairs
- precision_recall_curves.csv with threshold, precision, and recall columns per similarity measure

## How to apply

Split annotated spectra by unique molecule (e.g. the paper's 3601-test-spectrum set from 500 unique InChIKey compounds held out of training) and compute ground-truth structural similarity for every spectrum pair as Tanimoto scores over RDKit fingerprints (Morgan-2/Morgan-3, i.e. ECFP-4/ECFP-6 pendants). Compute pairwise spectral scores for all three or more measures — MS2DeepScore via its pretrained Siamese model (available as an installable Python library, e.g. version 0.2.0, integrated in matchms), modified Cosine via matchms, and Spec2Vec. Sweep the pair-selection decision threshold from 0 to 1.0; at each threshold, label selected pairs as positive when true Tanimoto > 0.6, and record precision and recall per measure. Output threshold/precision/recall columns (e.g. to precision_recall_curves.csv) and judge correctness by whether the deep-learning curve dominates the modified Cosine and Spec2Vec curves across the sweep.

## Related tools

- **MS2DeepScore** (Primary similarity measure under test; pretrained Siamese model predicting Tanimoto scores from MS/MS spectrum pairs) — https://github.com/matchms/ms2deepscore
- **matchms** (Python library used to import/handle spectra and compute modified Cosine scores; spectra are cleaned with its filtering pipeline) — https://github.com/matchms/matchms
- **Spec2Vec** (Unsupervised spectral similarity baseline compared against MS2DeepScore in the retrieval benchmark)
- **RDKit** (Computes molecular fingerprints (Morgan-2/Morgan-3 / ECFP pendants) for ground-truth Tanimoto structural similarity)

## Examples

```
from matchms.Pipeline import Pipeline, create_workflow; from matchms.filtering.default_pipelines import DEFAULT_FILTERS; from ms2deepscore import MS2DeepScore; pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS, score_computations=[[MS2DeepScore, {"model": model}]])); pipeline.run(spectrum_file_name)
```

## Evaluation signals

- The MS2DeepScore precision-recall curve dominates the curves of both modified Cosine variants and Spec2Vec across the 0–1 threshold sweep (paper Figs. 4/5)
- Positive-pair labels are consistent: pairs count as retrieved only when true Tanimoto > 0.6 computed from RDKit fingerprints
- Test set composition matches the held-out design: spectra from compounds unseen during training (paper: 3601 spectra of 500 unique compounds)
- precision_recall_curves.csv contains one (threshold, precision, recall) triple per measure with thresholds spanning 0 to 1.0
- MS2DeepScore RMSE against true Tanimoto on the test set falls in the paper-reported ~0.15 range (RMSE 0.13–0.2 for Tanimoto 0.1–0.9)

## Limitations

- MS2DeepScore scores are not deterministic and can change when a new model is trained (different parameters or training data)
- Spectral libraries are biased toward certain compound classes and mixed instrumentation/quality, so the maximum achievable precision is unclear
- Difficult to discriminate high Tanimoto scores (0.8–0.9) from near-complete chemical match; there is no consensus on how best to quantify chemical relatedness

## Evidence

- [abstract] MS2DeepScore outperforms classical measures in P/R retrieval: "we demonstrate that MS2DeepScore outperforms classical spectral similarity measures in retrieving chemically related compound pairs from large mass spectral datasets"
- [results] Outperforms modified Cosine and Spec2Vec at Tanimoto > 0.6: "MS2DeepScore clearly outperforms both classical measures (two forms of the modified Cosine) as well as the unsupervised spectral similarity measure Spec2Vec, with respect to identifying high Tanimoto"
- [results] Test-set size and compound holdout: "The test set, for instance, contains 3601 spectra of 500 unique compounds"
- [results] RDKit fingerprints define ground-truth structural similarity: "circular fingerprints (we used Morgan-2 and Morgan-3, the RDKit [23] pendants of ECFP-4 and ECFP-6)"
- [readme] Tool purpose per README: "Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) from pairs of mass spectrometry spectra"
