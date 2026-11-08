---
name: split-integrity-verification
description: Use when a machine-learning model on spectral pairs (e.g. MS2DeepScore
  Tanimoto prediction) is trained and evaluated on spectra derived from shared molecules,
  and you must confirm the split is molecule-disjoint before trusting test-set metrics.
when_to_use_negative:
- Input spectra lack InChIKey (or other molecule-level) annotations, so molecule identity
  cannot be collapsed and disjointness cannot be checked
- The downstream task is spectrum-level (not molecule-level) prediction, where molecule-disjoint
  splitting is not required
- The split is already certified molecule-disjoint by an upstream tool and only needs
  re-use, not re-verification
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: ms2deepscore
  role: Data-handling utilities used to load annotated spectra and extract/collapse
    14-character InChIKey prefixes for split assignment
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Underlying spectral import and handling library on which the ms2deepscore
    split utilities are built
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:faf2df162b6454a51f6bfa14fff24eb1b579117f873ad414def752b33941af7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - matchms
  - ms2deepscore
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# split-integrity-verification

## Summary

Verifies that a train/validation/test split of mass spectral data is molecule-disjoint: all spectra of a given InChIKey must map to exactly one split, and per-set unique-InChIKey counts (14,062/500/500 in the MS2DeepScore GNPS dataset) must match the declared design. This prevents structural-similarity leakage between model training and held-out evaluation.

## When to use

Use when a machine-learning model on spectral pairs (e.g. MS2DeepScore Tanimoto prediction) is trained and evaluated on spectra derived from shared molecules, and you must confirm the split is molecule-disjoint before trusting test-set metrics.

## When NOT to use

- Input spectra lack InChIKey (or other molecule-level) annotations, so molecule identity cannot be collapsed and disjointness cannot be checked
- The downstream task is spectrum-level (not molecule-level) prediction, where molecule-disjoint splitting is not required
- The split is already certified molecule-disjoint by an upstream tool and only needs re-use, not re-verification

## Inputs

- Cleaned, annotated GNPS MS/MS dataset (210,407 spectra; 184,698 annotated with InChIKey and SMILES/InChI) from https://zenodo.org/record/4699300
- Split assignments (e.g. inchikey_split.csv with InChIKey and split label columns)

## Outputs

- inchikey_split.csv mapping each of the 15,062 unique InChIKeys to a split label (train/validation/test)
- Verification report: pairwise-intersection emptiness, per-set counts (14,062/500/500), and spectrum counts per split (102,536 training spectra)

## How to apply

Load the annotated spectra (from the Zenodo record https://zenodo.org/record/4699300: 210,407 spectra, 184,698 annotated with InChIKey and SMILES/InChI) with the matchms-based ms2deepscore data-handling utilities, extract the 14-character InChIKey prefix for every annotated spectrum, and collapse duplicates so each unique molecule appears once — yielding 15,062 unique InChIKeys. Assign all spectra of a given molecule to exactly one of three disjoint sets (14,062 train / 500 validation / 500 test unique InChIKeys). Then verify: (a) pairwise set intersections of InChIKeys across splits are empty, (b) per-set unique InChIKey counts equal 14,062/500/500, and (c) the union reproduces all 15,062 molecules. Write assignments to inchikey_split.csv with columns for InChIKey and split label so the check is auditable. The rationale is that because MS2DeepScore is trained to predict Tanimoto structural similarity between molecules, spectra of the same compound leaking across splits would inflate test-set RMSE; the paper's test set therefore contains 3601 spectra of 500 unseen compounds.

## Related tools

- **ms2deepscore** (Data-handling utilities used to load annotated spectra and extract/collapse 14-character InChIKey prefixes for split assignment) — https://github.com/matchms/ms2deepscore
- **matchms** (Underlying spectral import and handling library on which the ms2deepscore split utilities are built) — https://github.com/matchms/ms2deepscore

## Examples

```
train, val, test = set(train_keys), set(val_keys), set(test_keys); assert not train & val and not train & test and not val & test; assert (len(train), len(val), len(test)) == (14062, 500, 500)
```

## Evaluation signals

- Set union of the three splits equals the full set of 15,062 unique InChIKeys with no duplicates
- Pairwise set intersections (train∩val, train∩test, val∩test) are empty
- Per-set unique InChIKey counts are exactly 14,062 (train), 500 (validation), 500 (test)
- Each annotated spectrum appears in exactly one split; training set contains 102,536 spectra; test set contains 3601 spectra of 500 unique compounds
- inchikey_split.csv schema contains the expected columns for InChIKey and split label with one row per unique InChIKey

## Limitations

- Results are not deterministic across retrained models — scores change with different parameters or training data, so split verification is a precondition for comparability, not a guarantee of reproducible predictions
- Spectral library bias toward certain compound classes means a balanced molecule-disjoint split does not guarantee balanced compound-class representation across splits
- Splitting by full InChIKey (not just the 14-character skeleton prefix) may be required if stereoisomer-level leakage is a concern; the workflow described here collapses on the 14-character prefix

## Evidence

- [discussion] Partition of 15,062 unique InChIKeys into molecule-disjoint splits: "The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500)."
- [other] Molecule-disjoint split construction on annotated GNPS spectra: [evidence span withheld — anchor status: not_found]
- [results] Test-set composition confirming molecule-disjointness: "The test set, for instance, contains 3601 spectra of 500 unique compounds"
- [discussion] Source of the annotated dataset to be split: "The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo: https://zenodo.org/record/4699300."
- [results] Tanimoto-score prediction task motivating the split: "MS2DeepScore generally performs very well and can predict Tanimoto scores between 0.1 and 0.9 with a RMSE between 0.13 and 0.2"
