---
name: compound-dataset-splitting
description: Use when training per-m/z peak-presence models (ANN, GNN or COM) from
  merged binarized MS/MS spectra and a compound-level split must be defined before
  ANNmodel.py / GNNmodel.py / COMmodel.py are invoked.
when_to_use_negative:
- The task is to merge spectra per compound or to binarize raw spectra — those are
  upstream preprocessing steps, not the split itself
- Prediction is requested for a compound already present in the training split (e.g.
  the supplied inputMols_git.xlsx toy set) — no split decision is needed for held-in
  compounds
- 'A different adduct (e.g. [M-H]−, [M+Na]+) is targeted: the 80/10/10 split described
  here applies to the experimental [M+H]+ dataset used to train SingleFrag classifiers'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: SingleFrag
  role: Source of the merged binarized dataset and the training scripts (ANNmodel.py,
    GNNmodel.py, COMmodel.py) that consume the split files
  repo: https://github.com/MaribelPR/SingleFrag
- name: 3DMolMS
  role: Baseline competitor whose reported performance on the test compounds SingleFrag
    is compared against; the split must keep test compounds held out so this comparison
    is valid
  repo: josiehong/3DMolMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:f7d8ab28f80839532c80428fd0794423ca4719a3b301f6447a4494fe30d93e78
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - SingleFrag
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# compound-dataset-splitting

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Splits the 30,591 unique compounds in the SingleFrag dataset into train, validation and test sets at the compound level (80/10/10) so that no compound contributes spectra to more than one split, preventing structural leakage into spectral-presence classifiers.

## When to use

Use when training per-m/z peak-presence models (ANN, GNN or COM) from merged binarized MS/MS spectra and a compound-level split must be defined before ANNmodel.py / GNNmodel.py / COMmodel.py are invoked.

## When NOT to use

- The task is to merge spectra per compound or to binarize raw spectra — those are upstream preprocessing steps, not the split itself
- Prediction is requested for a compound already present in the training split (e.g. the supplied inputMols_git.xlsx toy set) — no split decision is needed for held-in compounds
- A different adduct (e.g. [M-H]−, [M+Na]+) is targeted: the 80/10/10 split described here applies to the experimental [M+H]+ dataset used to train SingleFrag classifiers

## Inputs

- Merged binarized per-compound spectra table (30,591 compounds; 0.01 m/z bins, binary peak-presence values) derived from 434,480 experimental [M+H]+ MS/MS spectra
- Canonical SMILES and InChIKey per compound
- Split ratio specification (80% train / 10% validation / 10% test)

## Outputs

- Compound-level train/val/test membership table
- Split-indexed binarized spectra tables for ANNmodel.py, GNNmodel.py, COMmodel.py training
- Validation/test performance metrics on unseen compounds (e.g. accuracy per m/z position, comparison vs. CFM-ID and 3DMolMS baselines)

## How to apply

After merging all filtered experimental [M+H]+ spectra of each compound into a single binarized spectrum (m/z bins of 0.01 Da set to 1 when a peak appears in at least one of the compound's spectra), enumerate the resulting 30,591 unique compounds and split them into 80% train, 10% validation and 10% test at the compound level, so every spectrum of one compound stays in one split. Save the split membership table alongside the merged binarized spectra so training scripts (ANNmodel.py, GNNmodel.py, COMmodel.py) can load disjoint sets, and report validation metrics on compounds never seen during training. The rationale is that SingleFrag predicts peak presence per m/z bin from molecular structure (mol2vec embedding or graph); if the same compound appeared in both train and test, predictions would trivially memorize the binarized target rather than learn fragmentation patterns, and the reported performance over CFM-ID and 3DMolMS would not be generalizable.

## Related tools

- **SingleFrag** (Source of the merged binarized dataset and the training scripts (ANNmodel.py, GNNmodel.py, COMmodel.py) that consume the split files) — https://github.com/MaribelPR/SingleFrag
- **3DMolMS** (Baseline competitor whose reported performance on the test compounds SingleFrag is compared against; the split must keep test compounds held out so this comparison is valid) — josiehong/3DMolMS

## Examples

```
python ANNmodel.py --train data/split/train.pickle --valid data/split/valid.pickle --test data/split/test.pickle --mz_position 581
```

## Evaluation signals

- Verify the union of train, validation and test compounds equals 30,591 unique compounds with no overlap across splits
- Confirm split sizes are approximately 24,473 / 3,059 / 3,059 compounds (80/10/10)
- Check that each merged binarized spectrum (one row per compound, 0.01 m/z bins) maps to exactly one split
- Confirm that validation performance used to tune ANN/GNN/COM thresholds comes only from validation-split compounds, and test metrics come only from test-split compounds

## Limitations

- The 80/10/10 split is specific to the [M+H]+ experimental dataset; adduct-specific or collision-energy stratification is not described
- Splitting is at the compound level, so structurally similar compounds (e.g. isomers sharing InChIKey parents) may still be co-located in different splits unless InChIKey-based de-duplication has removed them
- Models are deliberately simple (mol2vec embedding or molecular graph) and their architectures are not optimized, so a compound-level split alone does not guarantee structural diversity within each split

## Evidence

- [other] SingleFrag models are trained on a compound-level 80/10/10 split of 30,591 merged binarized spectra: [evidence span withheld — anchor status: not_found]
