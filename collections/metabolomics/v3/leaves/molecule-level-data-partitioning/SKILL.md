---
name: molecule-level-data-partitioning
description: Use when training or benchmarking spectral similarity/spectrum embedding
  models on GNPS-derived spectra (e.g. 210,407 spectra, 184,698 annotated) and spectrum-level
  splits would leak identical molecules across train and test.
when_to_use_negative:
- When the goal is to evaluate within-library retrieval on already-seen compounds
  rather than generalization to unseen molecules — a spectrum-level split would inflate
  performance
- When spectra lack InChIKey annotations (only 184,698 of 210,407 are annotated);
  unannotated spectra cannot be assigned to a molecule group
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: ms2deepscore
  role: Provides data-handling utilities to load annotated spectra and extract/collect
    unique InChIKey prefixes for partitioning
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Python library used for mass spectra import, handling and cleaning prior to
    splitting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:039cec44d575123fc131b758334dafb173e8eb9840304aa746cac854dbb0172d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - matchms
  - ms2deepscore
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# molecule-level-data-partitioning

## Summary

Splits MS/MS spectral datasets into training/validation/test sets at the molecule level rather than the spectrum level, so that all spectra of a given InChIKey are assigned to exactly one split and model evaluation reflects generalization to unseen compounds.

## When to use

Use when training or benchmarking spectral similarity/spectrum embedding models on GNPS-derived spectra (e.g. 210,407 spectra, 184,698 annotated) and spectrum-level splits would leak identical molecules across train and test.

## When NOT to use

- When the goal is to evaluate within-library retrieval on already-seen compounds rather than generalization to unseen molecules — a spectrum-level split would inflate performance
- When spectra lack InChIKey annotations (only 184,698 of 210,407 are annotated); unannotated spectra cannot be assigned to a molecule group

## Inputs

- Cleaned, annotated GNPS MS/MS spectral dataset (210,407 spectra; 184,698 annotated with InChIKey and SMILES/InChI) from https://zenodo.org/record/4699300
- Per-spectrum InChIKey metadata

## Outputs

- inchikey_split.csv with columns for InChIKey and split label (train/validation/test)
- Three molecule-disjoint spectral subsets: 14,062 / 500 / 500 unique InChIKeys

## How to apply

Load the cleaned, annotated spectra (210,407 spectra, of which 184,698 carry InChIKey and SMILES/InChI, from Zenodo record 4699300) with the matchms/ms2deepscore data-handling utilities. Extract the 14-character InChIKey prefix for every annotated spectrum and collapse duplicates so each unique molecule is counted once, verifying a total of 15,062 unique InChIKeys. Partition the InChIKeys into three disjoint sets — 14,062 training, 500 validation, 500 test — assigning every spectrum of a molecule to the same set; this yields pairs drawn from e.g. 102,536 spectra of 14,062 training molecules. Write assignments to inchikey_split.csv with columns for InChIKey and split label, then confirm disjointness and per-set counts.

## Related tools

- **ms2deepscore** (Provides data-handling utilities to load annotated spectra and extract/collect unique InChIKey prefixes for partitioning) — https://github.com/matchms/ms2deepscore
- **matchms** (Python library used for mass spectra import, handling and cleaning prior to splitting)

## Examples

```
from ms2deepscore import data_handling  # load annotated spectra, extract InChIKey prefixes, assign each unique molecule to train/validation/test and save to inchikey_split.csv
```

## Evaluation signals

- The set of unique InChIKey prefixes totals exactly 15,062 molecules
- Split sizes are exactly 14,062 / 500 / 500 unique InChIKeys for training/validation/test
- The three InChIKey sets are pairwise disjoint (no molecule appears in more than one split)
- Every spectrum of a given molecule is assigned to exactly one split (e.g. 102,536 training spectra across 14,062 molecules)
- inchikey_split.csv contains one row per InChIKey with a valid split label

## Limitations

- Splitting by molecule reduces the effective number of spectra per split compared to spectrum-level splits, so pair sampling quality must be checked for small training sets
- Spectral libraries are inherently biased toward certain compound classes, so random molecule-level splits may still show class imbalance across splits
- Spectra come from different instrumentation types and of varying quality, which can be unevenly distributed across molecule-level splits

## Evidence

- [discussion] The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500).: "The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500)."
- [results] The test set, for instance, contains 3601 spectra of 500 unique compounds: "The test set, for instance, contains 3601 spectra of 500 unique compounds"
- [discussion] The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo: https://zenodo.org/record/4699300.: "The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo: https://zenodo.org/record/4699300."
