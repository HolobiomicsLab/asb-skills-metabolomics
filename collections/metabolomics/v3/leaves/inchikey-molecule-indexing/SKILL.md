---
name: inchikey-molecule-indexing
description: Use when annotated MS/MS spectra (InChIKey/SMILES metadata) must be grouped
  at the compound level, e.g. to prevent spectra of the same molecule from leaking
  across model train/validation/test splits.
when_to_use_negative:
- Spectra lack InChIKey (or equivalent structure) annotations, so molecules cannot
  be identified and collapsed
- A spectrum-level (rather than molecule-level) split is intended, e.g. when duplicate
  spectra of the same compound intentionally appear in multiple sets
- 'Applying to a different dataset: the 15,062/14,062/500/500 counts are dataset-specific
  and must be re-verified rather than assumed'
edam_topics:
- http://edamontology.org/topic_3370
tools:
- name: ms2deepscore
  role: Provides matchms-based data-handling utilities used to load annotated spectra
    and extract/deduplicate InChIKey prefixes for the molecule-disjoint split
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Python library for mass spectra import and metadata handling underlying the
    indexing and split construction
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:37f07ae31497abf1bfe4d9f8c6e518ed28b8af1af9553599f55d57dcafd4be16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - matchms
  - ms2deepscore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3370
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# inchikey-molecule-indexing

## Summary

Index annotated GNPS MS/MS spectra by the 14-character InChIKey prefix of their compound, collapsing duplicate spectra so each unique molecule appears once, in order to build molecule-disjoint training/validation/test splits (14,062/500/500 InChIKeys) for MS2DeepScore training.

## When to use

Use when annotated MS/MS spectra (InChIKey/SMILES metadata) must be grouped at the compound level, e.g. to prevent spectra of the same molecule from leaking across model train/validation/test splits.

## When NOT to use

- Spectra lack InChIKey (or equivalent structure) annotations, so molecules cannot be identified and collapsed
- A spectrum-level (rather than molecule-level) split is intended, e.g. when duplicate spectra of the same compound intentionally appear in multiple sets
- Applying to a different dataset: the 15,062/14,062/500/500 counts are dataset-specific and must be re-verified rather than assumed

## Inputs

- Cleaned, annotated GNPS MS/MS spectral dataset (210,407 spectra; 184,698 annotated with InChIKey and SMILES and/or InChI) from Zenodo record 4699300
- Per-spectrum metadata fields: InChIKey (for 14-character prefix extraction), SMILES/InChI

## Outputs

- inchikey_split.csv with columns InChIKey and split label (train/validation/test)
- Molecule-disjoint split of 14,062 train / 500 validation / 500 test unique InChIKeys (15,062 total), with per-molecule spectra assigned to exactly one set

## How to apply

Download the cleaned, annotated GNPS dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES/InChI) from Zenodo record 4699300, and load it with the matchms-based ms2deepscore data-handling utilities. Extract the 14-character InChIKey prefix for every annotated spectrum and collapse duplicates so each unique molecule is represented exactly once, verifying the count reaches 15,062 unique InChIKeys. Partition the unique InChIKeys into three disjoint sets of 14,062 (training), 500 (validation) and 500 (test), assigning all spectra of a given molecule to exactly one set so the split is molecule-disjoint. Write the assignments to inchikey_split.csv with columns for InChIKey and split label; the training set then contains pairs drawn from 102,536 spectra of 14,062 unique molecules.

## Related tools

- **ms2deepscore** (Provides matchms-based data-handling utilities used to load annotated spectra and extract/deduplicate InChIKey prefixes for the molecule-disjoint split) — https://github.com/matchms/ms2deepscore
- **matchms** (Python library for mass spectra import and metadata handling underlying the indexing and split construction) — https://github.com/matchms/matchms

## Examples

```
python -c "import pandas as pd; counts = pd.read_csv('inchikey_split.csv').groupby('split')['InChIKey'].nunique(); assert sorted(counts.tolist()) == [500, 500, 14062]"
```

## Evaluation signals

- Set of unique 14-character InChIKey prefixes totals exactly 15,062 after collapsing duplicate spectra
- The three split sets are pairwise disjoint, and per-set InChIKey counts equal 14,062 (train), 500 (validation), 500 (test)
- Every spectrum of a given InChIKey appears in exactly one split (no molecule leakage across train/val/test)
- Test set contains approximately 3601 spectra of 500 unique compounds, matching the article's reported composition
- inchikey_split.csv parses cleanly and groupby('split')['InChIKey'].nunique() reproduces the expected counts

## Limitations

- Split sizes (15,062 unique InChIKeys; 14,062/500/500) are specific to this GNPS snapshot; re-derive counts when indexing other datasets
- Spectral libraries are biased toward certain compound classes, so the molecule index inherits that bias and it should be considered when applying trained models to experimental data
- The dataset mixes spectra from different instrumentation types of varying quality, which affects downstream model use of the indexed splits

## Evidence

- [discussion] The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500).: "The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500)."
- [discussion] The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo.: "The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo: https://zenodo.org/record/4699300."
- [results] The test set, for instance, contains 3601 spectra of 500 unique compounds: "The test set, for instance, contains 3601 spectra of 500 unique compounds"
