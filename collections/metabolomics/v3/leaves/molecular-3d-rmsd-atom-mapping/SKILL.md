---
name: molecular-3d-rmsd-atom-mapping
description: Use when a generative model outputs 3D molecular structures (atom coordinates
  plus types) and you need a geometry-level fidelity metric beyond 2D metrics like
  FCD or exact-structure accuracy, by comparing each generated conformation to a ground-truth
  3D structure.
when_to_use_negative:
- Only 2D topological outputs (SMILES or molecular graphs without coordinates) are
  available — RMSD requires 3D geometry; use FCD or exact-structure accuracy instead
- No ground-truth 3D reference structures exist for the test molecules, making atom
  mapping and RMSD undefined
- Evaluating spectral encoder quality (e.g., SpecFormer pre-training) rather than
  generated-structure geometry — use contrastive/InfoNCE or reconstruction metrics
  instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3047
- http://edamontology.org/topic_2269
tools:
- name: DiffSpectra
  role: Generative framework whose sampled 3D molecules are the input to the RMSD
    evaluation; produces molecules saved via --config.eval.save_mols true
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Hungarian algorithm
  role: Solves the atom-to-atom assignment between generated and ground-truth 3D structures
    so RMSD is computed over correctly paired atoms
- name: eval_sampled_mols/rmsd.py
  role: Repository script that performs the Hungarian atom mapping and RMSD computation
    on generated molecules
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:fb8e9ac25a7e1bc8be7e952c4363845d98469589cde457448080a22efce745c2
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - Hungarian algorithm
  - SpecFormer
  - eval_sampled_mols/rmsd.py
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3047
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# molecular-3d-rmsd-atom-mapping

## Summary

Evaluates 3D structural quality of generated molecules by performing atom-to-atom mapping against ground-truth 3D structures with the Hungarian algorithm and computing root mean square deviation (RMSD). Used in DiffSpectra to quantify how faithfully sampled 3D conformations match reference geometries.

## When to use

Use when a generative model outputs 3D molecular structures (atom coordinates plus types) and you need a geometry-level fidelity metric beyond 2D metrics like FCD or exact-structure accuracy, by comparing each generated conformation to a ground-truth 3D structure.

## When NOT to use

- Only 2D topological outputs (SMILES or molecular graphs without coordinates) are available — RMSD requires 3D geometry; use FCD or exact-structure accuracy instead
- No ground-truth 3D reference structures exist for the test molecules, making atom mapping and RMSD undefined
- Evaluating spectral encoder quality (e.g., SpecFormer pre-training) rather than generated-structure geometry — use contrastive/InfoNCE or reconstruction metrics instead

## Inputs

- Generated 3D molecular structures (atom types and coordinates) saved by the DiffSpectra eval loop under exp/{exp_name}/eval/
- Ground-truth 3D structures from the QM9S test set (huggingface.co/datasets/AzureLeon1/DiffSpectra)

## Outputs

- Per-molecule and aggregate RMSD values quantifying 3D structural deviation after Hungarian-algorithm atom mapping
- 3D structural quality contribution to the consolidated results_table.csv in exp/{exp_name}/eval/

## How to apply

For each generated molecule, obtain its 3D coordinates and the ground-truth 3D structure from the dataset (e.g., QM9S, which augments QM9 with simulated spectra). Because atom ordering may differ between generated and reference molecules, first solve an optimal atom-to-atom assignment against the ground-truth 3D structures using the Hungarian algorithm, then compute the root mean square deviation (RMSD) over the matched atom coordinates. In the DiffSpectra repository this is implemented in eval_sampled_mols/rmsd.py, run after molecule generation (main.py in eval mode with --config.eval.save_mols true) has written the sampled molecules to exp/{exp_name}/eval/. Report RMSD alongside top-1/top-10 exact-structure accuracy (Acc@1 = 40.76%, Acc@10 = 99.49%), molecular stability (~98.2%), uniqueness, novelty, and FCD (0.088) to give a complete 2D + 3D quality picture.

## Related tools

- **DiffSpectra** (Generative framework whose sampled 3D molecules are the input to the RMSD evaluation; produces molecules saved via --config.eval.save_mols true) — https://github.com/AzureLeon1/DiffSpectra
- **Hungarian algorithm** (Solves the atom-to-atom assignment between generated and ground-truth 3D structures so RMSD is computed over correctly paired atoms)
- **eval_sampled_mols/rmsd.py** (Repository script that performs the Hungarian atom mapping and RMSD computation on generated molecules) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
python eval_sampled_mols/rmsd.py
```

## Evaluation signals

- eval_sampled_mols/rmsd.py completes without atom-mapping failures on all generated molecules in exp/{exp_name}/eval/
- RMSD is computed only after valid 1-to-1 atom correspondence is established by the Hungarian algorithm (each generated atom maps to exactly one ground-truth atom)
- Aggregate RMSD values are written to the eval directory and consolidated into results_table.csv alongside Acc@1 (~40.76%), Acc@10 (~99.49%), stability (~98.2%), and FCD (~0.088)
- RMSD is near zero for molecules that were also scored as top-1 exact-structure matches, serving as a consistency check between 2D identity and 3D geometry
- Generated molecule files referenced by the RMSD script exist in the eval directory (verified via --config.eval.save_mols true output)

## Limitations

- RMSD assessment is meaningful only for molecules whose atom counts and composition allow a feasible Hungarian mapping; degenerate or invalid generated molecules may fail mapping
- Validation in the source work is primarily computational — no experimental/laboratory validation of the 3D structures was performed
- Framework and evaluation cover only QM9S-style molecules with UV-Vis, IR, and Raman spectra; applicability to NMR or MS modalities was not explored

## Evidence

- [supplementary] Dataset with 3D ground truth: [evidence span withheld — anchor status: not_found]
