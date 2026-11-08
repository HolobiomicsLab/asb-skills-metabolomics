---
name: per-compound-spectrum-merging
description: Use when training spectra-presence predictors from multiple spectra per
  compound and each compound needs one canonical binary target spectrum rather than
  intensity-level prediction across varying collision energies.
when_to_use_negative:
- The goal is intensity-level spectral prediction or matching raw spectra across a
  single collision energy — the binary presence/absence merge discards intensity information
  by design
- Spectra are already one-per-compound (e.g. curated single-spectrum libraries); merging
  is a no-op and the 0.01-binning step alone is needed
- The spectra to merge are not positive-mode [M+H] experimental spectra; the filter
  in this skill is specific to the [M+H] adduct used in SingleFrag
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SingleFrag
  role: Provides the raw MS/MS dataset (Figshare) whose spectra are filtered, merged,
    and binarized into per-compound targets; its downstream ANNmodel.py/GNNmodel.py/COMmodel.py
    consume the merged binary spectra
  repo: https://github.com/MaribelPR/SingleFrag
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:300a7eee23eeebf3489765eb69d736232b4e04b19beb6f7e463f349cc277bbef
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# per-compound-spectrum-merging

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Merges all experimental [M+H] MS/MS spectra belonging to the same compound into a single binary peak-presence spectrum, discretized into 0.01 m/z bins, to build the per-compound training targets used by SingleFrag's per-bin classifiers.

## When to use

Use when training spectra-presence predictors from multiple spectra per compound and each compound needs one canonical binary target spectrum rather than intensity-level prediction across varying collision energies.

## When NOT to use

- The goal is intensity-level spectral prediction or matching raw spectra across a single collision energy — the binary presence/absence merge discards intensity information by design
- Spectra are already one-per-compound (e.g. curated single-spectrum libraries); merging is a no-op and the 0.01-binning step alone is needed
- The spectra to merge are not positive-mode [M+H] experimental spectra; the filter in this skill is specific to the [M+H] adduct used in SingleFrag

## Inputs

- SingleFrag dataset from Figshare (https://figshare.com/articles/dataset/SingleFrag/27266787) containing 2,291,119 raw MS/MS spectra
- Per-spectrum adduct/precursor annotations used to identify experimental [M+H] spectra
- Compound identifiers (InChIKeys) for SMILES standardization and duplicate resolution

## Outputs

- Binarized per-compound MS/MS spectra table on a 0.01 m/z grid (30,591 compounds)
- 80/10/10 train/validation/test split files over the 30,591 compounds
- Binary per-compound target vectors consumed by ANNmodel.py, GNNmodel.py, and COMmodel.py per MZ position

## How to apply

Start from the SingleFrag dataset downloaded from Figshare (2,291,119 raw spectra) and filter to the 434,480 experimental [M+H] spectra using the adduct annotation in the dataset. Resolve canonical SMILES via each compound's InChIKey to deduplicate and standardize structures. For each compound, merge all its filtered spectra into one binary spectrum where an m/z position is set to 1 if a peak is present in at least one of the compound's available spectra and 0 otherwise — presence-anywhere aggregation is the deliberate choice because intensity varies across collision energies. Discretize the merged spectrum into 0.01 m/z bins, then split the resulting 30,591 compounds into 80/10/10 train/validation/test sets before training per-MZ-position models (ANNmodel.py, GNNmodel.py, COMmodel.py).

## Related tools

- **SingleFrag** (Provides the raw MS/MS dataset (Figshare) whose spectra are filtered, merged, and binarized into per-compound targets; its downstream ANNmodel.py/GNNmodel.py/COMmodel.py consume the merged binary spectra) — https://github.com/MaribelPR/SingleFrag

## Examples

```
python ANNmodel.py --mz-position 24500 --train train.bin.pkl --valid val.bin.pkl --test test.bin.pkl
```

## Evaluation signals

- The merged dataset must contain exactly 30,591 unique compounds after deduplication via canonical SMILES
- The filtered input to the merge step must be exactly 434,480 experimental [M+H] spectra out of the 2,291,119 raw spectra
- Each merged spectrum must be binary-valued (0/1) on a 0.01 m/z grid — no intensity values should survive
- Train/validation/test splits must partition the 30,591 compounds at roughly 80/10/10 with no compound appearing in multiple splits
- Every per-compound target must match the 'presence in at least one spectrum' rule: a bin is 1 only if some constituent spectrum had a peak there

## Limitations

- Public spectral libraries such as GNPS, MSDial, and MassBank often lack sufficient curation and contain spectra acquired at only one or a few collision energies per compound, which limits the diversity of merged spectra
- The merge intentionally collapses collision-energy-dependent information; peaks present only under specific energies are indistinguishable from always-present peaks
- Compounds must have at least one experimental [M+H] spectrum in the source dataset to receive a merged target

## Evidence

- [other] Binary merge rule (presence in at least one spectrum): [evidence span withheld — anchor status: not_found]
