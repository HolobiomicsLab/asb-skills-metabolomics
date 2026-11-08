---
name: compound-fingerprint-generation
description: Use when building an ANN that maps binned MS/MS spectra to compound fingerprints,
  or when swapping a baseline fingerprint representation (e.g. MACCS/FP3/FP4) for
  an alternative such as RDKit topological fingerprints.
when_to_use_negative:
- Input is an already-trained fingerprint-prediction model and you only need candidate
  ranking from an InChIKey list — use the MetFID prediction/recranking workflow instead.
- You need a fingerprint type not covered by OpenBabel MACCS/FP3/FP4 or RDKit topological
  fingerprints (e.g. circular/atom-pair) — PyFingerprint offers a wider catalogue.
- Spectra are merged across collision energies or use the same first InChIKey block
  as training, breaking the structure-disjoint evaluation.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0154
tools:
- name: OpenBabel
  role: Computes MACCS, FP3, and FP4 fingerprints for training-set compounds to assemble
    the 528-bit binary vector
  repo: https://github.com/aohongyu/MetFID
- name: RDKit
  role: Generates topological fingerprints used as the alternative ANN target for
    the topology-vs-SMARTS comparison
- name: PyFingerprint
  role: Unified Python wrapper exposing CDK, RDKit, OpenBabel (fp2/fp3/fp4), and other
    fingerprint types via get_fingerprint/get_fingerprints
  repo: https://github.com/hcji/PyFingerprint
- name: MetFID
  role: Trains and applies the Keras/TensorFlow ANN that maps binned MS/MS spectra
    to fingerprint targets
  repo: https://github.com/aohongyu/MetFID
- name: InChIKey
  role: Compound identifier used to merge spectra and enforce structure-disjoint train/test
    splits
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:ffc7c906e2aa151444a90f8caa44183c2d371981081085ba8de7fd7323fc7a97
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - InChIKey
  - MetFID
  - MoNA
  - NIST 17
  - OpenBabel
  - PyFingerprint
  - RDKit
  - TensorFlow
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# compound-fingerprint-generation

## Summary

Generate 528-bit binary molecular fingerprints (OpenBabel MACCS + FP3 + FP4 concatenated) for the MoNA/NIST 17 training compounds and RDKit topological fingerprints for comparison, which serve as ANN targets in MetFID's fingerprint prediction pipeline.

## When to use

Use when building an ANN that maps binned MS/MS spectra to compound fingerprints, or when swapping a baseline fingerprint representation (e.g. MACCS/FP3/FP4) for an alternative such as RDKit topological fingerprints.

## When NOT to use

- Input is an already-trained fingerprint-prediction model and you only need candidate ranking from an InChIKey list — use the MetFID prediction/recranking workflow instead.
- You need a fingerprint type not covered by OpenBabel MACCS/FP3/FP4 or RDKit topological fingerprints (e.g. circular/atom-pair) — PyFingerprint offers a wider catalogue.
- Spectra are merged across collision energies or use the same first InChIKey block as training, breaking the structure-disjoint evaluation.

## Inputs

- Filtered, denoised, binned MS/MS spectra (11,784 MoNA + 122,481 NIST 17 spectra; 15,228 unique compounds) with InChIKey identifiers
- Structure-disjoint testing spectra (first InChIKey block excluded, single collision energy, no merging)

## Outputs

- 528-entry binary fingerprint vector per training compound (MACCS + FP3 + FP4)
- RDKit topological fingerprint vector per compound (alternative target)
- topology_fingerprint_metrics.csv with macro-averaged accuracy and F1 score

## How to apply

For every unique compound in the training set (15,228 unique compounds from 11,784 MoNA and 122,481 NIST 17 spectra), compute OpenBabel MACCS, FP3, and FP4 fingerprints and concatenate them into a single 528-entry binary vector that becomes the ANN's sigmoid-output target. To reproduce the topology-vs-SMARTS comparison in the paper, instead compute RDKit topological fingerprints for the same compounds and train the Keras/TensorFlow ANN (two hidden ReLU layers, binary cross-entropy, Adam, 30 epochs, batch size 100) on those targets. Evaluate on structure-disjoint testing spectra (same first InChIKey block excluded, single collision energy, no merging) using macro-averaged accuracy (Eq. 2) and F1 (Eq. 3); the topology-based model should show lower accuracy/F1 than the 528-bit MACCS/FP3/FP4 baseline, reproducing the reported performance decrease.

## Related tools

- **OpenBabel** (Computes MACCS, FP3, and FP4 fingerprints for training-set compounds to assemble the 528-bit binary vector) — https://github.com/aohongyu/MetFID
- **RDKit** (Generates topological fingerprints used as the alternative ANN target for the topology-vs-SMARTS comparison)
- **PyFingerprint** (Unified Python wrapper exposing CDK, RDKit, OpenBabel (fp2/fp3/fp4), and other fingerprint types via get_fingerprint/get_fingerprints) — https://github.com/hcji/PyFingerprint
- **MetFID** (Trains and applies the Keras/TensorFlow ANN that maps binned MS/MS spectra to fingerprint targets) — https://github.com/aohongyu/MetFID
- **InChIKey** (Compound identifier used to merge spectra and enforce structure-disjoint train/test splits)

## Examples

```
python -c "from PyFingerprint.fingerprint import get_fingerprints; fps=[get_fingerprints([smi],'fp3')[0].to_numpy() for smi in smiles]"
```

## Evaluation signals

- Fingerprint vector length equals 528 binary entries when MACCS+FP3+FP4 are concatenated
- ANN trains for 30 epochs with batch size 100, two hidden ReLU layers, sigmoid output, binary cross-entropy loss, Adam optimizer
- Topological-fingerprint model shows lower macro-averaged accuracy and F1 than the 528-bit MACCS/FP3/FP4 baseline on the same structure-disjoint test set
- Output metrics file topology_fingerprint_metrics.csv is produced containing accuracy (Eq. 2) and F1 (Eq. 3)
- No test compound shares its first InChIKey block with any training compound

## Limitations

- Merging multiple MS/MS spectra with different collision energies per compound relies on InChIKey as the compound identifier; InChIKey block collisions could cause leakage if not filtered
- Training-set fingerprint targets depend on OpenBabel's SMARTS-pattern definitions for FP3/FP4, which may change across OpenBabel versions (paper used 3.1.1 per README prerequisites)
- Performance of the topology-fingerprint alternative is expected to be worse; results must be compared on identical structure-disjoint splits to be interpretable
- The paper does not release pre-trained fingerprint-generation model weights; the 528-bit target generation is library-dependent rather than learned

## Evidence

- [readme] PyFingerprint catalogue of fingerprint types: [evidence span withheld — anchor status: external_doc]
