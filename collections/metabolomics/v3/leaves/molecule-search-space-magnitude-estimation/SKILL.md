---
name: molecule-search-space-magnitude-estimation
description: 'Use when explaining size-dependent accuracy trends in structure prediction:
  accuracy is tabulated by heavy-atom bin and needs to be juxtaposed against the combinatorial
  molecule-count growth at that size.'
when_to_use_negative:
- Evaluating substructure prediction quality (Tanimoto similarity, substructure vectors)
  rather than size-dependent exact-match accuracy against combinatorial counts
- Single-molecule structure elucidation runs (nmr_infer_single_spectrum) where no
  per-size accuracy statistics or search-space framing is needed
- Datasets containing heteroatoms beyond C, N, and O or molecules larger than 19 heavy
  atoms — outside the paper's demonstrated scope
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Canonicalizes SMILES and provides substructure matching / molecule parsing
    used to compute heavy-atom counts for binning
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: NMR2Struct
  role: Source of the multitask model's processed test-set predictions (processed_predictions.h5)
    used to build the accuracy-by-size table
  repo: https://github.com/MarklandGroup/NMR2Struct
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:95a6d19dd5230e80ccf3e8b9fe9f8434e69aaa6fa7f14c9eab73dc56cff369e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - RDKit
  - Zenodo record 13892026
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# molecule-search-space-magnitude-estimation

## Summary

Estimates how the combinatorial search space of possible molecular structures grows with heavy-atom count (e.g., ∼2 million molecules at 10 heavy atoms vs. trillions at 19 heavy atoms) to contextualize model accuracy declines such as the 25.5-percentage-point drop observed for the multitask NMR-to-structure model.

## When to use

Use when explaining size-dependent accuracy trends in structure prediction: accuracy is tabulated by heavy-atom bin and needs to be juxtaposed against the combinatorial molecule-count growth at that size.

## When NOT to use

- Evaluating substructure prediction quality (Tanimoto similarity, substructure vectors) rather than size-dependent exact-match accuracy against combinatorial counts
- Single-molecule structure elucidation runs (nmr_infer_single_spectrum) where no per-size accuracy statistics or search-space framing is needed
- Datasets containing heteroatoms beyond C, N, and O or molecules larger than 19 heavy atoms — outside the paper's demonstrated scope

## Inputs

- processed_predictions.h5 from a multitask-model test-set inference run (valid_predictions and bad_predictions groups, num_heavy_atoms and prediction_strings fields)
- Canonical SMILES targets for the test set (numpy .npy binary-encoded array, decode('utf-8') per README)
- Reference search-space counts for C/N/O-only molecule space (GDB-17-derived; e.g., ∼2×10^6 at 10 heavy atoms, trillions at 19)

## Outputs

- results.tsv with columns: heavy-atom bin, accuracy, molecule-count magnitude, accuracy decrease, plus a note on the ~1% training-molecule share at 18–19 heavy atoms
- Accuracy-versus-size trend (77.5% at 10 heavy atoms → 52.0% at 19 heavy atoms) framed against search-space growth

## How to apply

Group molecules (e.g., from the SpectraBase + GDB-17 test set of C/N/O-only compounds with up to 19 heavy atoms) into heavy-atom bins — here 10, 17, and 18–19 heavy atoms — using num_heavy_atoms per molecule. For each bin, quantify the search-space magnitude: ∼2 million possible molecules at 10 heavy atoms, trillions at 19, with roughly 5 orders of magnitude of growth between 17 and 18–19 heavy atoms. Pair each count with the model's binned accuracy computed as the fraction of targets whose canonical SMILES appears among the 15 sampled predictions, so the ~5-order-of-magnitude size increase can be set against the ~25.5-percentage-point accuracy drop. Note data sparsity as a confounder: only ~1% of training molecules have 18–19 heavy atoms, so search-space growth alone does not fully explain the decline. Write binned results (heavy-atom bin, accuracy, count magnitude, decrease) to a tab-separated results.tsv.

## Related tools

- **RDKit** (Canonicalizes SMILES and provides substructure matching / molecule parsing used to compute heavy-atom counts for binning) — https://github.com/MarklandGroup/NMR2Struct
- **NMR2Struct** (Source of the multitask model's processed test-set predictions (processed_predictions.h5) used to build the accuracy-by-size table) — https://github.com/MarklandGroup/NMR2Struct

## Examples

```
import numpy as np, h5py; from collections import Counter; smis=[x.decode('utf-8') for x in np.load('smiles.npy')]; print(Counter(sum(1 for c in s if c not in 'H') for s in smis))
```

## Evaluation signals

- The 10-heavy-atom and 17-heavy-atom substructure-to-structure reference accuracies reproduce at 96.0% and 92.8% respectively
- The 17 → 18–19 heavy-atom absolute accuracy decrease lands near 25.5 percentage points, with 18–19-atom accuracy below 80%
- The 10-heavy-atom bin's count magnitude matches the paper's ∼2 million figure
- Multitask-model accuracy is monotone decreasing with heavy-atom count (77.5% at 10 → 52.0% at 19)
- results.tsv is written in tab-separated form with one row per heavy-atom bin and a note that only ~1% of training molecules have 18–19 heavy atoms

## Limitations

- Counts are for C/N/O-only chemical space (GDB-17 sampling); introducing other elements or stereochemistry changes the combinatorics
- Search-space growth between 17 and 18–19 heavy atoms is confounded by training-set sparsity (~1% of training molecules at 18–19 heavy atoms)
- Simulated SpectraBase/MestreNova spectra were used for training, so size-dependent accuracy may not transfer to real experimental spectra

## Evidence

- [results] problem size ~2 million at 10 heavy atoms: "at 10 heavy atoms, where the problem size is ∼2 million, the accuracy is 96.0% and only drops to 92.8% by 17 heavy atoms"
- [abstract] trillions of possible structures at 19 heavy atoms: "We demonstrate the effectiveness of this framework on molecules with up to 19 heavy (non-hydrogen) atoms, a size for which there are trillions of possible structures."
- [abstract] 11 orders of magnitude search-space reduction: "our approach predicts the exact molecule 69.6% of the time within the first 15 predictions, reducing the search space by up to 11 orders of magnitude."
- [readme] combinatorial explosion as atoms increase: "remains an extremely challenging problem because of the combinatorial explosion of the number of possible molecules as the number of constituent atoms is increased"
