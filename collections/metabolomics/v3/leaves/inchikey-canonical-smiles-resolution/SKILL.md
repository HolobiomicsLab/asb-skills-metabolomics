---
name: inchikey-canonical-smiles-resolution
description: Use when building a per-compound dataset from many spectra entries and
  you must deduplicate compounds and standardize structures (e.g. resolving canonical
  SMILES from InChIKeys before merging MS/MS spectra).
when_to_use_negative:
- Input molecules are already provided as canonical SMILES with no duplicate InChIKeys
  to resolve (e.g. the inputMols_git.xlsx toy input where SMILES is already the model
  input)
- The task requires predicting spectra for new molecules rather than standardizing
  a training-set compound list
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: SingleFrag
  role: Source dataset and downstream model pipeline within which InChIKey-based canonical
    SMILES resolution is performed
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Consumes the resolved canonical SMILES to generate the 300-dimensional molecular
    embedding used as ANN model input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:08b0f57d984dc2070a49e1c2cea87f9f7a3c3988279ceea61c46a9c6433d336b
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# inchikey-canonical-smiles-resolution

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Resolves canonical SMILES for each compound via its InChIKey to remove duplicates and standardize molecular structures, preparing a compound table for training or merging MS/MS spectra.

## When to use

Use when building a per-compound dataset from many spectra entries and you must deduplicate compounds and standardize structures (e.g. resolving canonical SMILES from InChIKeys before merging MS/MS spectra).

## When NOT to use

- Input molecules are already provided as canonical SMILES with no duplicate InChIKeys to resolve (e.g. the inputMols_git.xlsx toy input where SMILES is already the model input)
- The task requires predicting spectra for new molecules rather than standardizing a training-set compound list

## Inputs

- Per-spectrum compound identifiers with associated InChIKeys (from the SingleFrag dataset of raw MS/MS spectra)
- Raw spectra records from the SingleFrag Figshare dataset

## Outputs

- Per-compound table of canonical SMILES structures
- Deduplicated compound list used for train/validation/test splitting

## How to apply

Iterate over each spectrum's compound identifier, derive its InChIKey, and use that InChIKey to look up the canonical SMILES of the corresponding molecular structure. Use the canonical SMILES as the standard structural representation for each compound so that spectra from the same compound can be merged and duplicates removed. In the SingleFrag workflow this step operates on the 434,480 experimental [M+H] spectra and resolves structures before the 30,591 resulting compounds are split into train/validation/test sets; it also underlies the mol2vec inputs used by the ANN models, which take a 300-dimensional vector generated from SMILES. The output is a per-compound table of canonical SMILES that serves as the structure key for downstream binarized-spectrum merging.

## Related tools

- **SingleFrag** (Source dataset and downstream model pipeline within which InChIKey-based canonical SMILES resolution is performed) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Consumes the resolved canonical SMILES to generate the 300-dimensional molecular embedding used as ANN model input)

## Examples

```
resolve_smiles(dataset=load_figshare('SingleFrag'), spectra=filter_adduct('[M+H]'), key='InChIKey')
```

## Evaluation signals

- Number of distinct compounds after resolution matches the reported 30,591 compounds derived from 434,480 experimental [M+H] spectra
- No duplicate compounds remain in the output table (deduplication succeeded)
- Every resolved entry has a valid canonical SMILES that the Mol2vec model can convert into a 300-dimensional vector
- Train/validation/test split proportions of 80/10/10 are reproducible from the deduplicated compound table

## Limitations

- Molecules whose InChIKey cannot be resolved to a canonical SMILES will be missing from the deduplicated compound table
- The workflow recommends storing the resolved structures alongside the SingleFrag Figshare dataset files; pickle files were created with Python 3.8.19, so reproducibility depends on matching that environment

## Evidence

- [methods] Resolution of canonical SMILES: [evidence span withheld — anchor status: not_found]
- [methods] Scope of spectra filtered: [evidence span withheld — anchor status: not_found]
