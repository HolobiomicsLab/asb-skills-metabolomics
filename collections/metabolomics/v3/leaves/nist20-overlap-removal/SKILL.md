---
name: nist20-overlap-removal
description: Use when building pretraining corpora from large compound libraries (e.g.
  ~30M ZINC20 SMILES) that will later be paired with a model finetuned on NIST 20
  experimental EI-MS spectra, to prevent train/test leakage.
when_to_use_negative:
- Corpus is being built for a model that will never be evaluated on NIST 20 spectra
  (e.g. purely synthetic-spectrum benchmark) — removal is unnecessary and discards
  usable structures.
- Corpus already only contains compounds absent from NIST 20 (e.g. it was generated
  from a disjoint library).
- The downstream model is trained only on synthetic spectra with no NIST 20 finetuning,
  so there is no leakage channel.
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0605
tools:
- name: SpecTUS
  role: Downstream model whose NIST 20 finetuning step makes this exclusion mandatory;
    the removal is applied to the corpora SpecTUS is pretrained on.
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Environment where the filtering notebooks in the SpecTUS repo perform the
    NIST 20 compound removal during data preprocessing.
  repo: https://github.com/ljocha/spectus-demo
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:2d8835d5d8a8b4c168680f7cc302103fa315a705e8775e642a19fccc49ee2881
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - SpecTUS
  - conda
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# NIST 20 overlap removal

## Summary

Removes all compounds that also appear in the NIST 20 experimental library from candidate pretraining corpora (e.g. ZINC-derived synth1/synth2 SMILES sets) so that finetuning on NIST 20 spectra cannot leak into pretraining data.

## When to use

Use when building pretraining corpora from large compound libraries (e.g. ~30M ZINC20 SMILES) that will later be paired with a model finetuned on NIST 20 experimental EI-MS spectra, to prevent train/test leakage.

## When NOT to use

- Corpus is being built for a model that will never be evaluated on NIST 20 spectra (e.g. purely synthetic-spectrum benchmark) — removal is unnecessary and discards usable structures.
- Corpus already only contains compounds absent from NIST 20 (e.g. it was generated from a disjoint library).
- The downstream model is trained only on synthetic spectra with no NIST 20 finetuning, so there is no leakage channel.

## Inputs

- SMILES list of candidate pretraining compounds (e.g. ~30M ZINC20-derived strings, later reduced to ~4.7M synth1 / ~4.8M synth2 after RASSP-processability filtering)
- NIST 20 compound list (canonical SMILES / InChIKeys) used as the exclusion reference

## Outputs

- Filtered SMILES corpus with NIST 20 compounds removed (rassp_filtered_compounds.csv-style tagged output, one row per compound with source-corpus tag synth1 or synth2)
- Count of removed compounds (audit line for leakage-prevention provenance)

## How to apply

Parse each SMILES string in the candidate corpus (synth1 or synth2, derived from ZINC20 via the 2D-standard-annotated-druglike query) and canonicalize it for lookup against NIST 20 compound identifiers. Drop every compound present in NIST 20 before any further filtering is persisted. The SpecTUS authors perform this removal explicitly after RASSP-processability filtering (8 most common elements, bounded number of possible subformulae) precisely because their model is subsequently finetuned on 232,025 NIST 20 spectra. Rationale: retaining NIST 20 compounds in synthetic pretraining data would let the model see test-set structures during pretraining, inflating evaluation Acc1 on the held-out 28,267-spectrum NIST 20 test split.

## Related tools

- **SpecTUS** (Downstream model whose NIST 20 finetuning step makes this exclusion mandatory; the removal is applied to the corpora SpecTUS is pretrained on.) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Environment where the filtering notebooks in the SpecTUS repo perform the NIST 20 compound removal during data preprocessing.) — https://github.com/ljocha/spectus-demo

## Examples

```
python notebooks/preprocess/filter_nist20_overlap.py --input synth1_smiles.csv --nist nist20_compounds.csv --output rassp_filtered_compounds.csv
```

## Evaluation signals

- Verify the filtered corpus size matches the paper's reported values (~4.7M for synth1, ~4.8M for synth2) after both RASSP-processability and NIST 20 removal filters.
- Cross-check a random sample of removed compounds against the NIST 20 compound list to confirm they were genuine overlaps, not false positives from SMILES canonicalization mismatches.
- Assert that no canonicalized NIST 20 SMILES remains in the persisted filtered output (set-intersection size == 0).
- Confirm the same filter was applied to both synth1 and synth2 halves so pretraining (NEIMS:RASSP 1:1 mix) remains symmetric in leakage protection.
- Downstream sanity: validation Acc1 of the finetuned SpecTUS should reflect genuine generalization, not inflated numbers that would suggest NIST 20 structures persisted in pretraining data.

## Limitations

- Removal depends on correct SMILES canonicalization / structure matching; tautomers, salts, or stereoisomer discrepancies between ZINC20 and NIST 20 representations could leave residual overlap undetected.
- Only compounds explicitly present in NIST 20 are removed; near-duplicates or analogues that would still inflate evaluation are not excluded.
- NIST 20 is proprietary — a license is required to obtain the compound list needed to perform this removal yourself.

## Evidence

- [methods] The model is finetuned on NIST 20, which motivates the overlap removal.: [evidence span withheld — anchor status: not_found]
