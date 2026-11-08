---
name: smiles-validation-and-canonicalization
description: Use when a molecular generator (e.g., DeepMet RNN sampling) emits millions
  of raw SMILES strings that must be validity-checked, deduplicated by canonical form,
  and annotated with mass and formula before frequency tabulation.
when_to_use_negative:
- Input structures are already canonicalized and validated (e.g., a published 34.4-million-structure
  resource with sampling frequencies already provided) — re-running would be redundant
- The analysis is on preprocessed training-set SMILES rather than newly generated
  molecules; the preprocessing pipeline (removing light fragments, neutralizing charges,
  filtering for valid elements, removing rare vocabulary tokens) already handles those
- You need de novo structure prioritization by mass-to-charge ratio or MS/MS spectral
  matching rather than validity checking and frequency tabulation
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Parse sampled SMILES, test chemical validity, canonicalize strings, and compute
    calculated molecular mass and formula; also used upstream to remove light fragments
    and neutralize charges during preprocessing
- name: Python 3.10
  role: Language in which the validation/canonicalization scripts and the CLM/DeepMet
    workflow are implemented
  repo: https://github.com/skinniderlab/CLM
- name: Snakemake
  role: Orchestrates the end-end Snakefile workflow (sampling, parsing, tabulation,
    and cross-fold aggregation steps)
  repo: https://github.com/skinniderlab/CLM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:e0e87c78db7ba00d6c3ca8d9a1e2d0ea5bf159c1267cfd1433d3dbf86e90fd11
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - PyTorch
  - Python
  - Python 3.10
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# SMILES validation and canonicalization

## Summary

Parse every structure produced by a generative chemical language model with RDKit, discard chemically invalid SMILES, and canonicalize the remainder so that frequency counting and InChIKey-based de-duplication are applied to a single, well-defined representation of each molecule.

## When to use

Use when a molecular generator (e.g., DeepMet RNN sampling) emits millions of raw SMILES strings that must be validity-checked, deduplicated by canonical form, and annotated with mass and formula before frequency tabulation.

## When NOT to use

- Input structures are already canonicalized and validated (e.g., a published 34.4-million-structure resource with sampling frequencies already provided) — re-running would be redundant
- The analysis is on preprocessed training-set SMILES rather than newly generated molecules; the preprocessing pipeline (removing light fragments, neutralizing charges, filtering for valid elements, removing rare vocabulary tokens) already handles those
- You need de novo structure prioritization by mass-to-charge ratio or MS/MS spectral matching rather than validity checking and frequency tabulation

## Inputs

- Raw sampled SMILES strings from a trained RNN/DeepMet model (per-fold output, e.g. 100 million per fold)
- Fold-specific training-set molecules for InChIKey-based exclusion
- RDKit molecule objects parsed from each SMILES string

## Outputs

- Table of unique canonical SMILES with per-molecule sampling frequency, calculated molecular mass, and formula (e.g., structure_frequencies.tsv, one row per unique canonical structure)
- Discarded set of chemically invalid SMILES strings (excluded from all downstream counts)
- InChIKeys per canonical structure used for training-set filtering

## How to apply

Load each fold's autoregressively sampled SMILES output (100 million per fold in the DeepMet cross-validation, 1 billion total), then parse every string with RDKit and drop any that fail chemical validity. Canonicalize each valid SMILES so structurally identical molecules collapse to one row, and compute the calculated molecular mass and molecular formula per canonical structure. Convert canonical structures to InChIKeys and remove any molecule whose InChIKey matches a fold's training-set molecule, so that sampling-frequency metrics are computed only over generated, non-training structures. Finally, record the per-molecule sampling frequency; downstream aggregation across train_seeds models and folds (e.g., metric=freq-avg) is only meaningful if this validation-canonicalization step is complete and one row per unique canonical structure is present in structure_frequencies.tsv.

## Related tools

- **RDKit** (Parse sampled SMILES, test chemical validity, canonicalize strings, and compute calculated molecular mass and formula; also used upstream to remove light fragments and neutralize charges during preprocessing)
- **Python 3.10** (Language in which the validation/canonicalization scripts and the CLM/DeepMet workflow are implemented) — https://github.com/skinniderlab/CLM
- **Snakemake** (Orchestrates the end-end Snakefile workflow (sampling, parsing, tabulation, and cross-fold aggregation steps)) — https://github.com/skinniderlab/CLM

## Examples

```
mols = [Chem.MolFromSmiles(s) for s in sampled]; valid = [Chem.MolToSmiles(m) for m in mols if m is not None]
```

## Evaluation signals

- structure_frequencies.tsv contains exactly one row per unique canonical structure with its averaged sampling frequency
- No InChIKeys from the fold's training set appear in the output table (training-set leakage check passes)
- No chemically invalid SMILES (unparseable by RDKit) are present in the final table
- Every output row carries a non-empty canonical SMILES, calculated molecular mass, and molecular formula
- Aggregate counts are consistent with the sampling budget (e.g., 1 billion sampled SMILES across ten folds after deduplication into the 34.4-million-structure resource scale)

## Limitations

- Validity filtering is permissive of near-duplicates: as shown by mgenerators-failure-modes, tiny changes to training-set molecules can yield 'novel' structures, so novelty metrics based on canonical SMILES alone can overestimate true novelty
- Sampling frequency is a model-dependent signal; the article notes inherent limitations in anticipating metabolites from divergent, undiscovered biosynthetic routes
- Very large sampling runs (billions of SMILES) are computationally heavy; the end-end workflow 'should take around 5 minutes' only where torch has access to a GPU

## Evidence

- [other] Parse every sampled SMILES with RDKit and discard chemically invalid outputs. Canonicalize the valid SMILES and compute the calculated molecular mass and formula for each.: [evidence span withheld — anchor status: not_found]
