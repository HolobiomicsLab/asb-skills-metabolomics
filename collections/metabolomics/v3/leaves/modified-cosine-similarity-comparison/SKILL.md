---
name: modified-cosine-similarity-comparison
description: Use when comparing spectral similarity measures on annotated MS/MS data
  (e.g. a 3601-spectrum test set of 500 unique InChIKey compounds from GNPS) to quantify
  which measure retrieves high-Tanimoto (Tanimoto > 0.6) compound pairs via precision/recall
  curves.
when_to_use_negative:
- The spectra lack InChIKey/SMILES annotations, so no RDKit fingerprint ground-truth
  Tanimoto similarities can be computed for labelling positive pairs
- The goal is compound identification (near-complete chemical match) rather than retrieval
  of chemically related pairs — the paper notes it is difficult to discriminate Tanimoto
  0.8–0.9 from a near-complete chemical match
- Applying MS2DeepScore scores trained on GNPS library biases to experimental data
  from compound classes poorly represented in spectral libraries
tools:
- name: MS2DeepScore
  role: Siamese neural network predicting structural similarity (Tanimoto) from MS/MS
    spectrum pairs; provides the learned similarity scores being benchmarked
  repo: https://github.com/matchms/ms2deepscore
- name: Spec2Vec
  role: Unsupervised spectral similarity baseline compared against MS2DeepScore and
    modified Cosine in the precision/recall sweep
- name: matchms
  role: Python library for spectrum import, handling, filtering, and computing the
    modified Cosine score comparisons
- name: RDKit
  role: Computes ground-truth molecular fingerprints (Morgan-2/Morgan-3) and Tanimoto
    similarities for labelling chemically related pairs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:c463b9a8cd7ed4f4e6d9ce6d1f449621550b5f41b2bbf8aea8d20929386b713c
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# modified-cosine-similarity-comparison

## Summary

Benchmark spectral similarity measures (MS2DeepScore, modified Cosine, Spec2Vec) by their ability to retrieve chemically related MS/MS spectrum pairs, using RDKit-derived Tanimoto similarity > 0.6 as ground truth over a swept score threshold.

## When to use

Use when comparing spectral similarity measures on annotated MS/MS data (e.g. a 3601-spectrum test set of 500 unique InChIKey compounds from GNPS) to quantify which measure best retrieves high-Tanimoto (Tanimoto > 0.6) compound pairs via precision/recall curves.

## When NOT to use

- The spectra lack InChIKey/SMILES annotations, so no RDKit fingerprint ground-truth Tanimoto similarities can be computed for labelling positive pairs
- The goal is compound identification (near-complete chemical match) rather than retrieval of chemically related pairs — the paper notes it is difficult to discriminate Tanimoto 0.8–0.9 from a near-complete chemical match
- Applying MS2DeepScore scores trained on GNPS library biases to experimental data from compound classes poorly represented in spectral libraries

## Inputs

- Annotated GNPS-derived MS/MS test set (e.g. 3601 spectra, 500 unique InChIKey compounds) with SMILES/InChI structural annotations
- Pretrained MS2DeepScore model (from ms2deepscore/matchms Python library)
- RDKit molecular fingerprints computed from the structural annotations
- Spec2Vec embeddings/model and matchms modified Cosine score implementations

## Outputs

- precision_recall_curves.csv with threshold, precision, and recall columns for MS2DeepScore, modified Cosine (two variants), and Spec2Vec
- Optionally a matrix of pairwise spectral similarity scores per measure

## How to apply

Load the held-out test-set spectra and their InChIKey/SMILES annotations from the GNPS-derived dataset, then compute ground-truth structural Tanimoto similarities for all spectrum pairs from Morgan circular fingerprints in RDKit (Morgan-2/Morgan-3, equivalents of ECFP-4/ECFP-6). Compute pairwise spectral similarity scores for the same spectra with each measure: the pretrained MS2DeepScore Siamese model (matchms-integrated), two variants of modified Cosine in matchms, and Spec2Vec. Sweep the pair-selection decision threshold from 0 to 1; at each threshold, compute precision and recall against pairs labelled positive when Tanimoto > 0.6. Write threshold/precision/recall columns for all measures to precision_recall_curves.csv. Correct application means the MS2DeepScore precision-recall curve dominates those of both modified Cosine variants and Spec2Vec across the sweep.

## Related tools

- **MS2DeepScore** (Siamese neural network predicting structural similarity (Tanimoto) from MS/MS spectrum pairs; provides the learned similarity scores being benchmarked) — https://github.com/matchms/ms2deepscore
- **Spec2Vec** (Unsupervised spectral similarity baseline compared against MS2DeepScore and modified Cosine in the precision/recall sweep)
- **matchms** (Python library for spectrum import, handling, filtering, and computing the modified Cosine score comparisons)
- **RDKit** (Computes ground-truth molecular fingerprints (Morgan-2/Morgan-3) and Tanimoto similarities for labelling chemically related pairs)

## Examples

```
pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS, score_computations=[[MS2DeepScore, {"model": model}]]))
```

## Evaluation signals

- The MS2DeepScore precision-recall curve should dominate (sit above) the curves of both modified Cosine variants and Spec2Vec across the 0–1 threshold sweep, consistent with the reported finding
- precision_recall_curves.csv must contain threshold, precision, and recall columns for all three (or five, counting both modified Cosine variants) measures
- Ground-truth Tanimoto positives are defined consistently as Tanimoto > 0.6 computed via RDKit fingerprints from the same SMILES/InChI annotations
- The test set composition should match the paper: 3601 spectra of 500 unique compounds held out from training, with reported MS2DeepScore RMSE between 0.13 and 0.2 on this set
- Pairwise spectral similarity scores must lie in [0, 1] for every measure before threshold sweeping

## Limitations

- MS2DeepScore scores are not deterministic and may change when a new model is trained (different parameters or training data), so exact reproduced values may vary
- It is difficult to discriminate high Tanimoto scores (0.8–0.9) from a near-complete chemical match; exact compound identification requires additional algorithms
- Current spectral libraries (and hence the GNPS-derived training/test data) are inherently biased toward certain compound classes, of mixed instrumentation types and varying quality

## Evidence

- [results] MS2DeepScore clearly outperforms both classical measures (two forms of the modified Cosine) as well as the unsupervised spectral similarity measure Spec2Vec, with respect to identifying high Tanimoto: "MS2DeepScore clearly outperforms both classical measures (two forms of the modified Cosine) as well as the unsupervised spectral similarity measure Spec2Vec, with respect to identifying high Tanimoto"
- [results] The test set, for instance, contains 3601 spectra of 500 unique compounds: "The test set, for instance, contains 3601 spectra of 500 unique compounds"
- [results] circular fingerprints (we used Morgan-2 and Morgan-3, the RDKit pendants of ECFP-4 and ECFP-6): [evidence span withheld — anchor status: not_found]
- [readme] ms2deepscore provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) from pairs of mass spectrometry spectra.: "`ms2deepscore` provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) 
from pairs of mass spectrometry spectra."
