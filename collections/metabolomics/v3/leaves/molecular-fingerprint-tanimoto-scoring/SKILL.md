---
name: molecular-fingerprint-tanimoto-scoring
description: Use when annotated spectra (SMILES/InChI per spectrum) must be converted
  into pairwise structural-similarity ground truth, e.g. labelling compound pairs
  as related (Tanimoto > 0.6) to evaluate a spectral similarity measure by precision/recall
  over a threshold sweep.
when_to_use_negative:
- Structures lack SMILES/InChI annotations — Tanimoto scoring requires computable
  molecular structures, whereas MS2DeepScore itself works on spectra alone
- 'You need to discriminate highly similar analogues from near-complete chemical matches:
  the paper notes difficulty separating Tanimoto 0.8–0.9 from a near-complete match,
  so binary high-Tanimoto labels blur this regime'
- Different spectral scores must be compared deterministically — predicted scores
  are not deterministic and may change when a new model is trained
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: RDKit
  role: Generates Morgan-2/Morgan-3 (ECFP-4/ECFP-6 pendant) and Daylight fingerprints
    from structures and computes pairwise Tanimoto coefficients for ground-truth labels
- name: matchms
  role: Imports/normalizes the spectra and computes the comparison scores (modified
    Cosine) that are evaluated against the Tanimoto-derived labels
- name: ms2deepscore
  role: Supplies the learned similarity scores whose retrieval precision/recall is
    benchmarked against the fingerprint Tanimoto ground truth
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:d272d9a7ee59bf07306728f09717a82497c6c74cc0016fe903ed96417177729e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - RDKit
  - Spec2Vec
  - matchms
  - ms2deepscore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# molecular-fingerprint-tanimoto-scoring

## Summary

Compute ground-truth structural similarity between MS/MS-annotated compounds by hashing SMILES/InChI structures into RDKit molecular fingerprints (Morgan/ECFP or Daylight topological) and scoring all pairs with the Tanimoto coefficient; thresholding at Tanimoto > 0.6 defines 'chemically related' positives for benchmarking spectral similarity measures such as MS2DeepScore, modified Cosine, and Spec2Vec.

## When to use

Use when annotated spectra (SMILES/InChI per spectrum) must be converted into pairwise structural-similarity ground truth, e.g. labelling compound pairs as related (Tanimoto > 0.6) to evaluate a spectral similarity measure by precision/recall over a threshold sweep.

## When NOT to use

- Structures lack SMILES/InChI annotations — Tanimoto scoring requires computable molecular structures, whereas MS2DeepScore itself works on spectra alone
- You need to discriminate highly similar analogues from near-complete chemical matches: the paper notes difficulty separating Tanimoto 0.8–0.9 from a near-complete match, so binary high-Tanimoto labels blur this regime
- Different spectral scores must be compared deterministically — predicted scores are not deterministic and may change when a new model is trained

## Inputs

- Per-spectrum structural annotations (SMILES and/or InChI, keyed by InChIKey) from an annotated MS/MS library, e.g. the GNPS-derived 210,407-spectrum dataset on Zenodo record 4699300
- Cleaned MS/MS spectra (MGF/MSP/mzML readable by matchms) to pair with the annotations
- Optional: candidate spectral similarity score matrices (MS2DeepScore, modified Cosine, Spec2Vec) to benchmark against the Tanimoto labels

## Outputs

- Symmetric pairwise Tanimoto similarity matrix over all spectrum (or compound) pairs, values in [0,1]
- Binary ground-truth labels: pair = positive if Tanimoto > 0.6
- precision_recall_curves.csv with threshold, precision, and recall columns per similarity measure (MS2DeepScore vs modified Cosine variants vs Spec2Vec)

## How to apply

Parse the SMILES/InChI annotations attached to each spectrum into RDKit Mol objects and generate fingerprints — the paper used circular fingerprints (Morgan-2 and Morgan-3, the RDKit pendants of ECFP-4 and ECFP-6) as well as topological Daylight fingerprints. Compute the Tanimoto coefficient for every spectrum pair (e.g. the 3601-spectrum test set spanning 500 unique held-out InChIKeys from the GNPS-derived Zenodo dataset, record 4699300). Label a pair positive when Tanimoto > 0.6. Then, to benchmark a spectral similarity score, sweep its decision threshold from 0 to 1 and, at each threshold, compute precision and recall of selected pairs against these ground-truth labels, writing threshold/precision/recall columns per measure to precision_recall_curves.csv. When several spectra exist per compound, MS2DeepScore predictions can be aggregated by taking the median score per compound pair before thresholding.

## Related tools

- **RDKit** (Generates Morgan-2/Morgan-3 (ECFP-4/ECFP-6 pendant) and Daylight fingerprints from structures and computes pairwise Tanimoto coefficients for ground-truth labels)
- **matchms** (Imports/normalizes the spectra and computes the comparison scores (modified Cosine) that are evaluated against the Tanimoto-derived labels)
- **ms2deepscore** (Supplies the learned similarity scores whose retrieval precision/recall is benchmarked against the fingerprint Tanimoto ground truth) — https://github.com/matchms/ms2deepscore

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; fp = lambda s: AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(s), 3); tanimoto = DataStructs.TanimotoSimilarity(fp(smiles1), fp(smiles2))
```

## Evaluation signals

- All computed Tanimoto values lie in [0,1]; each compound vs itself yields exactly 1.0, and the pair matrix is symmetric
- Positive-label fraction is stable and reproducible when re-run with the same fingerprint type and Tanimoto > 0.6 cutoff on the 3601-spectrum test set
- The precision/recall CSV has one (threshold, precision, recall) trajectory per similarity measure with thresholds monotonically sweeping 0 → 1
- Sanity benchmark: the MS2DeepScore precision-recall curve dominates the modified Cosine variants and Spec2Vec across the threshold sweep, matching the reported Figure 4/5 finding
- Reported prediction quality context: MS2DeepScore Tanimoto predictions on the test set have RMSE between 0.13 and 0.2 for scores between 0.1 and 0.9

## Limitations

- No consensus exists on how best to quantify chemical relatedness — results depend on fingerprint choice (Morgan-2/3 vs Daylight) and scoring metric, so the 0.6 cutoff is convention, not ground truth
- Some fingerprint features may not be represented in any fragments, limiting how far spectral scores can recover them
- GNPS-derived spectral libraries are biased toward certain compound classes and of varying instrumentation quality, which affects both labels and downstream retrieval metrics

## Evidence

- [results] Circular fingerprints capture substructure diversity: "circular fingerprints (we used Morgan-2 and Morgan-3, the RDKit [23] pendants of ECFP-4 and ECFP-6) as well as topological fingerprints (we used Daylight fingerprints in RDKit)"
- [results] Benchmark outcome against spectral measures: "MS2DeepScore clearly outperforms both classical measures (two forms of the modified Cosine) as well as the unsupervised spectral similarity measure Spec2Vec"
- [intro] Test-set composition for held-out evaluation: "The test set, for instance, contains 3601 spectra of 500 unique compounds"
- [results] Score aggregation across replicate spectra: "structural similarity score predictions can be improved by taking the median of the scores calculated for different pairs of spectra corresponding to the same compound pairs"
- [readme] Tooling integration: "`ms2deepscore` provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) from pairs of mass spectrometry spectra."
